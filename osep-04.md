---
layout: default
title: OpenSpending Data Package
osep: 4
discussion: https://github.com/openspending/osep/issues/6
created: 8 December 2013
updated: 23 July 2015
authors: Rufus Pollock, Paul Walsh
accepted:
redirect_from: "/04-openspending-data-package.html"
---

## Overview

The specification of an OpenSpending Data Package is an essential part of the move to a flat-file based DataStore (see [OSEP-02][osep-02] and [OSEP-6][osep-06]) and [Github Issue 669][issue-669].

OpenSpending Data Packages are a *[Profile][dp-profiles]* of the [Tabular Data Package][tdp] format, and also implement significant portions of the [Budget Data Package][bdp] specification, albeit in a different manner. We do plan to syncronize OpenSpending Data Package and Budget Data Package in the near future.

The OpenSpending Data Package specification aims to provide support for:

* Mapping the physical model, as represented by Resources and their data files, to a logical model that can be used by various OpenSpending services.
* Packaging both normalized and denormalized data sources for use in OpenSpending, in order to support a wider range of data sources.
* Packaging resources that are *referenced* by the spend data proper, but that do not actually contain spend data. This could mean, for example, rich data on the recipients of funds, or projects associated with a particular set of data.
* Progressive enhancement of data via a range of *recommended*, but not *required* metadata, in order to establish clear path for data providers to enhance data quality, and to address new use cases for the OpenSpending platform going forward.

Leveraging [Data Package][dp] also opens up new opportunities to reuse/remix data from Open Spending with other tools and platforms in the [Frictionless Data][fd] ecosystem.

## Background

### What is a Data Package?

The [Data Package][dp] specifications are a family of formats for standardised publishing of data.

Data Packages can have *[Profiles][dp-profiles]*, which extend base specifications towards more specific application.

[Tabular Data Package][tdp] describes a publishing standard specifically for tabular data (e.g.: CSV). [Budget Data Package][bdp] is a profile that extends Tabular Data Package to describe a publishing standard for both transactional and aggregate fiscal data.

### What is an OpenSpending Data Package?

An **OpenSpending Data Package** is a Data Package Profile that extends Tabular Data Package, with *spend data* resources that have a similar schema and required field set to resources in Budget Data Package.

The OpenSpending platform stores spend data of both agreggate and transactional form. Currently, data is managed in a relational database. In moving to a flat file DataStore for all "raw" data in OpenSpending, we require a way to provide metadata for the data in a structured form, as well as a canonical way to access the spend data from the sources provided by the data packager.

Additionally, we want to provide flexibility in how users structure their source data, while still providing a consistent interface that OpenSpending services (see [OSEP-01][osep-01]) can rely on to access it.

Finally, we want to widen the use cases that OpenSpending can support. Part of the solution for this is to provide a way to store and reference additional data that supports the core spend data. OpenSpending Data Package provides structure for this.

### Relationship to Budget Data Package

[Budget Data Package][bdp] is another specification that Open Knowledge contributes to. Budget Data Package is already in use, and has a number of partners contributing to the specification. So, why aren't we using it?

1. Budget Data Package makes no distinction between the physical model and the logical model of data. This puts more responsibility on data producers to create resource files that strictly conform with the specification, right down to the naming of columns. OpenSpending Data Package provides a way to map the ideal logical model to the actual physical model.
2. Budget Data Package *requires* some metadata, like COFOG codes, that we would rather have as *recommendations* for OpenSpending.
3. Budget Data Package places most metadata on Resource objects, and by extension, expects that each Resource in the package is a spend data resource. OpenSpending Data Package places most metadata on the top-level descriptor, and provides support for Resources of spend data mixed with other data that supports the spend data.

Our goal is that OpenSpending Data Package and Budget Data Package will eventually merge into a single specification.

## Proposal

An OpenSpending Data Package is a profile of Tabular Data Package, which is a profile of Data Package.

A valid OpenSpending Data Package package MUST be a valid [Data Package](http://dataprotocols.org/data-packages/) as defined in that specification. This means that it MUST:

* Contain a package descriptor (`datapackage.json`)
* Provide at least the minimum required package metadata as described in the Data Package specification
* Include a description of each data file in the package in the `resources`array of the package

It MUST also be a valid [Tabular Data Package](http://dataprotocols.org/tabular-data-package/):

* It MUST contain at least one data file
* All data files MUST be in CSV format.
* Every resource MUST have a `schema` following the [JSON Table Schema specification](http://dataprotocols.org/json-table-schema/)

### What an OpenSpending Data Package looks like

Usually, the datapackage.json and data files are bundled together, and collectively referred to as "the data package".

A simple example of an OpenSpending Data Package on disk, or in the DataStore:

```
datapackage.json
# data files - can be data/ subdirectory or just in the base directory, must be CSV
data/my-financial-data.csv
```

A more complex example, with additional files:

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

And, an example of a data package with normalized data could be:

```
datapackage.json
README.md

# data files, but only the first one actually contains the spend data. It may contain references (foreign keys) to the other files.
data/my-financial-data.csv # actually contains spend data
data/my-list-of-entities-receiving-money.csv # data that augmented the spend data
data/my-list-of-projects-the-money-is-associated-with.csv # additional augmenting data
```

### Top-level descriptor

`datapackage.json` is the central file in a OpenSpending Data Package.

*Attributes defined in Data Package are marked (DP), and those that come from Tabular Data Package are marked (TDP).*

The following properties `MUST` be on the top-level descriptor:

* `name` (DP): a url-compatible short name ("slug") for the package
* `title` (DP): a human readable title for the package
* `profiles`: an hash of [Data Package profile][dp-profiles]-version pairs which the data package conforms to.
* `resources` (DP): an array of [Data Resources][dp-resources]
* `mapping`: a hash that provides information to build out the logical model of the package

The following properties `SHOULD` be on the top-level descriptor:

* `location`: A valid ISO code (See [Budget Data Package][bdp-resources]), or, an array of valid ISO codes

<div class="alert alert-warning">
  <strong>Note:</strong> The specification does not currently attempt to provide any specific ways to describe a package as a <i>government</i> budget, or the <i>type</i> of administrative body - details that would be extremely useful in order to provide a layer of analysis over spend from governing bodies. We plan to address this in a future OSEP, based on more work with real data and needs.
</div>

The following properties `MAY` be on the top-level descriptor:

* `granularity`: a keyword that represents the type of spend data, being one of "aggregated" or "transactional". Defaults to "aggregated" (See [Budget Data Package][bdp-resources])
* `direction`: A keyword that represents the *direction* of the spend, being one of "expenditure" or "revenue". Defaults to "expenditure" (See [Budget Data Package][bdp-resources]).
* `status`: A keyword that represents the status of the data, being one of "proposed", "approved", "adjusted", or "executed". Defaults to "approved" (See [Budget Data Package][bdp-resources]).


In addition to the properties described above, the descriptor `MAY` contain any number of additional properties that are not declared in the specification.


### Resources

The following properties `MUST` be on each resource in `resources`:

* `url`, `path` or `data` (DP): which provides the actual data
* `name` (DP): a url-compatible short name ("slug")
* `schema` (DP): a [JSON Table Schema][jts] that describes the types of, and relations within, the data

The following properties `SHOULD` be on each resource in `resources`:

* `title` (DP): a human readable title
* `description` (DP): a human readable description of the resource contents


### Mapping

The `mapping` hash provides a way to derive our logical model from the physical model represented by the package's resources via **mapping attributes** and **attribute groups**.


#### Attributes

Attributes must be declared as follows:

* IF the attribute value is a `HASH`, the hash `MUST` contain a `source` property that provides the mapping for this attribute
* IF the attrubute value is a `HASH`, any other properties `MAY` be present. Specific attributes of the OpenSpending data model may enforce particular requirements on additional properties
* The value of `attribute.source` `MUST` be either a `STRING` or an `ARRAY`.
  * IF the value is a `STRING`, it is of the format "{RESOURCE_NAME}/{FIELD_NAME}". Glob patterns `MAY` be used for the {RESOURCE_NAME} portion in order to match multiple resources.
  * IF the value is an `ARRAY`, then it is an array of complying source strings.

A mapping attribute looks like this:

```
# full representation, using an object and the source property
"attribute_name": {
  "source": "",
  ... other properties of the attribute
}

# shorthand, directly declaring the source of the attribute
"attribute_name": ""

# a set of attributes, grouped
"group_name": {
  "attribute_name1": "",
  "attribute_name2": {
    "source": "",
    ... other properties
  }
}
```


#### Data model

<div class="alert alert-warning">
<strong>Note:</strong> The `mapping` object aims to provide the neccesary structure in which to fully support **any** mapping of the logical model to the physical model. However, OSEP-04 **does not** attempt to cover any and all possible use cases. Why? We want to be flexible to iterate on this as we work with real data and real needs.
</div>

The following properties `MUST` be on `mapping`:

#### `id`

* An **attribute** that uniquely identifies a transaction

```
# mapping to a single file
"id": "budget/pk"

# mapping to multiple files
"id": ["budget1/pk", "budget2/pk"]
```

#### `amount`

* An **attribute** that declares the monetary amount of this transaction

Additional properties:

* `currency`: (`MUST`) Any valid ISO 4217 currency code.
* `factor`: (`MAY`) A factor by which to multiple the raw monetary values to get the real monetary amount, eg `1000`. Defaults to `1`.

```
# mapping to a single file
"amount": {
  "source": "budget/budget_spend",
  "currency": "GBP",
  "factor": 1
}

# mapping to a single file in USD with a default factor of 1
"amount": { 
  "source": "budget/budget_spend",
  "currency": "USD"
}

# mapping to multiple files
"amount": [
  { "source": "budget1/budget_spend"
    "currency": "GBP" },
  { "source": "budget2/budget_spend",
    "currency": "GBP" }
]
```

#### `date`

* An **attribute** that declares the date of this transaction

```
# mapping to a single file
"date": "budget/year"

# mapping to multiple files
"date": ["budget1/year", "budget2/year"]
```


The following properties `SHOULD` be on `mapping`:

#### `title`

* An **attribute** that declares the title of this transaction

```
# mapping to a single file
"title": "budget/name"

# mapping to multiple files
"title": ["budget1/name", "budget2/name"]
```

#### `payer`

* An **attribute group** that declares the payer of this transaction
  * Has the following attributes:
    * `id`: (`MUST`) a unique identifier for the payer
    * `title`: (`SHOULD`) a title or name for the payer
    * `description`: (`MAY`) a short description of the payer

```
"payer": {
  "id": "entities/id",
  "title": "entities/name",
  "description": "entities/about"
}
```

#### `payee`

* An **attribute group** that declares the payee of this transaction
  * Has the following attributes:
    * `id`: (`MUST`) a unique identifier for the payee
    * `title`: (`SHOULD`) a title or name for the payee
    * `description`: (`MAY`) a short description of the payee

```
"payee": {
  "id": "entities/id",
  "title": "entities/name",
  "description": "entities/about"
}
```

#### `function`

* An **attribute group** that declares the functional classification of this transaction
  * Has the following attributes:
    * `id`: (`MUST`) a unique identifier for the functional classification
    * `title`: (`SHOULD`) a title or name for the functional classification
    * `description`: (`MAY`) a short description of the functional classification
    * `cofog`: (`MAY`) a [COFOG][cofog] code that maps to this functional classification

```
"function": {
  "id": "budget_tree/id",
  "title": "budget_tree/title",
  "description": "budget_tree/summary",
  "cofog": "budget_tree/cofog_code"
}
```

#### `description`

* An **attribute** that declares the text description of this transaction

```
# mapping to a single file
"description": "budget/notes"

# mapping to multiple files
"description": ["budget1/notes", "budget2/notes"]
```

### Examples

How does this all come together?

#### Minimal example

Here is the most basic example of an OpenSpending Data Package. The example describes a package that only has transactional data, with only required fields.

```
# budget.csv
{% include minimal/budget.csv %}

# datapackage.json
{% include minimal/datapackage.json %}
```

#### Minimal with spend over multiple files example

Here is an example with spend data spread over multiple files. This demonstrates mapping of attributes over multiple physical sources.

```
# budget1.csv
{% include multiple/budget1.csv %}

# budget2.csv
{% include multiple/budget2.csv %}

# datapackage.json
{% include multiple/datapackage.json %}
```

#### Example with entities (denormalized)

Here is an example with information on the payor and payee, denormalized.

```
# budget.csv
{% include entities-denormalized/budget.csv %}

# datapackage.json
{% include entities-denormalized/datapackage.json %}
```

#### Example with entities (normalized)

Here is the same example as previous, but with the entity data normalized. That means this is also an example of an OpenSpending Data Package with a resource that is not a spend resource.

```
# budget.csv
{% include entities-normalized/budget.csv %}

# entities.csv
{% include entities-normalized/entities.csv %}

# datapackage.json
{% include entities-normalized/datapackage.json %}
```

#### Example with functional classification, and mapped to COFOG (normalized)

Here we build on the previous example and add functional classification of the spend data, as well as a mapping that classification to COFOG.

```
# budget.csv
{% include with-classification/budget.csv %}

# entities.csv
{% include with-classification/entities.csv %}

# classification.csv
{% include with-classification/classification.csv %}

# datapackage.json
{% include with-classification/datapackage.json %}
```

### Migration of Current OpenSpending Metadata

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
  * `currency` => currency
  * `description` no change
  * `languages` => ??
  * `territories` => ??
* `mapping` => `mapping`
* `views` => ??

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
[dp-profiles]: https://github.com/dataprotocols/dataprotocols/issues/87
[dp-resources]: http://dataprotocols.org/data-packages/#resource-information
[fd]: http://data.okfn.org
[mapping]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[views]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[jts]: http://dataprotocols.org/json-table-schema/
[current-model]: http://docs.openspending.org/en/latest/model/design.html
[cofog]: http://unstats.un.org/unsd/cr/registry/regcst.asp?Cl=4
[imf-budget]: http://www.imf.org/external/pubs/ft/tnm/2009/tnm0906.pdf
