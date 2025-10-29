---
layout: default
title: WASM
parent: FlatCityBuf
nav_order: 5
has_children: false
permalink: /flatcitybuf/wasm/
---

# Using FlatCityBuf with WebAssembly

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Install for JavaScript/TypeScript (npm)

{% raw %}
```bash
npm install @cityjson/flatcitybuf
```
{% endraw %}

## Using FlatCityBuf in the browser (WASM)

FlatCityBuf provides WebAssembly bindings for efficient CityJSON processing in web browsers.

### Setup and installation

Install the npm package:

{% raw %}
```bash
npm init -y # create package.json file
npm install @cityjson/flatcitybuf
```
{% endraw %}

Or include it in your `package.json`:

{% raw %}
```json
{
  "dependencies": {
    "@cityjson/flatcitybuf": "^0.2.0"
  }
}
```
{% endraw %}

### Reading FCB files over HTTP

Create a file called `index.html` and paste the following code into it. Open the HTML file in your browser and you'll see the UI like this:
![HTML to interact with FlatCityBuf over HTTP](./fcb_demo.png)

You can test it to fetch data with bounding box or attribute query.

You can download the complete HTML file here: [fcb_demo.html](./fcb_demo.html)
