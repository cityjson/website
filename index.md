---
layout: default
title: Home
# nav_order: 1
nav_exclude: true
description: "CityJSON homepage"
permalink: /
---

<!-- <img src="{{ '/assets/images/cityjson_logo.svg' | prepend: site.baseurl }}" width="200"> -->
<img src="{{ '/assets/images/cityjson_logo.svg' | prepend: site.baseurl }}" width="600">

A JSON-based encoding for 3D city models
{: .fs-6 .fw-300 }

[Specs latest version: v{{ site.lastversion}}]({{ '/specs/' | append: site.lastversion | prepend: site.baseurl }}){: .btn .btn-purple .fs-5 .mb-4 .mb-md-0 .mr-2 } 

[Getting started]({{ '/tutorials/getting-started/' | prepend: site.baseurl }}){: .btn .fs-5 .mb-4 .mb-md-0 .mr-2 } 
[Web-viewer](https://ninja.cityjson.org/){: .btn  .fs-5 .mb-4 .mb-md-0 .mr-2}
[Validator](https://validator.cityjson.org/){: .btn  .fs-5 .mb-4 .mb-md-0 .mr-2}


---

## Latest news

{% for post in site.posts limit:3 %}
  <span class="text-delta">{{ post.date | date_to_string }}</span> <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
{% endfor %}

[All the news]({{ site.baseurl }}/news/)

---

## What is CityJSON?

CityJSON is a [JSON-based](http://json.org) encoding for storing 3D city models, also called digital maquettes or digital twins ([longer explanation]({{ site.baseurl }}/about/)).

The aim of CityJSON is to offer a compact and developer-friendly format, so that files can be easily visualised, manipulated, and edited.
It was designed with programmers in mind, so that tools and APIs supporting it can be quickly built, and [several]({{ "/software/" | prepend: site.baseurl }}) are available (most as open-source).

We believe that you should use CityJSON because: 

  1. its simplicity means adding CityJSON support to your software is easy, [many software already support it]({{ "/software/" | prepend: site.baseurl }}) 
  2. you can in one-click convert CityGML files to CityJSON files, and vice versa, with the open-source tool [citygml-tools](https://github.com/citygml4j/citygml-tools); we even have a [tutorial]({{ "/tutorials/conversion/" | prepend: site.baseurl }})
  3. files are on average [6X more compact](https://github.com/cityjson/specs/wiki/Compression-factor-for-a-few-open-CityGML-datasets) than their CityGML equivalent
  4. you can automatically convert to/from CityGML v3.0 XML files with the excellent [citygml-tools](https://github.com/citygml4j/citygml-tools) 
  5. there is a [web-viewer](https://viewer.cityjson.org) where you can drag-and-drop a file
  6. you can easily manipulate files with [cjio](https://github.com/cityjson/cjio), you can for instance merge files, remove/filter objects, change the CRS, manage the textures, etc.
  7. you can *easily* define [Extensions]({{ "/extensions/" | prepend: site.baseurl }}) to the core model 
  8. its development is [open on GitHub](https://github.com/cityjson/specs/issues/), it is supported by a vibrant community, and everyone is welcome to contribute

---

## Is it an international standard?

<img src="{{ '/assets/images/OGC-1.svg' | prepend: site.baseurl }}" width="200">

Yes, CityJSON is an official standard of the [Open Geospatial Consortium (OGC)](https://www.ogc.org/). 

[<i class="fas fa-external-link-alt"></i> document 20-072r2](https://docs.ogc.org/cs/20-072r2/20-072r2.html)

---

## What's its relationship to CityGML?

CityJSON is an encoding for a subset of the [OGC CityGML](http://www.opengeospatial.org/standards/citygml) data model (version 3.0.0), which is an open standardised data model and exchange format (in [GML](http://www.opengeospatial.org/standards/gml)) to store digital 3D models of cities and landscapes. 
The few features that are not currently supported are either because they are seldom used, or because they would overcomplicate the JSON encoding; see the [CityGML v3.0 implementation page]({{ "/citygml/v30/" | prepend: site.baseurl }}).

And because we offer [bidirectional conversion]({{ "/tutorials/conversion/" | prepend: site.baseurl }}) between CityJSON and CityGML, using CityJSON means that you are using the CityGML data model.

---

## Contributing to the project 

We invite anyone to contribute to the development and improvement of CityJSON, all discussions, issues, and developments are open to everyone on the [GitHub repository of CityJSON](https://github.com/cityjson/specs).

---

## If you use CityJSON in an academic context, please cite this article

Ledoux H, Arroyo Ohori K, Kumar K, Dukai B, Labetski A, Vitalis S (2019). CityJSON: A compact and easy-to-use encoding of the CityGML data model. **Open Geospatial Data, Software and Standards**, 4:4 [<i class="fas fa-bookmark"></i>](http://dx.doi.org/10.1186/s40965-019-0064-0) [<i class="fas fa-file-pdf"></i>](https://opengeospatialdata.springeropen.com/counter/pdf/10.1186/s40965-019-0064-0.pdf)



