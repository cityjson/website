---
layout: post
title: "FlatCityBuf now speaks C++, Python, TypeScript—and runs in your browser 🏙️"
categories: news
date: 2026-08-10
---

![]({{ site.baseurl }}/flatcitybuf/flatcitybuf_logo.png){:width="50%"}

[FlatCityBuf]({{ site.baseurl }}/flatcitybuf/), the cloud-optimised CityJSON format, has had a busy couple of months. The headline: it is no longer a Rust format with bindings around it. There are now **four independent implementations**—Rust, C++, Python and TypeScript—each reading the same files and checked against each other on a shared conformance corpus.

## A browser viewer, with no server and no WebAssembly

**[flatcitybuf-prototype.hideba.me](https://flatcitybuf-prototype.hideba.me)** opens the complete [3DBAG](https://3dbag.nl)—one ~68GB file, 10.7 million buildings—straight from cloud storage and lets you fly around it. Pan and zoom, and it queries the visible area over HTTP range requests, decoding only the buildings it needs; draw a bounding box or filter on an attribute to narrow it down, switch level of detail, colour by an attribute, click a building for its attributes, and export what you see as CityJSON, CityJSONSeq or OBJ.

Nothing is preprocessed for the viewer: it is the same `.fcb` file the CLI writes and the other implementations read, and there is no server component at all. See [the web viewer page]({{ site.baseurl }}/flatcitybuf/viewer/).

## Native C++, Python and TypeScript

Each binding was rewritten as a native implementation of the format, rather than a wrapper around the Rust core:

- **[C++]({{ site.baseurl }}/flatcitybuf/cpp/)** — a from-scratch C++17 library that both reads *and* writes FlatCityBuf. No Rust toolchain, no FFI bridge, no async runtime, and no TLS dependency unless you opt into the HTTP adapter. All IO goes through one synchronous `RangeReader` interface you can implement yourself, so it drops into engines and desktop applications that own their own threading.
- **[Python]({{ site.baseurl }}/flatcitybuf/python/) — pure Python.** One universal wheel, no compiled extension, no per-platform builds. Until the new reader reaches PyPI, [install it from source]({{ site.baseurl }}/flatcitybuf/python/) — the `flatcitybuf` package currently on PyPI is still the retired compiled extension. `numpy` is optional and makes bulk decoding ~2.4× faster.
- **[TypeScript]({{ site.baseurl }}/flatcitybuf/typescript/) — pure TypeScript.** The WebAssembly binding is retired; `@cityjson/flatcitybuf` now ships no `.wasm` and runs the same code in the browser and in Node.js, with one runtime dependency. It also fixes several bugs the WASM binding had, from attribute queries on non-double columns to a range client that accepted a full-body `200` as if it were the requested range.

Both the Python and TypeScript packages are new APIs rather than drop-in replacements; each page has a migration table.

## A CLI that also inspects

The `fcb` CLI moved to positional arguments (`fcb ser input.city.jsonl output.fcb`—no more `-i`/`-o`), and gained a way to look inside a file before reading it:

- `fcb info city.fcb` prints size, version, feature count, extent, transform and, most usefully, which attributes carry an index and are therefore queryable.
- `fcb inspect city.fcb` opens an interactive terminal UI with metadata, the full column schema and a map of the dataset's extent. It takes a **URL** as well as a path, and reads only the header—so inspecting that 68GB file over the network is instant.

Details on [the CLI page]({{ site.baseurl }}/flatcitybuf/conversion/).

## And the docs

Every FlatCityBuf page on this site has been rewritten around all this: [datasets]({{ site.baseurl }}/flatcitybuf/datasets/) (including how to serve your own `.fcb` files—the CORS header everyone trips over), the [CLI]({{ site.baseurl }}/flatcitybuf/conversion/), the four language pages, and the [performance tips and FAQ]({{ site.baseurl }}/flatcitybuf/faq/). Every example on those pages was run against the same 1115-building Delft file, and all four implementations return the same answers.

FlatCityBuf was developed by [Hidemichi Baba](https://3d.bk.tudelft.nl/hideba) for his MSc thesis in Geomatics at TU Delft; the paper is in the [ISPRS archives](https://doi.org/10.5194/isprs-archives-XLVIII-4-W15-2025-17-2025). The code lives at [github.com/cityjson/flatcitybuf](https://github.com/cityjson/flatcitybuf).
