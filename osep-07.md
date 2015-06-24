---
layout: default
osep: 7
discussion: https://github.com/openspending/osep/issues/9
title: Aggregations and Aggregation Workflow
created: 26 April 2015
updated: 26 April 2015
authors: [Rufus Pollock, Paul Walsh]
accepted:
redirect_from: "/07-aggregation.html"
---

To produce visualizations or other forms of analytics one must aggregate low-level transactions summing over particular dimensions.

This document details how the Aggregation component of [OSEP 1][osep1] will work.

[osep1]: ./01-approach-and-architecture-of-openspending.html

## Overview

<img src="https://docs.google.com/drawings/d/1sOkT6bFBKAtO55nqHUTnnp8dpyTXq_apkt-1wCRt8NM/pub?w=960&h=720" alt="" style="max-width: 85vw; margin-left: -33%;" />

*Fig 1: Overview of ETL Workflow ([source diagram][fig1])*

[fig1]: https://docs.google.com/drawings/d/1sOkT6bFBKAtO55nqHUTnnp8dpyTXq_apkt-1wCRt8NM/pub?w=960&h=720

## Proposal

Our initial focus is computing only a partial aggregrates - in OLAP terminology
partial "materialization" of the cube. The main aggregates are ones that serve
visualization and hence are towards the apex of the lattice of cuboids, i.e.
aggregates over all but one, two or, maximum, three dimensions.

In straightforward terms: suppose we have a standard finance dataset with
dimensions:

* Payor (from) = government department
* Payee (to) = the supplier
* Category = type of expense
* Date = day, month or just year

We then want aggregates that support hierarchical visualizations such as
treemaps and bubbletrees. To do that we want to aggregate over two or maybe
three of these dimensions (and perhaps a filter by date - e.g. restrict just to
current year). For example, we want aggregation by Department and Supplier (so
we can visualize total spending by department then spending per supplier per
department).

Our key tasks are:

* Define a data format for aggregates that can serve as data "API" between
  aggregation system and visualizations. This would be a JSON or CSV format
  that the aggregation system will produce - whether as flat files or API - and
  which the visualization systems will consume.
* Outline an aggregation system that will produce these aggregates. This will
  largely either use an existing OLAP system or draw on the existing literature
  on aggregation algorithms (see e.g. Agarwal et al 1996 or Han and Kamber).

## Appendix

This article assumes familiarity with standard Data Warehouse and OLAP
terminology and techniques.

Reading:

* Data Warehouse Toolkit, 2nd Edition, Kimball and Ross
* [On the Computation of Multidimensional Aggregates by Agarwal et al 1996 (VLDB)][agg]
* Data Mining - Concepts and Techniques, Han and Kamber, 2nd Edition - especially chapter 4

[agg]: http://www.vldb.org/conf/1996/P506.PDF

