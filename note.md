---
layout: page
title: Notes
description: The list of notes for this course
---

# Notes

{% for note in site.notes %}
  <h2>{{ note.title }} </h2>
  <p><a href="{{ site.baseurl }}{{ note.url }}">{{ note.summary | markdownify }}</a></p>
{% endfor %}



