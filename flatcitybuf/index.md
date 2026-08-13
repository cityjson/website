---
layout: default
title: "FlatCityBuf"
nav_order: 5
has_children: true
permalink: /flatcitybuf/
---

[![](flatcitybuf_logo.png){:width="250px"}](https://github.com/cityjson/flatcitybuf)

**FlatCityBuf** is a new cloud-optimised format for 3D city models based on [FlatBuffers](https://flatbuffers.dev/) and CityJSON.

<i class="fab fa-github"></i> The schemas and software for conversion to/from CityJSON and to read/write FlatCityBuf are publicly available at [https://github.com/cityjson/flatcitybuf](https://github.com/cityjson/flatcitybuf) under a permissive license.

A FlatCityBuf file stores CityJSON's semantics in FlatBuffers, next to a packed Hilbert R-tree (spatial index) and static B+trees (attribute indices). A client can therefore fetch *only* the bytes a query needs, over plain HTTP range requests: querying a 70GB file on cloud storage takes a handful of requests and no server component at all.

{: .highlight }
Try it right now in your browser: the [web viewer]({{ '/flatcitybuf/viewer/' | prepend: site.baseurl }}) opens the full 3DBAG (~10.7M buildings, ~68GB) over HTTP and queries it live.

<details markdown="block">
<summary>Main features</summary>
- **Fast deserialisation**: 9-250× faster than existing formats
- **Zero-copy data access**: Efficient memory usage without data copying
- **Compact storage**: 10-30% compression compared to CityJSON
- **Memory efficient**: Uses 2-6× less memory
- **Spatial and attribute indices**: Enables efficient queries to retrieve partial data
- **CityGML compliance**: Adheres to the established CityGML v3.0 data model
- **Partial data access**: Efficient queries through spatial and attribute indexing
</details>

## Implementations

FlatCityBuf has four independent reader implementations. They are validated against each other on a shared [conformance corpus](https://github.com/cityjson/flatcitybuf/tree/main/conformance) of `.fcb` files: the expected output is produced by the Rust reader, and the C++, Python and TypeScript readers must reproduce it line for line on the same bytes.

| Implementation | Package | Reads | Writes | Notes |
| --- | --- | --- | --- | --- |
| [Rust]({{ '/flatcitybuf/rust/' | prepend: site.baseurl }}) | `fcb_core`, `fcb_cli` (crates.io) | ✅ | ✅ | reference implementation, zero-copy, sync + async HTTP |
| [C++]({{ '/flatcitybuf/cpp/' | prepend: site.baseurl }}) | vcpkg (custom registry) or CMake | ✅ | ✅ | native C++17, no Rust toolchain, no TLS dependency by default |
| [Python]({{ '/flatcitybuf/python/' | prepend: site.baseurl }}) | `flatcitybuf` (PyPI) | ✅ | — | pure Python, no compiled extension |
| [TypeScript]({{ '/flatcitybuf/typescript/' | prepend: site.baseurl }}) | `@cityjson/flatcitybuf` (npm) | ✅ | — | pure TypeScript, browser + Node.js, no WebAssembly |

All four decode the same things: attributes, geometry, semantics, geometry templates, appearance (materials and textures), and the extents and relationships in the header.

The pages here are a tour; each implementation's full reference lives in the repository — [docs/rust.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/rust.md), [docs/cpp.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/cpp.md), [docs/py.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/py.md), [docs/ts.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/ts.md).

The [`fcb` command-line tool]({{ '/flatcitybuf/conversion/' | prepend: site.baseurl }}) converts CityJSON/CityJSONSeq to `.fcb` and back, and inspects a file — locally or over HTTP.

## Where to start

1. Grab an [example dataset]({{ '/flatcitybuf/datasets/' | prepend: site.baseurl }}), or convert your own with [the CLI]({{ '/flatcitybuf/conversion/' | prepend: site.baseurl }}).
2. Look at it: `fcb inspect city.fcb` — a terminal UI on a terminal, a static report when piped or with `--static`.
3. Query it from [Rust]({{ '/flatcitybuf/rust/' | prepend: site.baseurl }}), [C++]({{ '/flatcitybuf/cpp/' | prepend: site.baseurl }}), [Python]({{ '/flatcitybuf/python/' | prepend: site.baseurl }}) or [TypeScript]({{ '/flatcitybuf/typescript/' | prepend: site.baseurl }}).
4. Stuck on performance or indexing choices? See the [FAQ and performance tips]({{ '/flatcitybuf/faq/' | prepend: site.baseurl }}).

<details markdown="block">
<summary>If you use FlatCityBuf in an academic context</summary>

Baba, Hidemichi, Ledoux, Hugo, and Peters, Ravi (2025). <em>FlatCityBuf: A new cloud-optimised CityJSON format</em>, Int. Arch. Photogramm. Remote Sens. Spatial Inf. Sci., XLVIII-4/W15-2025, 17–24 <small><a href="https://doi.org/10.5194/isprs-archives-XLVIII-4-W15-2025-17-2025"><i class="fas fa-book" title="thesis"></i></a></small>

Baba, Hidemichi <em>FlatCityBuf: a new cloud-optimised CityJSON format</em>. MSc thesis in Geomatics, Delft University of Technology. 2025. <small><a href="https://repository.tudelft.nl/record/uuid:6727c979-5e46-4fe0-9349-a7803e825d02"><i class="fas fa-book" title="thesis"></i></a></small> 
</details>
