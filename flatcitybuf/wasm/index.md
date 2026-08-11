---
layout: default
title: WASM (retired)
parent: FlatCityBuf
nav_exclude: true
has_children: false
permalink: /flatcitybuf/wasm/
---

# WebAssembly bindings (retired)

{: .warning }
The WebAssembly binding has been replaced by a **pure TypeScript reader**, published under the same npm name `@cityjson/flatcitybuf`. It runs the same code in the browser and in Node.js, ships no `.wasm`, and fixes several defects the WASM binding had. This page is kept only so old links do not break.

- **Reading FlatCityBuf in the browser or in Node.js** → [TypeScript]({{ '/flatcitybuf/typescript/' | prepend: site.baseurl }}), including a [migration table]({{ '/flatcitybuf/typescript/#migrating-from-the-wasm-binding' | prepend: site.baseurl }}) from the old WASM API.
- **The browser demo that used to live here** → the [web viewer]({{ '/flatcitybuf/viewer/' | prepend: site.baseurl }}), which opens the full 3DBAG over HTTP and renders it with deck.gl.
