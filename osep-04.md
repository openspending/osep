---
layout: default
title: OpenSpending Data Package
osep: 4
discussion: https://github.com/openspending/osep/issues/6
created: 8 December 2013
updated: 28 April 2015
authors: Rufus Pollock, Paul Walsh
accepted:
redirect_from: "/04-openspending-data-package.html"
---

## Overview

The specification of an OpenSpending Data Package is an essential part of the move to a flat-file based DataStore (see [OSEP-02][osep-02] and [OSEP-6][osep-06]) and [Github Issue 669][issue-669].

OpenSpending Data Packages are a *[Profile][dp-profiles]* of the [Tabular Data Package][tdp] format, and also implement significant portions of the [Budget Data Package][bdp] specification, albeit in a different manner. We do plan to syncronize OpenSpending Data Package and Budget Data Package in the near future.

The OpenSpending Data Package specification aims to provide support for:

* Mapping source data to integral parts of the OpenSpending data model (measures such as monetary values, and dimensions such as time, related entities, and so on).
* Packaging either normalized or denormalized data sources for use in OpenSpending.
* Packaging resources that are *referenced* by the spend data proper, but that do not actually contain spend data. This could mean, for example, rich data on the recipients of funds, or projects associated with a particular set of data.
* Progressive enhancement of data via a range of *recommended*, but not *required* metadata, in order to address new use cases for the OpenSpending platform going forward.

Leveraging [Data Package][dp] also opens up new opportunities to reuse/remix data from Open Spending with other tools and platforms in the [Frictionless Data][fd] ecosystem.

## Background

The [Data Package][dp] specifications are a family of formats for standardised publishing of data.

Data Packages can have *[Profiles][dp-profiles]*, which extend base specifications towards more specific application.

[Tabular Data Package][tdp] describes a publishing standard specifically for tabular data (e.g.: CSV). [Budget Data Package][bdp] is a profile that extends Tabular Data Package to describe a publishing standard for both transactional and aggregate fiscal data.

An **OpenSpending Data Package** is a Data Package Profile that extends Tabular Data Package, with *spend data* resources that have a similar schema and required field set to resources in Budget Data Package.

OpenSpending stores spend data of both agreggate and transactional form. Currently, data is managed in a relational database. In moving to a flat file DataStore for all "raw" data in OpenSpending, we require a way to provide metadata for the data in a structured form, as well as a canonical way to access the spend data from the raw sources.

Additionally, we want to provide flexibility in how users structure their source data, while still providing a consistent interface that OpenSpending services (see [OSEP-01][osep-01]) can rely on to access raw data.

Finally, we want to widen the use cases that OpenSpending can support. Part of the solution for this is to provide a way to store and reference additional data that supports the core spend data. OpenSpending Data Package provides structure for this.

## Proposal

An OpenSpending Data Package is a Tabular Data Package Profile.

At the bare minimum, that means:

* Data is stored in well-structured CSV files, refered to as "resources"
* Resources and additional metadata is provided via a `datapackage.json` descriptor
* Usually, the descriptor and the resources will be co-located in a directory, and this directory represents the "Data Package"

a simple example of an OpenSpending Data Package on disk, or in the DataStore, will look like this:

```
datapackage.json
# data files - can be data/ subdirectory or just in the base directory, must be CSV
data/my-financial-data.csv
```

A more complex example may have additional files:

```
datapackage.json
README.md           # optional

# data files - can be data/ subdirectory or just in the base directory, must be CSV
data/my-financial-data.csv

# directory for storing original data and other 'archival' material
# (optional)
archive/my-original-data.xls

# scripts used in preparing the data package
# (optional)
scripts/scrape-and-clean-the-data.py
```

### Required and Recommended Data and Metadata

In order for OpenSpending services to work effectively with raw data sources, we define a set of *requirements* for data and metadata, as well as a set of *recommendations* for metadata and data.

OpenSpending Data Packages are a Tabular Data Package Profile, and therefore must implement all the requirements for Tabular Data Packages.

See the [Data Package][dp] and [Tabular Data Package][tdp] specifications for further details.

OpenSpending Data Packages should move towards full support of the [Budget Data Package][bdp] specification. However, this current proposal does not aim for this support. Budget Data Packages place most metadata on Resource objects, whereas in the current proposal, OpenSpending Data Packages place most metadata on the top-level descriptor, and expect conformity amongst all spend data resources in a single package.

The following properties are required on the top-level descriptor:

* `name`: a url-compatible short name ("slug")
* `title`: a human readable title
* `profiles`: an array which declares the profile type(s) of the Data Package [see here][dp-profiles]
* `currency`: a currency code for this data (See [Budget Data Package][bdp-resources])
* `granularity`: a keyword that represents the type of spend data (See [Budget Data Package][bdp-resources])
* `fiscalYear`: A year (See [Budget Data Package][bdp-resources])
* `status`: A keyword that represents the status of the data (See [Budget Data Package][bdp-resources])
* `openspending`: a hash that provides implementation metadata for OpenSpending (see below for detailed information)
* `resources`: an array of [Data Resources][dp-resources]

The following properties are recommended on the top-level descriptor:

* `location`: A valid ISO code (See [Budget Data Package][bdp-resources])
* `type`: A keyword that represents the *direction* of the spend (See [Budget Data Package][bdp-resources])

The following properties are required on each resource in `resources`:

* `url`, `path` or `data`: which provides the actual data
* `name`: a url-compatible short name ("slug")
* `title`: a human readable title
* `description`: a human readable description of the resource contents
* `schema`: a [JSON Table Schema][jts] that describes the structure of the CSV

The following `fields` are required in each `schema` (and therefore, the data that the schema represents):

* `id` (or an *alias*): a unique identifier for this spend line
* `amount` (or an *alias*): the monetary amount for this spend line

The following `fields` are recommended in each `schema` (and therefore, the data that the schema represents):

* `payee` (or an *alias*): a human readable name for the payee of this spend line
* `payeeId` (or an *alias*): a unique identifer for the payee
* `payer` (or an *alias*): a human readable name for the payer of this spend line
* `payerId` (or an *alias*): a unique identifier for the payer

**If aliases are used, they must be mapped in `openspending.mapping` (see below).**

#### The `openspending` property

The `openspending` property is a `HASH` on the top-level descriptor, and provides implementation metadata for OpenSpending.

For v1 of the OpenSpending Data Package we proposed to largely reuse structures from the current OpenSpending data model.

The following properties are required on the `openspending` object:

* `owner`: a string which is the username of the OpenSpending account that the package belongs to

The following properties `MAY` be present on the `openspending` object:

* `spend_resources`: An `ARRAY` which is a list of Resource names, where each name `MUST` be present in the `resources` `ARRAY`. If provided, then **only** these resources are considered spend data proper. If not provided, All resources are considered spend data proper.
* `mapping`: A `HASH` that maps *alias* field names found in the schema/data to OpenSpending fields. Each key should be a valid OpenSpending field, and each value the name of the field in the data.
* `views`: `MAY` have a `views` attribute defining visualizations or views onto the data. If present, it MUST follow the structure defined in the [views specification in the OpenSpending documentation][views].

Here's an example of the `datapackage.json` top-level structure:

```
"name": ...,
"title": ...,
"profiles": [
  "openspending": "*"
},
"currency": ...,
"granularity": ...,
"fiscalYear": ...,
"openspending": {
  "owner": "me",
  "spend_resources": ["my-budget"]
  "mapping": {
    "id": "pk",
    "amount": "total"
  }
}
"resources": [
  {
    "name": "my-budget",
    "title": ...,
    "path": ...
  },
  {
    "name": "my-counties",
    "title": ...,
    "path": ...
  }
]
```

### Migration of Current OpenSpending Metadata

Full [documentation of the current OpenSpending JSON data model is
here][current]. The top-level structure is as follows:

[current]: http://docs.openspending.org/en/latest/model/design.html

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

You can just the dataset information by visiting:

    http://openspending.org/{dataset}.json

Migration of metadata is relatively simple:

* `dataset` attributes will be mapped across to the core `datapackage.json`
  metadata.
  * `name`: no change
  * `label` => title
  * `currency` => currency
  * `description` no change
  * `languages` => ??
  * `territories` => ??
* `mapping` - moves to `openspending.mapping`
* `views` - moves to `openspending.views`

### Structuring Data

* Recommendations about structuring the data
  * For datasets like to be below 10MB store into one file
  * For datasets > 10 MB suggest starting to partition in some sensible way.
    Most obvious is
    * by date: e.g. storing by month or year (yyyy or yyyy-mm(-dd))
    * by entity and date e.g. department-date
  * This approach has benefits that as new data arrives we don't need to keep
    overwriting one massive file (s3 does not support append operations!)
* Versioning data - we recommend turning on s3 versioning so we don't need to
  worry about accidental data loss

### Extras (under discussion)

* `archive` directory is for data that is not for openspending directly but is
  useful to archive raw data files from which data was extracted
* Note it is possible to store the data elsewhere than in data.openspending.org -
  you can have datapackage.json point to files somewhere else on the web
    (though we may still want to cache that at some point)

## Appendix

There is no material for the appendix at present.


[issue-669]: https://github.com/openspending/openspending/issues/669
[osep-01]: http://labs.openspending.org/osep/osep-01.html
[osep-02]: http://labs.openspending.org/osep/osep-02.html
[osep-06]: http://labs.openspending.org/osep/osep-06.html
[dp]: http://dataprotocols.org/data-packages/
[tdp]: http://dataprotocols.org/tabular-data-package/
[bdp]: https://github.com/openspending/budget-data-package
[bdp-resources]: https://github.com/openspending/budget-data-package/blob/master/specification.md#budget-specific-metadata
[dp-profiles]: https://github.com/dataprotocols/dataprotocols/issues/183
[dp-resources]: http://dataprotocols.org/data-packages/#resource-information
[fd]: http://data.okfn.org
[mapping]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[views]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[jts]: http://dataprotocols.org/json-table-schema/
