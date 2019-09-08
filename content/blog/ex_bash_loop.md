---
title: "Example: Bash Loops"
date: 2017-12-13T18:27:56+03:00 
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

My study notes and examples to remember how to loop in bash.

<!--more-->

<!-- toc -->

## Read csv: loop over rows, then parse columns

``` bash
csv_file=./data/bash_sample_csv.csv
while IFS=, read col1 col2
do 
  echo "$col1 $col2"
done < $csv_file
  ## col1 col2
  ## row11 row12
  ## row21 row22
```

`IFS`: field separator. Default value: `space`

## Loop over file names matching a pattern

``` bash
for filename in *.Rmd; do
  echo $filename 
done
  ## ex_bash_loop.Rmd
```

## Loop over a range of numbers

``` bash
for i in {1..5}; do echo $i; done
```

## Loop using variables for end points

Read: https://stackoverflow.com/questions/6191146/variables-in-bash-seq-replacement-1-10#6191382

Using `seq` command:

``` bash
END=2
for i in $(seq 1 $END); do echo "row: $i"; done
  ## row: 1
  ## row: 2
```

The following doesn't work because brace expansion happens before variable expansion:

``` bash
start=1
end=10
echo {$start..$end}
  ## Ouput: {1..10}
```

Using `eval` to make variable expansion first:

``` bash
END=2
for i in $(eval echo 1 $END); do echo "row: $i"; done
  ## row: 1
  ## row: 2
```

Using arithmetic for loop:

``` bash
start=1
end=2
for ((i=start; i<=end; i++))
do
   echo "row: $i"
done
  ## row: 1
  ## row: 2
```

