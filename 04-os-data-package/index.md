---
layout: default
title: OpenSpending Data Package
osep: 4
discussion: https://github.com/openspending/osep/issues/6
created: 8 December 2013
updated: 4 November 2015
authors: Rufus Pollock, Paul Walsh
accepted:
redirect_from:
  - "/04-openspending-data-package.html"
  - "/osep-04.html"
---

# Overview

The fiscal data that OpenSpending imports and manages comes in all shapes and sizes. To be able to efficiently import and manage that data we need some way to describe and structure the data.

We want to do this in a way that is easy for publishers to use but, at the same time, provides enough information that consumers - most importantly tools and services within the OpenSpending - can also use the data easily.

OpenSpending Data Package is our proposed solution.

# Proposal

For OpenSpending we will use [Fiscal Data Package][fdp]. There are no modifications or extensions to the base spec.

Thus, OpenSpending Data Package is [Fiscal Data Package][fdp].

[fdp]: http://fiscal.dataprotocols.org/spec/

# Appendix

## Migration of Current OpenSpending Metadata

Full [documentation of the current OpenSpending JSON data model is here][current-model]. The top-level structure is as follows:

```
{
  "dataset": {
    ... basic dataset attributes ...
    },
  "mapping": {
    ... dimension descriptions ...
    },
  "views": [
    ... pre-defined views ...
    ]
}
```

You can retrieve this information for each dataset by visiting:

    https://openspending.org/{dataset}/model.json

You can just get the dataset information by visiting:

    http://openspending.org/{dataset}.json

Migration of metadata is relatively simple:

* `dataset` attributes will be mapped across to the core `datapackage.json`
  metadata.
  * `name`: no change
  * `label` => title
  * `currency` => currency on measure
  * `description` no change
  * `languages` => ??
  * `territories` => countryCode
* `mapping` => `mapping` with appropriate modifications
* `views` => TODO

## Structuring Data

Recommendations about structuring the data

* For datasets like to be below 10MB store into one file
* For datasets > 10 MB suggest starting to partition in some sensible way.
  Most obvious is
  * by date: e.g. storing by month or year (yyyy or yyyy-mm(-dd))
  * by entity and date e.g. department-date
* This approach has benefits that as new data arrives we don't need to keep
  overwriting one massive file (s3 does not support append operations!)
* Versioning data - we recommend turning on s3 versioning so we don't need to
  worry about accidental data loss
* `archive` directory: is for data that is not for openspending directly but is
  useful to archive raw data files from which data was extracted.

[current-model]: http://docs.openspending.org/en/latest/model/design.html

