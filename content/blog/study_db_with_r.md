---
title: "Study: Database with R"
date: 2017-12-15T18:27:56+03:00 
draft: false
description: ""
tags:
categories: ["database", "r"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: datascience.mertnuhoglu.com
resource_files:

-
path: ~/projects/study/ds/study_db_with_r.md

---

Various examples on using databases in R

<!--more-->

<!-- toc -->

```r
library(dbplyr, warn.conflicts = F)
library(dplyr, warn.conflicts = F)
```

You can check official documentation vignettes for dbplyr:

``` r
vignette("dbplyr")
browseVignettes("dbplyr")
```

``` r
con <- DBI::dbConnect(RSQLite::SQLite(), path = ":memory:")
```

``` r
con <- DBI::dbConnect(RPostgreSQL::PostgreSQL()
  , user = Sys.getenv("SUPER_USER")
  , password = Sys.getenv("SUPER_USER_PASSWORD")
  , dbname = "app"
  , host = "localhost"
  , port = "5432"
)
```

## How to Run SQL Expressions

PostgreSQL expects schema name in all SQL expressions. If it is not given, PostgreSQL cannot find the relations (tables).

There are at least three alternatives to pass schema name to PostgreSQL:

### Option 1: dbplyr::in_schema(schema, table)

``` r
tbl(con, dbplyr::in_schema("recommendation_engine", "product"))
```

Note that the following expressions won't work because PostgreSQL expects the schema name as search path.

``` r
tbl(con, "product")
tbl(con, "recommendation_engine.product")
```

### Option 2: options = "-c search_path=schema"

``` r
con2 <- DBI::dbConnect(RPostgreSQL::PostgreSQL()
  , user = Sys.getenv("SUPER_USER")
  , password = Sys.getenv("SUPER_USER_PASSWORD")
  , dbname = "app"
  , host = "localhost"
  , port = "5432"
  , options = "-c search_path=recommendation_engine"
)
tbl(con2, "product")
```

### Option 3: SELECT * FROM schema.table

``` r
sql("SELECT * FROM recommendation_engine.product") %>%
  tbl(src = con)
```

## Example Using JDBC and Returning SELECT Result as Dataframe

``` r
get_db_emdk = function() {
	library("RJDBC")
	username = "btg_mis"
	password = Sys.getenv("LERIS_ORACLE_BTG_MIS_PASSWORD")
	conStr =  "jdbc:oracle:thin:@31.170.236.10:1521:tekuistest"
	drv <- JDBC("oracle.jdbc.driver.OracleDriver",
		"other/ojdbc6.jar",
		identifier.quote="`")
	conn = dbConnect(drv, conStr, username, password)
	return(conn)
}
sql_fk_listing = function(conn) {
	sql = "SELECT a.table_name, a.column_name, a.constraint_name, c.owner, 
				 c.r_owner, c_pk.table_name r_table_name, c_pk.constraint_name r_pk
						FROM all_cons_columns a
						JOIN all_constraints c ON a.owner = c.owner
																	AND a.constraint_name = c.constraint_name
						JOIN all_constraints c_pk ON c.r_owner = c_pk.owner
																		 AND c.r_constraint_name = c_pk.constraint_name
						WHERE c.constraint_type = 'R'
						AND c.r_owner = 'BTG_MIS'"
	df = dbGetQuery(conn, sql)
	return(df)
}
conn = get_db_emdk()
df = sql_fk_listing(conn)
```


