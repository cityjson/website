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


For maximum performance and control, you can use FlatCityBuf directly in Rust applications.

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
fcb_core = "0.5.0"

# For HTTP support
fcb_core = { version = "0.5.0", features = ["http"] }
```

{% endraw %}

## Basic reading and spatial queries

Use bounding box queries with the packed R-tree index:

{% raw %}

```rust
use fcb_core::{FcbReader, packed_rtree::Query};
use std::fs::File;
use std::io::BufReader;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let input_file = File::open("../delft.fcb")?;
    let input_reader = BufReader::new(input_file);

    // Define bounding box
    let minx = 84227.77;
    let miny = 445377.33;
    let maxx = 85323.23;
    let maxy = 446334.69;

    let mut reader = FcbReader::open(input_reader)?.select_query(
        Query::BBox(minx, miny, maxx, maxy),
        None,
        None,
    )?;

    let mut count = 0;
    while let Some(feature_buf) = reader.next()? {
        let cj_feature = feature_buf.cur_cj_feature()?;
        println!("Feature in bbox: {}", cj_feature.id);
        count += 1;
    }

    println!("Found {} features in bounding box", count);

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

## HTTP streaming

For cloud-based FCB files, use the async HTTP reader:
Don't forget to add `tokio` to your `Cargo.toml`:

{% raw %}

```toml
[dependencies]
tokio = { version = "1.48.0", features = ["rt-multi-thread", "macros"] }

```

{% endraw %}

{% raw %}
```rust
use fcb_core::{FcbReader, HttpFcbReader, packed_rtree::Query};
use std::fs::File;
use std::io::BufReader;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let http_reader =
        HttpFcbReader::open("https://storage.googleapis.com/flatcitybuf/3dbag_all_index.fcb")
            .await?;

    // Get header information
    let header = http_reader.header();
    println!("Features: {}", header.features_count());

    // Spatial query over HTTP
    let minx = 84227.77;
    let miny = 445377.33;
    let maxx = 85323.23;
    let maxy = 446334.69;

    let mut iter = http_reader
        .select_query(Query::BBox(minx, miny, maxx, maxy))
        .await?;

    while let Some(feature) = iter.next().await? {
        let cj_feature = feature.cj_feature()?;
        println!("Feature: {}", cj_feature.id);
    }

    Ok(())
}

```
{% endraw %}
