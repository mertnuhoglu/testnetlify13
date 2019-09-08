---
title: "Example: Bash Arrays"
date: 2017-12-14T18:27:56+03:00 
draft: false
description: ""
tags: ["examples", "bash"]
categories: ["software", "bash"]
type: post
url:
author: "Mert Nuhoglu"
output: rmarkdown::html_document
blog: mertnuhoglu.com
resource_files:
- ./data/bash_sample_csv.csv
---

My study notes and examples to remember how to use bash arrays.

<!--more-->

<!-- toc -->

ref:

- http://www.tldp.org/LDP/abs/html/arrays.html
- http://www.thegeekstuff.com/2010/06/bash-array-tutorial
- http://www.yourownlinux.com/2016/12/bash-scripting-arrays-examples.html

## Declare Array and Assigning Values

``` bash
arr[index]=value
```

``` bash
arr[0]='a'
arr[1]='b'
echo ${arr[1]}
  ## b
```

## Initializing during declaration

Two options:

``` bash
declare -a arr=(el1 el2)
arr=(el1 el2)
```

``` bash
arr=('a' 'b')
echo ${arr[1]}
  ## b
```

## Print all elements 

``` bash
echo ${arr[@]}
  ## a b
```

## Read a File into an Array 

``` bash
arr=(`cat ./data/bash_sample_csv.csv`)
echo ${arr[@]}
  ## col1,col2 row11,row12 row21,row22
```

## Loop over array

``` bash
arr=('a' 'b')
for el in "${arr[@]}"; do 
  echo $el
done
  ## a
  ## b
```

## Loop over a space separated string

``` bash
str="a b"
for el in $str; do 
  echo $el
done
  ## a 
  ## b
```

