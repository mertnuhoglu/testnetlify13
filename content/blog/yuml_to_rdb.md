---
title: "Convert yuml to RDB Meta Data"
date: 2018-01-17T18:30:12+03:00 
draft: false
description: ""
tags: ["r", "rdb"]
categories: ["software", "r", "rdb"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: mertnuhoglu.com
resource_files:
-
path: ~/projects/study/r/yuml_to_rdb.md
state: wip

---

I wanted to automate the process from data modelling to testing it with sample data in actual database.

<!--more-->

<!-- toc -->

```r
library(dplyr, warn.conflicts = F)
data_model_dir = "/Users/mertnuhoglu/projects/itr/vrp_doc/data_model/"
```

# Implementation of `yumltordbschema::yuml_to_rdb()` function

`yuml_to_rdb()` takes as input `yuml` formatted data model and converts it into normalized structural data in RDB style.

Example input:

    [enum_value| enum_value_id PK; enum_var_id; enum_value_name TEXT; ]
    [enum_var| enum_var_id PK; enum_var_name TEXT; ]
    [enum_var] 1-n [enum_value]

Final output:

    data_entity.tsv
    | data_entity_id | entity_name |
    | 7              | enum_value  |
    | 8              | enum_var    |

    data_field.tsv
    | data_field_id | data_field_name | type   | pk_fk   | data_entity_id | fk_data_entity_id | enum_var_name |
    | 26            | enum_value_id   | BIGINT | PK      | 7              |                   |               |
    | 27            | enum_var_id     | BIGINT | FK      | 7              | 8                 |               |
    | 28            | enum_value_name | TEXT   | NOT_KEY | 7              |                   |               |
    | 29            | enum_var_id     | BIGINT | PK      | 8              |                   |               |
    | 30            | enum_var_name   | TEXT   | NOT_KEY | 8              |                   |               |

``` r
yuml_to_rdb = function(data_model_dir) {
  build_datamodel_sdb.sh = system.file("bash/build_datamodel_sdb.sh", package = "yumltordbschema")
  system2(build_datamodel_sdb.sh, data_model_dir)

	yuml_lines = r_datamodel_sdb(data_model_dir) %>% 
    rutils::grepv("\\|")
	ydm = build_yuml_data_model(yuml_lines)
	den = update_new_entities(ydm)
  dfl = update_new_fields(ydm, den)

  write_yuml_data_model(ydm, data_model_dir = data_model_dir)
  write_data_entity(den, data_model_dir)
  write_data_field(dfl, data_model_dir)
  return(list(data_entity = den, data_field = dfl))
}
``` 

Calling `yuml_to_rdb` requires the parameter `data_model_dir` which is the path of the `yuml` files.

``` r
data_model_dir =  "/Users/mertnuhoglu/projects/itr/vrp_doc/data_model/"
rdt = yuml_to_rdb(data_model_dir)
``` 

## `build_datamodel_sdb.sh` script: Concatenate `yuml` files and fix them

@tbd

## `build_yuml_data_model()` function: Convert free format yuml content into structural data

Input: The unified and fixed yuml file: `datamodel_sdb.yuml`

Output: Structural data: `yuml_data_model.csv`

```r
lines = c("[enum_value| enum_value_id INT PK; enum_var_id INT FK @NN; enum_value_name TEXT; ]",
  "[enum_var| enum_var_id INT PK; enum_var_name TEXT; ]")
lines
  ## [1] "[enum_value| enum_value_id INT PK; enum_var_id INT FK @NN; enum_value_name TEXT; ]"
  ## [2] "[enum_var| enum_var_id INT PK; enum_var_name TEXT; ]"
```

``` r
ydm_p1 = lines %>%
  stringr::str_replace_all( "^[ \\[]*", "" ) %>%
  stringr::str_replace_all( "[ \\]]*$", "" ) %>%
  stringr::str_replace_all( "\\|\\s*", "\\|" ) %>%
  stringr::str_replace_all( ";\\s*", ";" ) 
ydm_p1 
  ## [1] "enum_value|enum_value_id INT PK;enum_var_id INT FK @NN;enum_value_name TEXT;"
  ## [2] "enum_var|enum_var_id INT PK;enum_var_name TEXT;"
```

``` r
ydm_p2 = ydm_p1 %>%
  dplyr::data_frame( ln = . ) 
ydm_p2 
  ## # A tibble: 2 x 1
  ##                                                                             ln
  ##                                                                          <chr>
  ## 1 enum_value|enum_value_id INT PK;enum_var_id INT FK @NN;enum_value_name TEXT;
  ## 2                              enum_var|enum_var_id INT PK;enum_var_name TEXT;
ydm_p3 = ydm_p2 %>%
  tidyr::separate( ln, c("entity_name", "columns"), "\\|" ) %>%
  tibble::glimpse()
ydm_p4 = ydm_p3 %>%
  tidyr::unnest( columns = strsplit( columns, ";") ) 
ydm_p4 
  ## # A tibble: 5 x 2
  ##   entity_name                columns
  ##         <chr>                  <chr>
  ## 1  enum_value   enum_value_id INT PK
  ## 2  enum_value enum_var_id INT FK @NN
  ## 3  enum_value   enum_value_name TEXT
  ## 4    enum_var     enum_var_id INT PK
  ## 5    enum_var     enum_var_name TEXT
ydm_p5 = ydm_p4 %>%
  tidyr::separate( columns, c("data_field_name", "type", "pk_fk"), " " ) %>%
  dplyr::mutate( pk_fk = ifelse( is.na(pk_fk), "NON_KEY", pk_fk)) %>%
  dplyr::mutate( type = ifelse( is.na(type), "TEXT", type)) %>%
  dplyr::mutate( entity_name = tolower(entity_name) ) %>%
  dplyr::mutate( data_field_name = tolower(data_field_name) ) 
ydm_p5 
  ## # A tibble: 5 x 4
  ##   entity_name data_field_name  type   pk_fk
  ##         <chr>           <chr> <chr>   <chr>
  ## 1  enum_value   enum_value_id   INT      PK
  ## 2  enum_value     enum_var_id   INT      FK
  ## 3  enum_value enum_value_name  TEXT NON_KEY
  ## 4    enum_var     enum_var_id   INT      PK
  ## 5    enum_var   enum_var_name  TEXT NON_KEY
```

### Handling new field attribute `@NN` in yuml 

I noted that `ydm_p4` manipulation gives warning: "Warning: Too many values at 2 locations: 2"

Checking why this warning is issued:

``` r
ydm_p4[2, ]
  ## # A tibble: 1 x 2
  ##   entity_name                columns
  ##         <chr>                  <chr>
  ## 1  enum_value enum_var_id INT FK @NN
```

These two fields have an additional attribute as `@NN`, but `tidyr::separate( columns, c("data_field_name", "type", "pk_fk"), " " )` expects only 3 items.

This `@NN` attribute comes from original `yuml` file:

    [enum_value| enum_value_id PK; enum_var_id @NN; enum_value_name TEXT; ]

The following `tidyr::separate` function call will parse additional `@NN` attribute:

``` r
ydm_p6 = ydm_p4 %>%
  tidyr::separate( columns, c("data_field_name", "type", "pk_fk", "not_null"), " " ) %>%
  dplyr::mutate( pk_fk = ifelse( is.na(pk_fk), "NON_KEY", pk_fk)) %>%
  dplyr::mutate( type = ifelse( is.na(type), "TEXT", type)) %>%
  dplyr::mutate( not_null = ifelse( is.na(not_null), FALSE, TRUE)) %>%
  dplyr::mutate( entity_name = tolower(entity_name) ) %>%
  dplyr::mutate( data_field_name = tolower(data_field_name) ) 
ydm_p6
  ## # A tibble: 5 x 5
  ##   entity_name data_field_name  type   pk_fk not_null
  ##         <chr>           <chr> <chr>   <chr>    <lgl>
  ## 1  enum_value   enum_value_id   INT      PK    FALSE
  ## 2  enum_value     enum_var_id   INT      FK     TRUE
  ## 3  enum_value enum_value_name  TEXT NON_KEY    FALSE
  ## 4    enum_var     enum_var_id   INT      PK    FALSE
  ## 5    enum_var   enum_var_name  TEXT NON_KEY    FALSE
``` 

No, need to make any changes in `update_new_entities` and `update_new_fields`

``` r
ydm = ydm_p6
den = yumltordbschema::update_new_entities(ydm)
den
  ## # A tibble: 2 x 2
  ##   data_entity_id entity_name
  ##            <int>       <chr>
  ## 1              1  enum_value
  ## 2              2    enum_var
dfl = yumltordbschema::update_new_fields(ydm, den)
dfl
  ## # A tibble: 5 x 8
  ##   data_field_id data_field_name  type   pk_fk not_null data_entity_id fk_data_entity_id enum_var_name
  ##           <int>           <chr> <chr>   <chr>    <lgl>          <int>             <int>         <chr>
  ## 1             1   enum_value_id   INT      PK    FALSE              1                NA          <NA>
  ## 2             2     enum_var_id   INT      FK     TRUE              1                 2          <NA>
  ## 3             3 enum_value_name  TEXT NON_KEY    FALSE              1                NA          <NA>
  ## 4             4     enum_var_id   INT      PK    FALSE              2                NA          <NA>
  ## 5             5   enum_var_name  TEXT NON_KEY    FALSE              2                NA          <NA>
``` 

