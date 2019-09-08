---
title: "Example: rmarkdown"
date: 2017-12-20T18:27:56+03:00 
draft: false
description: ""
tags: ["examples", "r", "rmarkdown"]
categories: ["software", "r", "rmarkdown"]
type: post
url:
author: "Mert Nuhoglu"
output: rmarkdown::html_document
blog: mertnuhoglu.com
resource_files:
- ./globals.R
---

Rmarkdown examples

<!--more-->

<!-- toc -->

## Read yaml header parameters

``` r
yml = rmarkdown::yaml_front_matter("~/projects/study/r/ex_r_rmarkdown.Rmd")
str(yml)
  #> List of 10
  #>  $ title      : chr "Example: rmarkdown"
  #>  $ date       : chr "`r strftime(Sys.time(), "%Y-%m-%dT%H:%M:%S+03:00")`"
  #>  $ draft      : logi FALSE
  #>  $ description: chr ""
  #>  $ tags       : NULL
  #>  $ categories : chr "examples r rmarkdown"
  #>  $ type       : chr "post"
  #>  $ url        : NULL
  #>  $ author     : chr "Mert Nuhoglu"
  #>  $ output     : chr "rmarkdown::html_document"
```

yaml header parameters in raw format are here:

``` bash
cat ~/projects/study/r/ex_r_rmarkdown.Rmd | head 
## ---
## title: "Example: rmarkdown"
## date: '`r strftime(Sys.time(), "%Y-%m-%dT%H:%M:%S+03:00")`'
## draft: false
## description: ""
## tags:
## categories: examples r rmarkdown
## type: post
## url:
## author: "Mert Nuhoglu"
```

## Find external resource references / dependencies

Assume that I have an external dependency such as the following code:

``` bash
cat ./globals.R
```

When first run, `find_external_resources()` couldn't find external dependencies:

``` r
rmarkdown::find_external_resources("~/projects/study/r/ex_r_rmarkdown.Rmd")
  #> [1] path     explicit web     
  #> <0 rows> (or 0-length row.names)
```

Then I added the following parameter to the yaml header of the Rmd file:

``` 
resource_files:
- ./globals.R
``` 

``` r
df = rmarkdown::find_external_resources("~/projects/study/r/ex_r_rmarkdown.Rmd")
df
  #>        path explicit   web
  #> 1 globals.R     TRUE FALSE
str(df)
  #> 'data.frame':    1 obs. of  3 variables:
  #>  $ path    : chr "globals.R"
  #>  $ explicit: logi TRUE
  #>  $ web     : logi FALSE
```

