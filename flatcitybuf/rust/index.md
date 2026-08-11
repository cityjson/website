---
layout: default
title: Rust
parent: FlatCityBuf
nav_order: 4
has_children: false
permalink: /flatcitybuf/rust/
---

# Using FlatCityBuf with Rust

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Rust is the reference implementation: `fcb_core` both reads and writes FlatCityBuf, and is what the [`fcb` CLI]({{ '/flatcitybuf/conversion/' | prepend: site.baseurl }}) and the other implementations are validated against. Its API reference is on [docs.rs/fcb_core](https://docs.rs/fcb_core), and the repository's own guide — crate layout, features, tooling — is [docs/rust.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/rust.md).

## Adding to your project

Initialise a new Rust project:

{% raw %}
```bash
cargo new fcb_demo
cd fcb_demo
```
{% endraw %}

Add FlatCityBuf to your `Cargo.toml`:

{% raw %}
```toml
[dependencies]
fcb_core = "0.7.6"

# The `http` feature is on by default. For a local-only reader without
# reqwest, turn it off:
fcb_core = { version = "0.7.6", default-features = false }
```

{% endraw %}

## Reading a local file

{% raw %}

```rust
use fcb_core::{deserializer::to_cj_metadata, FcbReader};
use std::fs::File;
use std::io::BufReader;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let file = BufReader::new(File::open("delft.fcb")?);
    let mut features = FcbReader::open(file)?.select_all()?;

    // The CityJSON metadata object: the first line of the equivalent
    // CityJSONSeq document.
    let cj = to_cj_metadata(&features.header())?;
    println!(
        "CityJSON {}, {} features",
        cj.version,
        features.header().features_count()
    );

    while let Some(feature) = features.next()? {
        let cj_feature = feature.cur_cj_feature()?;
        println!("{}", cj_feature.id);
    }

    Ok(())
}
```

{% endraw %}

then run it with:

{% raw %}

```bash
cargo run
```

{% endraw %}

## Spatial queries

`select_query` walks the packed R-tree and skips straight to the matching features. The bounding box is `(min_x, min_y, max_x, max_y)` in the file's CRS; the two `Option`s are `limit` and `offset`.

{% raw %}

```rust
use fcb_core::{FcbReader, SpatialQuery};
use std::fs::File;
use std::io::BufReader;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let file = BufReader::new(File::open("delft.fcb")?);
    let bbox = SpatialQuery::BBox(84227.77, 445377.33, 85323.23, 446334.69);

    let mut hits = FcbReader::open(file)?.select_query(bbox, None, None)?;

    let mut count = 0;
    while let Some(feature) = hits.next()? {
        println!("in bbox: {}", feature.cur_cj_feature()?.id);
        count += 1;
    }
    println!("{count} features in the bounding box");

    Ok(())
}
```

{% endraw %}

`SpatialQuery` also has `PointIntersects(x, y)` and `PointNearest(x, y)`.

## Attribute queries

`select_attr_query` uses the static B+tree indices, so the attribute must have been indexed at write time (see [`fcb ser --attr-index`]({{ '/flatcitybuf/conversion/#with-attribute-indexing' | prepend: site.baseurl }})). A query is a list of `(column, operator, value)` triples, AND-ed together, and the value's `KeyType` must match the column's type on disk.

{% raw %}

```rust
use fcb_core::{AttrQuery, FcbReader, KeyType, Operator};
use std::fs::File;
use std::io::BufReader;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let file = BufReader::new(File::open("delft.fcb")?);

    let query: AttrQuery = vec![(
        "b3_h_dak_50p".to_string(),
        Operator::Gt,
        KeyType::Float64(20.0.into()),
    )];

    let mut hits = FcbReader::open(file)?.select_attr_query(query)?;
    while let Some(feature) = hits.next()? {
        println!("tall: {}", feature.cur_cj_feature()?.id);
    }

    Ok(())
}
```

{% endraw %}

Operators are `Eq`, `Ne`, `Gt`, `Ge`, `Lt`, `Le`. For string columns use `KeyType::StringKey50(FixedStringKey::from_str("..."))` — the index stores keys truncated to 50 bytes, so it answers with candidates that the reader verifies against the full value.

## HTTP streaming

For cloud-hosted files, use the async HTTP reader. Only the bytes a query needs are fetched, so a query against a 68GB file costs a handful of range requests.

Add `tokio` to your `Cargo.toml`:

{% raw %}

```toml
[dependencies]
fcb_core = "0.7.6"   # the `http` feature is on by default
tokio = { version = "1", features = ["rt-multi-thread", "macros"] }
```

{% endraw %}

{% raw %}
```rust
use fcb_core::{HttpFcbReader, SpatialQuery};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let reader =
        HttpFcbReader::open("https://flatcitybuf.open3d.city/data/3dbag_all_index.fcb")
            .await?;

    println!("{} features", reader.header().features_count());

    let bbox = SpatialQuery::BBox(120000.0, 486000.0, 120200.0, 486200.0);
    let mut iter = reader.select_query(bbox).await?;

    while let Some(feature) = iter.next().await? {
        // note: `cj_feature()` here, not `cur_cj_feature()`
        println!("{}", feature.cj_feature()?.id);
    }

    Ok(())
}
```
{% endraw %}

Attribute queries work the same way over HTTP with `select_attr_query(&query)`, and both have `_paged` variants (`select_query_paged`, `select_attr_query_paged`) taking a limit and an offset.

## Zero-copy access

Each feature can be read in two ways:

- `feature.cur_feature()` returns the FlatBuffers view — **zero-copy**, only the fields you touch are decoded;
- `feature.cur_cj_feature()` returns a `CityJSONFeature`, which parses the FlatBuffers into owned CityJSON structures.

If you only need a few attributes per feature, the first is considerably cheaper. See the [performance tips]({{ '/flatcitybuf/faq/#zero-copy-benefits' | prepend: site.baseurl }}).

## Writing files

`FcbWriter` takes the CityJSON metadata object plus a stream of `CityJSONFeature`s, and assembles the header, the indices and the feature data when you call `write`. The [CLI's source](https://github.com/cityjson/flatcitybuf/blob/main/src/rust/cli/src/main.rs) is the fully worked example: it builds the attribute schema in a first pass, then adds every feature, then writes.
