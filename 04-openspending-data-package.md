---
layout: default
title: OpenSpending Data Package
osep: 4
created: 8 December 2013
updated: 25 August 2014
authors: Rufus Pollock
accepted:
---

The specification of an OpenSpending Data Package is an essential part of the
move to a flat-file based DataStore (see OSEP 2) and [Github Issue
669][issue-669].

[issue-669]: https://github.com/openspending/openspending/issues/669

## OpenSpending Data Package

An OpenSpending data package is an extension of the [Budget Data Package][bdp]
which is, in turn, an extension of the [Tabular Data Package][tdp].

[bdp]: https://github.com/openspending/budget-data-package

Key points:

* It is a [Data Package][dp] and stores descriptor information in
  `datapackage.json`
* Data is stored in well-structured CSV files

[tdp]: http://dataprotocols.org/tabular-data-package/
[dp]: http://dataprotocols.org/data-packages/

To summarize, an OpenSpending Data Package on disk or in our store at
http://data.openspending.org/ will look something like this:

```
datapackage.json
# data files - can be data/ subdirectory or just in the base directory, must be CSV
data/my-financial-data.csv
```

A more complex version may have additional files 

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

### OpenSpending Data Package Profile

This documents the specific structure of the JSON metadata stored in the
`datapackage.json`.

* Base metadata is as per [Budget Data Package spec][bdp]
* Subsection under key "openspending" 

```
"name": ...
"title": ...
"currency": ...
"profiles": [
  "openspending": "1"
  "budget-data-package": ...
},
"openspending": {
  ... openspending specific profile info - see below
}
"resources": [
  ... as per Tabular Data Package and Budget Data Package ...
]
...
```

#### `openspending` entry

The `openspending` entry in the `datapackage.json` is for OpenSpending specific
additions. For v1 of the OS Data Package Profile we proposed to largely reuse
structures from current OpenSpending.

Specifically `openspending` entry in `datapackage.json`:

* MUST be a JSON object `{}`
* MAY have a `mapping` attribute providing a mapping of the data (CSV file in
  `resources`) to the Database. If present it MUST follow the structure defined
  in the [mapping specification in the OpenSpending documentation][mapping].
* MAY have a `views` attribute defining visualizations or views onto the data.
  If present, it MUST follow the structure defined in the [views specification
  in the OpenSpending documentation][views].

> Note: Budget data package has some required and standardized fields in the
> CSV. One may be able to automatically generate a mapping on this basis.
> However, this may be something for a separate `mapping` generator tool.

[mapping]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations 
[views]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations

#### Migration of Current OpenSpending Metadata

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

## Extras (under discussion)

* `archive` directory is for data that is not for openspending directly but is
  useful to archive raw data files from which data was extracted 
* Note it is possible to store the data elsewhere than in data.openspending.org -
  you can have datapackage.json point to files somewhere else on the web
    (though we may still want to cache that at some point)

