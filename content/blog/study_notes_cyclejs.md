---
title: "Study Notes for CycleJs"
date: 2018-02-26T22:04:09+03:00 
draft: false
description: ""
tags: ["cyclejs", "js"]
categories: ["software", "js"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: mertnuhoglu.com
resource_files:
- ex/cyclejs_ex17.js
- ex/cyclejs_ex16.js
- ex/cyclejs_ex15.js
- ex/cyclejs_ex14.js
- ex/cyclejs_ex13.js
- ex/cyclejs_ex12.js
- ex/cyclejs_ex11.js
- ex/cyclejs_ex10.js
- ex/cyclejs_ex09.js
- ex/cyclejs_ex08.js
- ex/cyclejs_ex07.js
- ex/cyclejs_ex06.js
- ex/cyclejs_ex05.js
- ex/cyclejs_ex04.js
- ex/cyclejs_ex03.js
- ex/cyclejs_ex02.js
- ex/cyclejs_ex17.html
- ex/cyclejs_ex16.html
- ex/cyclejs_ex15.html
- ex/cyclejs_ex14.html
- ex/cyclejs_ex13.html
- ex/cyclejs_ex12.html
- ex/cyclejs_ex11.html
- ex/cyclejs_ex10.html
- ex/cyclejs_ex09.html
- ex/cyclejs_ex08.html
- ex/cyclejs_ex07.html
- ex/cyclejs_ex06.html
- ex/cyclejs_ex05.html
- ex/cyclejs_ex04.html
- ex/cyclejs_ex03.html
- ex/cyclejs_ex02.html
- ex/cyclejs_ex01.html
path: ~/projects/study/js/study_notes_cyclejs.md
state: wip

---

This blog post is simply my work through notes while studying great video course of [CycleJs Fundamentals](https://egghead.io/courses/cycle-js-fundamentals) given by [Andre Staltz](https://twitter.com/andrestaltz) freely on [egghead](https://egghead.io/).

I really love Andre's style of teaching. He explains hard topics in a very easy to understand way. 

<!--more-->

<!-- toc -->

## v01

<iframe src="ex/cyclejs_ex01.html" width="200" height="30"></iframe>

``` bash
cat ex/cyclejs_ex01.html | sed -n "/<body>/,/<\/body>/ p"
  ##> <body>
  ##> <div id="app"></div>
  ##> <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/rxjs/5.5.6/Rx.min.js"></script>
  ##> <script>
  ##>   Rx.Observable.timer(0, 1000)
  ##>     .map(i => `Seconds ${i}`)
  ##>     .subscribe(text => {
  ##>       const container = document.querySelector('#app');
  ##>       container.textContent = text;
  ##>     })
  ##> </script>
  ##> </body>
``` 

This code consists of two parts: 

1. Logic

        Rx.Observable.timer(0, 1000)
          .map(i => `Seconds ${i}`)

2. Effects

          .subscribe(text => {
            const container = document.querySelector('#app');
            container.textContent = text;
          })

We want to separate these two parts.

Effects are everything that changes external world. Here we change the web page or DOM. console.log and http requests are all effects. These codes are put into `subscribe()` function.

Logic part doesn't change external world.

Effects part is imperative. Logic part is functional. We need to separate these two parts. Effects part needs to be handled by the framework. Application should consist only of the logic part. 

## v02

Now, let's encapsulate logic and effects parts into two separate functions.

``` bash
cat ex/cyclejs_ex02.js
  ##> function main() {
  ##>   return Rx.Observable.timer(0, 1000)
  ##>     .map(i => `Seconds ${i}`);
  ##> }
  ##> 
  ##> function DOMEffect(text$) {
  ##>   text$.subscribe(text => {
  ##>     const container = document.querySelector('#app');
  ##>     container.textContent = text;
  ##>   });
  ##> }
  ##> 
  ##> DOMEffect(main());
``` 

<iframe src="ex/cyclejs_ex02.html" width="200" height="30"></iframe>

We can use the rx observable in other effects too. For example:

``` bash
cat ex/cyclejs_ex03.js | sed -n "/function consoleLogEffect/,$ p"
  ##> function consoleLogEffect(msg$) {
  ##>   msg$.subscribe(msg => console.log(msg));
  ##> }
  ##> 
  ##> const sink = main();
  ##> consoleLogEffect(sink);
  ##> DOMEffect(sink);
``` 

<iframe src="ex/cyclejs_ex03.html" width="200" height="30"></iframe>

## v03

We have two types effects: consoleLogEffect and DOMEffect. But they both show the same stream from main(). 

Now, we want to show two different streams.

<iframe src="ex/cyclejs_ex04.html" width="200" height="30"></iframe>


``` js
function main() {
  return {
    DOM: Rx.Observable.timer(0, 1000)
      .map(i => `Seconds ${i}`),
    Log: Rx.Observable.timer(0, 2000)
      .map(i => 2*i),
  };
}

...
const sink = main();
consoleLogEffect(sink.Log);
DOMEffect(sink.DOM);
``` 

Logic is put into main() function. Effects are put into their own respective functions.

## v04

The last part joins logic part and effects part. We will encapsulate it into `run()` function.

``` js
function run(main) {
  const sinks = main();
  consoleLogEffect(sinks.Log);
  DOMEffect(sinks.DOM);
}
run(main);
``` 

But now the effects are hard-coded. Let's parameterize it too:

``` js
function run(main, effects) {
  const sinks = main();
  Object.keys(effects).forEach(key => {
    effects[key](sinks[key])
  })
}

const effects = {
  DOM: DOMEffect,
  Log: consoleLogEffect,
}
run(main, effects);
``` 

<iframe src="ex/cyclejs_ex05.html" width="200" height="30"></iframe>

Now, rename `effects` function as `drivers`. The reason for this renaming is that drivers are interfaces between software and hardware. Our drivers are interfaces between program (logic part) and effects.

``` js
const drivers = {
  DOM: DOMDriver,
  Log: consoleLogDriver,
}
``` 

<iframe src="ex/cyclejs_ex06.html" width="200" height="30"></iframe>

## v05

All the effects above are write effects. We don't have any input from external world until now.

Our program (main) should be able to get input from external world.

Inputs from external world are called as read (input) effects and as sources. Outputs to external world are called as write (output) effects and as sinks. 

Source/sink or input/output are named from the perspective of `main()` function. The logic part of the application is in `main()` function. There are some external inputs or sources to this program. They are called as input effects. And there are some external outputs or sinks from this program. They are called as write effects.

``` js
function DOMDriver(text$) {
  ...
  const DOMSource = Rx.Observable.fromEvent(document, 'click');
  return DOMSource;
}

function run(main, drivers) {
  const sinks = main(DOMSource);
  const DOMSource = drivers.DOM(sinks.DOM);
  //Object.keys(drivers).forEach(key => {
    //drivers[key](sinks[key])
  //})
}
``` 

But there is a cyclic dependency problem here:

    const sinks = main(DOMSource);
    const DOMSource = drivers.DOM(sinks.DOM);

This is similar to the following problem:

    a = f(b)
    b = g(a)

To solve this cyclic dependency, we need to introduce bProxy as the first input argument:

    bProxy = ...
    a = f(bProxy)
    b = g(a)
    bProxy.imitate(b)

So, we pass an empty stream as the initial input argument of `main()` 

    const proxyDOMSource = new Rx.Subject();
    const sinks = main(proxyDOMSource);
    const DOMSource = drivers.DOM(sinks.DOM);
    DOMSource.subscribe(click => proxyDOMSource.next(click));

Now, let's use this input source stream in our example. Instead of using `Rx.Observable.timer` stream once, let's use click stream to restart `timer`.

``` js
function main() {
  return {
    DOM: Rx.Observable.timer(0, 1000)
      .map(i => `Seconds ${i}`),
    Log: Rx.Observable.timer(0, 2000)
      .map(i => 2*i),
  };
}

--->>>

function main(DOMSource) {
  const click$ = DOMSource;
  return {
    DOM: click$
      .startWith(null)
      .switchMap(() =>
        Rx.Observable.timer(0, 1000)
          .map(i => `Seconds ${i}`)
      ),
    Log: Rx.Observable.timer(0, 2000).map(i => 2*i),
  };
}
``` 

Now, clicking anywhere in the web page, restarts the `timer` count from zero.

<iframe src="ex/cyclejs_ex07.html" width="200" height="30"></iframe>

## v06

Now, parameterize input source arguments to main() function. There can be different types of drivers.

``` js
function main(DOMSource) {

--->>>

function main(sources) {
  const click$ = sources.DOM;
``` 

``` js
function run(main, drivers) {
  const proxyDOMSource = new Rx.Subject();
  const sinks = main(proxyDOMSource);

--->>>

function run(main, drivers) {
  const proxySources = {}
  Object.keys(drivers).forEach(key => {
    proxySources[key] = new Rx.Subject();
  })
  const sinks = main(proxySources);
  Object.keys(drivers).forEach(key => {
    const source = drivers[key](sinks[key]);
    source.subscribe(x => proxySources[key].next(x));
  })
}
``` 

This `run` function is totally generic. It doesn't contain any application specific code. Therefore, we can move it to an external framework. This is what `cycle-core` library does.

``` js
Cycle.run(main, drivers);
``` 

`Cycle.run` function is defined in:

    <script src="https://rawgit.com/cyclejs/cycle-core/v6.0.0/dist/cycle.js"></script>

<iframe src="ex/cyclejs_ex08.html" width="200" height="30"></iframe>

## v07 Improving DOM Driver

Now, let's return HTML element stream instead of text stream inside the DOM driver.

``` js
      .flatMapLatest(() =>
        Rx.Observable.timer(0, 1000)
          .map(i => `Seconds ${i}`)
      ),

      --->>>

      .flatMapLatest(() =>
        Rx.Observable.timer(0, 1000)
          .map(i => {
              return {
                tagName: 'H1',
                children: [
                  `Seconds ${i}`
                ]
              }
            }
          )
      ),
``` 

``` js
  text$.subscribe(text => {
    const container = document.querySelector('#app');
    container.textContent = text;
  });

  --->>>

  function createElement(obj) {
    const element = document.createElement(obj.tagName);
    element.innerHTML = obj.children[0];
    return element;
  }
  obj$.subscribe(obj => {
    const container = document.querySelector('#app');
    container.innerHTML = '';
    const element = createElement(obj);
    container.appendChild(element);
  });
``` 

<iframe src="ex/cyclejs_ex09.html" width="200" height="30"></iframe>

But this solution contains hard-coded code inside `createElement`. We might have different types of elements in the stream.

Let's make `createElement` more generic:

``` js
          .map(i => {
              return {
                tagName: 'H1',
                children: [
                  `Seconds ${i}`
                ]
              }
            }

          --->>>

          .map(i => {
              return {
                tagName: 'H1',
                children: [
                  {
                    tagName: 'SPAN',
                    children: [
                      `Seconds ${i}`
                    ]
                  }
                ]
              }
            }
``` 

``` js
  function createElement(obj) {
    const element = document.createElement(obj.tagName);
    element.innerHTML = obj.children[0];
    return element;
  }

  --->>>

  function createElement(obj) {
    const element = document.createElement(obj.tagName);
    obj.children
      .filter(c => typeof c === 'object')
      .map(createElement)
      .forEach(c => element.appendChild(c));
    obj.children
      .filter(c => typeof c === 'string')
      .forEach(c => element.innerHTML += c);
    return element;
  }
``` 

<iframe src="ex/cyclejs_ex10.html" width="200" height="30"></iframe>

## v08: Different Types of Input Events

Currently, we have only click type input events. But there are other types of input events too, such as `mouseover`.

``` js
  const DOMSource = Rx.Observable.fromEvent(document, 'click');

  --->>>

  const DOMSource = {
    selectEvents: function(tagName, eventType) {
      return Rx.Observable.fromEvent(document, eventType)
        .filter(ev => ev.target.tagName === tagName.toUpperCase());
    }
  }
``` 

``` js
  const click$ = sources.DOM;

  --->>>

  const mouseover$ = sources.DOM.selectEvents('span', 'mouseover');
``` 

<iframe src="ex/cyclejs_ex11.html" width="200" height="30"></iframe>

## v09

Our view code can be automated a little further:

``` js
              return {
                tagName: 'H1',
                children: [
                  {
                    tagName: 'SPAN',
                    children: [
                      `Seconds ${i}`
                    ]
                  }
                ]
              }

--->>> 

function h(tagName, children) {
  return {
    tagName: tagName,
    children: children,
  }
}

              return {
                h('H1', [
                  h('SPAN', [
                    `Seconds ${i}`
                  ])
                ])
``` 

<iframe src="ex/cyclejs_ex12.html" width="200" height="30"></iframe>

Now, we can further simplify by defining a new helper function for each `tagName`:

``` js
function h1(children) {
  return {
    tagName: 'H1',
    children: children,
  }
}

function span(children) {
  return {
    tagName: 'SPAN',
    children: children,
  }
}

                ...
                h1([
                  span([
                    `Seconds ${i}`
                  ])
                ])
``` 

Now, we can use js functions instead of a markup language such as HTML or templating language such as PUG or Mustache.

## v10

Can we move DOMDriver into an external framework? Yes, only application specific code inside DOMDriver is HTML element id `#app`.

``` js
function DOMDriver(obj$) {
  function createElement(obj) {
    const element = document.createElement(obj.tagName);
    obj.children
      .filter(c => typeof c === 'object')
      .map(createElement)
      .forEach(c => element.appendChild(c));
    obj.children
      .filter(c => typeof c === 'string')
      .forEach(c => element.innerHTML += c);
    return element;
  }
  obj$.subscribe(obj => {
    const container = document.querySelector('#app');
    container.innerHTML = '';
    const element = createElement(obj);
    container.appendChild(element);
  });
  const DOMSource = {
    selectEvents: function(tagName, eventType) {
      return Rx.Observable.fromEvent(document, eventType)
        .filter(ev => ev.target.tagName === tagName.toUpperCase());
    }
  }
  return DOMSource;
}

--->>>


function makeDOMDriver(mountSelector) {
  return function DOMDriver(obj$) { ... }
   
----

const drivers = {
  DOM: DOMDriver,

--->>>

const drivers = {
  DOM: makeDOMDriver('#app'),

``` 

<iframe src="ex/cyclejs_ex13.html" width="200" height="30"></iframe>

One critical issue performance-wise is the following line because we are recreating the complete inner DOM tree from scratch after every event:

    container.innerHTML = '';

Another issue is that we don't yet support CSS selectors except `tagName`. We should support CSS selectors such as `.class`.

To solve these issues, we can use the actual cyclejs library instead of helper functions and driver functions.

``` bash
cat ex/cyclejs_ex14.js
  ##> const {h, h1, span, makeDOMDriver} = CycleDOM;
  ##> 
  ##> function main(sources) {
  ##>   const mouseover$ = sources.DOM.select('span').events('mouseover');
  ##>   const sinks = {
  ##>     DOM: mouseover$
  ##>       .startWith(null)
  ##>       .flatMapLatest(() =>
  ##>         Rx.Observable.timer(0, 1000)
  ##>           .map(i => 
  ##>             h1( {style: {background: 'yellow'}}, [
  ##>               span([
  ##>                 `Seconds ${i}`
  ##>               ])
  ##>             ])
  ##>           )
  ##>       ),
  ##>     Log: Rx.Observable.timer(0, 2000).map(i => 2*i),
  ##>   };
  ##>   return sinks;
  ##> }
  ##> 
  ##> function consoleLogDriver(msg$) {
  ##>   msg$.subscribe(msg => console.log(msg));
  ##> }
  ##> 
  ##> const drivers = {
  ##>   DOM: makeDOMDriver('#app'),
  ##>   Log: consoleLogDriver,
  ##> }
  ##> 
  ##> Cycle.run(main, drivers);
``` 

<iframe src="ex/cyclejs_ex14.html" width="200" height="30"></iframe>

``` js
const {h, h1, span, makeDOMDriver} = CycleDOM;
...
  const mouseover$ = sources.DOM.select('span').events('mouseover');
``` 

CycleDOM actually uses virtual DOM objects. This improves the performance of DOM updates a lot.

You can also specify attributes of HTML elements:

            h1( {style: {background: 'yellow'}}, [

## v11 Hello World App

``` bash
cat ex/cyclejs_ex15.js
  ##> const {label, input, hr, div, h1, makeDOMDriver} = CycleDOM;
  ##> 
  ##> function main(sources) {
  ##>   const inputEv$ = sources.DOM.select('.field').events('input');
  ##>   const name$ = inputEv$.map(ev => ev.target.value).startWith('');
  ##>   return {
  ##>     DOM: name$.map(name =>
  ##>       div([
  ##>         label('Name:'),
  ##>         input('.field', {type: 'text'}),
  ##>         hr(),
  ##>         h1(`Hello ${name}!`)
  ##>       ])
  ##>     )
  ##>   };
  ##> }
  ##> 
  ##> const drivers = {
  ##>   DOM: makeDOMDriver('#app'),
  ##> }
  ##> 
  ##> Cycle.run(main, drivers);
``` 

<iframe src="ex/cyclejs_ex15.html" width="200" height="150"></iframe>

## v12: Decrement Increment: State

``` bash
cat ex/cyclejs_ex16.js
  ##> const {button, p, label, div, makeDOMDriver} = CycleDOM;
  ##> 
  ##> function main(sources) {
  ##>   const decrementClick$ = sources.DOM
  ##>     .select('.decrement').events('click');
  ##>   const incrementClick$ = sources.DOM
  ##>     .select('.increment').events('click');
  ##>   const decrementAction$ = decrementClick$.map(ev => -1);
  ##>   const incrementAction$ = incrementClick$.map(ev => +1);
  ##>   const number$ = Rx.Observable.of(0)
  ##>     .merge(decrementAction$).merge(incrementAction$)
  ##>     .scan((prev, curr) => prev + curr);
  ##>   return {
  ##>     DOM: number$.map(number =>
  ##>       div([
  ##>         button('.decrement', 'Decrement'),
  ##>         button('.increment', 'Increment'),
  ##>         p([
  ##>           label(String(number))
  ##>         ])
  ##>       ])
  ##>     )
  ##>   };
  ##> }
  ##> 
  ##> const drivers = {
  ##>   DOM: makeDOMDriver('#app'),
  ##> }
  ##> 
  ##> Cycle.run(main, drivers);
``` 

<iframe src="ex/cyclejs_ex16.html" width="200" height="150"></iframe>

## v13: HTTP Driver

``` bash
cat ex/cyclejs_ex17.js
  ##> const {button, h1, h4, a, div, makeDOMDriver} = CycleDOM;
  ##> const {makeHTTPDriver} = CycleHTTPDriver;
  ##> 
  ##> // DOM read effect: button clicked
  ##> // HTTP write effect: request sent
  ##> // HTTP read effect: response received
  ##> // DOM write effect: data displayed
  ##> 
  ##> function main(sources) {
  ##>   const clickEvent$ = sources.DOM
  ##>     .select('.get-first').events('click');
  ##>   
  ##>   const request$ = clickEvent$.map(() => {
  ##>     return {
  ##>       url: 'http://jsonplaceholder.typicode.com/users/1',
  ##>       method: 'GET',
  ##>     };
  ##>   });
  ##>   
  ##>   const response$$ = sources.HTTP
  ##>     .filter(response$ => response$.request.url ===
  ##>            'http://jsonplaceholder.typicode.com/users/1');
  ##>   
  ##>   const response$ = response$$.switch();
  ##>   const firstUser$ = response$.map(response => response.body)
  ##>     .startWith(null);
  ##>   
  ##>   return {
  ##>     DOM: firstUser$.map(firstUser =>
  ##>       div([
  ##>         button('.get-first', 'Get first user'),
  ##>         firstUser === null ? null : div('.user-details', [
  ##>           h1('.user-name', firstUser.name),
  ##>           h4('.user-email', firstUser.email),
  ##>           a('.user-website', {href: firstUser.website}, firstUser.website)
  ##>         ])
  ##>       ])
  ##>     ),
  ##>     HTTP: request$,
  ##>   };
  ##> }
  ##> 
  ##> const drivers = {
  ##>   DOM: makeDOMDriver('#app'),
  ##>   HTTP: makeHTTPDriver(),
  ##> }
  ##> 
  ##> Cycle.run(main, drivers);
  ##> 
``` 

<iframe src="ex/cyclejs_ex17.html" width="200" height="250"></iframe>

## v14: BMI Calculator

