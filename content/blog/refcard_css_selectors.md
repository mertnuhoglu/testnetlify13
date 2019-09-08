﻿---
title: "Curated Refcard for CSS Selectors and Elements"
date: 2018-02-19T10:23:13+03:00 
draft: false
description: ""
tags: ["css"]
categories: ["software", "css"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: mertnuhoglu.com
resource_files:
-

---

This blog post is curated from different sources such as:

- https://developer.mozilla.org/en-US/docs/Web/CSS
- https://adam-marsden.co.uk/css-cheat-sheet

But mostly, I summarized MDN documentation according to my own needs.

<!--more-->

<!-- toc -->

## Basic Selectors

Universal Selectors:

    /* * Selects all elements of any type */
    * { color: green; }

`*.warning` and `.warning` are equivalent

Id Selectors:

    #id {}

equivalent to `attribute selector`: `[id=id_value]`

Class Selectors

    .class_name {}
    li.class_name {}
    /* both class1 and class2 required: */
    li.class1.class2 {}

equivalent to `attribute selector`: `[class~=class_name]`

Type Selectors:

    a {}
    span {}
    /* apply to both h1 and h2 */
    h1, h2 {}

Adjacent sibling combinator `+`

    /* p follows img immediately */
    img + p {}
    former_element + target_element {}

Child combinator `>`

    /* target is direct children of ul */
    ul > target {}

General sibling selectors `~`

    /* span follows p and both are children of the same parent */
    p ~ span {}

    <span>not selected.</span>
    <p>anything.</p>
    <code>....</code>
    <span>SELECTED!</span>
    
Descendant combinator ` ` (`space`) or `>>`

    /* li is descendant of ul */
    ul li {}

    <ul>...</ul>
      <div>...
        <li>SELECTED</li>
      </div>
      <li>SELECTED</li>

Attribute selector

    /* <a> elements with a title attribute */
    a[title] { }

    /* <a> elements with an href matching "https://example.org" */
    a[href="https://example.org"] { }

    /* <a> elements with an href containing "example" */
    a[href*="example"] { }

    /* <a> elements with an href ending ".org" */
    a[href$=".org"] { }

    /* attr contains a list of words one of which matches value */
    a[attr~="value"] { }

    /* case sensitive: i I */
    a[href*="insensitive" i] {}

Examples

    | description                     | example                 |
    | select all elements of any type | *{}                     |
    | select some_id                  | #some_id{}              |
    | select class1 and class2 elems  | .class1.class2 {}       |
    | select h1 h3 types              | h1, h3 {}               |
    | p follows img directly          | img + p {}              |
    | li direct children of ul        | ul > li {}              |
    | span follows p                  | p ~ span {}             |
    | li descendant of ul             | ul li {}                |
    | <a> with title attribute        | a[title] {}             |
    | <a> href matches g.com          | a[href="https://g.com"] |
    | <a> href contains g             | a[href*="g"]            |
    | <a> href ends with ".org"       | a[href$=".org"]         |
    | <a> attr contains "val" word    | a[attr~="val"]          |

## Combinators

## Pseudo selectors

    | description                               | selector        |
    | mouse over selector                       | a:hover         |
    | active link selector                      | a:active        |
    | focus selector                            | input:focus     |
    | visited links selector                    | a:visited       |
    | link (not yet visited) selector           | a:link          |
    | checked elements (toggled to an on state) | input:checked   |
    | disabled elements                         | input:disabled  |
    | enabled elements                          | input:enabled   |
    | not a specified element                   | :not(p)         |
    | first line                                | p::first-line   |
    | first letter                              | p::first-letter |
    | first child                               | p:first-child   |
    | last child                                | p:last-child    |
    | nth child (every 4th child)               | p:nth-child(4n) |
    | first element of its type among siblings  | p:first-of-type |
    | element with no children                  | p:empty         |

    /* odd rows of a table */
    tr:nth-child(odd)
    /* seventh element */
    :nth-child(7)

## Font Styling

    | description  | css             | values                             |
    | font style   | font-style:     | normal italic oblique              |
    | font variant | font-variant    | normal small-caps                  |
    | boldness     | font-weight     | normal bold bolder lighter 100-900 |
    | vertical     | vertical-align  | baseline 10px top ...              |
    | transform    | text-transform  | capitalise lowercase uppercase     |
    | size         | font-size       | 12px 0.8em 80%                     |
    | space        | letter-spacing  | normal 4px                         |
    | height       | line-height     | normal 3em 34%                     |
    | horizontal   | text-align      | left right center justify          |
    | decoration   | text-decoration | none underline overline ...        |
    | indent       | text-indent     | 25px                               |
    | font         | font-family     | 'Open Sans', sans-serif            |

## Position

Use CSS Grid instead

## Background

    | description | css               | values       |
    | image       | background-image  | url()        |
    | repeat      | background-repeat | repeat-x ... |
    | attachment  | ..                | ..           |
    | color       | background-color  | ..           |
    | position    | ..                |              |

## Box Properties

    | description | css          | values                 |
    | sizing      | box-sizing   | border-box             |
    | margin      | margin       | 2px 4px 6px 8px        |
    | padding     | padding      | ..                     |
    | color       | border-color |                        |
    | style       | border-style | none hidden dotted ... |
    | width       | border-width | 10px                   |

## List Styling

    | description | css                 | values         |
    | type        | list-style-type     | disc circle .. |
    | positon     | list-style-position | inside outside |
    | image       | list-style-image    | url()          |



