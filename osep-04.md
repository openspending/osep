---
layout: default
title: OpenSpending Data Package
osep: 4
discussion: https://github.com/openspending/osep/issues/6
created: 8 December 2013
updated: 31 May 2015
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

Additionally, we want to provide flexibility in how users structure their source data, while still providing a consistent interface that OpenSpending services (see [OSEP-01][osep-01]) can rely on to access raw data.

Finally, we want to widen the use cases that OpenSpending can support. Part of the solution for this is to provide a way to store and reference additional data that supports the core spend data. OpenSpending Data Package provides structure for this.

### Relationship to Budget Data Package

[Budget Data Package][bdp] is another specification that Open Knowledge contributes to. Budget Data Package is already in use, and has a number of partners contributing to the specification. So, why aren't we using it?

1. Budget Data Package makes no distinction between the physical model and the logical model of data. This puts more responsibility on data producers to create resource files that strictly conform with the specification, right down to the naming of columns. OpenSpending Data Package provides a way to map the ideal logical model to the actual physical model.
2. Budget Data Package *requires* some metadata, like COFOG codes, that we would rather have as *recommendations* for OpenSpending.
3. Budget Data Package places most metadata on Resource objects, and by extension, expects that each Resource in the package is a spend data resource. OpenSpending Data Package places most metadata on the top-level descriptor, and provides support for Resources of spend data mixed with other data that supports the spend data.

Our goal is that OpenSpending Data Package and Budget Data Package will eventually merge into a single specification.

## Proposal

An OpenSpending Data Package is a Tabular Data Package Profile.

### What an OpenSpending Data Package looks like

An OpenSpending Data Package is:

* A descriptor file, named `datapackage.json`, which provides metadata
* Data stored in one or many well-structured CSV files, referred to from the Data Package descriptor in the `resources` property, where each resource represents a file and its specific metadata
* Usually, the descriptor and the resources will be co-located in a directory, and this directory represents the "Data Package"

A simple example of an OpenSpending Data Package on disk, or in the DataStore, will look like this:

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

And, an example of a data package with normalized data could be:

```
datapackage.json
README.md

# data files, but only the first one actually contains the spend data. It may contain references (foreign keys) to the other files.
data/my-financial-data.csv # actually contains spend data
data/my-list-of-entities-receiving-money.csv # data that augmented the spend data
data/my-list-of-projects-the-money-is-associated-with.csv # additional augmenting data
```

### OpenSpending Data Package specification

As stated, this specification builds on the [Data Package][dp] and [Tabular Data Package][tdp] specifications, so please reference these for further information.

### Top-level descriptor

`datapackage.json` is the central file in a OpenSpending Data Package.

The following properties `MUST` be on the top-level descriptor:

* `name`: a url-compatible short name ("slug") for the package
* `title`: a human readable title for the package
* `profiles`: an array which declares the profile type(s) of the Data Package [see here][dp-profiles]
* `resources`: an array of [Data Resources][dp-resources]
* `mapping`: a hash that provides information to build out the logical model of the package
* `owner`: The username of the account on OpenSpending that owns this package

The following properties `SHOULD` be on the top-level descriptor:

* `location`: A valid ISO code (See [Budget Data Package][bdp-resources]), or, an array of valid ISO codes

<div class="alert alert-warning">
  <strong>Note:</strong> The specification does not currently attempt to provide any specific ways to describe a package as a *government* budget, or the *type* of administrative body - details that would be extremely useful in order to provide a layer of analysis over spend from governing bodies. We plan to address this in a future OSEP, based on more work with real data and needs.
</div>

The following properties `MAY` be on the top-level descriptor:

* `granularity`: a keyword that represents the type of spend data, being one of "aggregated" or "transactional". Defaults to "aggregated" (See [Budget Data Package][bdp-resources])
* `direction`: A keyword that represents the *direction* of the spend, being one of "expenditure" or "revenue". Defaults to "expenditure" (See [Budget Data Package][bdp-resources]).
* `status`: A keyword that represents the status of the data, being one of "proposed", "approved", "adjusted", or "executed". Defaults to "approved" (See [Budget Data Package][bdp-resources]).


In addition to the properties described above, the descriptor `MAY` contain any number of additional properties that are not declared in the specification.


### Resources

The following properties `MUST` be on each resource in `resources`:

* `url`, `path` or `data`: which provides the actual data
* `name`: a url-compatible short name ("slug")
* `schema`: a [JSON Table Schema][jts] that describes the types of, and relations within, the data

The following properties `SHOULD` be on each resource in `resources`:

* `title`: a human readable title
* `description`: a human readable description of the resource contents


### Mapping

The mapping hash provides a way to derive our logical model from the physical model represented by the package's resources. The mapping is based around OpenSpending's four model types. Not all model types are required.

#### Model types

The following properties `MUST` be on the `mapping` hash:

* `transaction`: A hash of the key measures and attributes of the spend data that the package describes

The following properties `SHOULD` be on the `mapping` hash:

* `taxonomy`: a hash of the classification system(s) used in the spend data that the package describes

The following properties `MAY` be on the `mapping` hash:

* `entity`: a hash of the entities that are paid or paying in the spend data that the package describes
* `project`: a hash of the projects that provide context for the transactions that the package describes

#### Attributes

Each model type `MUST` have an `attributes` hash, where each key represents a field in the logical model, and the value is an array of strings in the physical model that provide this data. Each string in this array is of the format "{RESOURCE_NAME}/{FIELD_NAME}". Glob patterns `MAY` be used for the {RESOURCE_NAME} portion in order to match multiple resources.

Each model type is described in detail below.

#### Transaction

The following properties `MUST` be on the `transaction` hash:

* `value`: a hash that provides information on the monetary values described by the data package
  * `currency`: any valid ISO 4217 currency code
  * `factor`: a factor by which to multiple the raw monetary values to get the real monetary amount. Defaults to 1
* `attributes`: is a hash with the following properties:
  * `id`: (`MUST`) a unique identifier for the transaction
  * `amount`: (`MUST`) a monetary amount for the transaction
  * `date`: (`MUST`) a date for the transaction
  * `title`: (`SHOULD`) a title or name for the transaction
  * `description`: (`MAY`) a short description of the transaction

#### Entity

The following properties `MUST` be on the `entity` hash, if the entity hash is present:

* `attributes`: is a hash with the following properties:
  * `payer`:
    * `id`: (`MUST`) a unique identifier for the payer
    * `title`: (`SHOULD`) a title or name for the payer
    * `description`: (`MAY`) a short description of the payer
  * `payee`:
    * `id`: (`MUST`) a unique identifier for the payee
    * `title`: (`SHOULD`) a title or name for the payee
    * `description`: (`MAY`) a short description of the payee

#### Taxonomy

Following from this [IMF document][imf-budget] on budget classification, we identify 3 classification types:

* **Functional**: categorisation according to purpose and objective
* **Economic**: categorisation according to type (salaries, transfers, goods and services)
* **Administrative**: categorisation according to administering entity

The following properties `MUST` be on the `taxonomy` hash, if the taxonomy hash is present:

* `attributes`: is a hash with at least one of the following properties:
  * `functional`:
    * `id`: (`MUST`) a unique identifier for the functional classification
    * `title`: (`SHOULD`) a title or name for the functional classification
    * `description`: (`MAY`) a short description of the functional classification
    * `cofog`: (`SHOULD`) a [COFOG][cofog] code that maps to this functional classification
  * `economic`:
    * `id`: (`MUST`) a unique identifier for the transaction
    * `title`: (`SHOULD`) a title or name for the transaction
    * `description`: (`MAY`) a short description of the transaction
  * `administrative`:
    * `id`: (`MUST`) a unique identifier for the transaction
    * `title`: (`SHOULD`) a title or name for the transaction
    * `description`: (`MAY`) a short description of the transaction

#### Project

The following properties `MUST` be on the `project` hash, if the project hash is present:

* `attributes`: is a hash with the following properties:
  * `id`: (`MUST`) a unique identifier for the transaction
  * `amount`: (`MUST`) a monetary amount for the transaction
  * `date`: (`MUST`) a date for the transaction
  * `title`: (`SHOULD`) a title or name for the transaction
  * `description`: (`MAY`) a short description of the transaction


### Examples

How does this all come together?

#### Minimal example

Here is the most basic example of an OpenSpending Data Package. The example describes a package that only has transactional data, with only required fields.

```
# budget.csv
pk,budget,budget_date
1,10000,01/01/2015

# datapackage.json
{
  "name": "my-openspending-datapackage",
  "title": "My OpenSpending Data Package",
  "profiles": [
    "openspending": "*",
    "tabular": "*"
  ],
  "owner": "my-username",
  "mapping": {
    "transaction": {
      "value": {
        "currency": "USD",
        "factor": 1
      },
      "attributes": {
        "id": ["budget/pk"],
        "amount": ["budget/budget"],
        "date": ["budget/budget_date"]
      }
    }
  },
  "resources": [
    {
      "name": "budget",
      "title": "Budget",
      "path": "budget.csv",
      "schema": {
        "fields": [
          {
            "name": "id",
            "type": "string"
          },
          {
            "name": "amount",
            "type": "number",
            "format": "currency"
          },
          {
            "name": "date",
            "type": "date"
          }
      ],
      "primaryKey": "id"
      }
    }
  ]
}
```

# Example with entities (denormalized)

Here is an example with information on the payor and payee, denormalized.

```
# budget.csv
pk,budget,budget_date,payee_id,payor_id,payee_name,payor_name
1,10000,01/01/2015,1,2,Acme 1,Acme 2
2,20000,01/02/2015,1,2,Acme 1,Acme 2

# datapackage.json
{
  "name": "my-openspending-datapackage",
  "title": "My OpenSpending Data Package",
  "profiles": [
    "openspending": "*",
    "tabular": "*"
  ],
  "owner": "my-username",
  "mapping": {
    "transaction": {
      "value": {
        "currency": "USD",
        "factor": 1
      },
      "attributes": {
        "id": ["budget/pk"],
        "amount": ["budget/budget"],
        "date": ["budget/budget_date"]
      }
    },
    "entity": {
      "payee": {
        "id": ["budget/payee_id"],
        "title": ["budget/payee_name"]
      },
      "payor": {
        "id": ["budget/payor_id"],
        "title": ["budget/payor_name"]
      }
    }
  },
  "resources": [
    {
      "name": "budget",
      "title": "Budget",
      "path": "budget.csv",
      "schema": {
        "fields": [
          {
            "name": "id",
            "type": "string"
          },
          {
            "name": "amount",
            "type": "number",
            "format": "currency"
          },
          {
            "name": "date",
            "type": "date"
          },
          {
            "name": "payee_id",
            "type": "string"
          },
          {
            "name": "payee_name",
            "type": "string"
          },
          {
            "name": "payor_id",
            "type": "string"
          },
          {
            "name": "payor_name",
            "type": "string"
          }
      ],
      "primaryKey": "id"
      }
    }
  ]
}
```

# Example with entities (normalized)

Here is the same example as previous, but with the entity data normalized. That means this is also an example of an OpenSpending Data Package with a resource that is not a spend resource.

```
# budget.csv
pk,budget,budget_date,payee,payor
1,10000,01/01/2015,1,2
2,20000,01/02/2015,1,2

# entities.csv
id,name,description,since
1,Acme 1,They are the first acme company,1973
2,Acme 2,They are the sceond acme company,1974

# datapackage.json
{
  "name": "my-openspending-datapackage",
  "title": "My OpenSpending Data Package",
  "profiles": [
    "openspending": "*",
    "tabular": "*"
  ],
  "owner": "my-username",
  "mapping": {
    "transaction": {
      "value": {
        "currency": "USD",
        "factor": 1
      },
      "attributes": {
        "id": ["budget/pk"],
        "amount": ["budget/budget"],
        "date": ["budget/budget_date"]
      }
    },
    "entity": {
      "payee": {
        "id": ["budget/payee"],
        "title": ["name"]  # note: we really want a way of saying this has be dereferenced!
      },
      "payor": {
        "id": ["budget/payor"],
        "title": ["name"]  # note: we really want a way of saying this has be dereferenced!
      }
    }
  },
  "resources": [
    {
      "name": "budget",
      "title": "Budget",
      "path": "budget.csv",
      "schema": {
        "fields": [
          {
            "name": "id",
            "type": "string"
          },
          {
            "name": "amount",
            "type": "number",
            "format": "currency"
          },
          {
            "name": "date",
            "type": "date"
          },
          {
            "name": "payee",
            "type": "string"
          },
          {
            "name": "payee",
            "type": "string"
          }
      ],
      "primaryKey": "id"
      },
      "foreignKeys": [
        {
          "fields": "payee",
          "reference": {
            "datapackage": "my-openspending-datapackage",
            "resource": "entities",
            "fields": "id"
          }
        },
        {
          "fields": "payor",
          "reference": {
            "datapackage": "my-openspending-datapackage",
            "resource": "entities",
            "fields": "id"
          }
        }
      ]
    },
    {
      "name": "entities",
      "title": "Entities",
      "path": "entities.csv",
      "schema": {
        "fields": [
          {
            "name": "id",
            "type": "string"
          },
          {
            "name": "title",
            "type": "string"
          },
          {
            "name": "description",
            "type": "string"
          },
          {
            "name": "since",
            "type": "date"
          }
      ],
      "primaryKey": "id"
      }
    }
  ]
}
```

# Example with functional classification, and mapped to COFOG (normalized)

Here we build on the previous example and add functional classification of the spend data, as well as a mapping that calssification to COFOG.

```
# budget.csv
pk,budget,budget_date,payee,payor,category
1,10000,01/01/2015,1,2,1
2,20000,01/02/2015,1,2,2

# entities.csv
id,name,description,since
1,Acme 1,They are the first acme company,1973
2,Acme 2,They are the sceond acme company,1974

# classification.csv
id,name,description,cofog_code
1,Rubbish Disposal,Cleaning the rubbish,05.1
2,River regeneration,Cleaning the river,05.4

# datapackage.json
{
  "name": "my-openspending-datapackage",
  "title": "My OpenSpending Data Package",
  "profiles": [
    "openspending": "*",
    "tabular": "*"
  ],
  "owner": "my-username",
  "mapping": {
    "transaction": {
      "value": {
        "currency": "USD",
        "factor": 1
      },
      "attributes": {
        "id": ["budget/pk"],
        "amount": ["budget/budget"],
        "date": ["budget/budget_date"]
      }
    },
    "entity": {
      "payee": {
        "id": ["budget/payee"],
        "title": ["name"]  # note: we really want a way of saying this has be dereferenced!
      },
      "payor": {
        "id": ["budget/payor"],
        "title": ["name"]  # note: we really want a way of saying this has be dereferenced!
      }
    },
    "taxonomy": {
      "functional": {
        "id": ["budget/category"],
        "title": ["name"], # note: we really want a way of saying this has be dereferenced!
        "cofog": ["cofog_code"]   # note: we really want a way of saying this has be dereferenced!
      }
    }
  },
  "resources": [
    {
      "name": "budget",
      "title": "Budget",
      "path": "budget.csv",
      "schema": {
        "fields": [
          {
            "name": "id",
            "type": "string"
          },
          {
            "name": "amount",
            "type": "number",
            "format": "currency"
          },
          {
            "name": "date",
            "type": "date"
          },
          {
            "name": "payee",
            "type": "string"
          },
          {
            "name": "payee",
            "type": "string"
          },
          {
            "name": "category",
            "type": "string"
          }
      ],
      "primaryKey": "id"
      },
      "foreignKeys": [
        {
          "fields": "payee",
          "reference": {
            "datapackage": "my-openspending-datapackage",
            "resource": "entities",
            "fields": "id"
          }
        },
        {
          "fields": "payor",
          "reference": {
            "datapackage": "my-openspending-datapackage",
            "resource": "entities",
            "fields": "id"
          }
        },
        {
          "fields": "category",
          "reference": {
            "datapackage": "my-openspending-datapackage",
            "resource": "classification",
            "fields": "id"
          }
        }
      ]
    },
    {
      "name": "entities",
      "title": "Entities",
      "path": "entities.csv",
      "schema": {
        "fields": [
          {
            "name": "id",
            "type": "string"
          },
          {
            "name": "title",
            "type": "string"
          },
          {
            "name": "description",
            "type": "string"
          },
          {
            "name": "since",
            "type": "date"
          }
      ],
      "primaryKey": "id"
      }
    }
  ]
}
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
[dp-profiles]: https://github.com/dataprotocols/dataprotocols/issues/183
[dp-resources]: http://dataprotocols.org/data-packages/#resource-information
[fd]: http://data.okfn.org
[mapping]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[views]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[jts]: http://dataprotocols.org/json-table-schema/
[current-model]: http://docs.openspending.org/en/latest/model/design.html
[cofog]: http://unstats.un.org/unsd/cr/registry/regcst.asp?Cl=4
[imf-budget]: http://www.imf.org/external/pubs/ft/tnm/2009/tnm0906.pdf
