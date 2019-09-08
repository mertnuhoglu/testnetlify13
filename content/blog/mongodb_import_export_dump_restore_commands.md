---
title: "mongodb: import/export dump/restore command snippets"
date: 2018-02-05T14:20:16+03:00 
draft: false
description: ""
tags: ["mongo"]
categories: ["software", "mongo"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: mertnuhoglu.com
resource_files:
-
path: ~/projects/study/r/read_json.md

---

I have a mongodb database. I need to export import its data frequently. In this post, I keep mongo terminal commands to remember how to do it.

<!--more-->

<!-- toc -->

# JSON vs BSON (Dump vs Partial Data)

First issue is what the data format is going to be. 

mongodb has two pairs of commands:

- `mongodump` and `mongorestore` that exports and imports **everything** in BSON format. 
- `mongoexport` and `mongoimport` that exports and imports only requested collection in JSON format.

## Dump and Restore BSON Files

`mongodump` command dumps everything into the directory given by `-o` argument in BSON format.

``` 
mongodump -h ${HOSTNAME} -u ${DBUSER} -p ${DBPASS} -d ${DBNAME} -o /srv/app/mongo
``` 

Note that the following expressions are environment variables defined in bash terminal: `-h ${HOSTNAME} -u ${DBUSER} -p ${DBPASS} -d ${DBNAME}`

`mongorestore` command reads all BSON files in the directory and restores the database.

``` 
mongorestore -h ${HOSTNAME} -u ${DBUSER} -p ${DBPASS} --db ${DBNAME} /srv/app/mongo
``` 

## Export and Import JSON Files

`mongoexport` and `mongoimport` commands export and import only one collection per call.

``` 
mongoexport -h ${HOSTNAME} -u ${DBUSER} -p ${DBPASS} -d ${DBNAME} -c admin_locations -o admin_locations.json --jsonArray 
``` 

This statement exports a single collection (aka table) called `admin_locations` into the file `admin_locations.json`. 

Note the argument `--jsonArray`. This argument lets mongodb to put multiple json records into an array. If you don't specify it, all json records are separated by whitespace and many JSON readers will fail to read the file then.

``` 
mongoimport -h ${HOSTNAME} -u ${DBUSER} -p ${DBPASS} -d ${DBNAME} --file admin_locations.json
``` 

This statement imports the file `admin_locations.json` into a collection. Since the collection name is not given, mongodb assumes that it has the same name as json file ie. `admin_locations`.
