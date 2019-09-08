---
title: "Study RamdaJs"
date: 2018-03-19T19:24:58+03:00 
draft: false
description: "ramdajs hints"
summaryLength: 10
tags: ["js", "functional_programming", "ramdajs"]
categories: ["software", "js", "functional_programming"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: mertnuhoglu.com
resource_files:
- ex/study_ramda/package.json
- ex/study_ramda/ex01.js
- ex/study_ramda/ex02.js
- ex/study_ramda/ex03.js
- ex/study_ramda/ab01.js
- ex/study_ramda/ab02.js
- ex/study_ramda/ab02b.js
- ex/study_ramda/ab03.js
- ex/study_ramda/ab04.js
- ex/study_ramda/ab05.js
- ex/study_ramda/ab06.js
path: ~/projects/study/js/study_ramda.Rmd

---

Examples for ramdajs

<!--more-->

<!-- toc -->

Source code in https://github.com/mertnuhoglu/study/tree/master/js/ex/study_ramda/

``` bash
mkdir -p ex/study_ramda && cd $_ && npm init -y && pnpm i parcel-bundler ramda
``` 

### v01: Basics

``` js
const data = [
  {'id': 1, 'title': "a"},
  {'id': 2, 'title': "b"},
];
const getId = R.map(R.prop('id'));
console.log(getId(data));
``` 

``` bash
node ex/study_ramda/ex01.js
## [ 1, 2 ]
``` 

### v02: Basics

``` js
const data = [
  {'sub': {'id': 1, 'title': "a"}},
  {'sub': {'id': 2, 'title': "b"}},
];
const getId = R.map(R.compose(R.prop('id'), R.prop('sub')));
console.log(getId(data));
``` 

``` bash
node ex/study_ramda/ex02.js
## [ 1, 2 ]
``` 

### v03: Basics

I changed the input function of `R.map` to common object access notation:

``` js
const getId = R.map(R.compose(R.prop('id'), R.prop('sub')));
--->>>
const getId = R.map(e => e.sub.id);
``` 

This looks simpler than previous expression. 

``` bash
node ex/study_ramda/ex03.js
## [ 1, 2 ]
``` 

### Tutorial by Andrew Burgess

Here, I follow the tutorial in [Ramda By Example](https://www.youtube.com/watch?v=tN4wyJ9DdtM&t=14s&list=PL_L_J_Lv0U2qJepM0FMF73BHGZsNNu-nk) made by Andrew Burgess and the revised code written by [Francisco Neto](https://www.youtube.com/user/tartaneto)

The goal is to convert the following data structure into the next data structure:

``` js
const data = {
  'group1-perm1': true,
  'group1-perm2': false,
  'group2-perm1': false,
  'group2-perm2': true,
  'perm3': true,
  'perm4': false
};
// --->>>
const target = {
  'group1': [
    { value: 'group1-perm1', checked: true, 'label': 'perm1' }
  ],
// ...
};
``` 

#### v01: Convert key-value pairs into 2-tuples

First, we will convert the input data into an array of 2-tuples like that:

    [ [ 'group1-perm1', true ], ... ]

``` js
R.map(([k, v]) => global[k] = v, R.toPairs(R));
const fn = compose(toPairs);
console.log(fn(data));
``` 
 
``` bash
node ex/study_ramda/ab01.js
## [ [ 'group1-perm1', true ],
##   [ 'group1-perm2', false ],
##   [ 'group2-perm1', false ],
##   [ 'group2-perm2', true ],
##   [ 'perm3', true ],
##   [ 'perm4', false ] ]
``` 

#### v02: append an element

Now, we will convert 2-tuples into this structure:

    [ { value: 'group1-perm1', checked: true, label: 'group1-perm1' }, ...

We can do this in two ways. First way:

``` js
const addLabel = ([value, checked]) => ({value, checked, label: value});
const fn = compose(map(addLabel), toPairs);
console.log(fn(data));
``` 

``` bash
node ex/study_ramda/ab02.js
## [ { value: 'group1-perm1', checked: true, label: 'group1-perm1' },
##   { value: 'group1-perm2', checked: false, label: 'group1-perm2' },
##   { value: 'group2-perm1', checked: false, label: 'group2-perm1' },
##   { value: 'group2-perm2', checked: true, label: 'group2-perm2' },
##   { value: 'perm3', checked: true, label: 'perm3' },
##   { value: 'perm4', checked: false, label: 'perm4' } ]
``` 

The second way is to use `chain` function:

``` js
const addLabel = chain(append, head);
const fn = compose(map(addLabel), toPairs);
console.log(fn(data));
``` 

This code produces the following data structure which needs one more step:

    [ [ 'group1-perm1', true, 'group1-perm1' ], ...

``` bash
node ex/study_ramda/ab02b.js
## [ [ 'group1-perm1', true, 'group1-perm1' ],
##   [ 'group1-perm2', false, 'group1-perm2' ],
##   [ 'group2-perm1', false, 'group2-perm1' ],
##   [ 'group2-perm2', true, 'group2-perm2' ],
##   [ 'perm3', true, 'perm3' ],
##   [ 'perm4', false, 'perm4' ] ]
``` 

The first version is easier to understand. Let's keep that one.

#### v03: Text substitution using regex

Now we want to update `label`'s value as such:

    [ { value: 'group1-perm1', checked: true, label: 'group1-perm1' }, ...
    --->>>
    [ { value: 'group1-perm1', checked: true, label: 'perm1' }, ...

``` js
const getLabel = R.compose(R.head, R.match(/perm[0-9]/g));
const addLabel = ([value, checked]) => ({value, checked, label: getLabel(value)});
``` 

``` bash
node ex/study_ramda/ab03.js
## [ { value: 'group1-perm1', checked: true, label: 'perm1' },
##   { value: 'group1-perm2', checked: false, label: 'perm2' },
##   { value: 'group2-perm1', checked: false, label: 'perm1' },
##   { value: 'group2-perm2', checked: true, label: 'perm2' },
##   { value: 'perm3', checked: true, label: 'perm3' },
##   { value: 'perm4', checked: false, label: 'perm4' } ]
``` 

#### v04: Group items using `groupBy`

    [ { value: 'group1-perm1', checked: true, label: 'perm1' }, ...
    --->>>
    { 'group1-perm1': [ { value: 'group1-perm1', checked: true, label: 'perm1' } ], ...

``` js
const groupName = R.prop('value');
const fn = compose(R.groupBy(groupName), map(addLabel), toPairs);
``` 

``` bash
node ex/study_ramda/ab04.js
## { 'group1-perm1': [ { value: 'group1-perm1', checked: true, label: 'perm1' } ],
##   'group1-perm2':
##    [ { value: 'group1-perm2', checked: false, label: 'perm2' } ],
##   'group2-perm1':
##    [ { value: 'group2-perm1', checked: false, label: 'perm1' } ],
##   'group2-perm2': [ { value: 'group2-perm2', checked: true, label: 'perm2' } ],
##   perm3: [ { value: 'perm3', checked: true, label: 'perm3' } ],
##   perm4: [ { value: 'perm4', checked: false, label: 'perm4' } ] }
``` 

#### v05: Group items 02

    { 'group1-perm1': [ { value: 'group1-perm1', checked: true, label: 'perm1' } ], ...
    --->>>
    { group1:
       [ { value: 'group1-perm1', checked: true, label: 'perm1' },
         { value: 'group1-perm2', checked: false, label: 'perm2' } ],

``` js
const getGroup = R.compose(R.defaultTo('general'), R.head, R.match(/group[0-9]/g));
const groupName = R.compose(getGroup, R.prop('value'));
const fn = compose(R.groupBy(groupName), map(addLabel), toPairs);
``` 

``` bash
node ex/study_ramda/ab05.js
## { group1:
##    [ { value: 'group1-perm1', checked: true, label: 'perm1' },
##      { value: 'group1-perm2', checked: false, label: 'perm2' } ],
##   group2:
##    [ { value: 'group2-perm1', checked: false, label: 'perm1' },
##      { value: 'group2-perm2', checked: true, label: 'perm2' } ],
##   general:
##    [ { value: 'perm3', checked: true, label: 'perm3' },
##      { value: 'perm4', checked: false, label: 'perm4' } ] }
``` 

#### v06: `pipe` instead of `compose`

`R.pipe` and `R.compose` are similar but they execute the functions in reverse order. `R.pipe` executes the input functions from left-to-right. `compose` executes from right-to-left. Thus `pipe` is easier to read.

    const fn = compose(R.groupBy(groupName), map(addLabel), toPairs);
    --->>>>

``` js
const fn = pipe(
  toPairs,
  map(addLabel), 
  R.groupBy(groupName), 
);
``` 

``` bash
node ex/study_ramda/ab06.js
## { group1:
##    [ { value: 'group1-perm1', checked: true, label: 'perm1' },
##      { value: 'group1-perm2', checked: false, label: 'perm2' } ],
##   group2:
##    [ { value: 'group2-perm1', checked: false, label: 'perm1' },
##      { value: 'group2-perm2', checked: true, label: 'perm2' } ],
##   general:
##    [ { value: 'perm3', checked: true, label: 'perm3' },
##      { value: 'perm4', checked: false, label: 'perm4' } ] }
``` 

