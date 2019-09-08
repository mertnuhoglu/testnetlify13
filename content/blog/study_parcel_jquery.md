---
title: "Studying Parcel + Jquery"
date: 2018-03-16T12:55:20+03:00 
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
path: ~/projects/study/js/study_parcel_jquery.Rmd
wip: true
---

How to package jquery using parcel?

<!--more-->

<!-- toc -->

## v01: require('jquery')

Resource:

https://github.com/potato4d/parcel-examples/tree/master/parcel-jquery

Source code in https://github.com/mertnuhoglu/study/tree/master/js/ex/study_parcel_jquery/

``` bash
mkdir study_parcel_jquery && cd $_ && npm init -y && npm i parcel-bundler jquery jquery-ui-dist
``` 

I basically copied the code from `potato4d/parcel-examples` repo:

`index.html` is very basic:

``` html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="utf-8">
  <title>Parcel for jQuery</title>
</head>
<body>
  <div id="root"></div>
  <script src="./src/index.js"></script>
</body>
</html>
``` 

`index.js` is very basic:

``` js
window.$ = require('jquery')
$(document).ready(function () {
  $('#root').html('')
  $('#root').append(`
    <h1>Parcel for jQuery</h1>
    <button>Click me!</button>
  `)
  $(document).on('click', 'button', function () {
    alert('Hello, Parcel')
  })
})
``` 

`package.json`

    ...
    "start": "parcel index.html",
    "build": "parcel build index.html --public-url ./",

Run `npm start`
 
Open http://localhost:1234

## v02: import jquery using es6 syntax

Source code in https://github.com/mertnuhoglu/study/tree/master/js/ex/study_parcel_jquery02/

Following the instructions given in https://stackoverflow.com/questions/47968529/how-do-i-use-jquery-and-jquery-ui-with-parcel-bundler to import jquery using es6 syntax

``` bash
mkdir study_parcel_jquery02 && cd $_ && npm init -y && npm i parcel-bundler jquery jquery-ui-dist
cp ../study_parcel_jquery/index.html .
mkdir -p src
cp ../study_parcel_jquery/src/index.js src/
``` 

Differences in `index.js` 

    window.$ = require('jquery')
    --->>>
    import "./import-jquery";

So we created a separate file `import-jquery.js` to import `jquery`:

    import jquery from "jquery";
    export default (window.$ = window.jQuery = jquery);

Run `npm start`
 
Open http://localhost:1234

## v03: Use jquery-ui

Source code in https://github.com/mertnuhoglu/study/tree/master/js/ex/study_parcel_jquery03/

Following the instructions given in https://stackoverflow.com/questions/47968529/how-do-i-use-jquery-and-jquery-ui-with-parcel-bundler to import jquery using es6 syntax

``` bash
mkdir study_parcel_jquery03 && cd $_ && npm init -y && npm i parcel-bundler jquery jquery-ui-dist
cp ../study_parcel_jquery02/index.html . && mkdir -p src && cp ../study_parcel_jquery02/src/index.js src/
``` 

`src/index.js`

``` js
const jquery = require("jquery")
window.$ = window.jQuery = jquery;
require("jquery-ui-dist/jquery-ui.css")
require("jquery-ui-dist/jquery-ui.js")
$(document).ready(function () {
  $('#root').html('')
  $('#root').append(`
    Date: <input type="text" id="datepicker">
  `)
  $(function() {
    $("#datepicker").datepicker();
  });
})
``` 

Run `npm start`
 
Open http://localhost:1234

First, the resulting web page had some style issues. Chrome's default translator somehow interfered with the resulting web page and broke the style code. I disabled it to correct the issues.

## v03b: jquery-ui as implicit dependencies

Source code in https://github.com/mertnuhoglu/study/tree/master/js/ex/study_parcel_jquery03/index_jquery_ui_01.html

``` html
  <body>
    <input id="datepicker">
    <script src="https://unpkg.com/jquery@3.1.1/dist/jquery.js"></script>
    <script src="https://unpkg.com/jquery-ui-dist@1.12.1/jquery-ui.js"></script>
    <script>
      $( "#datepicker" ).datepicker();
    </script>
  </body>
``` 

Result in:

https://rawgit.com/mertnuhoglu/study/master/js/ex/study_parcel_jquery03/index_jquery_ui_01.html

## v04: ES Modules Mixed with Implicit Dependencies

Source code in https://github.com/mertnuhoglu/study/tree/master/js/ex/study_parcel_jquery04/

``` bash
mkdir study_parcel_jquery04 && cd $_ && npm init -y && npm i parcel-bundler jquery jquery-ui-dist
cp ../study_parcel_jquery03/index.html . && mkdir -p src && cp ../study_parcel_jquery03/src/index.js src/
``` 

`index.html` sources `jquery` libraries globally:

``` html
    <div id="root"></div>
    <script src="https://unpkg.com/jquery@3.1.1/dist/jquery.js"></script>
    <script src="https://unpkg.com/jquery-ui-dist@1.12.1/jquery-ui.js"></script>
    <script src="./src/index.js"></script>
``` 

`index.js` uses `jquery` as implicit dependencies and `app.js` that is imported as ES6 module:

``` js
import {addText} from './app.js'
$('#root').append(`
  Date: <input type="text" id="datepicker">
`)
$("#datepicker").datepicker();
addText("Look up!")
``` 

`app.js` exports a function:

``` js
export function addText(text) {
  $('#root').append(`<p>${text}</p>`)
}
``` 

Run `npm start`
 
Open http://localhost:1234

## v05: ES Modules mixed with CommonJs require()

Source code in https://github.com/mertnuhoglu/study/tree/master/js/ex/study_parcel_jquery05/

``` bash
mkdir study_parcel_jquery05 && cd $_ && npm init -y && npm i parcel-bundler jquery jquery-ui-dist
cp ../study_parcel_jquery04/index.html . && mkdir -p src && cp ../study_parcel_jquery04/src/index.js src/
``` 

`index.js` sources `jquery` with CommonJs `require()` and `app.js` with ES6 module syntax:

``` js
const jquery = require("jquery")
window.$ = window.jQuery = jquery;
require("jquery-ui-dist/jquery-ui.css")
require("jquery-ui-dist/jquery-ui.js")
import {addText} from './app.js'
``` 

Run `npm start`
 
Open http://localhost:1234

