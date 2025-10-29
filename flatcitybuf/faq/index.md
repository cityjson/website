---
layout: default
title: FAQ
parent: FlatCityBuf
nav_order: 7
has_children: false
permalink: /flatcitybuf/faq/
---

# Performance tips and FAQ

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Indexing tips

### Branching factor considerations for attribute indexing

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

### Cardinality of attributes

Since the attribute index is a B+tree, attributes with higher cardinality (more unique values) will be better for indexing, whilst lower cardinality attributes will be less efficient or sometimes not worth indexing. As an extreme example, if you have an attribute with only 2 unique values (e.g. true/false), you won't get any benefit from indexing it.

## HTTP and cloud optimisation

### Range request batching

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

## Zero-copy benefits

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


## Can I edit FCB files directly?

No, FCB files are binary and not designed for direct editing. To modify data:

1. Convert FCB back to CityJSONSeq: `fcb deser -i data.fcb -o data.city.jsonl`
2. Edit the CityJSONSeq file
3. Convert back to FCB: `fcb ser -i data.city.jsonl -o data.fcb`

## How do I update a single feature?

FCB files are immutable. To update features, you must regenerate the entire file. For frequently updated data, consider:

- Keeping source data in CityJSONSeq format
- Regenerating FCB files periodically
- Using FCB for read-heavy, write-light scenarios

## What's the maximum file size?

FlatCityBuf has been tested with files up to 70GB (complete 3DBAG dataset). Theoretical limits are much higher, constrained mainly by:

- Available disk space
- Memory for index construction during writing
- HTTP server capabilities for remote access

## Can I use FCB with other GIS tools?

Currently, FCB is primarily used with its own libraries.

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

## Does FlatCityBuf support textures and appearances?

Yes, FlatCityBuf preserves all CityJSON data including textures and appearances.
