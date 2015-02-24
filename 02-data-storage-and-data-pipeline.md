---
layout: default
osep: 2
title: Data Storage, ETL and the Data Processing Pipeline
created: 19 April 2013
updated: 8 June 2013
authors: Rufus Pollock
accepted: 1 August 2013
---


This document sets out a new data processing pipeline. It draws on OSEP-01
"Approach and Architecture" and proposes concrete changes to the core
architecture and data pipeline of OpenSpending.

Key proposed changes:

* Switch primary data storage to flat files on disk or "cloud" storage like s3
* Partition our ETL process in several better defined and smaller stages
  * Discovery and location of data
  * Basic cleaning and transformation (output Simple Data Format data packages at data.openspending.org)
  * Enhancement / restructuring to OpenSpending format (addition of specific metadata etc)
  * Registration (and, as appropriate) load into OpenSpending.org
* Separate the ETL process (and code) from the code to run OpenSpending.org
  * The ETL software does not need be integrated with the code for the website,
    analytics or visualization and separating will provide a better
    componentized design allowing much easier enhancement of the relevant
    components 

## Background

We face several significant challenges

* **Scaling**. We already encounter challenges with our current architecture
  and problems will likely worsen as the pace of data accumulation accelerates.
  Specific problems include:

  * DB level - core DB is Postgresql RDBMS. Given data size we are facing challenges both in terms of cost and management.
  * ETL process already has issues with import performance

* **Complexity of ETL process**

  * We have made huge strides since 2011 but the process is still complex and
    difficult for many people to navigate. This represents a huge blocker to
    community contribution
  * Lack of separation hinders code development and progressive enhancement
  * Hard to document

* **Extracting data in bulk from the main DB is difficult**

  * This blocks analysis and use by 3rd parties cf
    https://github.com/openspending/thingstodo/issues/4
  * It can also risk crashing the system if you attempt to download a very
    large dataset

The proposed approach assists on each of these:

* Scaling

  * nothing is easier or cheaper than flat file
  * easier to get donated space

* Partition the ETL process into discrete steps

  * Makes contribution easier
  * Easier to develop new processes and code


## Part I - to data.openspending.org

Outline of the initial process

1. (Identify data we are looking for)
2. Locate data
3. Retrieve data (and cache?)
   1. Designate an identifier for this data e.g. gov-spending-uk
4. Extract and transform => Simple Data Format (see below)
5. Save to data.openspending.org
   1. Locate at data.openspending.org/datasets/{id}/datapackage.json
   2. Suggest splitting data files up by time periods so as
      * have manageable size (many < 100 mb files rather than one huge multi-gb
        or terabyte file)
      * support incremental updates better (e.g. you do not need to replace the
        entire data file but just the latest one)

Note one could even separate steps 1-3 into a discrete research and discovery
phase further enhancing contribution and collaboration.

<img src="https://docs.google.com/a/okfn.org/drawings/d/1quRc7emv7fPrezqX9YWfqiAfyk_A7KPZ3N8gux2NSh4/pub?w=480&amp;h=210" />

## Part IIa - to OpenSpending.org

1. Create mapping
2. Do import (similar to the current process)

Aside: what does OS.org provide

* Web home
* Searchability
* Easy to create viz


## Appendix: Current Instructions used by RP on Open Data Days

These are the instructions RP uses on Open Data Maker nights:

1. Identify your local council (log your name + authority in this months list)
2. Grab the CSVs for that council from their website (link it below)
   * Search for {council name} spending 500
   * Check out http://schoolofdata.org/handbook/courses/finding-data/
3. Prepare a clean CSV
   * (basically CSV with no blank lines at the top)
   * Dates as yyyy-mm-dd
   * Currencies: remove "," but leave "."
4. Prepare them in OpenSpending format: http://openspending.org/help/data-cleansing.html 
   * have column called time
   * have column called amount
5. Upload the csv file somewhere (if you have had to clean up) e.g. DropBox, Google Drive, DataHub, ...
   * Suggest google docs (Make sure you publish to the web (File menu -> publish to the web))
   * [[ data.openspending.org - coming soon ]]
   * Alternative: http://datahub.io/ (if large) or gist.github.com (if small).
     * If you are using gist get the url from gist.github.com, create public
       gist and Create a column ‘amount’, and a column ‘time’ in the dataset
6. [Optional: Create a DataPackage.json describing the file - example - generator]
7. Register on OpenSpending if you don’t have an account
8. Create a new dataset - name it as gb-local-{council-name} e.g. gb-local-cambridge
   * Do NOT put the date / year ...
   * (put link below)
9. Create a visualization (treemap or bubblemap) and share here
10. Add your city or area to the Spending Map Spreadsheet here
    https://docs.google.com/a/okfn.org/spreadsheet/ccc?key=0AqR8dXc6Ji4JdHZZNUpWQ2paY3FfYTdFNXkxZXZDTWc#gid=0

