---
title: "Process: From yuml to ddl and test data"
date: 2018-01-20T18:30:12+03:00 
draft: false
description: ""
tags: ["r", "sql", "datafiller", "yuml"]
categories: ["software", "r", "sql"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: mertnuhoglu.com
resource_files:
-
path: ~/projects/study/problem/data_generation/yuml_to_data_process.md
state: wip

---

I wanted to automate the process from data modelling to testing it with sample data in actual database.

<!--more-->

<!-- toc -->

To describe the data models, I use a very simple domain specific language (DSL) called as `yuml`. `yuml` is developed and used by some UML tool called [yuml.me](http://yuml.me "yuml.me").

Starting from `yuml` the flow of information is as follows:

    | yuml -> uml | yuml_to_uml.sh | yuml_to_uml.Rmd |
    | yuml -> rdb | yuml_to_rdb.R  | yuml_to_rdb.Rmd |
    | rdb -> ddl  | rdb_to_ddl.R   | rdb_to_ddl.Rmd  |
    | ddl -> data | ddl_to_data.sh | ddl_to_data.Rmd | ex_sql_data_generator_datafiller.Rmd |
    | rdb -> data | rdb_to_data.R  | rdb_to_data.Rmd |

`rdb` is short for "requirements database". The name is actually a little misleading. Maybe, I should call it as "structural meta database". `rdb` files are some tsv/excel files that store meta data about the final product. I prefer to use `excel` instead of `tsv/csv` because it is easier to edit and view manually.

## `yuml -> uml` 

First step is converting `yuml` text to `uml` graphs. This is actually not required for data generation. But it is very easy to do and helps me visualize and understand the data model better.

    yuml -i datamodel_enum.md -o datamodel_enum.png

This step is explained in [yuml_to_uml](/tech/yuml_to_uml/). 

## `yuml -> rdb`

Second step is converting `yuml` text to `RDB` data. 

[RDB](/tech/what_is_rdb/) is an abbreviation that I use for requirements database. I try to store all software specifications as structured data in a relational database format.

Conversion of `yuml` to `RDB` is done by an R function called `yuml_to_rdb`:

    rdb = yuml_to_rdb(data_model_dir)

This step is explained in [yuml_to_rdb](/tech/yuml_to_rdb/). 

## `rdb -> ddl`

Third step is converting `RDB` meta data to database schema defined as `DDL` statements.

Conversion of `RDB` to `DDL` is done by an R function called `rdb_to_ddl`:

    ddl = rdb_to_ddl( data_entity = den, data_field = dfl)

This step is explained in [rdb_to_ddl](/tech/rdb_to_ddl/). 

## Generating Data: `ddl -> data` and `rdb -> data`

Fourth step is generating sample data. This consists of two steps that are independent of each other: `ddl -> data` and `rdb -> data` 

In  `ddl -> data` step, sample data is generated using `datafiller` directives embedded as comments inside `DDL` statements. This step is explained in [ddl_to_data](/tech/ddl_to_data/). 

In `rdb -> data` step, sample data is generated from `RDB` data. In general, I generate sample data from `DDL`. But some data must be predefined in order to make the system work. This data is mostly special knowledge such as enum values. For example, different types of package, different types of weight units are of this kind. I define this kind of knowledge in `RDB` database inside tsv files. Then I generate their `SQL` statements automatically using `rdb_to_ddl` function. 

This step is explained in [rdb_to_data](/tech/rdb_to_data/)


