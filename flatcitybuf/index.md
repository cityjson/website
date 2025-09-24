---
layout: default
title: "FlatCityBuf"
nav_order: 5
permalink: /flatcitybuf/
---

<h1>{{ page.title }}</h1>

[![](flatcitybuf_logo.png){:width="250px"}](https://github.com/cityjson/flatcitybuf)

**FlatCityBuf** is a new cloud-optimised format for 3D city models based on [FlatBuffers](https://flatbuffers.dev/) and CityJSON.

## Key Features

- **Zero-copy data access**: Efficient memory usage without data copying
- **Fast deserialisation**: 9-250× faster than existing formats
- **Compact storage**: 10-30% compression compared to CityJSON
- **Memory efficient**: Uses 2-6× less memory
- **Spatial and attribute indices**: Enables efficient queries to retrieve partial data
- **CityGML compliance**: Adheres to the established CityGML data model

## Performance Benefits

FlatCityBuf demonstrates significant advantages over existing formats:

- **Compression**: Achieves 10-30% compression compared to the already compact CityJSON format
- **Speed**: 9-250× faster deserialisation performance
- **Memory**: 2-6× less memory usage during processing
- **Partial data access**: Efficient queries through spatial and attribute indexing

## Software and Schemas

The schemas and accompanying software (Rust + Python + WASM) for conversion to/from CityJSON are publicly available at [https://github.com/cityjson/flatcitybuf](https://github.com/cityjson/flatcitybuf) under a permissive license.

## Citation


The reserach paper has been published on 20th 3D GeoInfo conference in 2025. The paper is publicly availabe on [ISPRS achives](https://isprs-archives.copernicus.org/articles/XLVIII-4-W15-2025/17/2025/) and its DOI is `10.5194/isprs-archives-XLVIII-4-W15-2025-17-2025`

If you use FlatCityBuf in your research, please cite:

```bibtex
@inproceedings{25_3dgeoinfo_fcb,
 author = {Baba, Hidemichi and Ledoux, Hugo and Peters, Ravi},
 title = {{FlatCityBuf}: {A} new cloud-optimised {CityJSON} format},
 booktitle = {Proceedings 20th 3D GeoInfo Conference},
 year = {2025},
 volume = {XLVIII-4/W15-2025},
 pages = {17--24},
 address = {Tokyo, Japan},
 publisher = {ISPRS},
 doi = {10.5194/isprs-archives-XLVIII-4-W15-2025-17-2025}
}
```

## MSc thesis related to this topic

[![](msc-hidemichi.png){:width="250px"}](https://repository.tudelft.nl/record/uuid:6727c979-5e46-4fe0-9349-a7803e825d02)

Hidemichi Baba <em>FlatCityBuf: a new cloud-optimised CityJSON format</em>. MSc thesis in Geomatics, Delft University of Technology. 2025. <small><a href="https://repository.tudelft.nl/record/uuid:6727c979-5e46-4fe0-9349-a7803e825d02"><i class="fas fa-book" title="thesis"></i></a></small> <small><a href="https://github.com/cityjson/flatcitybuf/tree/main"><i class="fab fa-github" title="github"></i></a></small>

## 
