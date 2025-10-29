---
layout: default
title: to/from CityJSONSeq 
parent: FlatCityBuf
nav_order: 2
has_children: false
permalink: /flatcitybuf/conversion/
---

# Conversion CityJSON <=> FlatCityBuf

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .highlight }
If you have CityJSON files, you first need to [convert them to the CityJSONSeq format]({{ '/cityjsonseq/#cjseq-cityjson--cityjsonseq' | prepend: site.baseurl }}).

## Rust CLI to manipulate FlatCityBuf files

We offer a CLI, written in Rust ([code on GitHub](https://github.com/cityjson/flatcitybuf)) to convert FlatCityBuf files to/from different formats.

**Option 1: Install from crates.io**

{% raw %}

```bash
cargo install fcb_cli
```

{% endraw %}

**Option 2: Build from source**

{% raw %}

```bash
git clone https://github.com/cityjson/flatcitybuf.git
cd flatcitybuf/src/rust
cargo build --workspace --all-features --exclude fcb_wasm --release
```

{% endraw %}

The binary will be available at `target/release/fcb`.


## Conversion CityJSONSeq => FlatCityBuf

{% raw %}
```bash
$ fcb ser -i delft.city.jsonl -o delft.fcb
Successfully encoded to FCB
```
{% endraw %}

This creates a FlatCityBuf file with spatial indexing enabled by default.

### With attribute indexing

To enable fast queries on specific attributes:

{% raw %}
```bash
$ fcb ser -i delft.city.jsonl -o delft.fcb \
  --attr-index identificatie,b3_h_dak_50p,b3_is_glas_dak \
  --attr-branching-factor 16
```
{% endraw %}

The `--attr-index` flag takes a comma-separated list of attribute names to index. The branching factor (default: 256) controls the B+tree structure – higher values mean flatter trees and faster queries but slightly larger file sizes.

### Index all attributes

If you want to index every attribute found in the dataset:

{% raw %}
```bash
$ fcb ser -i delft.city.jsonl -o delft.fcb -A
Successfully encoded to FCB
```
{% endraw %}

This is convenient but will increase file size and conversion time.

### Filtering by bounding box

You can filter features during conversion to create a subset:

{% raw %}

```bash
$ fcb ser -i delft.city.jsonl -o filtered.fcb \
  --bbox "84227.77,445377.33,85323.23,446334.69"
Successfully encoded to FCB
```

{% endraw %}

The bounding box format is: `minx,miny,maxx,maxy`

## Conversion FlatCityBuf => CityJSONSeq

To convert an FCB file back to CityJSON Text Sequences:

{% raw %}

```bash
$ fcb deser -i delft.fcb -o delft.city.jsonl
Successfully decoded from FCB
```

{% endraw %}


## Inspecting FCB files

The CLI also allows us to view information about an FCB file:

{% raw %}
```bash
$ fcb info -i delft.fcb
FCB File Info:
    File size: 6 MB
  Version: 2.0
  Features count: 1115
  bbox: Some(GeographicalExtent { min: Vector { x: 84501.5546875, y: 445805.03125, z: -3.746997833251953 }, max: Vector { x: 85675.234375, y: 446983.46875, z: 95.04200744628906 } })
  attr_index: []
  Title: 3DBAG
  Geographical extent:
    Min: [84501.5546875, 445805.03125, -3.746997833251953]
    Max: [85675.234375, 446983.46875, 95.04200744628906]
```
{% endraw %}
