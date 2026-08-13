---
layout: default
title: FAQ
parent: FlatCityBuf
nav_order: 8
has_children: false
permalink: /flatcitybuf/faq/
---

# Performance tips and FAQ

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## How do I choose between CityJSON and FlatCityBuf?

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

## Which implementation should I use?

All four read the same files and produce the same CityJSON; they differ in what else they can do.

| | [Rust]({{ '/flatcitybuf/rust/' | prepend: site.baseurl }}) | [C++]({{ '/flatcitybuf/cpp/' | prepend: site.baseurl }}) | [Python]({{ '/flatcitybuf/python/' | prepend: site.baseurl }}) | [TypeScript]({{ '/flatcitybuf/typescript/' | prepend: site.baseurl }}) |
| --- | --- | --- | --- | --- |
| Read | ✅ | ✅ | ✅ | ✅ |
| Write | ✅ | ✅ | — | — |
| HTTP | ✅ async | ✅ (libcurl, opt-in) | ✅ synchronous | ✅ browser + Node.js |
| Install | `cargo add fcb_core` | vcpkg or CMake, no Rust toolchain | `pip install flatcitybuf`, no compiler | `npm install @cityjson/flatcitybuf`, no WASM |
| Best for | pipelines, servers, maximum speed | native apps, engines, plugins | analysis and scripting | web apps and Node.js tools |

If you only need to convert or inspect files, you do not need any of them — use [the `fcb` CLI]({{ '/flatcitybuf/conversion/' | prepend: site.baseurl }}).

## Can I edit FCB files directly?

No, FCB files are binary and not designed for direct editing. To modify data:

1. Convert FCB back to CityJSONSeq: `fcb deser data.fcb data.city.jsonl`
2. Edit the CityJSONSeq file
3. Convert back to FCB: `fcb ser data.city.jsonl data.fcb`

## How do I update a single feature?

FCB files are immutable. To update features, you must regenerate the entire file. For frequently updated data, consider:

- Keeping source data in CityJSONSeq format
- Regenerating FCB files periodically
- Using FCB for read-heavy, write-light scenarios

## What's the maximum file size?

FlatCityBuf has been tested with files up to ~68GB (the complete 3DBAG, 10.7M features). Theoretical limits are much higher, constrained mainly by:

- Available disk space
- Memory for index construction during writing
- HTTP server capabilities for remote access

## Can I use FCB with other GIS tools?

Currently, FCB is primarily used with its own libraries. Converting to CityJSON/CityJSONSeq (`fcb deser`) gets you into every CityJSON-aware tool, and the [web viewer]({{ '/flatcitybuf/viewer/' | prepend: site.baseurl }}) also exports a query result as CityJSON, CityJSONSeq or OBJ.

## Does FlatCityBuf support textures and appearances?

Yes. FlatCityBuf preserves all CityJSON data including materials and textures, and all four readers decode them: `appearance` (with `materials`, `textures` and `vertices-texture`) comes back on the converted CityJSON feature, and the per-surface `material`/`texture` mappings come back on the geometry.

## Why does my remote file fail to open in the browser?

Almost always CORS: the server must expose the `Content-Range` header. See [serving your own `.fcb` files]({{ '/flatcitybuf/datasets/#serving-your-own-fcb-files' | prepend: site.baseurl }}).

## Indexing tips

### Which attributes can be queried?

Only the ones that were given a B+tree index when the file was written. Everything else is still readable, just not queryable. `fcb inspect` lists what a file has — in its *Columns* tab, or in the `--static` report.

### Branching factor considerations for attribute indexing

The branching factor controls the B+tree structure:

{% raw %}
```bash
# Default (256) - good for most cases
fcb ser data.city.jsonl data.fcb --attr-index height

# Higher (512) - faster queries, slightly larger files
fcb ser data.city.jsonl data.fcb \
  --attr-index height --attr-branching-factor 512

# Lower (128) - smaller files, slightly slower queries
fcb ser data.city.jsonl data.fcb \
  --attr-index height --attr-branching-factor 128
```
{% endraw %}

The larger the branching factor, the more nodes are fetched per one round trip to the server. In many cases, especially over HTTP, the round trip time dominates the total time more than the time to fetch redundant nodes.

**Recommendations:**

- **Default (256)**: Good balance for most datasets
- **Higher (512-1024)**: For query-heavy applications with huge datasets (100GB or more)
- **Lower (64-128)**: For datasets with a relatively small size (10GB or less)

{: .info }
`--attr-branching-factor` is the attribute B+tree's; the spatial R-tree has its own, unrelated `--index-node-size` (default 16). The readers always use whatever node size the header declares, so a non-default choice is safe.

### Cardinality of attributes

Since the attribute index is a B+tree, attributes with higher cardinality (more unique values) will be better for indexing, whilst lower cardinality attributes will be less efficient or sometimes not worth indexing. As an extreme example, if you have an attribute with only 2 unique values (e.g. true/false), you won't get any benefit from indexing it.

### Long string values

String index keys are stored truncated to 50 bytes (100 for JSON and binary columns), so for long values the index returns *candidates* rather than answers. Every reader verifies each candidate against the full, untruncated attribute value before handing it to you, so results are exact — but a column whose values only differ after 50 bytes will do more work than one that differs early.

## HTTP and cloud optimisation

### Stream, don't collect

Once the reader is initialised, features are fetched from the server as you ask for them. Materialising the whole result set (`collect()`, `list()`, and the like) fetches everything at once, which is exactly what the format is designed to avoid. Iterate instead.

**Good:**

{% raw %}

```python
for hit in hits:
    process(reader.feature_at(hit))
```

{% endraw %}

**Avoid:**

{% raw %}

```python
all_features = [reader.feature_at(hit) for hit in hits]
process(all_features)
```

{% endraw %}

The same applies in TypeScript (`for await (const feature of cursor)` rather than collecting the cursor into an array) and in Rust (`while let Some(feature) = iter.next()`).

### Use `limit` when you only need a page

The readers' paged queries (`limit`/`offset` in TypeScript and Rust) stop after the features you asked for, while still reporting the total number of matches. That is what the [web viewer]({{ '/flatcitybuf/viewer/' | prepend: site.baseurl }}) uses to keep a country-scale dataset interactive.

## Zero-copy benefits

Especially when you use FlatCityBuf in Rust, there are two ways to access data:

1. Use the `cur_feature()` method to get the current feature in FlatBuffer format
2. Use the `cur_cj_feature()` method to get the current feature in CityJSON format

The first one returns data in FlatBuffer format and achieves zero-copy deserialisation. If you access a specific field of the feature, only that field is loaded. On the other hand, the second one returns features in CityJSON format, which internally parses FlatBuffers into CityJSON objects (not zero-copy). Choose the appropriate one based on your use case.

**Zero-copy deserialisation:**

{% raw %}

```rust
while let Some(feature_buf) = reader.next()? {
    let feature = feature_buf.cur_feature()?; // You get flatbuffer feature
    process(&feature);
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

The same trade-off exists in the other implementations: C++'s `it.current()` is a view over the bytes, and `to_cityjson_feature` is what builds a JSON tree; Python's `Feature` and TypeScript's feature object are likewise cheaper than the `to_cityjson_feature` conversion.

## How do I know the implementations agree?

The repository has a [conformance corpus](https://github.com/cityjson/flatcitybuf/tree/main/conformance): `.fcb` files covering the awkward cases — single-feature files, prefix-colliding strings, duplicate keys, zero-area extents, geometry templates, appearance at every nesting depth, files declaring an unknown feature count. Each has an expected output produced by the Rust reader, and the C++, Python and TypeScript test suites must reproduce it line for line on the same bytes.
