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

We offer a few example FlatCityBuf files for testing:

- [Delft (7MB)](https://github.com/cityjson/flatcitybuf/blob/main/examples/data/delft.fcb) — 1115 buildings from the 3DBAG, all 44 attributes indexed. The file every example on this site uses.
- [3DBAG small (3.4GB)](https://storage.googleapis.com/flatcitybuf/3dbag_subset_all_index.fcb)
- [3DBAG all (~68GB)](https://storage.googleapis.com/flatcitybuf/3dbag_all_index.fcb) — the complete 3DBAG, 10 771 547 features, with spatial indexing and all attributes indexed

The two large files are hosted on Google Cloud Storage with range requests and CORS enabled, so you can query them directly from a script or from the browser — including from the [web viewer]({{ '/flatcitybuf/viewer/' | prepend: site.baseurl }}), which opens the 68GB one by default.

You can always convert any CityJSONSeq file to a FlatCityBuf (and vice-versa), see [the CLI page]({{ '/flatcitybuf/conversion/' | prepend: site.baseurl }}).

## Looking inside a file

Before writing any code, `fcb info` (a one-shot summary) and `fcb inspect` (an interactive terminal UI) tell you what a file contains — how many features, in which CRS, and, crucially, **which attributes are indexed** and therefore queryable. `fcb inspect` accepts a URL and reads only the header, so it is instant even on the 68GB file:

{% raw %}

```bash
fcb info delft.fcb
fcb inspect https://storage.googleapis.com/flatcitybuf/3dbag_all_index.fcb
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

For a Google Cloud Storage bucket:

{% raw %}

```bash
echo '[{"maxAgeSeconds":3600,"method":["GET","HEAD","OPTIONS"],"origin":["*"],"responseHeader":["Content-Type","Content-Range","Accept-Ranges"]}]' > cors.json
gsutil cors set cors.json gs://your-bucket
```

{% endraw %}

After changing CORS, hard-reload the page: the browser may have cached the earlier failed response.
