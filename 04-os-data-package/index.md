---
layout: default
title: OpenSpending Data Package
osep: 4
discussion: https://github.com/openspending/osep/issues/6
created: 8 December 2013
updated: 17 August 2015
authors: Rufus Pollock, Paul Walsh
accepted:
redirect_from:
  - "/04-openspending-data-package.html"
  - "/osep-04.html"
---

# Overview

The fiscal data that OpenSpending imports and manages comes in all shapes and sizes. To be able to efficiently import and manage that data we need some way to describe and structure the data.

We want to do this in a way that is easy for publishers to use but, at the same time, provides enough information that consumers - most importantly tools and services within the OpenSpending - can also use the data easily.

OpenSpending Data Package is our proposed solution. It is a simple set of requirements on the structure of the data itself - the CSV, Excel or other files - plus a metadata specification to describe those files in a way that makes them easily usuable in the platform.

## Table of Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}


# Background

This proposal assumes some familiarity with fiscal data - e.g. budgets, spending etc - as this is the data we are structuring and describing.

Often, this data takes the form of rows in a spreadsheet or database with each row describing some kind of expenditure or receipt of money. The data can get considerably more complex but keep this simple model in mind for what follows.

TODO: simple ascii diagram of a classic spreadsheet

This proposal also builds one and reuses the [Data Package][dp] specifications. These are a family of simple, lightweight formats for publishing data. If you are unfamiliar with these more information can be found in the Appendix.

# Proposal

An OpenSpending Data Package has a simple structure:

* Data: the data MUST be stored in CSV files.
* Descriptor: there must a descriptor for the data and the "package" as a whole (e.g. who created it, high-level description etc). This comes in the form of a single `datapackage.json` file.

OpenSpending Data Package builds on the existing [Data Package][dp] specifications. In particular, OpenSpending is a Tabular Data Package which is, in turn, a Data Package. Specifically, an OpenSpending Data Package package MUST be a valid [Data Package][dp] and MUST be a valid [Tabular Data Package][tdp]. We will spell out key implications of this as we proceed.

Here's an overview diagram that not only illustrates the basic setup but also the relation with the Data Package specifications:

![Basic diagram of OpenSpending Data Package](img/open-spending-data-package.svg){: .center-block}

Basic overview of the OpenSpending Data Package
{: style="text-align: center"}

## Data Packages on Disk

Here are some examples of what an OpenSpending Data Package looks like on disk. Usually, the datapackage.json and data files are bundled together, and collectively referred to as "the data package".

A simple example of an OpenSpending Data Package:

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

## The Data

The data in your Data Package intended for use by OpenSpending MUST:

* Be in CSV format.
* Have well-structured CSVs- no blank rows, columns etc. [Tabular Data Package][tdp] speels this out in detail.

*Note: you can store other data files in your data package - for example, you may want to archive the original xls or data files you used. However, we do not consider these the "OpenSpending" data.*

## The Descriptor - `datapackage.json`

An OpenSpending data package MUST contain a `datapackage.json` - it is the central file in an OpenSpending Data Package.

The `datapackage.json` contains information in three key areas:

* Package Metadata - title, author etc
* Resources - describing data files
* Mapping - mapping the source data to a "Logical" model

We will detail each in turn.

## General Package Metadata

The following properties `MUST` be on the top-level descriptor:

* `name` (DP): a url-compatible short name ("slug") for the package
* `title` (DP): a human readable title for the package

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


## Resources

The Data Package MUST have a `resources` property. 

The definition and behaviour of the `resources` property is described in detail in the [Data Package][dp-resources] and [Tabular Data Package][tdp] specifications.

The only point we emphasize here is that each data file MUST have an entry in the `resources` array.


## Mapping

The OpenSpending Data Package MUST provide a `mapping` property.  `mapping` MUST be a hash.

The `mapping` hash provides a way to derive our logical model from the physical model represented by the package's resources.

<img src="https://docs.google.com/drawings/d/1krRsqOdV_r9VEjzDSliLgmTGcbLhnvd6IH-YDE8BEAY/pub?w=710&h=357" alt="" />

*Diagram illustrating how the mapping connects the "physical" model (raw CSV files) to the "logical", conceptual, model. The conceptual model is heavily oriented around OLAP.  ([Source on Gdocs](https://docs.google.com/drawings/d/1krRsqOdV_r9VEjzDSliLgmTGcbLhnvd6IH-YDE8BEAY/edit))*

*Note: a strong focus of this spec is simplicity, especially for publishers. Hence, in regard of the mapping we tend to follow a parsimonious approach and only require features that directly relate to key data consumption use cases, most specifically easy aggregation. As such, we do not seek to provide a complete description of the data.*

### Logical Model

The logical model is heavily based on [OLAP][olap]. Key aspects for our purpose are:

* Numerical *measures*: these will usually be the monetary amounts in the spending data
* Dimensions including common ones like:
  * Dates / times: almost all spending data has a temporal aspect.
  * Payor and Payee: entities spending or receiving money
  * Project / Programs: expenditure is often linked to a specific project or program
  * Taxonomies or classifications: expenditure is often classified in standard ways

From an OLAP perspective many of these dimensions may not split out in actual separate tables but map to attributes on the fact table if they are very simple (e.g. a given classification may just be a single field).

### Details

The `mapping` has the following structure:

```
  "measures": {
    "measure-name": {
      measure-descriptor
    }
  }
  "dimension-1": ...
  "dimension-2": ...
```

**Describing sources**: one common feature that we will need repeatedly is to indicate that the data for a given part of the logical model comes from a given field/column in a CSV file. Our common pattern for this is:

```
# full representation, using an object and the source property
"property-on-logical-model": {
  "source": "name-of-field-on-the-resource",
  # Optional - if not present it implicitly defaults to first resource in resources list
  "resource": "name-of-resource"

  ...
}
```

### Measures

Measures are numerical and usually correspond to finanical amounts in the source data. Structure is like the following:

```
{
  "source": "amount"
  "currency": "USD"
  "factor": 1
}
```

Properties:

* `currency`: (`MUST`) Any valid ISO 4217 currency code.
* `factor`: (`MAY`) A factor by which to multiple the raw monetary values to get the real monetary amount, eg `1000`. Defaults to `1`.

### Dimensions

A dimension has the structure:

```
"dimension-name": {
  "fields": {
    "field-1": ...,
    "field-2": ...
  }
  other properties ...
}
```

Each `field` is a property on the dimension - think of it as column on that dimension in a database. At a minimum it must have "source" information - i.e. where the data comes from for that property:

```
"field-1": {
  "source": "abc"
}
```

A dimension MUST have at least one field with the property `primaryKey` set on it e.g.

```
"field-1": {
  "primaryKey": true # note being true is optional it can just be set to ""
  "source": "..."
}
```

#### Common Dimensions

**`date`**

An **attribute** that declares the date of the transaction.

## Examples

### Minimal example

Here is the most basic example of an OpenSpending Data Package. The example describes a package that only has transactional data, with only required fields.

```
# budget.csv
{% include minimal/budget.csv %}

# datapackage.json
{% include minimal/datapackage.json %}
```

### Example with entities (denormalized)

Here is an example with information on the payor and payee, denormalized.

```
# budget.csv
{% include entities-denormalized/budget.csv %}

# datapackage.json
{% include entities-denormalized/datapackage.json %}
```

### Example with entities (normalized)

Here is the same example as previous, but with the entity data normalized. That means this is also an example of an OpenSpending Data Package with a resource that is not a spend resource.

```
# budget.csv
{% include entities-normalized/budget.csv %}

# entities.csv
{% include entities-normalized/entities.csv %}

# datapackage.json
{% include entities-normalized/datapackage.json %}
```

<!--
### Example with functional classification, and mapped to COFOG (normalized)

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
-->


----

# Appendix

## Related OSEPs

* OpenSpending Next Platform Architecture
* Move to a flat-file based DataStore (see [OSEP-02][osep-02] and [OSEP-6][osep-06]) and [Github Issue 669][issue-669]

The OpenSpending Data Package specification aims to provide support for:

* Mapping the physical model, as represented by Resources and their data files, to a logical model that can be used by various OpenSpending services.
* Packaging both normalized and denormalized data sources for use in OpenSpending, in order to support a wider range of data sources.
* Packaging resources that are *referenced* by the spend data proper, but that do not actually contain spend data. This could mean, for example, rich data on the recipients of funds, or projects associated with a particular set of data.
* Progressive enhancement of data via a range of *recommended*, but not *required* metadata, in order to establish clear path for data providers to enhance data quality, and to address new use cases for the OpenSpending platform going forward.

## Relationship to Data Packages

OpenSpending Data Packages are a *[Profile][dp-profiles]* of the [Tabular Data Package][tdp] format, and also implement significant portions of the [Budget Data Package][bdp] specification, albeit in a different manner. We do plan to syncronize OpenSpending Data Package and Budget Data Package in the near future.

Leveraging [Data Package][dp] also opens up new opportunities to reuse/remix data from Open Spending with other tools and platforms in the [Frictionless Data][fd] ecosystem.

## What is a Data Package?

The [Data Package][dp] specifications are a family of formats for standardised publishing of data.

Data Packages can have *[Profiles][dp-profiles]*, which extend base specifications towards more specific application.

[Tabular Data Package][tdp] describes a publishing standard specifically for tabular data (e.g.: CSV). [Budget Data Package][bdp] is a profile that extends Tabular Data Package to describe a publishing standard for both transactional and aggregate fiscal data.

## Relationship to Budget Data Package

[Budget Data Package][bdp] is another specification that Open Knowledge contributes to. Budget Data Package is already in use, and has a number of partners contributing to the specification. So, why aren't we using it?

1. Budget Data Package makes no distinction between the physical model and the logical model of data. This puts more responsibility on data producers to create resource files that strictly conform with the specification, right down to the naming of columns. OpenSpending Data Package provides a way to map the ideal logical model to the actual physical model.
2. Budget Data Package *requires* some metadata, like COFOG codes, that we would rather have as *recommendations* for OpenSpending.
3. Budget Data Package places most metadata on Resource objects, and by extension, expects that each Resource in the package is a spend data resource. OpenSpending Data Package places most metadata on the top-level descriptor, and provides support for Resources of spend data mixed with other data that supports the spend data.

Our goal is that OpenSpending Data Package and Budget Data Package will eventually merge into a single specification.

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
  * `currency` => currency
  * `description` no change
  * `languages` => ??
  * `territories` => ??
* `mapping` => `mapping`
* `views` => ??

## Structuring Data

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

## `archive` directory

`archive` directory is for data that is not for openspending directly but is useful to archive raw data files from which data was extracted.


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
[olap]: https://en.wikipedia.org/wiki/Online_analytical_processing


