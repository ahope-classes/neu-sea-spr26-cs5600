---
layout: page
title: Calendar
description: Listing of course modules and topics.
---

# Calendar

* This schedule may change as the semester progresses. 
* It is expected that readings will be completed before class. 

{% for module in site.modules %}
{{ module }}
{% endfor %}
