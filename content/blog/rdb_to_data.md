---
title: "Convert RDB to SQL Data Insert/Update Statements"
date: 2018-01-20T18:30:12+03:00 
draft: false
description: ""
tags: ["r", "sql", "rdb"]
categories: ["software", "r", "sql", "rdb"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: mertnuhoglu.com
resource_files:
-
---

This blog post is on a personal need. It is in a draft state currently. I hope to write on this later.

<!--more-->

<!-- toc -->

``` r
library(dplyr, warn.conflicts = F)
```

``` r
evr = tibble::tribble(
  ~enum_var_id,     ~enum_var_name,
            1L,  "route_stop_enum",
            2L,      "action_enum",
            3L,       "place_enum"
  )
evl = tibble::tribble(
  ~enum_value_id,          ~enum_value_name,      ~enum_var_id,
           1000L,         "departure depot",                1L,
           1001L,            "return depot",                1L,
           1002L,               "pick stop",                1L,
           1003L,               "drop stop",                1L,
           1004L,          "pick/drop stop",                1L,
           1005L,  "departure/return depot",                1L,
           2000L,               "pick/load",                2L,
           2001L,             "drop/unload",                2L,
           3000L,                   "depot",                3L,
           3001L,                 "factory",                3L,
           3002L,                "customer",                3L
  )
```

Here are the test data:

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

I didn't type this data manually. I reproduced this data using excellent life-saver library `datapasta` by running the following code on terminal once:

``` r
yumltordbschema::read_data_entity() %>% dplyr::as_data_frame() %>% datapasta::dpasta()
yumltordbschema::read_data_field() %>% dplyr::as_data_frame() %>% datapasta::dpasta()
```

`read_data_entity` and `read_data_field` functions import data from the file system. In order to make this `Rmd` file completely reproducible, I don't import data from the file system here.

The main entry function for sql generation is `rdb_to_data`:

``` r
rdb_to_data = function(entity, data_entities, dfl, den) {
  dflf = dfl %>%
    dplyr::left_join(den, by = "data_entity_id") %>%
    dplyr::filter(entity_name == entity) 
  columns = dflf$data_field_name
  id_column = dflf %>%
    dplyr::filter(pk_fk == "PK") %>%
    magrittr::extract2("data_field_name")
	df = data_entities[[entity]] %>%
    dplyr::select_(.dots = columns)
	build_sql(df, entity, id_column, den)
}
```

This will be called by the following lines in actual program:

``` r
data_model_dir = "/Users/mertnuhoglu/projects/itr/vrp_doc/data_model/"
r_entity = function(entity, data_model_dir = env_data_model_dir()) {
  rio::import(sprintf("%s/../rdb/%s.xlsx", data_model_dir, entity))
}
entities = c("enum_var", "enum_value")
data_entities = lapply(entities, r_entity, data_model_dir) %>%
	setNames(entities)
data_entities
lapply( entities, rdb_to_data, data_entities, dfl, den)
```

Note that, these statements access file system. In this Rmd file, I don't want to import data from file system. Instead I will initialize `data_entities` from sample dataframes:

``` r
entities = c("enum_var", "enum_value")
data_entities = list( enum_var = evr, enum_value = evl )
data_entities
  ## $enum_var
  ## # A tibble: 3 x 2
  ##   enum_var_id enum_var_name  
  ##         <int> <chr>          
  ## 1           1 route_stop_enum
  ## 2           2 action_enum    
  ## 3           3 place_enum     
  ## 
  ## $enum_value
  ## # A tibble: 11 x 3
  ##    enum_value_id enum_value_name        enum_var_id
  ##            <int> <chr>                        <int>
  ##  1          1000 departure depot                  1
  ##  2          1001 return depot                     1
  ##  3          1002 pick stop                        1
  ##  4          1003 drop stop                        1
  ##  5          1004 pick/drop stop                   1
  ##  6          1005 departure/return depot           1
  ##  7          2000 pick/load                        2
  ##  8          2001 drop/unload                      2
  ##  9          3000 depot                            3
  ## 10          3001 factory                          3
  ## 11          3002 customer                         3
``` 

Step by step, data flow inside `rdb_to_data()` is as follows:

``` r
entity = "enum_var"
dflf = dfl %>%
  dplyr::left_join(den, by = "data_entity_id") %>%
  dplyr::filter(entity_name == entity) 
columns = dflf$data_field_name
df = data_entities[[entity]] %>%
	dplyr::select_(.dots = columns)
df
  ## # A tibble: 3 x 2
  ##   enum_var_id enum_var_name  
  ##         <int> <chr>          
  ## 1           1 route_stop_enum
  ## 2           2 action_enum    
  ## 3           3 place_enum
```

So, we get meta data from `dfl` which stores all data fields for each entity (table). From that, we obtain the names of the columns (fields) which we will use while generating SQL commands.

Now, let's implement `build_sql()`

``` r
insert_sql_template = function( entity, columns ) {
	template = "INSERT INTO %s (%s) VALUES (%s);"
	column_names = columns %>% paste(collapse=",")
	data_placeholders = rep( "'%s'", length(columns ) ) %>% paste(collapse=",") 
	result = sprintf( template, entity, column_names, data_placeholders )
	return(result)
}

insert_sql = function( data, template ) {
	arg = c( list(template), as.list(data) )
	do.call( 'sprintf', arg ) %>%
		stringr::str_replace_all( "'NA'", "null" )
}

sql_insert = function(df, entity) {
	template_df = dplyr::data_frame( entity = entity, insert_template = insert_sql_template(entity, names(df) ))
	insert_sql( df, template_df$insert_template )
}

sql = sql_insert( df, entity )
sql
  ## [1] "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('1','route_stop_enum');" "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('2','action_enum');"    
  ## [3] "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('3','place_enum');"
```

`df` contains the data for which we generate `SQL INSERT` commands.

`sql_insert()` function converts data contained in a `data_frame` into `SQL INSERT` statements.

``` r
build_sql = function(df, entity) {
	sql = sql_insert( df, entity )
  return(list(
              sql_insert = sql
              ))
}
``` 

I put `sql_insert` inside `build_sql` function because later I will also create `UPDATE` and other commands in addition to `INSERT` inside `build_sql()`.

So, let's update definition of `rdb_to_data()` with this `build_sql()` and run the program again:

``` r
rdb_to_data = function(entity, data_entities, dfl, den) {
  dflf = dfl %>%
    dplyr::left_join(den, by = "data_entity_id") %>%
    dplyr::filter(entity_name == entity) 
  columns = dflf$data_field_name
	df = data_entities[[entity]] %>%
    dplyr::select_(.dots = columns)
	build_sql(df, entity)
}
lapply( entities, rdb_to_data, data_entities, dfl, den)
  ## [[1]]
  ## [[1]]$sql_insert
  ## [1] "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('1','route_stop_enum');" "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('2','action_enum');"    
  ## [3] "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('3','place_enum');"     
  ## 
  ## 
  ## [[2]]
  ## [[2]]$sql_insert
  ##  [1] "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1000','1','departure depot');"        "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1001','1','return depot');"          
  ##  [3] "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1002','1','pick stop');"              "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1003','1','drop stop');"             
  ##  [5] "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1004','1','pick/drop stop');"         "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1005','1','departure/return depot');"
  ##  [7] "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('2000','2','pick/load');"              "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('2001','2','drop/unload');"           
  ##  [9] "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('3000','3','depot');"                  "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('3001','3','factory');"               
  ## [11] "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('3002','3','customer');"
```

Now, we can define functions that will generate `SQL UPDATE` and `SQL DELETE` commands. The functions are very similar to `SQL INSERT` generating functions.

Let's add `SQL UPDATE` and `SQL DELETE` statements into `build_sql()` function:

``` r
delete_sql = function( df, template ) {
	arg = c( list(template), as.list(df) )
	do.call( 'sprintf', arg ) 
}

delete_sql_template = function( entity, id_column) {
	template = "DELETE FROM %s WHERE %s = %%s;"
	sprintf( template, entity, id_column)
}

sql_delete = function(df, entity) {
	template = dplyr::data_frame( entity= entity, delete_template = delete_sql_template(entity, names(df) ))
	delete_sql( df, template$delete_template )
}

update_sql = function( data, template ) {
	arg = c( list(template), as.list(data) )
	do.call( 'sprintf', arg ) %>%
		stringr::str_replace_all( "'\\bNA\\b'", "null" ) 
}

update_sql_template = function( entity, columns ) {
	template = "    UPDATE %s SET %s WHERE %s = %%s;"
	columns_to_set = head(columns, -1)
	set_column_clause = sprintf("%s = '%%s'", columns_to_set) %>% 
		paste(collapse=", ")
  id_column = tail(columns, 1)
	sprintf( template, entity, set_column_clause, id_column )
}

sql_update = function(df, entity) {
	template = dplyr::data_frame( entity= entity, update_template = update_sql_template(entity, names(df) ))
	update_sql( df, template$update_template )
}
```

Now, let's update `build_sql` with `sql_update`, `sql_delete`:

``` r
build_sql = function(df, entity, id_column, den) {
  df_id_at_end = df %>%
		dplyr::select( -dplyr::one_of(id_column), dplyr::everything(), id_column )
  df_no_fk = dplyr::select( df, -dplyr::ends_with("_id"), id_column) %>%
    dplyr::select( id_column, dplyr::everything() )
  if ( any("invalid" %in% names(df)) ) {
    df_invalid = df %>%
      dplyr::filter( invalid == 1 ) %>%
      dplyr::select(id_column)
  } else {
    df_invalid = df %>%
      dplyr::select(id_column)
  }
  list(
       sql_insert = sql_insert( df, entity )
       , sql_update = sql_update( df_id_at_end, entity )
       , sql_insert_no_fk = sql_insert( df_no_fk, entity )
       , sql_delete = sql_delete( df_invalid, entity )
       )
} 

rdb_to_data = function(entity, data_entities, dfl, den) {
  dflf = dfl %>%
    dplyr::left_join(den, by = "data_entity_id") %>%
    dplyr::filter(entity_name == entity) 
  columns = dflf$data_field_name
  id_column = dflf %>%
    dplyr::filter(pk_fk == "PK") %>%
    magrittr::extract2("data_field_name")
	df = data_entities[[entity]] %>%
    dplyr::select_(.dots = columns)
	build_sql(df, entity, id_column, den)
}
ent_m_cmdtype_m_cmds = lapply( entities, rdb_to_data, data_entities, dfl, den) %>%
  setNames(entities)
str(ent_m_cmdtype_m_cmds)
  ## List of 2
  ##  $ enum_var  :List of 4
  ##   ..$ sql_insert      : chr [1:3] "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('1','route_stop_enum');" "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('2','action_enum');" "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('3','place_enum');"
  ##   ..$ sql_update      : chr [1:3] "    UPDATE enum_var SET enum_var_name = 'route_stop_enum' WHERE enum_var_id = 1;" "    UPDATE enum_var SET enum_var_name = 'action_enum' WHERE enum_var_id = 2;" "    UPDATE enum_var SET enum_var_name = 'place_enum' WHERE enum_var_id = 3;"
  ##   ..$ sql_insert_no_fk: chr [1:3] "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('1','route_stop_enum');" "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('2','action_enum');" "INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('3','place_enum');"
  ##   ..$ sql_delete      : chr [1:3] "DELETE FROM enum_var WHERE enum_var_id = 1;" "DELETE FROM enum_var WHERE enum_var_id = 2;" "DELETE FROM enum_var WHERE enum_var_id = 3;"
  ##  $ enum_value:List of 4
  ##   ..$ sql_insert      : chr [1:11] "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1000','1','departure depot');" "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1001','1','return depot');" "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1002','1','pick stop');" "INSERT INTO enum_value (enum_value_id,enum_var_id,enum_value_name) VALUES ('1003','1','drop stop');" ...
  ##   ..$ sql_update      : chr [1:11] "    UPDATE enum_value SET enum_var_id = '1', enum_value_name = 'departure depot' WHERE enum_value_id = 1000;" "    UPDATE enum_value SET enum_var_id = '1', enum_value_name = 'return depot' WHERE enum_value_id = 1001;" "    UPDATE enum_value SET enum_var_id = '1', enum_value_name = 'pick stop' WHERE enum_value_id = 1002;" "    UPDATE enum_value SET enum_var_id = '1', enum_value_name = 'drop stop' WHERE enum_value_id = 1003;" ...
  ##   ..$ sql_insert_no_fk: chr [1:11] "INSERT INTO enum_value (enum_value_id,enum_value_name) VALUES ('1000','departure depot');" "INSERT INTO enum_value (enum_value_id,enum_value_name) VALUES ('1001','return depot');" "INSERT INTO enum_value (enum_value_id,enum_value_name) VALUES ('1002','pick stop');" "INSERT INTO enum_value (enum_value_id,enum_value_name) VALUES ('1003','drop stop');" ...
  ##   ..$ sql_delete      : chr [1:11] "DELETE FROM enum_value WHERE enum_value_id = 1000;" "DELETE FROM enum_value WHERE enum_value_id = 1001;" "DELETE FROM enum_value WHERE enum_value_id = 1002;" "DELETE FROM enum_value WHERE enum_value_id = 1003;" ...
```

## The Essence of SQL Generation

The essence of SQL commands generation are done by the following functions:

``` r
insert_sql_template = function( entity, columns ) {
	template = "INSERT INTO %s (%s) VALUES (%s);"
	column_names = columns %>% paste(collapse=",")
	data_placeholders = rep( "'%s'", length(columns ) ) %>% paste(collapse=",") 
	result = sprintf( template, entity, column_names, data_placeholders )
	return(result)
}

insert_sql = function( data, template ) {
	arg = c( list(template), as.list(data) )
	do.call( 'sprintf', arg ) %>%
		stringr::str_replace_all( "'NA'", "null" )
}

sql_insert = function(df, entity) {
	template_df = dplyr::data_frame( entity = entity, insert_template = insert_sql_template(entity, names(df) ))
	insert_sql( df, template_df$insert_template )
}

sql = sql_insert( df, entity )
```

`insert_sql_template()` function returns a template string:

``` r
template_df = dplyr::data_frame( entity = entity, insert_template = insert_sql_template(entity, names(df) ))
template_df
  ## # A tibble: 1 x 2
  ##   entity   insert_template                                                     
  ##   <chr>    <chr>                                                               
  ## 1 enum_var INSERT INTO enum_var (enum_var_id,enum_var_name) VALUES ('%s','%s');
```

This template string is filled with the data in `df`. 

``` r
df
  ## # A tibble: 3 x 2
  ##   enum_var_id enum_var_name  
  ##         <int> <chr>          
  ## 1           1 route_stop_enum
  ## 2           2 action_enum    
  ## 3           3 place_enum
```

