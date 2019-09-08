---
title: "Refcard Loops in R"
date: 2017-12-16T18:27:56+03:00 
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

There are too many ways of looping in R data structures. Here is an unfinished summary of them.

<!--more-->

<!-- toc -->

# Looping Basic

``` r
mylist = list(10, 20)
for (e in mylist) print(e)
  #> [1] 10
  #> [1] 20
```

``` r
myvec = c(10, 20)
for (e in myvec) print (e)
  #> [1] 10
  #> [1] 20
```

``` r
myvec = c(10, 20)
for (i in seq_along(myvec)) print (myvec[i])
  #> [1] 10
  #> [1] 20
```

``` r
dfa = data.frame( a_id = 1:3, a_title = letters[1:3], b_id = c(10,10,11) )
for (i in 1:nrow(dfa))
  print(dfa[i, ])
  #>   a_id a_title b_id
  #> 1    1       a   10
  #>   a_id a_title b_id
  #> 2    2       b   10
  #>   a_id a_title b_id
  #> 3    3       c   11
```

``` r
dfa = data.frame( a_id = 1:3, a_title = letters[1:3], b_id = c(10,10,11) )
rutils::applyr(dfa, print)
  #>    a_id a_title    b_id 
  #>     "1"     "a"    "10" 
  #>    a_id a_title    b_id 
  #>     "2"     "b"    "10" 
  #>    a_id a_title    b_id 
  #>     "3"     "c"    "11"
```

# Looping over Data Frame Rows

Setup 

``` r
library(dplyr, warn.conflicts = F)
dfa = data_frame( a_id = 1:3, a_title = letters[1:3], b_id = c(10,10,11) )
dfb = data_frame( b_id = 10:12, b_title = letters[10:12] )
```

# When Can We Not Use Loop Functions?

There are cases when we cannot use loop functions like `lapply`, `apply`, `Map`

1. When we loop over each distinct value of a field of a table. See: [ex_sql_generation.Rmd](/tech/ex_sql_generation/ "SQL Generation")
