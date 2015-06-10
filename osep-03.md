---
layout: default
osep: 3
discussion: https://github.com/openspending/osep/issues/5
title: Flask web framework
created: 30 September 2013
updated: 30 September 2013
accepted: 10 June 2015
authors: [Tryggvi Björgvinsson, Friedrich Lindenberg]
redirect_from: "/03-flask-web-framework.html"
---

This document proposes a replacement for the Pylons web framework currently being used for OpenSpending. Pylons is a deprecated (in favour of Pyramid). This document proposes Flask as a new web framework for OpenSpending.

## Key proposed changes

* Make existing code as web framework independent as possible
* Remove Pylons dependency
* Use Flask as OpenSpending's framework
* Use Flask extensions if possible

## Background

Pylons has been deprecated in favour of Pyramid (after merging with repoze.bfg). There haven't been any changes to the code base in about one year and won't be any enhancements.

This has numerous problems.

One is that openspending will have to maintain its version of pylons if any new additions to the framework are necessary or go out of its way to bend pylons to do it.

Another problem (similar to the previous one) is that issues like security and framework tests will not receive the same focus as they did before the merger so openspending would have to manage that as well.

A third problem is developers contributing to openspending. Handing people interested in contributing a pylons code base which is hard to find documentation for makes the whole process a lot more difficult. Using another more well known and simpler framework is more likely to make it easier for developers to contribute code to openspending.

As discussed in an [openspending ticket](https://github.com/openspending/openspending/issues/247) work towards migrating to [Flask](http://flask.pocoo.org/) has already been undertaken. All work should try to stay as web framework independent as possible but switching to Flask after starting the work (as discussed in the ticket) seems to be the way forward.

Since Flask is a micro-framework it makes developing framework independent code simpler. In addition to a similar syntactic sugar as Flask-SQLAlchemy and using Jinja2 (maintained by Flask people), OpenSpending also uses Babel for internationalisation and localisation. Babel was recently adopted by the Flask people and will be maintained by them.

## Alternatives

There are other web frameworks that could be used instead of Flask. We list them here (not all of them) with some

* Pyramid
    * The straight-forward migration would be to switch to Pyramid and should be considered thoroughly. It has good features and design decisions but from a contributing perspective Flask might be more known and easier to grasp for new comers.
* Django
    * Django is a very popular web framework but it makes some assumptions about the code and coupling. We wouldn't be developing a web app but a django app. Not good for framework independent code. Requires an extensive overhaul of the code.
* web2py
    * Not as popular as the other frameworks. Makes some assumptions and includes some magic which might make maintenance and user contributions trickier. Requires some overhaul of the code.
* Tornado
    * Asynchronous framework which makes it more efficient to support many users. This requires a major overhaul of the code so this is not an option.
* Node.js
    * Another programming language (but asynchronous) which makes migrating more difficult (due to the same reasons as tornado). This is just left in here for fun and since there are many node.js developers out there (but you could say that about all frameworks).

## Appendix: Work needed

* Migrate to SQLAlchemy (Flask-style): **Done**
* Use SQLAlchemy migrate: **Done**
* Migrate to Jinja2: *In progress*
* Look at and examine migration from nosetests to another test framework: Not started
* Look at and examine migration from beaker to another caching framework: Not started
* Look at and examine migration from repoze to another security/authentication framework: Not started
* Look at and examine migration from colander to another forms framework: Not started
* Look at and examine static file hierarchy setup/access: **Done**
* Look at and examine if Webhelpers migration is necessary: Not started
* Flask style routing: Not started
