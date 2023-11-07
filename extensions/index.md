---
layout: default
title: Extensions
nav_order: 2
permalink: /extensions/
---

# Extensions

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


CityJSON uses [JSON Schemas](http://json-schema.org/) to document and validate the data model, schemas should be seen as basically validating the syntax of a JSON document.

A CityJSON *Extension* is a JSON file that allows us to document how the core data model of CityJSON may be extended, and to validate CityJSON files containing new objects and/or attributes.
This is conceptually akin to the *Application Domain Extensions* (ADEs) in CityGML; see Section 10.13 of the [official CityGML documentation](https://portal.opengeospatial.org/files/?artifact_id=47842).


[<i class="fas fa-external-link-alt"></i> CitySON v2.0 Extension specifications](https://www.cityjson.org/specs/#extensions)


## How to create an extension?

See the [Extensions specifications for v{{site.lastversion}}]({{ site.url }}{{ site.baseurl }}/specs/#extensions) for an overview of how the file should be structured.

This [tutorial to create a noise extension]({{ '/tutorials/extension/' | prepend: site.baseurl }}) guides you through the process, and explains how to validate the extension itself and datasets that uses the extension.

As a template/example, you can have a look at an [Extension for CityJSON v2.0 having `+GenericCityObect` with a new semantic surface]({{ '/extensions/download/v20/generic.ext.json' | prepend: site.baseurl }}).



## Registry of current Extensions

{% assign extensions = site.data.extensions | sort_natural: 'name' %}

{% for i in extensions %}
<p>
  <b>{{ i.name }}</b>
  <br/>
  {{ i.description | markdownify | remove: '<p>' | remove: '</p>' }} 
  <br/>
  {% if i.status %}
  v{{ i.status }} |
  {% endif %}
  <a href="{{ i.url }}">schema</a>
  {% if i.details %}
  | 
  <a href="{{ i.details }}">details</a>
  {% endif %}
  {% if i.organisation %}
  | 
  {{ i.organisation | markdownify | remove: '<p>' | remove: '</p>' }} 
  {% endif %}
</p>
{% endfor %}
