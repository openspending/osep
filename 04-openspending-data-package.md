---
title: An OpenSpending Data Package
created: 2013-12-08
updated: 2014-01-09
status: Draft
accepted:
---

# An OpenSpending Data Package

The specification of an OpenSpending Data Package is an essential part of the
move to a flat-file based DataStore (see OSEP 2) and [Github Issue
669][issue-669].

issue-669: https://github.com/openspending/openspending/issues/669

## OpenSpending Data Package

An OpenSpending data package is an extension of the [Simple Data Format Data
Package][sdf]. Key points:

* It is a [Data Package][dp] and stores descriptor information in `datapackage.json`
* Data is stored in well-structured CSV files

[sdf]: http://dataprotocols.org/simple-data-format/
[dp]: http://dataprotocols.org/data-packages/

To summarize an OpenSpending data package on disk or in our store at
http://data.openspending.org/ will look something like this:

```
datapackage.json
README.md           # optional
# data files - can be data/ subdirectory or just in the base directory, must be CSV
data/my-financial-data.csv
```

## OpenSpending Data Package Profile (specific metadata)

cf http://docs.openspending.org/en/latest/model/design.html

* Base metadata is as per (Simple Data Format) Data Package spec
* Resources is as per Simple Data Format 
* Subsection under key "openspending" with `mapping` and `views` section (as per current OS spec)
  * Propose new attribute in openspending section: openspendingVersion with value of 1

```
"name": ...
"title": ...
"currency": ...
"openspending": {
  "openspendingVersion": 1
  "mapping": ...
  "views": ...
}
"resources": [
]
...
```

### Implied Changes

* `dataset` entry changes as follows
  * `name`: no change
  *  `label` => title
  * `currency` => currency (new)
  * `description` no change

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

