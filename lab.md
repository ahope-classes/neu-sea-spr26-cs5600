---
layout: page
title: Labs
description: The list of labs for this course
---

# Labs


Jan 5
: [Processes](#)
  : **READING**{: .label .label-purple }[Chapter 4](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf)
  **LAB**{: .label .label-yellow }[Lab 0](labs/lab0)


{% for lab in site.labs %}
  <h2>
    <a href="{{ site.baseurl }}/{{ lab.url }}">
      {{ lab.title }} 
    </a>
  </h2>
  
{% endfor %}