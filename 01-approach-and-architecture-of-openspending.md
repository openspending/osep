# OpenSpending - Thoughts on Approach and Architecture

* **Created**: 10 April 2013
* **Updated**: 8 June 2013
* **Accepted**: 
* **Status**: Draft

This document represents reflections that distil understanding (and
thoughts) on where OpenSpending should be going with our approach and
architecture.

## Purpose and Approach

Single statement summary:

> We want to centralize data but decentralize "presentation" ("views")

By "presentation" (views) I mean presentations of that data to people in the
broadest sense - it could be a visualization and discussion in a news article
or a dedicated site like Where Does My Money Go.

To elaborate this a bit, it means:

1. OS provides a single central repository of open data on government (and
   corporate) finances
2. OS provides good access (APIs, dumps) but quite basic presentation of that
   data (browser, some viz)
3. Most of the presentation of that data happens on non-OS sites but using OS
   data (via the API, via dump etc)

Some of 3 may be done by members of the "OpenSpending" community and we care a
great deal about 3 (that stuff is the point of having 1+2).

BUT OpenSpending, at least as a technical project, is focused on 1+2.

This means OpenSpending technically is primarily about:

- DB: Maintaining that central repository (note this need *not* be a classic
  relational DB - it could be files on s3 or ...)
- ETL: Providing means to get data into that repository (ETL)
- API + Dumps: Providing means to get data out of that repository

For core there is *limited* provision of:

- Viz: providing off the shelf visualizations
- Analytics: providing ways to do analysis on that data

Note that on Viz and Analytics *in core* we provide only limited functionality
of the demonstrator or essential kind - there are lots of visualizations and
analyses that can be done and many ways to do it and OS at the core can and
should do only a little. (This does *not* mean that substantial analysis and
presentation cannot take place within the wider OpenSpending *project* -- there
can, and should be, lots of this but it would be *outside* of core).

Aside: analogies with OpenStreetMap. I continue to find analogies with OSM
incredibly useful. Few people see OSM data or maps via openstreetmap.org.
Instead they see or use that data in sites or products elsewhere (e.g.
FourSquare). OSM's core is the central DB, the data adding tools and the
API/Dumps. Viz even in the form of essential things like mapnik and tile
production now largely happens in other projects that are a part of the
community but not OSM "core".

## Implications

There’s more to think through here. These are just some immediate thoughts

1. The DB is not necessarily a (relational) DB
   * We need something that we can reliably store into not something that does
     all our analytics too. This could be flat files in s3
2. Optimize ETL
   * Getting data in is essential
   * This is about people as much as tools
   * Maximize structure and reliability
3. We should not care about OS.org traffic or SEO for normal users. What we
   care about is API usage.
   * We should start measuring API usage asap ...
4. Enabling people to build satellite sites or embed viz is our priority
   * We have made huge strides in this direction ... but we can do more
   * E.g. why focus on satellite sites in wordpress
   * Make it easier to get data slices

