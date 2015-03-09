---
layout: default
title: OpenSpending Enhancement Proposals (OSEP)
---

{% assign sorted_pages = site.pages | sort:"name" %}
{% for page in sorted_pages %}
  {% if page.name != 'index.md' %}
  * [#{{page.osep}}: {{ page.title }}]({{ page.url | remove_first:'/' }}) (Status: {% if page.accepted %}Accepted &ndash; {{page.accepted}}{% else %} Draft{% endif %})
  {% endif %}
{% endfor %}

----

## Submitting an Enhancement Proposal

Anyone can submit an Enhancement Proposal.

To submit an OSEP:

* Create a draft document in markdown format
* Add it to the [OSEP github respository][repo]
  * Either: sending to openspending-dev mailing list (and we will add it for you)
  * Or: submit directly to the [OSEP github repository][repo] using a pull request

[repo]: https://github.com/openspending/osep

An OSEP does **not have to be complete to be submitted** - you can submit an OSEP
in draft form and then revise it with the community.

