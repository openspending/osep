---
layout: default
osep: 6
title: New DataStore in Detail
created: 26 April 2015
updated: 26 April 2015
authors: [Rufus Pollock, Paul Walsh]
accepted:
redirect_from: "/06-datastore.html"
---

At the heart of [OSEP 1][osep1] architecture is the DataStore which is the primary repository for the datasets in OpenSpending. This document details around the DataStore component will work.

[osep1]: ./01-approach-and-architecture-of-openspending.html

## Overview

OSEP 1 envisions a new, simple file-oriented DataStore. This OSEP puts flesh on those bare bones.

Key questions to answer include:

* What actual backend is used for the DataStore
* What structure should the DataStore have (how do datasets map to files on disk)
* Does the DataStore support versioning and if so how
* What is the authorization structure and mechanism
* What changelog (and audit log) does the DataStore provide (e.g. recently added or updated datasets)

## Details

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
