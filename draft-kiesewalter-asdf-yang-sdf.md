---
title: >
  Mapping between YANG and SDF
abbrev: Mapping between YANG and SDF
docname: draft-kiesewalter-asdf-yang-sdf-latest
date: 2021-06-22

stand_alone: true

ipr: trust200902
keyword: Internet-Draft
cat: info

pi: [toc, sortrefs, symrefs, compact, comments]

author:
  - name: Jana Kiesewalter
    org: Universität Bremen
    email: jankie@uni-bremen.de
  - name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org
    role: editor

normative:
  I-D.ietf-asdf-sdf: sdf
  RFC7950: yang
informative:
  SDF-YANG-CONVERTER:
    -: impl
    target: sdf-yang-converter.org
    title: SDF YANG converter playground
    author:
      name: Jana Kiesewalter
  SDF-YANG-CONVERTER-IMPL:
    -: implsrc
    target: https://github.com/jkiesewalter/sdf-yang-converter
    title: SDF YANG converter
    author:
      name: Jana Kiesewalter

--- abstract

(briefly explain what we are trying to do, e.g.:)

YANG and SDF are two languages for modelling the interaction with and
the data interchanged with devices in the network.
As their areas of application (network management, IoT, resp.)
overlap, it is useful to be able to translate between the two.

The present specification provides information about how models in one
of the two languages can be translated into the other.
This specification is not intended to be normative, but to help with
creating translators.


--- middle

Introduction        {#intro}
============

(See Abstract for now, add references {{-yang}} and {{-sdf}}.)

Pairing SDF and YANG features
======================

{{yang-to-sdf}} gives an overview over how language features of YANG
can be mapped to SDF features.  In many cases, several translations
are possible, and the right choice depends on the context.

| YANG      | SDF                                                                                        |
|-----------|--------------------------------------------------------------------------------------------|
| module    | SDF model (i.e. info block, namespace section & definitions)                               |
| container | sdfObject (top-level container)                                                            |
|           | sdfProperty of compound-type (container that is a child node of top-level container)       |
|           | property of a compound-type element (container on any other level)                         |
| list      | sdfProperty of type array, items of type object (top-level list or one level below)        |
|           | property of a compound-type element of type array, items of type object (any other level)  |
| leaf-list | sdfProperty of type array, items of simple types (top-level leaf-list or one level below)  |
|           | property of a compound-type element of type array, items of simple types (any other level) |
| leaf      | sdfProperty of a simple type (top-level leaf or one level below)                           |
|           | property of a compound-type element of a simple type (any other level)                     |
| typedef   | sdfData of a simple type                                                                   |
| grouping  | sdfData of compound-type                                                                   |
{: #yang-to-sdf title="Mapping YANG to SDF"}

{{sdf-to-yang}} provides the inverse mapping.


| SDF                                                          | YANG                                                       |
| SDF model (i.e. info block, namespace section & definitions) | module                                                     |
| sdfThing, sdfObject                                          | container                                                  |
| sdfProperty  of compound-type                                | container                                                  |
| sdfProperty of simple types                                  | Leaf                                                       |
| sdfProperty of type array with items of simple types         | Leaflist                                                   |
| sdfProperty of type array with items of compound-type        | List                                                       |
| sdfAction                                                    | RPC with input/output nodes for sdfInputData/sdfOutputData |
| sdfEvent                                                     | notification with nodes for sdfOutputData                  |
| sdfData  of compound-type                                    | grouping                                                   |
| sdfData of simple types                                      | typedef                                                    |
| sdfData of type array with items of simple types             | grouping with leaflist                                     |
| sdfData of type array with items of compound-type            | grouping with list                                         |
{: #sdf-to-yang title="Mapping SDF to YANG"}

Implementation Considerations
=================================

An implementation of an initial converter between SDF and YANG can be
found at {{-impl}}; the source code can be found at {{-implsrc}}.

IANA Considerations
==================

This document makes no requests of IANA.


Security considerations
=======================

The security considerations of {{-yang}} and {{-sdf}} apply.

--- back

Acknowledgements
================
{: numbered="no"}

TBD.

