---
layout: default
title: Python bindings
parent: FlatCityBuf
nav_order: 3
has_children: false
permalink: /flatcitybuf/python/
---

# Using FlatCityBuf with Python

## Table of contents

{: .no_toc .text-delta }

1. TOC
{:toc}

The Python bindings provide a convenient interface for reading and querying FlatCityBuf files.

## Installation Python bindings

{% raw %}

```bash
pip install flatcitybuf
```

{% endraw %}

For development or building from [source (the main FlatCityBuf repository)](https://github.com/cityjson/flatcitybuf):

{% raw %}

```bash
pip install maturin
cd flatcitybuf/src/rust/fcb_py
maturin develop --features http
```

{% endraw %}

## Basic reading

### Opening local files

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

### Iterating through features

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

## Spatial queries

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

## Attribute queries

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

### Available operators

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

## HTTP and cloud access

Now you'll see the most powerful part of FlatCityBuf: retrieving data from a huge remote file (70GB!) over HTTP.

For remote FCB files, use the async reader:

Query a huge FlatCityBuf over HTTP in Python (async, streaming)

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

```shell
$ python3 main.py
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

No matter how big the file is, you can query it in milliseconds! :D
