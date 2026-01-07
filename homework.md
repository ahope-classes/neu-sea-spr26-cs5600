---
layout: page
title: Homeworks
description: The list of homeworks for this course
---

# Homeworks





{% for hw in site.homeworks %}
  <h2>
    <a href="{{ hw.url }}">
      {{ hw.title }} 
    </a>
  </h2>
  
{% endfor %}

## Homeworks in a table


|Week |Title |Released |Due |
|:---- |:----- |:-------- |:--- |
{% for hw in site.homeworks %}
| {{ hw.week }}   | <a href="{{ site.url }}/{{ hw.url }}">{{ hw.title }} </a> | {{ hw.released }} | {{ hw.due }}     | 
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



| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |




