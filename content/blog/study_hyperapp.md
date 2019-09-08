---
title: "Studying Hyperapp.js"
date: 2018-03-16T10:44:32+03:00 
draft: false
description: ""
tags: ["js"]
categories: ["software", "js"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: mertnuhoglu.com
resource_files:
- 
path: ~/projects/study/js/study_hyperapp.md
wip: true
---

hyperapp.js is a minimalistic web app framework.

<!--more-->

<!-- toc -->

## Hyperapp + Parcel Tutorial

Following the tutorial explained in: https://blog.daftcode.pl/hyperapp-parcel-71823bd93f1c

Major points:

- Parcel: zero configuration as opposed to Webpack
- Hyperapp: minimalist frontend framework
- inspired by Elm architecture

## v01: Basic Setup

Source code in https://github.com/mertnuhoglu/study/tree/master/js/ex/study_hyperparcel/

``` bash
mkdir study_hyperparcel && cd $_ && npm init -y && npm i hyperapp parcel-bundler 
``` 

The original tutorial uses jsx. I don't like jsx because of additional mental burden and no benefit over hyperscript as explained by Andre Staltz in https://staltz.com/nothing-new-in-react-and-flux-except-one-thing.html

`index.html` is very basic:

``` html
<html>
  <body>
    <script src="./index.js"></script>
  </body>
</html>
``` 

`index.js` is very basic:

``` js
console.log('hello parcel')
``` 

`package.json`

    ...
    "start": "parcel index.html",
    "build": "parcel build index.html --public-url ./",

Run `npm start`
 
Open http://localhost:1234

The speed of bundling and running server is amazing in comparison to webpack. 

``` bash
❯ npm start

> study_hyperparcel@1.0.0 start /Users/mertnuhoglu/projects/study/js/ex/study_hyperparcel
> parcel index.html

Server running at http://localhost:1234
✨  Built in1.2s.
``` 

## v02: Using Hyperscript in Frontend View

Source code in https://github.com/mertnuhoglu/study/tree/master/js/ex/study_hyperparcel02/

``` bash
mkdir study_hyperparcel02 && cd $_ && npm init -y && npm i hyperapp parcel-bundler 
``` 

`index.html` and `package.json` are the same.

``` bash
cp ../study_hyperparcel/index.html .
cp ../study_hyperparcel/package.json .
``` 

Add hyperscript to `index.js`:

``` js
console.log('hello parcel')
``` 

## v03

@tbd

