---
layout: default
osep: 6
discussion: https://github.com/openspending/osep/issues/8
title: New DataStore in Detail
created: 26 April 2015
updated: 10 August 2015
authors: [Rufus Pollock, Paul Walsh]
accepted:
redirect_from: "/06-datastore.html"
---

At the heart of [OSEP 1][osep1] architecture is the DataStore which is the
primary repository for the datasets in OpenSpending. This document details
around the DataStore component will work.

[osep1]: ./01-approach-and-architecture-of-openspending.html

## Overview

OSEP 1 envisions a new, simple file-oriented DataStore. This OSEP puts flesh on
those bare bones.

Key questions to answer include:

* What actual backend is used for the DataStore
* What structure should the DataStore have (how do datasets map to files on
  disk)
* Does the DataStore support versioning and if so how
* What is the authorization structure and mechanism
* What changelog (and audit log) does the DataStore provide (e.g. recently
  added or updated datasets)

## Proposal

The key aspects of the DataStore are as follows:

* A dataset *can* be stored anywhere that provides a reasonable flat-file
  structure
* We specifically support github (with lfs) and S3 as storage backends. github
  is the primary backend. Specifically
  * Each dataset is stored in one git repositories on github
* There is a registry of datasets stored in a github repository called `registry` in the `os-team` organization. The registry is itself a Data Package.

As we allow datasets to be stored "anywhere" we need to be clear about
what a dataset being "in" OpenSpending means:

* A dataset is **"in"** OpenSpending if it is listed in the OpenSpending dataset
  registry
* Datasets which are conformant to the OpenSpending Data Package spec but are
  not in the dataset registry can be termed "OpenSpending compatible" but they
  are not "in" OpenSpending
* We introduce the concept of "core" datasets and "user" datasets. Core
  datasets are specifically maintained by the OpenSpending "data team".
* Core datasets MUST live in the github.com/os-data organization

### Dataset Structure

This is largely set out in OSEP-04 on OpenSpending Data Package. We only cover
here particularities of serialization to the backend.

* Aggregates: we propose storing aggregates in a directory called
  `/aggregates/`
  * Because of the size of aggregates and because of their frequent
    recalculation by a separate service it may be the case that the aggregates
    are stored on S3

### Github structure

* One dataset to one repository

### S3 structure

We use a S3 bucket called http://data.openspending.org/ for storage.

Within that each dataset is stored at:

```
/datasets/{owner}/{dataset-name}/
```

### Registry

The registry has the following structure:

```
datapackage.json
data/catalog.csv
```

catalog.csv is a CSV file with the following structure:

```
url,name,owner
...
```

* `url`: url to the dataset, usually the URL to the github repository
* `name`: the name of the dataset as set in its `datapackage.json` (will
  usually be the same as the name of the repository)
* `owner`: the username of the owner of the package. For datasets in github
  this will be the github username

### Authorization and Access Control



### Integration with Aggregations


### Changelog and Notifications

OSEP 01 envisages various independent services with the DataStore being one.
For the platform to work well, various of these services need to
intercommunicate. For example:

* When a new Dataset is added to the registry there must be a way for the
  frontend site to update with relatively limited latency
* When data is added or updated in a dataset, aggregates may need to be
  re-calculated.




### Other Issues

* "caching": we will probably want to back up each dataset that is "in" OpenSpending but not core. This is avoid datasets disappearing if someone deletes their dataset accidentally or intentionally. 

## Appendix

### What backend do we use?

Options are:

* s3
* git(hub) + s3
* github (now it has lfs)

### What Structure does the DataStore have?

* We have a two level name hierarachy: `{user-or-org}/{dataset-name}`

```
/datasets/{user-or-org}/{dataset-name}/
```
