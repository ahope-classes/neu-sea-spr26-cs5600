---
layout: page
title: Labs
description: The list of labs for this course
---

# Labs

This is an overview of the labs and timing of the labs for this semester. 

* Details of each lab will be released by the posted "release" date. 
* Dates may change as the quarter goes on; all date changes will be reflected in an announcement. 
* This overview is provided to give you a chance to plan ahead. 




|Title |Released |Due |
|:----- |:-------- |:--- |
{% for lab in site.labs -%}
| <a href="{{ site.baseurl }}/{{ lab.url }}">{{ lab.title }} </a> <br> {{ lab.summary }}| {{ lab.released }} | {{ lab.due }} | 
{% endfor %}