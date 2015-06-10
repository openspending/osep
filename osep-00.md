---
layout: default
osep: 0
discussion: https://github.com/openspending/osep/issues/11
title: OSEP Guidelines
created: 27 April 2015
updated: 27 April 2015
accepted: 10 June 2015
authors: [Paul Walsh]
---

## Overview

This document describes a set of lightweight guidelines for creating *Open Spending Enhancement Proposals*, or, *OSEPs*. The document itself follows the guidelines, and can therefore be used as a template for new OSEPs.

## Background

The need for a standard structure for OSEPs was first raised [here](https://github.com/openspending/osep/issues/1). However, to date the existing OSEPs do not follow any discernable structure.

## Proposal

### Structure

All OSEP documents should adhere to the following structure. This document is an implementation of this structure.

* Front matter
    * From matter provides meta data on the proposal. The following fields are required:
      * **osep**: (integer) The indentifying number for this OSEP. OSEP numbering is sequential.
      * **discussion**: (url) The URL of the discussion thread for this OSEP.
      * **title**: (string) The title of this OSEP
      * **created**: (date) The date that this OSEP was created
      * **updated**: (date) The date that this OSEP was last updated
      * **accepted**: (date) empty, or the date that the proposal was accepted
* Overview
    * A short (one-two paragraphs) description of the proposal.
* Background
    * A description of  any other background information that provides support or rationale for this proposal
* Proposal
    * The complete proposal, with as much detail as required to present the goals of the proposal.
* Appendix
    * Any related links or information that is of interest to consideration of the proposal.

### Acceptance

There is currently no clear acceptance process for OSEPs. This needs to be defined and clearly stated as part of OSEP-00.

### Additional

* Each proposal must have a place for discussing the proposal. This should be accessible via a URL on the `discussion` key of the front matter.
* Each proposal should be written in Markdown.
* Each proposal should be in a file with the following naming convention: `osep-{NUMBER}.md`

## Appendix

* The guidelines have been influenced by the Python Enhancement Proposal process, particularly [PEP 1](https://www.python.org/dev/peps/pep-0001/) and [PEP 9](https://www.python.org/dev/peps/pep-0009/)
