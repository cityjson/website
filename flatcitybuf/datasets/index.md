---
layout: default
title: Datasets
parent: FlatCityBuf
nav_order: 1
has_children: false
permalink: /flatcitybuf/datasets/
---

# FlatCityBuf example datasets

We offer a few example FlatCityBuf files for testing:

<!-- TODO: add actual URLs to example files -->

- [Delft (6MB)](https://github.com/cityjson/flatcitybuf/blob/bf270c345715e1020d314b7235ede8fffc6e0d85/examples/data/delft.fcb)
- [3DBAG small (3.4GB)](https://storage.googleapis.com/flatcitybuf/3dbag_subset_all_index.fcb)
- [3DBAG all (70GB)](https://storage.googleapis.com/flatcitybuf/3dbag_all_index.fcb): Complete 3DBAG dataset with spatial indexing and all attributes indexed

These files are hosted on Google Cloud Storage and can be accessed directly via HTTP.

You can always convert any CityJSONSeq file to a FlatCityBuf (and vice-versa), see our [conversion page]({{ '/flatcitybuf/conversion/' | prepend: site.baseurl }}).
