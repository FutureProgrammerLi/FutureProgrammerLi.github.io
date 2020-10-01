---
title: YDKJSY
date: 2020-01-07 00:13:01
tags: You Don't Know JS Yet
excerpt: Characteristics of JS
categories:
- 书本
---
## Paradigm
1. Procedural    C(linear progression)
2. Objected-oriented Programming  Java/C++ (together in units)
3. Functiontional Programming  Haskell(?)
(ALL-OR-NOTHING)
JS, all the above kinds.

## Backwards && Forwards
Javascript:
Backwards means once something is accepted as valid JS, there will not be a future change to the language that causes that code to become invalid JS.
HTML/CSS:
Forwards means that including a new addition to the language in a program would not cause that program to break if it were run in an older JS engine. 
(the unrecognized CSS / HTML is skipped over, while the rest of the CSS / HTML would be processed accordingly.)
The reason why JS is not being forward-capability:
HTML&CSS are declarative.`skipp-able,ignore the unrecognized`
But for JS, it's impossible to ensure that a subsequent part of the program wasn't expecting the skipped over part to have been processed.

## Jumping the Gap
new syntax
the transpiler -- Babel 
```js
if (something) {
    let x = 3;
    console.log(x);
}
else {
    let x = 4;
    console.log(x);
}
//converted version
var x$0;
var x$1;
if (something) {
    x$0 = 3;
    console.log(x$0);
}
else {
    x$1 = 4;
    console.log(x$1);
}
```

## Filling the Gap
Provide a definition for that missing API method that stands in and acts as if the older environment had already had it natively defined.
new API--polyfill
```js
if (!Promise.prototype.finally) {
    Promise.prototype.finally = function f(fn){
        return this.then(fn,fn);
    };
}
```