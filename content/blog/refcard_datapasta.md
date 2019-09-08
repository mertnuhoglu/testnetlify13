---
title: "Refcard datapasta"
date: 2017-12-18T18:27:56+03:00 
draft: false
description: ""
tags: ["r", "examples"]
categories: ["software", "r"]
type: post
url:
author: "Mert Nuhoglu"
output: rmarkdown::html_document
blog: mertnuhoglu.com
resource_files:
-
---

datapasta is an R tool to generate reproducible code documentation.

<!--more-->

<!-- toc -->

# Start from data.frame

Initially I have some data.frame. `dpasta()` generates reproducible code for the data.frame

``` r
library(magrittr)
library(datapasta)
#> Warning: package 'datapasta' was built under R version 3.3.2
df = data.frame(stringsAsFactors=FALSE,
    data_field_aid = c(1L, 2L),
   data_field_name = c("vehicle_id", "type"),
              type = c(NA, "TEXT"),
             pk_fk = c(NA, NA),
   data_entity_aid = c(1L, 1L),
       entity_name = c("Vehicle", "Vehicle")
)
df %>% dpasta()
#> data.frame(stringsAsFactors=FALSE,
#>     data_field_aid = c(1L, 2L),
#>    data_field_name = c("vehicle_id", "type"),
#>               type = c(NA, "TEXT"),
#>              pk_fk = c(NA, NA),
#>    data_entity_aid = c(1L, 1L),
#>        entity_name = c("Vehicle", "Vehicle")
#> )
df
#>   data_field_aid data_field_name type pk_fk data_entity_aid entity_name
#> 1              1      vehicle_id <NA>    NA               1     Vehicle
#> 2              2            type TEXT    NA               1     Vehicle
```

# Start from tibble

Initially I have some tibble. I want to generate reproducible code for df tibble:

``` r
library(dplyr)
library(datapasta)
df = tibble::tribble(
  ~data_field_aid, ~data_field_name,   ~type, ~pk_fk, ~data_entity_aid, ~entity_name,
               1L,     "vehicle_id",      NA,     NA,               1L,    "Vehicle",
               2L,           "type",  "TEXT",     NA,               1L,    "Vehicle"
)
df %>% dpasta()
#> tibble::tribble(
#>   ~data_field_aid, ~data_field_name,   ~type, ~pk_fk, ~data_entity_aid, ~entity_name,
#>                1L,     "vehicle_id",      NA,     NA,               1L,    "Vehicle",
#>                2L,           "type",  "TEXT",     NA,               1L,    "Vehicle"
#>   )
```

# Vector and List

Use `dpasta()` or `vector_paste()` for reproducing vector data:

``` r
library(magrittr)
library(datapasta)
mtcars$mpg %>% head() %>% dpasta()
#> c(21, 21, 22.8, 21.4, 18.7, 18.1)
```

Works with lists too:

``` r
library(magrittr)
library(datapasta)
#> Warning: package 'datapasta' was built under R version 3.3.2
alist = list(mpg = head(mtcars$mpg), cyl = head(mtcars$cyl))
alist %>% dpasta()
#> c(c(21, 21, 22.8, 21.4, 18.7, 18.1), c(6, 6, 4, 6, 8, 6))
```

Vertical printing:

``` r
library(magrittr)
library(datapasta)
mtcars$mpg %>% head() %>% vector_paste_vertical()
#> c(21,
#>   21,
#>   22.8,
#>   21.4,
#>   18.7,
#>   18.1)
```

# Copy Table from Excel/TSV

Assume that I copied the following excel/tsv data into the clipboard:

    "data_entity_aid"	"entity_name"
    1	"Vehicle"
    2	"Shipment"
    3	"Shipment_Action"

Now, I can reproduce data.frame code for this data using `df_paste()`

``` r
df_paste()
#> data.frame(stringsAsFactors=FALSE,
#>    X.data_entity_aid. = c(1L, 2L, 3L),
#>        X.entity_name. = c("\"Vehicle\"", "\"Shipment\"", "\"Shipment_Action\"")
#> )
```

# Read Data from Excel/TSV

This time, instead of copying data manually from Excel/TSV, I will read it inside R and then reproduce it:

``` {r}
library(magrittr)
readr::read_tsv("data/refcard_datapasta_data_entity.tsv") %>%
  datapasta::dpasta()
  ##> tibble::tribble(
  ##>   ~data_entity_id,               ~entity_name,
  ##>                 1,          "address_variant",
  ##>                 2,                  "address",
  ##>                 3,  "administrative_location"
  ##>   )
``` 

Instead of `readr`, I can also use `rio::import` but this time the output of `dpasta()` is not as nicely formatted as `tibble`:

``` {r}
rio::import("data/refcard_datapasta_data_entity.tsv") %>%
  datapasta::dpasta()
  ##> data.frame(stringsAsFactors=FALSE,
  ##>    data_entity_id = c(1L, 2L, 3L),
  ##>       entity_name = c("address_variant", "address", "administrative_location")
  ##> )
``` 

I can also read using the base `read.csv`

``` {r}
read.csv("data/refcard_datapasta_data_entity.tsv", sep = "\t") %>%
  datapasta::dpasta()
  ##> data.frame(
  ##>    data_entity_id = c(1L, 2L, 3L),
  ##>       entity_name = as.factor(c("address_variant", "address",
  ##>                                 "administrative_location"))
  ##> )
``` 

# Code for Markdown

`dmdclip()` produces code that is intended with 4 spaces to use in markdown documents.

``` r
library(magrittr)
library(datapasta)
mtcars %>% head() %>% dmdclip()
```

    data.frame(
             mpg = c(21, 21, 22.8, 21.4, 18.7, 18.1),
             cyl = c(6, 6, 4, 6, 8, 6),
            disp = c(160, 160, 108, 258, 360, 225),
              hp = c(110, 110, 93, 110, 175, 105),
            drat = c(3.9, 3.9, 3.85, 3.08, 3.15, 2.76),
              wt = c(2.62, 2.875, 2.32, 3.215, 3.44, 3.46),
            qsec = c(16.46, 17.02, 18.61, 19.44, 17.02, 20.22),
              vs = c(0, 0, 1, 1, 0, 1),
              am = c(1, 1, 1, 0, 0, 0),
            gear = c(4, 4, 4, 3, 3, 3),
            carb = c(4, 4, 1, 1, 2, 1)
    )

