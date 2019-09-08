---
title: "Debug: Using `data.table` Bracket `[]` in Own Package"
date: 2017-12-21T14:27:56+03:00 
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

Bug: Using data.table inside own package messes up bracket `[]` operator

<!--more-->

# Bug: Using data.table inside own package messes up bracket `[]` operator

In a `data.table` object, the bracket `[]` operator can be used to extract a row.

``` r
dt = data.table::data.table(a = 1:3, b = 10:12)
data.table::setkey(dt, a)
dt[2]$b = 20
print(dt[2])
  #>    a  b
  #> 1: 2 20
```

This is different than the usual bracket in a `data.frame` object. Here, the above idiom gives an error:

``` r
df = data.frame(a = 1:3, b = 10:12)
print(dt[2])
  #> Error in dt[2]: object of type 'closure' is not subsettable
```

But when using that idiom inside my own package, the behaviour of the bracket `[]` operator is different. Instead of returning a row, it returns a column then:

``` r
dt = data.table::data.table(a = 1:3, b = 10:12)
data.table::setkey(dt, a)
dt[2]$b = 20
print(dt[2])
  #>     b
  #> 1: 20
  #> 2: 20
  #> 3: 20
```

To solve the problem, put `import(data.table)` line into `NAMESPACE` or put the following lines into any R script file in the package, so that `devtools::document` puts `import` into `NAMESPACE`.

``` r
  #' @import data.table
NULL
```

