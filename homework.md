---
layout: page
title: Homeworks
description: The list of homeworks for this course
---

# Labs


Jan 5
: [Processes](#)
  : **READING**{: .label .label-purple }[Chapter 4](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf)
  **LAB**{: .label .label-yellow }[Lab 0](labs/lab0)


{% for hw in site.homeworks %}
  <h2>
    <a href="{{ hw.url }}">
      {{ lab.title }} 
    </a>
  </h2>
  
{% endfor %}

## Homeworks in a table


| Week | Title | Released | Due |
| ---- | ----- | -------- | --- |
{% for hw in site.homeworks %}
  
| {{ hw.week }}   | <a href="{{ hw.url }}">{{ hw.title }} </a> | {{ hw.released }} | {{ hw.due }}     |

  
{% endfor %}


<ul>
  {% for hw in site.homeworks %}
    <li>
      <a href="{{ hw.url }}">{{ hw.title }}</a>
    </li>
  {% endfor %}
</ul>



|Title  |Link  |
|---|---|
{% for hw in site.homeworks -%}
{% if hw.title -%}
|{{ hw.title }}  |[Click Here]({{ hw.url }})  |
{% endif %}
{%- endfor -%}




