---
title: "Example: VimL Vimscript File Expansion"
date: 2018-01-20T18:25:31+03:00 
draft: false
description: ""
tags: ["vimscript", "examples"]
categories: ["software", "vimscript"]
type: post
url:
author: "Mert Nuhoglu"
output: rmarkdown::html_document
blog: mertnuhoglu.com
resource_files:
-

---

How to print current path and similar stuff in vimscript?

<!--more-->

<!-- toc -->

## File expansions

Get help from Vim about filename and path expansion modifiers:

``` vim
:help filename-modifiers
``` 

Path of file. `%` expands into current file path.

``` vim
echo expand("%")
  ## /Users/mertnuhoglu/projects/study/vim/ex_viml_file_expansion.Rmd
``` 

Path of directory. `h`: head. `p`: path. `:p:h` head of path of file

``` vim
echo expand("%:p:h")
  ## /Users/mertnuhoglu/projects/study/vim
``` 

Path of file. `p`: path of file

``` vim
echo expand("%:p")
  ## /Users/mertnuhoglu/projects/study/vim/ex_viml_file_expansion.Rmd
``` 

Path of file without extension. `r`: root of file name

``` vim
echo expand("%:r")
  ## /Users/mertnuhoglu/projects/study/vim/ex_viml_file_expansion
``` 


## Run an R command to render rmarkdown files from within Vim

In bash, the following command renders an `.Rmd` file to `.html` file

``` bash
R -e 'rmarkdown::render("%")'
``` 

To run this command from within Vim easily, I defined a custom command in `.vimrc` file:

``` vim
command! Rmarkdown :!R -e 'rmarkdown::render("%")'
``` 

Now simply typing the following auto-completable command inside Vim is enough:

``` vim
:Rmarkdown
``` 

## Open the generated html file after rendering rmarkdown

`:Rmarkdown` command above renders `.Rmd` file into `.html` file. Now, I want to open it from within Vim easily. 

The following custom command let's me open generated `html` file easily:

``` vim
command! OpenHtml :execute '!open ' . expand("%:r") . '.html'
command! Ohtml OpenHtml 
``` 

