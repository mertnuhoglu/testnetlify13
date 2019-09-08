---
title: "Example: fzf fuzzy file finder"
date: 2017-12-30T18:26:59+03:00 
draft: false
description: ""
tags: ["vimscript", "example"]
categories: ["software", "vimscript"]
type: post
url:
author: "Mert Nuhoglu"
output: rmarkdown::html_document
blog: mertnuhoglu.com
resource_files:
-
---

Most of the examples are taken from fzf wiki: https://github.com/junegunn/fzf/wiki/Examples-(vim)

<!--more-->

<!-- toc -->

## Ex: Run `fzf` with shell find and send output to Vim's `e` command

``` vim
call fzf#run({'sink': 'e'})
``` 

`fzf#run()` function takes a dictionary as argument. This dictionary has two important keys: `sink` and `source`. `sink` is the channel where `fzf` output is redirected. `source` is the data stream where `fzf` makes search.

For example, the above function call uses the default `source` which is the default find command. The `sink` is `e` command of Vim. Thus it will open the selected file with `e`. 

Instead of `e` we could also use `tabedit`

``` vim
call fzf#run({'sink': 'tabedit'})
``` 

## Ex: Use other shell comands in input data stream

Or instead of using the default find command as input data stream, we can use any shell command that produces some data stream. For example:

``` vim
call fzf#run({'source': 'git ls-files', 'sink': 'e'})
``` 

## Ex: Use a Vim array as input data stream

`source` can be a vim array too. The following uses names of open buffers as source:

``` vim
call fzf#run({'source': map(split(globpath(&rtp, 'colors/*.vim'))
            \               , 'fnamemodify(v:val, ":t:r")')
            \ , 'sink': 'e'})
``` 

## Ex: List all open buffers:

``` vim
command! Buffers call fzf#run(fzf#wrap( {'source': map(range(1, bufnr('$')), 'bufname(v:val)')}))
``` 

## Ex: Open `Fzf` buffer as normal Vim buffer

`fzf#wrap` opens fzf using `g:fzf_layout` setting

``` vim
call fzf#run( fzf#wrap({'sink': 'e'}) )
``` 

## Ex: `locate` or `mdfind` command to open any file

`mdfind` command is used in osx to search contents of all documents. 

``` vim
command! -nargs=1 -bang Mdfind call fzf#run(fzf#wrap( {'source': 'mdfind <q-args>', 'options': '-m'}, <bang>0))
command! -nargs=1 -bang Mdfind call fzf#run(fzf#wrap(
      \ {'source': 'mdfind <q-args>', 'options': '-m'}, <bang>0))
``` 

## Ex: FZF

``` vim
" Look for files under current directory
:FZF

" Look for files under your home directory
:FZF ~
``` 

## Keybindings: Enter CTRL-T CTRL-X CTRL-V

		Enter: open in current window
		CTRL-T: in new tab
		CTRL-X: in horizontal split
		CTRL-V: in vertical split
