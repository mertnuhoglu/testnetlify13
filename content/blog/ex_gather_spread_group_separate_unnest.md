---
title: "Example: gather spread group_by separate unnest"
date: 2017-12-23T18:27:56+03:00 
description: ""
tags:
categories: ["datascience", "r", "dplyr"]
type: post
draft: false
url:
author: "Mert Nuhoglu"
output: html_document
blog: datascience.mertnuhoglu.com
resource_files:

---

Aggregation (dplyr::summarise) functions and window functions (dplyr::lead, lag, top_n etc.) operate on rows only. Aggregation functions output single row from an input of a group of rows. Window functions output one row per input row.

<!--more-->

<!-- toc -->

```r
library(dplyr, warn.conflicts = F)
library(tidyr, warn.conflicts = F)
```

Aggregation (dplyr::summarise) functions and window functions (dplyr::lead, lag, top_n etc.) operate on rows only. Aggregation functions output single row from an input of a group of rows. Window functions output one row per input row.

gather, spread, separate, unite are different. They reshape data that is they work not only on rows, but also on columns. Their input and output have some effect on columns in addition to rows. So, they change the shape of the data.

There is a great [visual cheatsheet on data wrangling by RStudio](https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf)

## gather: gather columns into key-value pairs

**Usage**

    gather(data, key = "key", value = "value", ..., na.rm = FALSE, convert = FALSE, factor_key = FALSE)

Input: multiple columns. Output: two columns as key-value columns and multiple rows. 

### gather example 01

``` r
set.seed(1)
stocks <- data.frame(
  time = as.Date('2009-01-01') + 0:1,
  X = rnorm(2, 0, 1),
  Y = rnorm(2, 0, 2)
)
stocks
  #>         time          X         Y
  #> 1 2009-01-01 -0.6264538 -1.671257
  #> 2 2009-01-02  0.1836433  3.190562
```

``` r
stocks %>% gather(stock, price, -time)
  #>         time stock      price
  #> 1 2009-01-01     X -0.6264538
  #> 2 2009-01-02     X  0.1836433
  #> 3 2009-01-01     Y -1.6712572
  #> 4 2009-01-02     Y  3.1905616
```

`stock` and `price` are `key` - `value` pairs.

Note that, `X` and `Y` are actually not variables. `stock` is the variable. 

### gather example 02

``` r
mini_iris <- iris[c(1, 51), c(1,2,5)]
mini_iris
  #>    Sepal.Length Sepal.Width    Species
  #> 1           5.1         3.5     setosa
  #> 51          7.0         3.2 versicolor
``` 

`Sepal.Length` and `Sepal.Width` are not variables. They are `key` values. `values` for these keys are `5.1`, `3.5` etc.

``` r
tidyr::gather(mini_iris, key = flower_att, value = measurement,
              Sepal.Length, Sepal.Width)
  #>      Species   flower_att measurement
  #> 1     setosa Sepal.Length         5.1
  #> 2 versicolor Sepal.Length         7.0
  #> 3     setosa  Sepal.Width         3.5
  #> 4 versicolor  Sepal.Width         3.2
```

``` r
gathered_iris = mini_iris %>% 
  tidyr::gather(key = flower_att, value = measurement, -Species)
gathered_iris
  #>      Species   flower_att measurement
  #> 1     setosa Sepal.Length         5.1
  #> 2 versicolor Sepal.Length         7.0
  #> 3     setosa  Sepal.Width         3.5
  #> 4 versicolor  Sepal.Width         3.2
```

## spread: ungather key-value pair across multiple columns

``` r
gathered_iris %>%
  tidyr::spread(flower_att, measurement)
  #>      Species Sepal.Length Sepal.Width
  #> 1     setosa          5.1         3.5
  #> 2 versicolor          7.0         3.2
```

### Error: spread fails when there are redundant columns

``` r
s1 = tibble::tribble(
  ~id,   ~key, ~value,
  1,   "a",  2,
  1,   "b",  3,
  2,   "a",  2
  ) %>%
  tidyr::spread(key, value)
s1
  #> # A tibble: 2 x 3
  #>      id     a     b
  #> * <dbl> <dbl> <dbl>
  #> 1     1     2     3
  #> 2     2     2    NA
```

``` r
library(dplyr, warn.conflicts = F)
s2 = tibble::tribble(
  ~id,   ~key, ~value, ~amount
  1,   "a",  2, 10,
  1,   "b",  3, 20,
  2,   "a",  2, 15
  ) %>%
  tidyr::spread(key, value)
  #> Error: <text>:4:3: unexpected numeric constant
  #> 3:   ~id,   ~key, ~value, ~amount
  #> 4:   1
  #>      ^
```

## separate: separate one column into several

**Usage**

    separate(data, col, into, sep = "[^[:alnum:]]+", remove = TRUE, convert = FALSE, extra = "warn", fill = "warn", ...)

``` r
tb = tibble::tribble(
  ~year,   ~demo, ~n,
  2015 ,   "m04",  2,
  2015 ,   "f04",  3,
  2015 ,   "m05",  1,
  2015 ,   "f05",  0
  )
sep_tb = tb %>%
  tidyr::separate(demo, c("sex", "age"), 1)
sep_tb
  #> # A tibble: 4 x 4
  #>    year   sex   age     n
  #> * <dbl> <chr> <chr> <dbl>
  #> 1  2015     m    04     2
  #> 2  2015     f    04     3
  #> 3  2015     m    05     1
  #> 4  2015     f    05     0
```

Note: `spread` is similar to `separate` because both reshape one column into multiple columns. But `spread` takes as input two columns: `key-value` whereas `separate` takes as input only one column.

## unite: unseparate several columns into one

**Usage**

    unite(data, col, ..., sep = "_", remove = TRUE)

``` r
sep_tb %>%
  tidyr::unite(demo, sex, age, sep = "")
  #> # A tibble: 4 x 3
  #>    year  demo     n
  #> * <dbl> <chr> <dbl>
  #> 1  2015   m04     2
  #> 2  2015   f04     3
  #> 3  2015   m05     1
  #> 4  2015   f05     0
```

### unnest: unnest a list column

A column can be a list or dataframe. `unnest` converts multiple values in a row into multiple rows.

**Usage**

    unnest(data, ..., .drop = NA, .id = NULL, .sep = NULL)

``` r
df1 = tibble::tribble(
  ~x,    ~y,
   1,   "a",
   2, "d,e"
)
``` 

``` r
df2 = df1 %>%
  transform(y = strsplit(y, ","))
str(df2)
  #> 'data.frame':    2 obs. of  2 variables:
  #>  $ x: num  1 2
  #>  $ y:List of 2
  #>   ..$ : chr "a"
  #>   ..$ : chr  "d" "e"
```

Note that, y column contains a list of character vectors.

``` r
df2 %>%
  unnest(y)
  #>   x y
  #> 1 1 a
  #> 2 2 d
  #> 3 2 e
```

Alternatively use `unnest` directly:

``` r
df2 = df1 %>%
  unnest(y = strsplit(y, ","))
df2
  ## # A tibble: 3 x 2
  ##       x     y
  ##   <dbl> <chr>
  ## 1     1     a
  ## 2     2     d
  ## 3     2     e
```

### nest: reverse of unnest

``` r
df3 = df2 %>%
  nest(y)
str(df3)
  ## Classes 'tbl_df', 'tbl' and 'data.frame':    2 obs. of  2 variables:
  ##  $ x   : num  1 2
  ##  $ data:List of 2
  ##   ..$ :Classes 'tbl_df', 'tbl' and 'data.frame': 1 obs. of  1 variable:
  ##   .. ..$ y: chr "a"
  ##   ..$ :Classes 'tbl_df', 'tbl' and 'data.frame': 2 obs. of  1 variable:
  ##   .. ..$ y: chr  "d" "e"
```

### unnest and group_by: take last element in each group

``` r
ef1 = data.frame( a = c("ali,veli", "can,cin" ) )
ef1
  ##          a
  ## 1 ali,veli
  ## 2  can,cin
``` 

I want to get the last word in `a` column for each row.

``` r
ef2 = ef1 %>%
  dplyr::mutate( b = stringr::str_split(a, ",") ) %>%
  tidyr::unnest(b) %>%
  dplyr::group_by(a) 
ef2
  #> # A tibble: 4 x 2
  #> # Groups:   a [2]
  #>          a     b
  #>     <fctr> <chr>
  #> 1 ali,veli   ali
  #> 2 ali,veli  veli
  #> 3  can,cin   can
  #> 4  can,cin   cin
``` 

``` r
ef2 %>%
  dplyr::filter(row_number()==n())
  #> # A tibble: 2 x 2
  #> # Groups:   a [2]
  #>          a     b
  #>     <fctr> <chr>
  #> 1 ali,veli  veli
  #> 2  can,cin   cin
``` 

What does `row_number()==n()` mean? Let's check what `row_number()` and `n()` produce by themselves?

``` r
ef2 %>%
  dplyr::mutate(row = row_number())
  #> # A tibble: 4 x 3
  #> # Groups:   a [2]
  #>          a     b   row
  #>     <fctr> <chr> <int>
  #> 1 ali,veli   ali     1
  #> 2 ali,veli  veli     2
  #> 3  can,cin   can     1
  #> 4  can,cin   cin     2

ef2 %>%
  dplyr::mutate(row = n())
  #> # A tibble: 4 x 3
  #> # Groups:   a [2]
  #>          a     b   row
  #>     <fctr> <chr> <int>
  #> 1 ali,veli   ali     2
  #> 2 ali,veli  veli     2
  #> 3  can,cin   can     2
  #> 4  can,cin   cin     2
```

