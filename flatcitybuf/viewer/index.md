---
layout: default
title: Web viewer
parent: FlatCityBuf
nav_order: 3
has_children: false
permalink: /flatcitybuf/viewer/
---

# The FlatCityBuf web viewer

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

<i class="fas fa-external-link-alt"></i> **[flatcitybuf-prototype.hideba.me](https://flatcitybuf-prototype.hideba.me)**

A browser viewer for FlatCityBuf, built on the [TypeScript reader]({{ '/flatcitybuf/typescript/' | prepend: site.baseurl }}). There is **no server component and no WebAssembly in the read path**: the browser opens a `.fcb` over HTTP range requests, walks its indices, decodes only the features a query matches, and renders them with [deck.gl](https://deck.gl/) on a [MapLibre](https://maplibre.org/) basemap.

By default it opens the complete [3DBAG](https://3dbag.nl) as one ~68GB FlatCityBuf file with ~10.7 million buildings. Nothing is preprocessed or tiled for the viewer: it is the same file the CLI writes and the other implementations read.

The source is in [`examples/web`](https://github.com/cityjson/flatcitybuf/tree/main/examples/web), and it is deployed on every push to `main`. It supersedes the earlier WASM-based prototype.

## What you can do

- **Open a file** — the default full-3DBAG URL, another URL, or a local `.fcb` (pick or drop it; local files never leave your machine).
- **Follow camera mode** (on by default) — query whatever is on screen, re-querying as you pan and zoom. Zoom in past the "get closer" hint to fetch; the rectangle drawn on the map shows the bounding box each query actually fetched.
- **Draw a bounding box** to query one area, or set an **attribute query** on any indexed column.
- **Level of Detail** — switch between LoD 1.2, 1.3, 2.2 and LoD 0 roofprints.
- **Colour by an attribute**, and click a building for a popup with its attributes.
- **Inspect the header** — feature count, CRS, transform, and the full column schema with which columns are queryable.
- **Export the current result** (see below).

## Export

The export downloads the **current query result** — exactly the features on screen (the active query, up to the render limit) — not the whole dataset. For the default 3DBAG file that is the difference between a few thousand buildings and 10.7 million.

| Format | Output |
| --- | --- |
| CityJSON | every rendered feature merged into a single `.city.json` |
| CityJSONSeq | `.city.jsonl`: one metadata line, then one feature per line |
| OBJ | a triangulated Wavefront `.obj` mesh |

Conversion runs entirely in the browser. CityJSONSeq is assembled in pure TypeScript; the merged CityJSON and the OBJ conversion reuse a prebuilt WebAssembly helper that is lazy-loaded on first use, so the `.wasm` is only fetched if you actually export to one of those two formats. The OBJ includes every LoD present in the data, not just the LoD currently rendered.

## Running it locally

The viewer consumes the TypeScript reader from the same repository, so build that first:

{% raw %}

```bash
git clone https://github.com/cityjson/flatcitybuf.git
cd flatcitybuf/src/ts && npm install && npm run build

cd ../../examples/web
npm install
npm run dev
```

{% endraw %}

## Pointing it at your own data

Any `.fcb` on any HTTP server works, as long as the server supports range requests **and** exposes the `Content-Range` header to the browser. See [serving your own `.fcb` files]({{ '/flatcitybuf/datasets/#serving-your-own-fcb-files' | prepend: site.baseurl }}) — a missing CORS header is by far the most common reason a remote file fails to open.
