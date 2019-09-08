﻿---
title: "CSS Grid Examples from Learn CSS Grid Course at Scrimba"
date: 2018-02-16T14:22:38+03:00  
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
- cssgrid01.html
- cssgrid02.html
- cssgrid03.html
- cssgrid04.html
- cssgrid05.html
- cssgrid06.html
- cssgrid07.html
- cssgrid08.html
- cssgrid09.html
- cssgrid10.html
- cssgrid11.html
- cssgrid12.html
- cssgrid13.html
- cssgrid14.html
- cssgrid15.html
- cssgrid16.html
- cssgrid17.html
- cssgrid17b.html
- cssgrid19.html
- cssgrid20.html
- cssgrid21.html
- cssgrid22.html
- cssgrid23.html
- cssgrid24.html
- cssgrid25.html
- cssgrid26.html
- cssgrid01.css
- cssgrid02.css
- cssgrid03.css
- cssgrid04.css
- cssgrid05.css
- cssgrid06.css
- cssgrid07.css
- cssgrid08.css
- cssgrid09.css
- cssgrid10.css
- cssgrid11.css
- cssgrid12.css
- cssgrid13.css
- cssgrid14.css
- cssgrid15.css
- cssgrid16.css
- cssgrid17.css
- cssgrid17b.css
- cssgrid19.css
- cssgrid20.css
- cssgrid21.css
- cssgrid22.css
- cssgrid23.css
- cssgrid24.css
- cssgrid25.css
- cssgrid26.css
path: ~/projects/study/html/cssgrid_01/cssgrid.md

---

This blog post contains my notes from the great interactive video course [Learn CSS Grid for free](https://scrimba.com/g/gR8PTE) authored by [Per Harald Borgen](https://scrimba.com/@perborgen). Please watch the video courses. They explain CSS Grid in a very easy to understand way. 

<!--more-->

The videos are interactive. The watcher can edit the source code at any time directly on the video screen and see the effects immediately.

<!-- toc -->

## v01: Stacking without CSS Grid

```bash
cat cssgrid01.html
  ##> ﻿<html>
  ##>     <head>
  ##>         <link rel="stylesheet" href="basic.css">
  ##>         <style>
  ##>             .container {
  ##>                 display: grid;
  ##>             }
  ##>         </style>
  ##>     </head>
  ##>     <body>
  ##>         <div class="container">
  ##>             <div>1</div>
  ##>             <div>2</div>
  ##>             <div>3</div>
  ##>             <div>4</div>
  ##>             <div>5</div>
  ##>             <div>6</div>
  ##>         </div>
  ##>     </body>
  ##> </html>
  ##> 
```

<iframe src="cssgrid01.html" width="600" height="300"></iframe>

## v02: Basic Layout with CSS Grid

Sizes of columns and rows are specified with: `grid-template-columns` and `grid-template-rows`

```{css}
            .container {
                display: grid;
                grid-template-columns: 100px auto;
                grid-template-rows: 50px 50px 100px;
                grid-gap: 3px;
            }
```

Note that, `auto` makes the second column responsive.

Let's convert from 3 columns x 2 rows to 2 columns x 3 rows:

<iframe src="cssgrid02.html" width="600" height="150"></iframe>

```{css}
                grid-template-columns: 100px auto;
                grid-template-rows: 50px 50px 100px;
```

<iframe src="cssgrid03.html" width="600" height="250"></iframe>

We can move the style code into a separate css file:

```{html}
        <link rel="stylesheet" href="cssgrid04.css">
```

```bash
cat cssgrid04.css
  ##> ﻿.container {
  ##>     display: grid;
  ##>     grid-template-columns: 100px auto;
  ##>     grid-template-rows: 50px 50px 100px;
  ##>     grid-gap: 3px;
  ##> }
  ##> 
```

The result is the same.

## v03: Fraction Units and repeat()

`2fr` column is twice as large as `1fr` column.

```{css}
    grid-template-columns: 1fr 2fr 1fr;
    grid-template-rows: 50px 50px;
```

<iframe src="cssgrid05.html" width="600" height="150"></iframe>

To make all columns equal in width, we can either do this:

```{css}
    grid-template-columns: 1fr 1fr 1fr;
```

or this:

```{css}
    grid-template-columns: repeat(3, 1fr);
```

<iframe src="cssgrid06.html" width="600" height="150"></iframe>

Let's make 6 columns:

```{css}
    grid-template-columns: repeat(6, 1fr);
    grid-template-rows: 50px 50px;
```

<iframe src="cssgrid07.html" width="600" height="150"></iframe>

We can use `repeat()` also with rows:

    grid-template-rows: repeat(2, 50px);

As shorthand, we can specify column and row sizes in one line:

    grid-template: repeat(2, 50px) / repeat(3, 1fr);

<iframe src="cssgrid08.html" width="600" height="250"></iframe>

## v04: Positioning items

```bash
cat cssgrid09.html | sed -n '/container/,+5 p'
  ##>         <div class="container">
  ##>             <div class="header">HEADER</div>
  ##>             <div class="menu">MENU</div>
  ##>             <div class="content">CONTENT</div>
  ##>             <div class="footer">FOOTER</div>
  ##>         </div>
```

    grid-template: 40px 40px / repeat(2, 1fr);

<iframe src="cssgrid09.html" width="600" height="400"></iframe>

Let's give `content` more area to expand:

    grid-template: 40px 200px 40px / repeat(2, 1fr);

But we need to separate `content` section from `footer` section.

```{css}
.header {
  grid-column-start: 1;
  grid-column-end: 3;
}

.menu {}

.content {}

.footer {
  grid-column-start: 1;
  grid-column-end: 3;
}
```

Let's make `content` section wider too:

    grid-template: 40px 200px 40px / 1fr 4fr;

<iframe src="cssgrid10.html" width="600" height="400"></iframe>

Alternatively, we can write in shorthand form:

    grid-column: 1 / 3;

Or: 

    grid-column: 1 / span 2;

Or:

    grid-column: 1 / -1;

`-1` is useful if we may change number of columns later.

Let's use 12 column layout:

    grid-template-columns: repeat(12, 1fr);

We don't need to change the followings:

    grid-column: 1 / -1;

But we need to specify the positioning of `content`

    .content {
      grid-column: 2 / -1;
    }

<iframe src="cssgrid11.html" width="600" height="400"></iframe>

Now, let's change the overall layout. Let the `menu` section start from the top and slide `header` section beside it.

To do it, `header` should start from 2. column border and `menu` should start from 1. row border:

    .header {
      grid-column: 2 / -1;
    }

    .menu {
      grid-row: 1 / 3;
    }

<iframe src="cssgrid12.html" width="600" height="400"></iframe>

## v05: Template Areas

"Template Area" feature allows to experiment with different layouts very rapidly.

```bash
cat cssgrid13.css
  ##> ﻿.container {
  ##>   height: 100%;
  ##>   display: grid;
  ##>   grid-template-columns: repeat(12, 1fr);
  ##>   grid-template-rows: 40px auto 40px;
  ##>   grid-gap: 3px;
  ##>   grid-template-areas:
  ##>     "h h h h h h h h h h h h"
  ##>     "m c c c c c c c c c c c"
  ##>     "f f f f f f f f f f f f"
  ##> }
  ##> 
  ##> .header {
  ##>   grid-area: h;
  ##> }
  ##> 
  ##> .menu {
  ##>   grid-area: m;
  ##> }
  ##> 
  ##> .content {
  ##>   grid-area: c;
  ##> }
  ##> 
  ##> .footer {
  ##>   grid-area: f;
  ##> }
  ##> 
  ##> 
```

The layout of the page is specified with the following expression:

    grid-template-areas:
      "h h h h h h h h h h h h"
      "m c c c c c c c c c c c"
      "f f f f f f f f f f f f"

`h m c f` are `header menu content footer` sections respectively.

<iframe src="cssgrid13.html" width="600" height="400"></iframe>

To experiment with different layouts, let's change top h and bottom f with m:

    grid-template-areas:
      "m h h h h h h h h h h h"
      "m c c c c c c c c c c c"
      "m f f f f f f f f f f f"

<iframe src="cssgrid14.html" width="600" height="400"></iframe>

`.` dots will make blank cells

But all sections should be rectangular.

    grid-template-areas:
      "m . . h h h h h h h h h"
      "m c c c c c c c c c c c"
      "m f f f f f f f f f f f"

<iframe src="cssgrid15.html" width="600" height="400"></iframe>

## v06 Auto-fit and minmax

`auto-fit` adjusts the number of columns according to the size of the window:

    grid-template-columns: repeat(auto-fit, 100px);

<iframe src="cssgrid16.html" width="600" height="400"></iframe>

<iframe src="cssgrid16.html" width="300" height="400"></iframe>

Instead of `100px` we can use `1fr` together with minmax. A column will be at least `100px` and at max `1fr`.

    grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));

<iframe src="cssgrid17.html" width="600" height="400"></iframe>

<iframe src="cssgrid17.html" width="200" height="400"></iframe>

## v07 Implicit rows

Note that, in the previous examples the rows after second row have a broken height. Let's define default size for all rows:

    grid-auto-rows: 100px;

instead of:

    grid-template-rows: repeat(2, 100px);

-->

    grid-auto-rows: 100px;

<iframe src="cssgrid17b.html" width="200" height="400"></iframe>

## v08 An awesome image grid

@tbd

    .horizontal {
      grid-column: span 2;
    }

    .vertical {
      grid-row: span 2;
    }

    .big {
      grid-column: span 2;
      grid-row: span 2;
    }

blank spots

    grid-auto-flow: dense;

source order independence

## v09 Named Lines

Until now, we specified the layout using the border line numbers such as `1/3` where `1` is the starting column border line and 3 is the ending column border line:

    .container {
      display: grid;
      grid-template-columns: 1fr 5fr;
      grid-template-rows: 40px auto 40px;
    }
    .header {
      grid-column: 1 / 3;
    }
    ...

<iframe src="cssgrid19.html" width="600" height="200"></iframe>

Now, instead of line numbers we can give names to the lines:

    grid-template-columns: [main-start] 1fr [content-start] 5fr [content-end main-end];

Here, [main-start] is the name of first line. [content-start] is the name of second line.

Note that, third line has two names: `content-end` and `main-end`.

So, instead of `1 / 3` we can say `main-start / main-end`

    .header {
      grid-column: main-start / main-end;
    }

    .content {
      grid-column: content-start / content-end;
    }

<iframe src="cssgrid20.html" width="600" height="200"></iframe>

We can further simplify these expressions by eliminating `-start` and `-end` words:

    .header {
      grid-column: main;
    }

    .content {
      grid-column: content;
    }

<iframe src="cssgrid21.html" width="600" height="200"></iframe>

We can do the same also for naming row lines:

    .container {
      grid-template-columns: [main-start] 1fr [content-start] 5fr [content-end main-end];
      grid-template-rows: [main-start] 40px [content-start] auto [content-end] 40px [main-end];
      ...

    .footer {
      grid-column: main;
    }

<iframe src="cssgrid22.html" width="600" height="200"></iframe>

We can still make one more simplification. Note that, `content` section's 4 borders are named as `content`. So we can say:

    .content {
      grid-area: content;
    }

<iframe src="cssgrid23.html" width="600" height="200"></iframe>

But we cannot do this for `footer` section. Because footer's starting row is named as `content-end` whereas its ending row is named as `main-end`. Since, their names are different, we cannot specify it as a `grid-area`.

## v10: justify-content and align-content

Use `justify-content` to justify content as `start`, `center` or `end`:

    .container {
      justify-content: end;
      ...

<iframe src="cssgrid24.html" width="600" height="250"></iframe>

`align-content` aligns the elements vertically:

    align-content: end;

To distribute elements around space use: `space-evenly` `space-between` `space-around`

    justify-content: space-between;

<iframe src="cssgrid25.html" width="600" height="250"></iframe>

## v11: auto-fit vs. auto-fill

There is a very slight difference between `auto-fit` and `auto-fill`:

    .container {
      grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
      grid-template-rows: repeat(2, 100px);
    }

    .container2 {
      grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
      grid-template-rows: repeat(2, 100px);
    }

<iframe src="cssgrid26.html" width="260" height="550"></iframe>

When we increase the width, both behave similarly:

<iframe src="cssgrid26.html" width="460" height="550"></iframe>

But if we keep expanding, then they differ:

<iframe src="cssgrid26.html" width="660" height="550"></iframe>
