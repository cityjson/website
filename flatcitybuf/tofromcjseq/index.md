---
layout: default
title: CLI (fcb)
parent: FlatCityBuf
nav_order: 2
has_children: false
permalink: /flatcitybuf/conversion/
---

# The `fcb` command-line tool

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .highlight }
If you have CityJSON files, you first need to [convert them to the CityJSONSeq format]({{ '/cityjsonseq/#cjseq-cityjson--cityjsonseq' | prepend: site.baseurl }}).

`fcb` converts between CityJSON/CityJSONSeq and FlatCityBuf, and inspects `.fcb` files. It is written in Rust ([code on GitHub](https://github.com/cityjson/flatcitybuf)); the crate-level reference is [docs/rust.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/rust.md).

## Installation

**Option 1: install from crates.io**

{% raw %}

```bash
cargo install fcb_cli --locked
```

{% endraw %}

This installs the `fcb` binary into your Cargo bin directory (usually `~/.cargo/bin/`).

**Option 2: build from source**

{% raw %}

```bash
git clone https://github.com/cityjson/flatcitybuf.git
cd flatcitybuf/src/rust
cargo build --release -p fcb_cli
```

{% endraw %}

The binary is then at `target/release/fcb`.

{: .warning }
`inspect` gained its static report — and absorbed the older summary subcommand — after the current crates.io release. If your `fcb` still has a separate `info` subcommand, build it from source (option 2) to follow this page.

## Commands

{% raw %}

```bash
$ fcb --help
CLI tool for CityJSON <-> FCB conversion

Usage: fcb <COMMAND>

Commands:
  ser      Convert CityJSON to FCB
  deser    Convert FCB to CityJSON
  cbor     Convert CityJSON to CBOR
  bson     Convert CityJSON to BSON
  inspect  Inspect an FCB file or URL: a terminal UI on a TTY, a static report otherwise
  help     Print this message or the help of the given subcommand(s)
```

{% endraw %}

{: .warning }
Input and output are **positional** arguments: the input comes first, the output last. Older versions of the CLI used `-i`/`-o` flags; those are gone.

Both accept `-` for stdin/stdout, so `fcb` composes in a pipeline.

## Conversion CityJSONSeq => FlatCityBuf

{% raw %}

```bash
$ fcb ser delft.city.jsonl delft.fcb

━━━ FlatCityBuf Serialization
━━━ ━━━━━━━━━━━━━━━━━━━━━━━━

▶ Configuration
  Input: 1 file(s)
    1. delft.city.jsonl
  Output: delft.fcb
  Spatial Index: enabled
  Geospatial Extent: not set

▶ Reading CityJSON...
  ✓ 1115 features
▶ Building attribute schema...
  ✓ 44 unique attributes found
▶ Building indices...
  ✓ Spatial R-tree index (node size: 16)

▶ Writing FCB file...
  ✓ File written successfully

━━━ Serialization Complete
```

{% endraw %}

The spatial (R-tree) index is written by default; `--no-spatial-index` turns it off.

### Multiple inputs and glob patterns

`ser` takes any number of inputs before the output, so it merges as it converts. When the inputs have different coordinate transforms, vertices are aligned to the first file's transform.

{% raw %}

```bash
# several files -- the last positional is always the output
fcb ser file1.city.jsonl file2.city.jsonl merged.fcb

# glob patterns (quote them, so fcb expands them itself)
fcb ser 'data/*.city.jsonl' output.fcb
fcb ser 'cities/**/*.city.json' all_cities.fcb

# a plain CityJSON file works too
fcb ser city.city.json output.fcb

# stdin to stdout
cat input.city.jsonl | fcb ser - - > output.fcb
```

{% endraw %}

### With attribute indexing

To enable fast queries on specific attributes:

{% raw %}

```bash
$ fcb ser delft.city.jsonl delft.fcb \
  --attr-index identificatie,b3_h_dak_50p,b3_is_glas_dak \
  --attr-branching-factor 256
```

{% endraw %}

`--attr-index` (`-a`) takes a comma-separated list of attribute names to index. The branching factor (default: 256) controls the B+tree structure – higher values mean flatter trees and fewer round trips per query, but slightly larger files. See the [performance tips]({{ '/flatcitybuf/faq/#indexing-tips' | prepend: site.baseurl }}).

### Index all attributes

If you want to index every attribute found in the dataset:

{% raw %}

```bash
$ fcb ser delft.city.jsonl delft.fcb -A
```

{% endraw %}

This is convenient but increases file size and conversion time.

### Filtering by bounding box

You can filter features during conversion to create a subset:

{% raw %}

```bash
$ fcb ser delft.city.jsonl filtered.fcb \
  --bbox "84227.77,445377.33,85323.23,446334.69"
```

{% endraw %}

The bounding box format is `minx,miny,maxx,maxy`, in the CRS of the data.

### All `ser` options

| Option | Meaning |
| --- | --- |
| `-a, --attr-index <LIST>` | comma-separated attribute names to index |
| `-A, --index-all-attributes` | index every attribute found |
| `--attr-branching-factor <N>` | B+tree branching factor (default 256) |
| `-s, --no-spatial-index` | do not write the R-tree |
| `--index-node-size <N>` | R-tree node size (default 16) |
| `-b, --bbox "minx,miny,maxx,maxy"` | keep only features inside the bbox |
| `-g, --ge` | compute and write the geographical extent in the header |

{: .info }
`--attr-branching-factor` and `--index-node-size` are unrelated knobs: the first is the attribute B+tree's, the second the spatial R-tree's.

## Conversion FlatCityBuf => CityJSONSeq

{% raw %}

```bash
$ fcb deser delft.fcb delft.city.jsonl
Successfully decoded to CityJSON
```

{% endraw %}

The output is a CityJSONSeq document: one metadata line, then one `CityJSONFeature` per line, in stored (Hilbert) order.

## Inspecting a file

`fcb inspect` reads the header of a **local file or an HTTP(S) URL**. For a remote file it fetches only the header bytes with a range request, so pointing it at a 68GB file on cloud storage is instant.

{% raw %}

```bash
fcb inspect delft.fcb
fcb inspect https://flatcitybuf.open3d.city/data/3dbag_all_index.fcb
```

{% endraw %}

### `fcb inspect --static` — a one-shot report

With `--static`, or whenever stdout is not a terminal (a pipe, a redirect, CI), `inspect` prints a plain-text report instead of the UI, so it composes in a script:

{% raw %}

```bash
$ fcb inspect --static delft.fcb

▶ File Details
  Source: delft.fcb
  Size: 7.31 MB
  Version: 2.0
  Title: 3DBAG

▶ Dataset
  Features: 1115
  Columns: 44
  Geospatial Extent: Yes
    Min: [84501.55, 445805.03, -3.75]
    Max: [85675.23, 446983.47, 95.04]
    Dimensions: 1173.68 × 1178.44 × 98.79

▶ Indices
  Spatial R-tree: Yes (node size: 16)
  Attribute Indices: 44 (B+Tree)
    1. b3_bag_bag_overlap
    2. b3_dak_type
    3. b3_h_dak_50p
    ... 41 more attributes...

▶ Coordinate Reference System
  CRS: EPSG:7415

▶ Coordinate Transform
  Scale: [0.001000, 0.001000, 0.001000]
  Translate: [85088.390625, 446394.250000, 45.648003]
```

{% endraw %}

The list of attribute indices is the list of attributes you can actually query: an attribute that was not indexed at write time is still readable, but not queryable.

### `fcb inspect` — an interactive terminal UI

On a terminal, the same command opens a full-screen UI with three tabs. **Metadata**:

{% raw %}

```
┌Header Categories─────────────────────────────────────────────────────────────┐
│ Metadata │ Columns │ Map                                                     │
└──────────────────────────────────────────────────────────────────────────────┘
┌Metadata──────────────────────────────────────────────────────────────────────┐
│Title: 3DBAG                                                                  │
│CityJSON Version: 2.0                                                         │
│Features: 1115                                                                │
│Columns: 44                                                                   │
│Spatial Index R-Tree Node Size: 16                                            │
│Attribute Indices: 44                                                         │
│Bounds: [84501.5547, 445805.0312, -3.7470] .. [85675.2344, 446983.4688, ...]  │
│Dimensions: 1173.68 x 1178.44 x 98.79                                         │
│Scale: [0.001000, 0.001000, 0.001000]                                         │
│Translate: [85088.391, 446394.250, 45.648]                                    │
│CRS Code: EPSG:7415                                                           │
└──────────────────────────────────────────────────────────────────────────────┘
```

{% endraw %}

**Columns** — the attribute schema, scrollable:

{% raw %}

```
┌Columns (1 of 44)─────────────────────────────────────────────────────────────┐
│Name                        Type      Description   Nullable  Primary Key  ... │
│b3_bag_bag_overlap          Double    -             true      false           │
│b3_dak_type                 String    -             true      false           │
│b3_h_dak_50p                Double    -             true      false           │
│b3_kas_warenhuis            Bool      -             true      false           │
│b3_pw_datum                 ULong     -             true      false           │
└──────────────────────────────────────────────────────────────────────────────┘
```

{% endraw %}

**Map** — for a geographic CRS, the dataset's extent drawn on a world coastline; for a projected CRS (like the EPSG:7415 above) it prints the extent instead of guessing a reprojection.

Keys: `Tab`/`←`/`→` (or `h`/`l`) switch tabs, `↑`/`↓` (or `k`/`j`) scroll, `g`/`G` jump to top/bottom, `q`/`Esc`/`Ctrl-C` quit. The UI needs a real terminal; without one you get the static report above rather than escape codes in your file.

## Other conversions

`fcb cbor` and `fcb bson` convert CityJSON to CBOR and BSON. They exist mainly to compare encodings and file sizes, and do not produce FlatCityBuf files:

{% raw %}

```bash
fcb cbor city.city.json city.cbor
fcb bson city.city.json city.bson
```

{% endraw %}
