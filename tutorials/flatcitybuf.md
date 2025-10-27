---
layout: page
title: Using FlatCityBuf
parent: Tutorials
nav_order: 5
permalink: /tutorials/flatcitybuf/
---

## Table of contents

{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Introduction

FlatCityBuf is a cloud-optimised binary format for storing and retrieving 3D city models, combining CityJSON with FlatBuffers binary serialisation and spatial indexing.

For details about FlatCityBuf's features, performance benefits, and research background, see the [FlatCityBuf overview page]({{ '/flatcitybuf/' | prepend: site.baseurl }}).

## Tutorial goals

In this tutorial, you will learn how to:

1. Convert CityJSON files to FlatCityBuf format using the CLI
2. Access and query FlatCityBuf files with Python
3. Use FlatCityBuf in web browsers with WebAssembly
4. Integrate FlatCityBuf into Rust applications
5. Optimise performance with spatial and attribute indexing

---

## Prerequisites and installation

### System requirements

Depending on your use case, you may need:

- **Rust toolchain** (1.83.0 or later) - for CLI and building from source
- **Python** (3.7 or higher) - for Python bindings
- **Node.js/npm** - for JavaScript/TypeScript usage
- **Modern web browser** - for WebAssembly applications

### Installation options

#### CLI tool

The command-line interface is useful for converting files and inspecting FCB data.

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

#### Python bindings

{% raw %}

```bash
pip install flatcitybuf
```

{% endraw %}

For development or building from source:

{% raw %}

```bash
pip install maturin
cd flatcitybuf/src/rust/fcb_py
maturin develop --features http
```

{% endraw %}

#### JavaScript/TypeScript (npm)

{% raw %}

```bash
npm install @cityjson/flatcitybuf
```

{% endraw %}

#### Rust library

Add to your `Cargo.toml`:

{% raw %}

```toml
[dependencies]
fcb_core = "0.5.0"

# For HTTP support
fcb_core = { version = "0.5.0", features = ["http"] }
```

{% endraw %}

---

## Working with the CLI

The FlatCityBuf CLI provides commands for converting between formats and inspecting FCB files.

### Converting CityJSONSeq to FlatCityBuf

The most common operation is converting CityJSON Text Sequences (`.city.jsonl` file) to FlatCityBuf format.

You can download a sample CityJSON Text Sequences file from here: [delft.city.jsonl](https://github.com/cityjson/flatcitybuf/blob/bf270c345715e1020d314b7235ede8fffc6e0d85/examples/data/delft.city.jsonl).

### Basic conversion

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

### Converting FlatCityBuf back to CityJSONSeq

To convert an FCB file back to CityJSON Text Sequences:

{% raw %}

```bash
$ fcb deser -i delft.fcb -o delft.city.jsonl
Successfully decoded from FCB
```

{% endraw %}

### Inspecting FCB files

To view information about an FCB file:

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

---

## Getting example data

### Download sample FlatCityBuf files

Several example FCB files are available for testing:

<!-- TODO: add actual URLs to example files -->

- [3DBAG all (70GB)](https://storage.googleapis.com/flatcitybuf/3dbag_all_index.fcb): Complete 3DBAG dataset with spatial and attribute indexing
- [3DBAG small (3.4GB)](https://storage.googleapis.com/flatcitybuf/3dbag_subset_all_index.fcb)
- [Delft (6MB)](https://github.com/cityjson/flatcitybuf/blob/bf270c345715e1020d314b7235ede8fffc6e0d85/examples/data/delft.fcb)

These files are hosted on Google Cloud Storage and can be accessed directly via HTTP.

### Creating your own FCB files

If you have CityJSON files, you first need to convert them to CityJSONTextSequences format.

For more details, see the [CityJSONSeq software page]({{ '/cityjsonseq/' | prepend: site.baseurl }})

---

## Using FlatCityBuf with Python

The Python bindings provide a convenient interface for reading and querying FlatCityBuf files.

### Basic reading

#### Opening local files

You can show the metadata of a FlatCityBuf file with the following code:

{% raw %}

```python
import flatcitybuf as fcb

# Open a local FCB file
reader = fcb.Reader("delft.fcb")

# Get file information
info = reader.info()
print(f"Features: {info.feature_count}")
print(f"Bounding box: {info.bbox}")

# Get CityJSON header with transform and metadata
cityjson = reader.cityjson_header()
print(f"CityJSON version: {cityjson.version}")
print(f"Transform scale: {cityjson.transform.scale}")
print(f"Transform translate: {cityjson.transform.translate}")
```

{% endraw %}

#### Iterating through features

This will print the first 2 features:

{% raw %}

```python
feature_count = 0
    for feature in reader:
        print(f"Feature {feature_count + 1}:")
        print(f"  ID: {feature.id}")
        print(f"  Type: {feature.type}")
        print(f"  Vertices: {len(feature.vertices)} vertices")
        print(f"  City Objects: {len(feature.city_objects)} objects")

        # Iterate over all city objects in the feature
        if feature.city_objects:
            for obj_id, city_obj in feature.city_objects.items():
                print(f"    Object ID: {obj_id}")
                print(f"    Object type: {city_obj.type}")
                print(f"    Geometries: {len(city_obj.geometry)}")

                # Show geometry with nested boundaries
                if city_obj.geometry:
                    for geom in city_obj.geometry:
                        if geom is not None:
                            print(f"      Geometry type: {geom.geometry_type}")
                            print(f"      Vertices index: {geom.vertices}")
                            print(f"      Boundaries: {geom.boundaries}")
                            if geom.semantics:
                                print(f"      Has semantics: {geom.semantics}")
                        else:
                            print("      Geometry is None")

        feature_count += 1
        # Limit output for demo
        if feature_count >= 2:
            print("  ... (showing first 2 features only)")
            break
# features will be printed here
```

{% endraw %}

### Spatial queries

FlatCityBuf's spatial indexing enables fast bounding box queries:

{% raw %}

```python
# Query features within a bounding box
# Format: min_x, min_y, max_x, max_y
features = list(reader.query_bbox(84227.77, 445377.33, 85323.23, 446334.69))
print(f"Found {len(features)} features in bounding box")

# Process spatially filtered features
for feature in features:
    # Access only features within the specified area
    print(f"Feature {feature.id} is within the bounding box")

# This will shows like this:
# Found 101 features in bounding box
# Feature NL.IMBAG.Pand.0503100000019446 is within the bounding box
# ...
```

{% endraw %}

### Attribute queries

To query features based on attribute values, you must serialise the file with attribute indexing enabled.

{% raw %}

```bash
$ fcb ser -i delft.city.jsonl -o delft.fcb -A --attr-branching-factor 16
Successfully encoded to FCB
```

{% endraw %}

Query features based on attribute values:

{% raw %}

```python
# Create attribute filters
# Format: (attribute_name, operator, value)

# Exact match
id_filter = fcb.AttrFilter(
    "identificatie", fcb.Operator.Eq, "NL.IMBAG.Pand.0503100000019581"
)

one_building = list(reader.query_attr([id_filter]))
print(f"Found {len(one_building)} matching buildings")

# Numeric comparison
height_filter = fcb.AttrFilter("b3_h_dak_50p", fcb.Operator.Gt, 20.0)
buildings = list(reader.query_attr([height_filter]))
print(f"Found {len(buildings)} matching buildings")

# Query with multiple filters (AND logic)
tall_glass_buildings = list(
    reader.query_attr(
        [
            fcb.AttrFilter("b3_h_dak_50p", fcb.Operator.Gt, 30.0),
            fcb.AttrFilter("b3_is_glas_dak", fcb.Operator.Eq, True),
        ]
    )
)

print(f"Found {len(tall_glass_buildings)} tall glass buildings")

# This will show like this:
# Found 1 matching buildings
# Found 4 matching buildings
# Found 0 tall glass buildings
```

{% endraw %}

#### Available operators

You can try the following operators:

{% raw %}

```python
fcb.Operator.Eq    # Equal
fcb.Operator.Ne    # Not equal
fcb.Operator.Gt    # Greater than
fcb.Operator.Ge    # Greater than or equal
fcb.Operator.Lt    # Less than
fcb.Operator.Le    # Less than or equal
```

{% endraw %}

### HTTP and cloud access

Now you'll see the most powerful part of FlatCityBuf: retrieving data from a huge remote file (70GB!) over HTTP.

For remote FCB files, use the async reader:

<details>
<summary>Click to show example: Query a huge FlatCityBuf over HTTP in Python (async, streaming)</summary>

{% raw %}

```python
import asyncio
import flatcitybuf as fcb

async def read_remote_fcb():
    # Create async reader for HTTP URL
    async_reader = fcb.AsyncReader(
        "https://storage.googleapis.com/flatcitybuf/3dbag_all_index.fcb"
    )
    opened_reader = await async_reader.open()

    # Get file info
    info = opened_reader.info()
    print(f"Remote file has {info.feature_count} features")

    # Get CityJSON header
    cityjson = opened_reader.cityjson_header()
    print(f"CityJSON version: {cityjson.version}")

    # Async iteration - stream features one by one
    async_iter = opened_reader.select_all()

    count = 0
    for _ in range(2):  # Get first 2 features
        feature = await async_iter.next()
        if feature is None:
            break
        print(f"Feature {count}: {feature.id}")
        count += 1

    # Async spatial query
    bbox_iter = opened_reader.query_bbox(84227.77, 445377.33, 85323.23, 446334.69)
    count = 0
    for _ in range(2):  # Get first 2 features
        feature = await bbox_iter.next()
        if feature is None:
            break
        print(f"  Spatial feature {count + 1}: {feature.id}")
        count += 1

    # Async attribute query
    id_filter = fcb.AttrFilter(
        "identificatie",
        fcb.Operator.Eq,
        "NL.IMBAG.Pand.0503100000012869",
    )
    attr_iter = opened_reader.query_attr([id_filter])
    attr_features = await attr_iter.collect()
    print(f"Found {len(attr_features)} features with specific ID")
    if attr_features:
        print(f"  Found feature: {attr_features[0].id}")

# Run the async function

if __name__ == "__main__":
    asyncio.run(read_remote_fcb())
```

{% endraw %}

</details>

{% raw %}

```shell
python main.py
# This will show like this in milliseconds!
# Remote file has 10771547 features
# CityJSON version: 2.0
# Feature 0: NL.IMBAG.Pand.0983100000055544
# Feature 1: NL.IMBAG.Pand.0983100000061503
#   Spatial feature 1: NL.IMBAG.Pand.0503100000014644
#   Spatial feature 2: NL.IMBAG.Pand.0503100000019937
# Found 1 features with specific ID
#   Found feature: NL.IMBAG.Pand.0503100000012869
```

{% endraw %}

No matter how big the file is, you can query it in milliseconds! :D

---

## Using FlatCityBuf in the browser (WASM)

FlatCityBuf provides WebAssembly bindings for efficient CityJSON processing in web browsers.

### Setup and installation

Install the npm package:

{% raw %}

```bash
npm init -y # create package.json file
npm install @cityjson/flatcitybuf
```

{% endraw %}

Or include it in your `package.json`:

{% raw %}

```json
{
  "dependencies": {
    "@cityjson/flatcitybuf": "^0.2.0"
  }
}
```

{% endraw %}

### Reading FCB files over HTTP

Create a file called `index.html` and paste the following code into it. Open the HTML file in your browser and you'll see the UI like this:
![HTML to interact with FlatCityBuf over HTTP](../files/fcb_demo.png)

You can test it to fetch data with bounding box or attribute query.

You can download the complete HTML file here: [fcb_demo.html](../files/fcb_demo.html)

---

## Using FlatCityBuf with Rust

For maximum performance and control, you can use FlatCityBuf directly in Rust applications.

### Adding to your project

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

### Basic reading and spatial queries

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

### HTTP streaming

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

---

## Performance tips and best practises

### Indexing tips

#### Branching factor considerations for attribute indexing

The branching factor controls B+tree structure:

{% raw %}

```bash
# Default (256) - good for most cases
fcb ser -i data.city.jsonl -o data.fcb --attr-index height

# Higher (512) - faster queries, slightly larger files
fcb ser -i data.city.jsonl -o data.fcb \
  --attr-index height --attr-branching-factor 512

# Lower (128) - smaller files, slightly slower queries
fcb ser -i data.city.jsonl -o data.fcb \
  --attr-index height --attr-branching-factor 128
```

{% endraw %}

The larger the branching factor, the more nodes are fetched per one round trip to the server. In many cases, especially over HTTP, the round trip time dominates the total time more than the time to fetch redundant nodes.

**Recommendations:**

- **Default (256)**: Good balance for most datasets
- **Higher (512-1024)**: For query-heavy applications with huge datasets (100GB or more)
- **Lower (64-128)**: For datasets with a relatively small size (10GB or less)

#### Cardinality of attributes

Since the attribute index is a B+tree, attributes with higher cardinality (more unique values) will be better for indexing, whilst lower cardinality attributes will be less efficient or sometimes not worth indexing. As an extreme example, if you have an attribute with only 2 unique values (e.g. true/false), you won't get any benefit from indexing it.

### HTTP and cloud optimisation

#### Range request batching

Once the reader is initialised, the reader fetches features from the server when it's called with the `next()` method or relevant iterators (depending on which language you are using). If you use something like `collect()`, this fetches all features at once, which is not efficient. We recommend using a streaming approach instead (use `next` to fetch features when needed)

**Good:**

{% raw %}

```python
reader = reader.select_all()
for feature in reader:
    process(feature)
```

{% endraw %}

**Avoid:**

{% raw %}

```python
reader = reader.select_all().collect()
all_features = list(reader)
```

{% endraw %}

### Zero-copy benefits

Especially when you use FlatCityBuf in Rust, there are two ways to access data:

1. Use the `cur_feature()` method to get the current feature in FlatBuffer format
2. Use the `cur_cj_feature()` method to get the current feature in CityJSON format

The first one returns data in FlatBuffer format and achieves zero-copy deserialisation. If you access a specific field of the feature, only that field is loaded. On the other hand, the second one returns features in CityJSON format, which internally parses FlatBuffers into CityJSON objects (not zero-copy). Choose the appropriate one based on your use case.

**Zero-copy deserialisation:**

{% raw %}

```rust
while let Some(feature_buf) = reader.next()? {
    let cj_feature = feature_buf.cur_feature()?; // You get flatbuffer feature
    process(&cj_feature);
}
```

{% endraw %}

**Not zero-copy deserialisation:**

{% raw %}

```rust
while let Some(feature_buf) = reader.next()? {
    let cj_feature = feature_buf.cur_cj_feature()?;
    process(&cj_feature);
}
```

{% endraw %}

---

## Frequently asked questions

### Can I edit FCB files directly?

No, FCB files are binary and not designed for direct editing. To modify data:

1. Convert FCB back to CityJSONSeq: `fcb deser -i data.fcb -o data.city.jsonl`
2. Edit the CityJSONSeq file
3. Convert back to FCB: `fcb ser -i data.city.jsonl -o data.fcb`

### How do I update a single feature?

FCB files are immutable. To update features, you must regenerate the entire file. For frequently updated data, consider:

- Keeping source data in CityJSONSeq format
- Regenerating FCB files periodically
- Using FCB for read-heavy, write-light scenarios

### What's the maximum file size?

FlatCityBuf has been tested with files up to 70GB (complete 3DBAG dataset). Theoretical limits are much higher, constrained mainly by:

- Available disk space
- Memory for index construction during writing
- HTTP server capabilities for remote access

### Can I use FCB with other GIS tools?

Currently, FCB is primarily used with its own libraries.

### How do I choose between CityJSON and FlatCityBuf?

Use **CityJSON/CityJSONSeq** when:

- Files are small (<1GB)
- Human readability is important
- Frequent editing is needed
- Maximum tool compatibility is required

Use **FlatCityBuf** when:

- Files are large (>1GB)
- Read performance is critical
- Spatial/attribute queries are needed
- Cloud/HTTP access is required

### Does FlatCityBuf support textures and appearances?

Yes, FlatCityBuf preserves all CityJSON data including textures and appearances.
