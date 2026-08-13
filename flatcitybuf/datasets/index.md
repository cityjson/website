---
layout: default
title: Datasets
parent: FlatCityBuf
nav_order: 1
has_children: false
permalink: /flatcitybuf/datasets/
---

# FlatCityBuf example datasets

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Ready-made files

The smallest one first: [Delft (7MB)](https://github.com/cityjson/flatcitybuf/blob/main/examples/data/delft.fcb) — 1115 buildings from the 3DBAG, all 44 attributes indexed. It lives in the repository, and every example on this site uses it.

The published files are served from `https://flatcitybuf.open3d.city/data/`, on Cloudflare R2 with range requests and CORS enabled, so you can query any of them straight from a script or from the browser — including from the [web viewer]({{ '/flatcitybuf/viewer/' | prepend: site.baseurl }}), which opens the 68GB one by default. Sizes are decimal (1 MB = 10⁶ bytes).

| Name | Size | URL |
| --- | --- | --- |
| `3DBAG.city.fcb` | 7.1 MB | <https://flatcitybuf.open3d.city/data/3DBAG.city.fcb> |
| `3DBV.city.fcb` | 321.0 MB | <https://flatcitybuf.open3d.city/data/3DBV.city.fcb> |
| `3dbag_subset.city.fcb` | 2.32 GB | <https://flatcitybuf.open3d.city/data/3dbag_subset.city.fcb> |
| `Helsinki.city.fcb` | 388.1 MB | <https://flatcitybuf.open3d.city/data/Helsinki.city.fcb> |
| `Helsinki_tex.city.fcb` | 598.6 MB | <https://flatcitybuf.open3d.city/data/Helsinki_tex.city.fcb> |
| `Ingolstadt.city.fcb` | 3.3 MB | <https://flatcitybuf.open3d.city/data/Ingolstadt.city.fcb> |
| `Montreal.city.fcb` | 5.0 MB | <https://flatcitybuf.open3d.city/data/Montreal.city.fcb> |
| `NYC.fcb` | 85.0 MB | <https://flatcitybuf.open3d.city/data/NYC.fcb> |
| `Railway.city.fcb` | 3.9 MB | <https://flatcitybuf.open3d.city/data/Railway.city.fcb> |
| `Rotterdam.fcb` | 3.0 MB | <https://flatcitybuf.open3d.city/data/Rotterdam.fcb> |
| `Vienna.city.fcb` | 4.4 MB | <https://flatcitybuf.open3d.city/data/Vienna.city.fcb> |
| `Zurich.city.fcb` | 206.3 MB | <https://flatcitybuf.open3d.city/data/Zurich.city.fcb> |
| `plateau_takeshiba_bldg.city.fcb` | 83.8 MB | <https://flatcitybuf.open3d.city/data/plateau_takeshiba_bldg.city.fcb> |
| `plateau_takeshiba_brid.city.fcb` | 5.5 MB | <https://flatcitybuf.open3d.city/data/plateau_takeshiba_brid.city.fcb> |
| `plateau_takeshiba_rwy.city.fcb` | 4.5 MB | <https://flatcitybuf.open3d.city/data/plateau_takeshiba_rwy.city.fcb> |
| `plateau_takeshiba_tran.city.fcb` | 28.4 MB | <https://flatcitybuf.open3d.city/data/plateau_takeshiba_tran.city.fcb> |
| `plateau_takeshiba_tun.city.fcb` | 4.9 MB | <https://flatcitybuf.open3d.city/data/plateau_takeshiba_tun.city.fcb> |
| `plateau_takeshiba_veg.city.fcb` | 2.5 MB | <https://flatcitybuf.open3d.city/data/plateau_takeshiba_veg.city.fcb> |
| `tokyo_plateau.city.fcb` | 232.1 MB | <https://flatcitybuf.open3d.city/data/tokyo_plateau.city.fcb> |
| `3dbag_all_index.fcb` | 68.55 GB | <https://flatcitybuf.open3d.city/data/3dbag_all_index.fcb> |
| `3dbag_subset_all_index.fcb` | 3.81 GB | <https://flatcitybuf.open3d.city/data/3dbag_subset_all_index.fcb> |
| `3dbag_subset2_all_index.fcb` | 7.59 GB | <https://flatcitybuf.open3d.city/data/3dbag_subset2_all_index.fcb> |

The three `*_all_index.fcb` files at the bottom are the benchmark set: the whole 3DBAG (10 771 547 features) and two subsets, written with **every attribute indexed**. The rest is one file per source dataset.

Every dataset also exists as CityJSONSeq under `https://cityjson.open3d.city/cityjsonseq/`, same stem with a `.jsonl` extension. That second table, and the notes on range requests and CORS, are in the repository's canonical inventory: [docs/data.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/data.md).

You can always convert any CityJSONSeq file to a FlatCityBuf (and vice-versa), see [the CLI page]({{ '/flatcitybuf/conversion/' | prepend: site.baseurl }}).

## Looking inside a file

Before writing any code, `fcb inspect` tells you what a file contains — how many features, in which CRS, and, crucially, **which attributes are indexed** and therefore queryable. It opens a terminal UI when it has a terminal and prints a static report otherwise (or with `--static`), and it accepts a URL, reading only the header bytes — so it is instant even on the 68GB file:

{% raw %}

```bash
fcb inspect delft.fcb
fcb inspect --static delft.fcb
fcb inspect https://flatcitybuf.open3d.city/data/3dbag_all_index.fcb
```

{% endraw %}

See [inspecting a file]({{ '/flatcitybuf/conversion/#inspecting-a-file' | prepend: site.baseurl }}).

## Serving your own `.fcb` files

FlatCityBuf needs no special server: static hosting is enough, as long as the server

1. **supports HTTP range requests** (`Accept-Ranges: bytes`, and honours `Range` with a `206` response), and
2. **exposes the `Content-Range` header to the browser** when the file is on another origin.

The second one is the usual stumbling block. Browsers hide response headers on cross-origin requests unless the server explicitly exposes them, and a reader learns the file size from `Content-Range` — so it refuses to guess and reports something like *"sent a 206 response without an accessible Content-Range header"*. The fix is a CORS header:

{% raw %}

```
Access-Control-Expose-Headers: Content-Range, Accept-Ranges
```

{% endraw %}

If you host on Google Cloud Storage:

{% raw %}

```bash
echo '[{"maxAgeSeconds":3600,"method":["GET","HEAD","OPTIONS"],"origin":["*"],"responseHeader":["Content-Type","Content-Range","Accept-Ranges"]}]' > cors.json
gsutil cors set cors.json gs://your-bucket
```

{% endraw %}

After changing CORS, hard-reload the page: the browser may have cached the earlier failed response.

To check a host before pointing a reader at it, use the one-line `curl` recipe in [docs/data.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/data.md#notes) — a `206` answer with an `Accept-Ranges` and a `Content-Range` header is all FlatCityBuf needs.
