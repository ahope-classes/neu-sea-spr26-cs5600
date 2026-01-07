---
layout: page
title: Notes
description: The list of notes for this course
---

# Notes



{% for note in site.notes -%}
  <a href="{{ site.baseurl }}{{ note.url }}">{{ note.title }} </a>
{% endfor %}







