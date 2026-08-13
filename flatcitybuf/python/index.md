---
layout: default
title: Python
parent: FlatCityBuf
nav_order: 6
has_children: false
permalink: /flatcitybuf/python/
---

# Using FlatCityBuf with Python

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

The `flatcitybuf` package is a **pure-Python reader**: it parses the FlatBuffers bytes itself, so there is no compiled extension, no Rust toolchain and no per-platform wheel — one `py3-none-any` wheel on CPython 3.9+, with `flatbuffers` as its only required dependency.

{: .warning }
This page documents the pure-Python reader (version 0.3.0 and later). It is **not** a drop-in replacement for the older PyO3 bindings (0.2.0 and earlier) — see [Migrating from the old bindings](#migrating-from-the-old-bindings) at the bottom.

## Installation

{% raw %}

```bash
pip install flatcitybuf

# with numpy, worth it for large files
pip install "flatcitybuf[numpy]"
```

{% endraw %}

PyPI serves the pure-Python reader from version `0.3.1` onwards; `0.2.0` and earlier are the retired PyO3 extension. `numpy` is optional: it speeds up bulk vertex and geometry decoding by roughly 2.4×, and every code path has a pure-Python fallback when it is absent.

If you had the package pinned before, check what you have — anything below `0.3.0` is the old extension:

{% raw %}

```bash
python -c "import flatcitybuf; print(flatcitybuf.__version__)"
```

{% endraw %}

For a development checkout with [uv](https://docs.astral.sh/uv/), and the full API reference, see [docs/py.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/py.md).

## Opening a file and reading its header

{% raw %}

```python
import flatcitybuf as fcb

reader = fcb.FcbReader.open_file("delft.fcb")

info = reader.header.info
print(f"Features: {info.features_count}")
print(f"CityJSON version: {info.cityjson_version}")
print(f"CRS: {info.crs}")
print(f"Extent: {info.geographical_extent}")
print(f"Columns: {len(info.columns)}")

# The CityJSONSeq metadata line (transform, metadata, geometry templates...)
metadata = fcb.to_cityjson_metadata(reader.header)
print(metadata["transform"])

# Features: 1115
# CityJSON version: 2.0
# CRS: EPSG:7415
# Extent: (84501.5546875, 445805.03125, -3.746997833251953, 85675.234375, 446983.46875, 95.04200744628906)
# Columns: 44
# {'scale': [0.001, 0.001, 0.001], 'translate': [85088.390625, 446394.25, 45.64800262451172]}
```

{% endraw %}

## Iterating through features

`select_all()` streams every feature in stored (Hilbert) order. `to_cityjson_feature` turns one into a plain CityJSON dict — exactly the shape the CityJSON specification describes, so `cj["CityObjects"]`, `cj["vertices"]`, `cj["appearance"]` are all there.

{% raw %}

```python
for i, feature in enumerate(reader.select_all()):
    cj = fcb.to_cityjson_feature(feature, reader.header)
    print(f"{cj['id']}: {len(cj['CityObjects'])} city object(s)")

    for obj_id, city_object in cj["CityObjects"].items():
        print(f"  {obj_id} ({city_object['type']})")
        for geometry in city_object.get("geometry", []):
            print(f"    {geometry['type']}, LoD {geometry.get('lod')}")

    if i >= 1:
        break

# NL.IMBAG.Pand.0503100000031902: 2 city object(s)
#   NL.IMBAG.Pand.0503100000031902-0 (BuildingPart)
#     MultiSurface, LoD 0
#   ...
```

{% endraw %}

{: .info }
Vertices are quantised integers: the real coordinate is `v[n] * transform["scale"][n] + transform["translate"][n]`, and the transform lives on the **metadata** object, not on the feature.

For analysis you can skip CityJSON entirely: `fcb.raw_city_object(view)` and `fcb.raw_city_feature(feature)` return the generated FlatBuffers tables holding the **encoded** geometry — the format's own flat count arrays (`Solids`/`Shells`/`Surfaces`/`Strings`, plus the flat `Boundaries` index list) and the quantised vertices they index into — and `fcb.geometry_type_name` / `fcb.semantic_surface_type_name` turn the raw type tags into their CityJSON names. Nothing has to be nested, allocated or turned into JSON to get a number out of it; [`examples/geometry_analysis.py`](https://github.com/cityjson/flatcitybuf/blob/main/src/py/examples/geometry_analysis.py) sums surface area per semantic surface type that way, and the rest of the [runnable examples](https://github.com/cityjson/flatcitybuf/blob/main/src/py/examples/README.md) cover one capability each. Nesting depth comes from `Geometry.Type()`, never from which array is populated: a `Solid` with one shell and a `MultiSolid` with one solid flatten to byte-identical arrays.

## Spatial queries

`search_rtree` answers a bounding box from the packed R-tree. Like the attribute query below, it returns `SearchResultItem`s — byte offsets into the feature section — which `feature_at` turns into a feature. That is deliberate: you only pay for decoding the features you actually want.

{% raw %}

```python
hits = fcb.search_rtree(
    reader.range_reader,
    reader.header.layout.rtree_begin,
    reader.header.info.features_count,
    reader.header.info.index_node_size,
    (84227.77, 445377.33, 85323.23, 446334.69),  # min_x, min_y, max_x, max_y
)
print(f"Found {len(hits)} features in the bounding box")

for hit in hits[:3]:
    cj = fcb.to_cityjson_feature(reader.feature_at(hit), reader.header)
    print(" ", cj["id"])

# Found 101 features in the bounding box
#   NL.IMBAG.Pand.0503100000019446
#   ...
```

{% endraw %}

## Attribute queries

Attribute queries need the attribute to have been indexed when the file was written:

{% raw %}

```bash
$ fcb ser delft.city.jsonl delft.fcb -A --attr-branching-factor 256
```

{% endraw %}

A condition is a column name, an operator and a **typed** `KeyValue` whose type must match the column's type on disk. Multiple conditions are AND-ed.

{% raw %}

```python
# Numeric comparison
tall = reader.select_attr([
    fcb.AttrCondition("b3_h_dak_50p", fcb.Operator.GT, fcb.KeyValue.from_f64(20.0))
])
print(f"{len(tall)} buildings taller than 20m")
for hit in tall:
    print(" ", fcb.to_cityjson_feature(reader.feature_at(hit), reader.header)["id"])

# Exact string match
one = reader.select_attr([
    fcb.AttrCondition(
        "identificatie",
        fcb.Operator.EQ,
        fcb.KeyValue.from_string(fcb.KeyKind.STRING50, "NL.IMBAG.Pand.0503100000019581"),
    )
])
print(f"{len(one)} matching building")

# Several conditions, AND-ed
tall_and_flat = reader.select_attr([
    fcb.AttrCondition("b3_h_dak_50p", fcb.Operator.GT, fcb.KeyValue.from_f64(20.0)),
    fcb.AttrCondition("b3_dak_type", fcb.Operator.EQ,
                      fcb.KeyValue.from_string(fcb.KeyKind.STRING50, "slanted")),
])

# 4 buildings taller than 20m
#   NL.IMBAG.Pand.0503100000025026
#   NL.IMBAG.Pand.0503100000032914
#   NL.IMBAG.Pand.0503100000025170
#   NL.IMBAG.Pand.0503100000031390
# 1 matching building
```

{% endraw %}

Operators are `fcb.Operator.EQ`, `NE`, `GT`, `GE`, `LT`, `LE` (upper case).

`KeyValue` constructors follow the column type: `from_f64`, `from_f32`, `from_i64`, `from_u64`, `from_i32`, `from_u32`, `from_bool`, `from_string(KeyKind.STRING50, ...)`, and so on.

{: .info }
String index keys are truncated to 50 bytes, so the index returns *candidates*; `select_attr` re-checks each one against the full attribute value before returning it. Pass `exact_index_only=True` to skip that check and take the raw candidates.

## HTTP and cloud access

The same reader works on a remote file: swap the range reader. `HttpRangeReader` issues HTTP range requests with the standard library's `urllib.request` (no third-party dependency, and no `asyncio` — reads are synchronous), and `BufferedRangeReader` caches around it so index traversal does not re-fetch the same bytes.

{% raw %}

```python
import flatcitybuf as fcb

URL = "https://flatcitybuf.open3d.city/data/3dbag_all_index.fcb"

source = fcb.BufferedRangeReader(fcb.HttpRangeReader(URL))
reader = fcb.FcbReader.open(source)

info = reader.header.info
print(f"{info.features_count} features, CRS {info.crs}")

hits = fcb.search_rtree(
    reader.range_reader,
    reader.header.layout.rtree_begin,
    info.features_count,
    info.index_node_size,
    (120000, 486000, 120200, 486200),
)
print(f"{len(hits)} features in the bbox")

for hit in hits[:3]:
    print(" ", fcb.to_cityjson_feature(reader.feature_at(hit), reader.header)["id"])

# 10771547 features, CRS EPSG:7415
# 114 features in the bbox
#   NL.IMBAG.Pand.0363100012160936
#   ...
```

{% endraw %}

That file is ~68GB and the query never downloads more than the index nodes and the matching features.

## Migrating from the old bindings

Version 0.3.0 replaced the PyO3 extension with this pure-Python reader. The import name is the same, the API is not:

| Old (PyO3, ≤ 0.2.0) | New (pure Python, ≥ 0.3.0) |
| --- | --- |
| `fcb.Reader(path)` | `fcb.FcbReader.open_file(path)` |
| `reader.info()` | `reader.header.info` |
| `reader.cityjson_header()` | `fcb.to_cityjson_metadata(reader.header)` |
| `reader.query_bbox(minx, miny, maxx, maxy)` | `fcb.search_rtree(...)` + `reader.feature_at(hit)` |
| `reader.query_attr([...])` | `reader.select_attr([...])` + `reader.feature_at(hit)` |
| `fcb.AttrFilter(name, fcb.Operator.Eq, 20.0)` | `fcb.AttrCondition(name, fcb.Operator.EQ, fcb.KeyValue.from_f64(20.0))` |
| `feature.city_objects`, `feature.id` (classes) | `fcb.to_cityjson_feature(feature, reader.header)` → a CityJSON dict |
| `fcb.AsyncReader(url)`, `await reader.open()` | `fcb.FcbReader.open(fcb.HttpRangeReader(url))` — synchronous |
| `pip install flatcitybuf` + platform wheel | one universal wheel, no compiler |

The one real regression is the async API: the old bindings had `AsyncReader`/`AsyncFeatureIterator` on a tokio runtime, and the pure-Python reader deliberately has no `asyncio` story. If you need concurrent remote reads, run the synchronous reader in a thread pool — or use [Rust]({{ '/flatcitybuf/rust/' | prepend: site.baseurl }}) or [TypeScript]({{ '/flatcitybuf/typescript/' | prepend: site.baseurl }}), which are async.
