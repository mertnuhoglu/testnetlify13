---
title: "Example: jq"
date: 2017-12-12T18:27:56+03:00 
draft: false
description: ""
tags: ["examples", "bash", "jq", "json"]
categories: ["software", "bash"]
type: post
url:
author: "Mert Nuhoglu"
output: rmarkdown::html_document
blog: mertnuhoglu.com
resource_files:
- ./data/ex_jq_2.json
- ./data/ex_jq_2.yaml
- ./data/ex_jq_github.json
---

jq is a very powerful tool for json parsing. Here are a few examples on how to use it.

<!--more-->

<!-- toc -->

# Tutorial: String output

I simplified examples from here: https://medium.com/@frontman/how-to-parse-yaml-string-via-command-line-374567512303

``` bash
cat ./data/ex_jq_2.json

  ## {
  ##    "field": "a",
  ##    "object": {
  ##       "field": "b"
  ##    },
  ##    "array": [
  ##      {
  ##         "f1": "c",
  ##         "f2": "d"
  ##      }
  ##    ]
  ## }
```

## String output

Get by key: `.key`

``` bash
cat ./data/ex_jq_2.json | jq '.field'
  ## "a"
```

Get nested element by key: `.array[0].key`

``` bash
cat ./data/ex_jq_2.json | jq '.array[0].f1'
  ## "c"
```

## Json output

Get multiple fields: `{field1, field2}`

``` bash
cat ./data/ex_jq_2.json | jq '.object | {field}'

  ## {
  ##   "field": "b"
  ## }
```

# Using jq for yaml files

yq is a jq wrapper. Download: https://github.com/kislyuk/yq

``` bash
cat ./data/ex_jq_2.yaml

  ## ---
  ## field: a
  ## object:
  ##   field: b
  ## array:
  ## - f1: c
  ## - f2: d
```

## String output

Get by key: `.key`

``` bash
cat ./data/ex_jq_2.yaml | yq '.field'
  ## "a"
```

Get nested element by key: `.array[0].key`

``` bash
cat ./data/ex_jq_2.yaml | yq '.array[0].f1'
  ## "c"
```

## yaml output

Get multiple fields: `{field1, field2}`

``` bash
cat ./data/ex_jq_2.yaml | yq '.object | {field}'

  ## {
  ##   "field": "b"
  ## }
```

``` bash
cat ./data/ex_jq_2.yaml | yq -y '.object | {field}'

  ## field: b
```

# Official Tutorial jq

I simplified examples from here: https://stedolan.github.io/jq/tutorial/

``` bash
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=2' > ./data/ex_jq_github.json
head -n 6 ./data/ex_jq_github.json
```

``` bash
cat ./data/ex_jq_github.json | jq '.' | head -n 5
    ## [
  ##   {
  ##     "sha": "61edf3fa93f6177ef099b1b0cb2b49813a35c546",
  ##     "commit": {
  ##       "author": {
```

Get first item: `.[0]`

``` bash
cat ./data/ex_jq_github.json | jq '.[0]' | head -n 5
  ## {
  ##   "sha": "61edf3fa93f6177ef099b1b0cb2b49813a35c546",
  ##   "commit": {
  ##     "author": {
  ##       "name": "Larry Aasen",
```

Get first item: `.[0]`
Then get `.commit.message`

``` bash
cat ./data/ex_jq_github.json | jq '.[0] | {message: .commit.message}' | head -n 5
  ## {
  ##   "message": "Updated the compile-ios.sh script to fix issues with local oniguruma path."
  ## }
```

Get all items: `.[]`
Then get `.commit.message` from each

``` bash
cat ./data/ex_jq_github.json | jq '.[] | {message: .commit.message}' | head -n 5

  ## {
  ##   "message": "Updated the compile-ios.sh script to fix issues with local oniguruma path."
  ## }
  ## {
  ##   "message": "Update AUTHORS"
```

Wrap output as a single array, by wrapping the filter in brackets: `[<filter>]`

``` bash
cat ./data/ex_jq_github.json | jq '[.[] | {message: .commit.message}]' | head -n 5

  ## [
  ##   {
  ##     "message": "Updated the compile-ios.sh script to fix issues with local oniguruma path."
  ##   },
  ##   {
```

## Navigating relative parents

``` bash
cat ./data/ex_jq_github.json | jq '[.[] | {parents: [.parents[].html_url]}]' | head -n 5

  ## [
  ##   {
  ##     "parents": [
  ##       "https://github.com/stedolan/jq/commit/8eff744eecb9ab2f43e75c70030bc9985bac18b2"
  ##     ]
```

