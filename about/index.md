---
layout: default
title: What is CityJSON?
nav_exclude: true
permalink: /about/
---

# What is CityJSON?

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


![](Den-Haag-3D.jpg)
<br>*Excerpt of the 3D city model (LOD2) of The Hague, Netherlands (open dataset).*<br/><br/>
{: .text-delta}


## Overview

CityJSON is a [JSON-based](http://json.org) encoding for a subset of the [CityGML](https://www.opengeospatial.org/standards/citygml) data model (version 3.0.0), which is an open standardised data model and exchange format to store digital 3D models of cities and landscapes.
CityGML is an official standard of the [Open Geospatial Consortium](http://www.opengeospatial.org).

CityJSON is also an official international standard: [document 20-072r5 <i class="fas fa-external-link-alt"></i>](https://docs.ogc.org/cs/20-072r5/20-072r5.html)

CityJSON defines ways to describe most of the common 3D features and objects found in cities (such as buildings, roads, rivers, bridges, vegetation and city furniture) and the relationships between them.
It also defines different standard levels of detail (LoDs) for the 3D objects, which allows us to represent different resolutions of objects for different applications and purposes.

A CityJSON file describes both the geometry and the semantics of the city features of a given area, eg buildings, roads, rivers, the vegetation, and the city furniture.

The aim of CityJSON is to offer an alternative to the GML encoding of CityGML, which can be verbose and complex (and thus rather frustrating to work with). 
CityJSON aims at being easy-to-use, both for reading datasets, and for creating them.
It was designed with programmers in mind, so that tools and APIs supporting it can be quickly built.
It was also designed to be compact (it typically [compresses publicly available CityGML files by 7x]({{ '/filesize/' | prepend: site.baseurl }})), and to be friendly for web and mobile development.

A CityJSON object, representing a city, is as 'flat' as possible, ie the hierarchy of CityGML has been flattened out and only the city objects which are 'leaves' of this hierarchy are implemented.
This considerably simplifies the storage of a city model, and furthermore does not mean that information is lost.

CityJSON supports the extension of the core modules in a structured way (new city objects and complex attributes), see the [page about Extensions]({{ site.baseurl }}/extensions/) to know how to create yours or to download an existing one.

By using CityJSON, you can build on the expertise on 3D city modelling that has accumulated over the years. 
There is now a growing number of [software tools]({{ site.baseurl }}/software/) that support the format, and a long list of potential [applications]({{ site.baseurl }}/applications/).


## What can be stored in CityJSON?

CityJSON mainly describes the geometry, attributes, and semantics of different kinds of 3D city objects. 
These can be supplemented with textures and/or colours in order to give a better impression of their appearance. 
Specific relationships between different objects can also be stored using CityGML, eg that a building is decomposed into three parts, or that a building has a both a carport and a balcony.

The types of objects stored in CityJSON/CityGML are grouped into different modules. 
These are:

* __Appearance__: textures and materials for other types
* __Bridge__: bridge-related structures, possibly split into parts
* __Building__: the exterior and possibly the interior of buildings with individual surfaces that represent doors, windows, etc.
* __CityFurniture__: benches, traffic lights, signs, etc.
* __CityObjectGroup__: groups of objects of other types
* __Generics__: other types that are not explicitly covered
* __LandUse__: areas that reflect different land uses, such as urban, agricultural, etc.
* __Relief__: the shape of the terrain
* __Transportation__: roads, railways and squares
* __Tunnel__: tunnels, possibly split into parts
* __Vegetation__: areas with vegetation or individual trees
* __WaterBody__: lakes, rivers, canals, etc.

It is also possible to extend this list with new classes and attributes by defining [Extensions]({{ site.baseurl }}/extensions/).
There are already a few Extensions defined, and those can be reused.
<!-- e.g. for representing the 3D topographic objects in the Netherlands, for energy estimation in an urban context, and for estimating solar potential. -->


## How are objects stored in CityJSON?

CityJSON datasets consist of a set of plain text files (JSON files) and optionally accompanying image files that are used as textures. 
Each text file can represent a part of the dataset, such as a specific region, a specific type of object (such as a set of roads), or a predefined LoD.

The structure of a CityJSON file is a fairly simple to understand and can be easily read by humans, a simple example would look like this:

```js
{
  "type": "CityJSON",
  "version": "2.0",
  "extensions": {...},
  "transform": {...},
  "metadata": { "referenceSystem": "https://www.opengis.net/def/crs/EPSG/0/7415" },
  "CityObjects": {
    "id-1": {
      "type": "Building",
      "attributes": { "roofType": "gabled roof" },
      "geometry": [{
        "type": "Solid",
        "lod": 2.2,
        "boundaries": [...]
      }]
    },
    "id-56": {...}
  },
  "vertices": [
    [231, 2321, 11],
    [140, 2299, 14],
    ...
  ],
  "appearance": {
    "textures": [...]
  },
  "geometry-templates": {...}
}
```


## Which features of CityGML are supported?

Most of them, see the [CityGML v3 conformance page]({{ site.baseurl }}/citygml/v30/). 


## A JSON encoding of GML, huh?!?

While its name otherwise implies, CityGML is not only a GML encoding, but is actually an open standardised data model and it currently has 3 implementations:

  1. the GML encoding (not released yet for v3)
  2. a database schema called [3DCityDB](http://www.3dcitydb.org), which can be implemented both for [PostgreSQL](https://www.postgresql.org) and [Oracle Spatial](https://www.oracle.com/database/spatial/index.html). This is *not* an official standard.
  3. CityJSON v1.1 

