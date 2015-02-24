---
layout: default
title: OpenSpending Enhancement Proposals
---

{% assign sorted_pages = site.pages | sort:"name" %}
{% for page in sorted_pages %}
  {% if page.name != 'index.md' %}
  * [#{{page.osep}}: {{ page.title }}]({{ page.url | remove_first:'/' }})
    (Status:
      {% if page.accepted %}Accepted &ndash; {{page.accepted}}{% else %} Draft{% endif %})
  {% endif %}
{% endfor %}

