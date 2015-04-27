---
layout: default
title: OpenSpending Enhancement Proposals (OSEP)
---

{% assign sorted_pages = site.pages | sort:"name" %}
{% for page in sorted_pages %}
  {% if page.name contains 'osep-' %}
  * [#{{page.osep}}: {{ page.title }}]({{ page.url | remove_first:'/' }}) <span class="pull-right">{% if page.accepted %}<span class="label label-info">{{page.accepted}}</span> <span class="label label-success">Accepted</span>{% else %} <span class="label label-warning">Draft</span>{% endif %}</span>
  {% endif %}
{% endfor %}

----

## Submitting an Enhancement Proposal

Anyone can submit an Enhancement Proposal.

To submit an OSEP:

* Read [OSEP-00](http://labs.openspending.org/osep/osep-00.html) for guidelines on creating an OSEP
* Create a markdown document for your proposal, following the OSEP-00 guidelines
* Add it to the [OSEP github respository][repo]
  * Either: sending to openspending-dev mailing list (and we will add it for you)
  * Or: submit directly to the [OSEP github repository][repo] using a pull request

[repo]: https://github.com/openspending/osep

An OSEP does **not have to be complete to be submitted** - you can submit an OSEP
in draft form and then revise it with the community.
