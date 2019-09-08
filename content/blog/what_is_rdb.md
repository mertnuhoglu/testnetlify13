---
title: "What is RDB (Requirements Database)? - WIP"
date: 2018-01-17T18:30:12+03:00 
draft: false
description: ""
tags: ["rdb"]
categories: ["software", "rdb"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: mertnuhoglu.com
resource_files:
- data/datamodel_enum.png
path: ~/projects/study/problem/rdb/what_is_rdb.md
state: wip

---

`RDB` is the abbreviation for requirements database. Requirements database stores the requirements information in structured relational format in Excel or tsv files.

This document is a work-in-process. I will work on it.

<!--more-->

Here is an example data set for RDB:

``` r
den = tibble::tribble(
  ~data_entity_id,               ~entity_name,
               7L,               "enum_value",
               8L,                 "enum_var"
  )
dfl = tibble::tribble(
  ~data_field_id,         ~data_field_name,          ~type,     ~pk_fk, ~not_null, ~data_entity_id, ~fk_data_entity_id,     ~enum_var_name,
             26L,          "enum_value_id",          "INT",       "PK",     FALSE,              7L,                 NA,                 NA,
             27L,            "enum_var_id",          "INT",       "FK",      TRUE,              7L,                 8L,                 NA,
             28L,        "enum_value_name",         "TEXT",  "NON_KEY",     FALSE,              7L,                 NA,                 NA,
             29L,            "enum_var_id",          "INT",       "PK",     FALSE,              8L,                 NA,                 NA,
             30L,          "enum_var_name",         "TEXT",  "NON_KEY",     FALSE,              8L,                 NA,                 NA
  )
```

This data set stores metadata for two entities (tables): `enum_value` and `enum_var` 

This meta data actually describes attributes of these two entities. The UML diagram of these entities are as here:

![UML Diagram of the Entities: enum_var and enum_value](/images/datamodel_enum.png)

Having meta data of entities defined as structured data helps me to automate several development and management related tasks. For example, I can generate [DDL statements](/tech/rdb_to_ddl/) of the database tables and [sample data](/tech/rdb_to_data/) for these tables automatically from RDB.

