---
layout: default
title: TypeScript
parent: FlatCityBuf
nav_order: 7
has_children: false
permalink: /flatcitybuf/typescript/
---

# Using FlatCityBuf with TypeScript and JavaScript

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

`@cityjson/flatcitybuf` is a **pure TypeScript reader**. It runs the same code in the browser and in Node.js, has one runtime dependency (`flatbuffers`), and ships no WebAssembly at all.

{: .warning }
This replaces the earlier WebAssembly binding. If you used `HttpFcbReader`/`WasmSpatialQuery`/`select_spatial`, see [migrating from the WASM binding](#migrating-from-the-wasm-binding).

Requirements: **ESM only** (import it, `require` will not work) and **Node ≥ 22.12** for the Node entry point. The browser entry point needs only `fetch` and `Blob`. The rendered API reference is at [cityjson.github.io/flatcitybuf/typescript](https://cityjson.github.io/flatcitybuf/typescript/), and the repository's guide is [docs/ts.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/ts.md).

## Installation

{% raw %}

```bash
npm install @cityjson/flatcitybuf
```

{% endraw %}

## Opening a file

### From a URL — HTTP range reads

{% raw %}

```ts
import { FcbReader } from '@cityjson/flatcitybuf'

const reader = await FcbReader.fromUrl('https://example.com/city.fcb')

console.log(reader.header.info.featuresCount)
console.log(reader.header.info.referenceSystem) // e.g. "EPSG:7415"
console.log(reader.header.info.columns.map((c) => c.name))
```

{% endraw %}

`fromUrl` validates the server's range support strictly: a server that ignores `Range` and answers `200`, or whose CORS configuration hides the `Content-Range` header, is rejected rather than silently mis-read. See [serving `.fcb` over HTTP]({{ '/flatcitybuf/datasets/#serving-your-own-fcb-files' | prepend: site.baseurl }}).

### From a `Blob` or `File` — drag-and-drop in the browser

{% raw %}

```ts
const file: File = /* from an <input type="file"> or a drop event */
const reader = await FcbReader.fromBlob(file)
```

{% endraw %}

`FcbReader.fromBytes(uint8Array)` reads from an in-memory buffer.

### From a local file in Node

The Node file reader lives behind a separate subpath, so the package root never imports `node:*` and stays usable in the browser:

{% raw %}

```ts
import { fromFile } from '@cityjson/flatcitybuf/node'

await using reader = await fromFile('./delft.fcb')
for await (const feature of await reader.selectAll()) {
  console.log(feature.id)
}
// `await using` closes the file handle on scope exit; otherwise call
// `await reader.close()` yourself.
```

{% endraw %}

## Streaming a whole file as CityJSONSeq

{% raw %}

```ts
for await (const line of reader.cityjson()) {
  console.log(JSON.stringify(line)) // metadata line first, then one feature per line
}
```

{% endraw %}

## Queries

`reader.select(options)` returns a `FeatureCursor`: an async-iterable whose `featuresCount` is the **total** number of matches, unaffected by `limit`/`offset`.

{% raw %}

```ts
// Bounding box
const inBox = await reader.select({
  spatial: { kind: 'bbox', value: [84227.77, 445377.33, 85323.23, 446334.69] },
})
console.log(inBox.featuresCount) // 101

// Point intersection, and nearest feature to a point
const atPoint = await reader.select({ spatial: { kind: 'point', value: [x, y] } })
const nearest = await reader.select({ spatial: { kind: 'nearest', value: [x, y] } })

// Attribute query: operator is Eq | Ne | Gt | Ge | Lt | Le, conditions are AND-ed
const tall = await reader.select({
  where: [{ field: 'b3_h_dak_50p', operator: 'Gt', value: 20 }],
})
for await (const feature of tall) {
  console.log(feature.id)
}

// Spatial AND attribute, with paging
const page = await reader.select({
  spatial: { kind: 'bbox', value: [84227.77, 445377.33, 85323.23, 446334.69] },
  where: [{ field: 'b3_h_dak_50p', operator: 'Ge', value: 10 }],
  limit: 50,
  offset: 0,
})
console.log(page.featuresCount) // total matches, not the page size
```

{% endraw %}

Notes:

- Attribute queries only work on columns that were indexed at write time (`fcb ser --attr-index …`); `reader.header.info.columns` lists the schema and `header.info.attributeIndices` the indexed ones.
- A `String` index stores keys truncated to 50 bytes, so it answers with *candidates*; the reader post-filters them against each feature's full attributes, so what you iterate are exact matches.
- `nearest` cannot be combined with `where`.
- Every query accepts an `AbortSignal` via `signal`, which is threaded into the in-flight reads.
- Querying a file with no spatial index throws `NoIndex`.

## Converting to CityJSON

{% raw %}

```ts
import { toCityJSONMetadata, toCityJSONFeature } from '@cityjson/flatcitybuf'

const metadata = toCityJSONMetadata(reader.header)
for await (const feature of await reader.selectAll()) {
  const cjFeature = toCityJSONFeature(feature, reader.header)
}
```

{% endraw %}

`Long`/`Int64` attribute values can exceed `Number.MAX_SAFE_INTEGER`. Pass an `Int64Policy` to choose how they are emitted: a lossy JS number (the default, which keeps the output JSON-serialisable), an exact decimal string, or a throw on any unsafe value.

## Examples

The repository ships [nine runnable example scripts](https://github.com/cityjson/flatcitybuf/blob/main/src/ts/examples/README.md), one per capability — inspecting a header, reading local and remote files, bbox and attribute queries, the per-object attribute schema, implementing your own `RangeReader`, and the `Int64Policy` choices. Every one of them is executed as a real subprocess *and* type-checked by the test suite on each run, so they cannot drift from the API, and the README records the exact output each one printed.

One of them never touches CityJSON. `CityObjectView.rawObject()` hands back the **encoded** geometry — the format's own flat count arrays (`solids`/`shells`/`surfaces`/`strings`, plus the flat `boundaries` index list) — and the package now also exports what you need to interpret it: `GeometryType`, `SemanticSurfaceType`, `NULL_INDEX`, `geometryTypeName`, `semanticSurfaceTypeName`, `decodeBoundaries` and `decodeSemantics`. That is the cheapest representation to compute over, since nothing has to be nested, allocated or turned into JSON; [`geometry-analysis.ts`](https://github.com/cityjson/flatcitybuf/blob/main/src/ts/examples/geometry-analysis.ts) sums surface area per semantic surface type straight from it.

{: .warning }
Nesting depth comes from the geometry's `type`, never from which array is populated: a `Solid` with one shell and a `MultiSolid` with one solid flatten to byte-identical arrays.

## A complete example

The [web viewer]({{ '/flatcitybuf/viewer/' | prepend: site.baseurl }}) is built on this package: it opens a `.fcb` over HTTP (the full 3DBAG by default), queries it as you pan the map, and renders the result with deck.gl — all in the browser, with no server component.

## Migrating from the WASM binding

| Old WASM API | New TypeScript API |
| --- | --- |
| `new HttpFcbReader(url)` | `await FcbReader.fromUrl(url)` |
| *(browser only, not available)* | `FcbReader.fromBlob(blob)`, `FcbReader.fromBytes(bytes)`, `fromFile(path)` |
| `reader.meta()` | `reader.header` |
| `reader.cityjson()` | `toCityJSONMetadata(reader.header)`, or `reader.cityjson()` to stream metadata **and** features |
| `reader.select_all()` | `reader.selectAll()` |
| `reader.select_spatial(q)` | `reader.select({ spatial })` |
| `reader.select_attr_query(q)` | `reader.select({ where })` |
| `…_paged(q, limit, offset)` | `reader.select({ …, limit, offset })` |
| `new WasmSpatialQuery({ type: 'bbox', minX, … })` | `{ kind: 'bbox', value: [minX, minY, maxX, maxY] }` |
| `new WasmAttrQuery([[field, op, value]])` | `where: [{ field, operator, value }]` |
| `const f = await iter.next()` | `for await (const feature of cursor)` |
| `iter.features_count()` | `cursor.featuresCount` |
| `iter.cur_cj_feature()` | `toCityJSONFeature(feature, reader.header)` |
| `cjToObj(...)`, `cjseqToCj(...)` | dropped — conversion to OBJ or merging a CityJSONSeq are CityJSON-tooling concerns, not reader concerns |

Beyond the shape change, the native reader fixes several defects the WASM binding shipped with: attribute queries against non-`Double` columns, string query values longer than 50 bytes, non-default R-tree node sizes over HTTP, and a range client that accepted a `200` full-body response as if it were the requested range.
