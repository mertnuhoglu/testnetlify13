---
title: "Debug: mutate with ifelse overwrites a non-NA value with NA"
date: 2017-12-22T18:27:56+03:00 
draft: false
description: ""
tags: ["r", "debug"]
categories: ["software", "r"]
type: post
url:
author: "Mert Nuhoglu"
output: rmarkdown::html_document
blog: mertnuhoglu.com
resource_files:
-
---

Debugging a weird problem

<!--more-->

``` r
library(dplyr)
df = tibble::tribble(
  ~data_field_name,   ~type,
      "vehicle_id",      NA,
            "type",  "TEXT"
  )
df
  #> # A tibble: 2 x 2
  #>   data_field_name  type
  #>             <chr> <chr>
  #> 1      vehicle_id  <NA>
  #> 2            type  TEXT
```

After `mutate()`, second row's `type` field becomes `NA`. This is unexpected behaviour.

``` r
df %>%
  dplyr::mutate( type = ifelse( rutils::is_na(type), "TEXT", type)) 
  #> # A tibble: 2 x 2
  #>   data_field_name  type
  #>             <chr> <chr>
  #> 1      vehicle_id  TEXT
  #> 2            type  <NA>
```

Using `base::is.na()` instead of `rutils::is_na()` solves the problem.

``` r
df %>%
  dplyr::mutate( type = ifelse( is.na(type), "TEXT", type)) 
  #> # A tibble: 2 x 2
  #>   data_field_name  type
  #>             <chr> <chr>
  #> 1      vehicle_id  TEXT
  #> 2            type  TEXT
```

Here is the definition of `rutils::is_na()`:

``` r
is_na = function(x) is.na(x) | all(ifelse( class(x) == "character", x == "NA", FALSE))
```

Probably, the unexpected behaviour is caused by intermixing of column data and row data. Normally, we expect `mutate()` to work for each row separately. But in this case, probably `x` argument is bound to the whole column vector not just a single row of it.
