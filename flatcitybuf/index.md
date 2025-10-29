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


<details markdown="block">
<summary>Main features</summary>
- **Fast deserialisation**: 9-250× faster than existing formats
- **Zero-copy data access**: Efficient memory usage without data copying
- **Compact storage**: 10-30% compression compared to CityJSON
- **Memory efficient**: Uses 2-6× less memory
- **Speed**: 9-250× faster deserialisation performance
- **Spatial and attribute indices**: Enables efficient queries to retrieve partial data
- **CityGML compliance**: Adheres to the established CityGML v3.0 data model
- **Partial data access**: Efficient queries through spatial and attribute indexing
</details>


<details markdown="block">
<summary>If you use FlatCityBuf in an academic context</summary>

Baba, Hidemichi, Ledoux, Hugo, and Peters, Ravi (2025). <em>FlatCityBuf: A new cloud-optimised CityJSON format</em>, Int. Arch. Photogramm. Remote Sens. Spatial Inf. Sci., XLVIII-4/W15-2025, 17–24 <small><a href="https://doi.org/10.5194/isprs-archives-XLVIII-4-W15-2025-17-2025"><i class="fas fa-book" title="thesis"></i></a></small>

Baba, Hidemichi <em>FlatCityBuf: a new cloud-optimised CityJSON format</em>. MSc thesis in Geomatics, Delft University of Technology. 2025. <small><a href="https://repository.tudelft.nl/record/uuid:6727c979-5e46-4fe0-9349-a7803e825d02"><i class="fas fa-book" title="thesis"></i></a></small> 
</details>
