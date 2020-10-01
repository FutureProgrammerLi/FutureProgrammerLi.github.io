---
title: YDKJSY-book2
date: 2020-01-11 19:39:44
tags: Javascript
excerpt: Closures / Parsing&Compilation of JS
categories:
- 书本
---
# State
`Storing values in variables and retrieving them later.`
---
# Closures
Functions are values,meaning they can be assigned and passed around like strings and values.
Since *these functions hold and access variables, they maintain their original scope* no matter where in the program the functions are eventually executed.
1. Tokenizing/Lexing
2. Parsing (Abstract Syntax Tree)
var a =2;
Variation Declaration ,Identifier , Assignment Expression , Numeric Liberal
3. Code Generation

# Parsing/Compilation before execution
Three observable facts：
1. 
```js
var greeting = "Hello";
console.log(greeting);  //Syntax Error with no greeting being consoled.
greeting = ."Hi";
```
2. 
```js
console.log("Howdy");
saySomething("Hello","Hi");    //Same that no Howdy is consoled
// Uncaught SyntaxError: Duplicate parameter name not allowed in this context
//Knowing duplciated parameters which are constrained by the inner strict mode , before execution,
function saySomething(greeting,greeting) {
    "use strict";                 
    console.log(greeting);
}
```
3. 
```js
function saySomething() {
    var greeting = "Hello";
    {
        greeting = "Howdy";   //knowing there will be a 'let' declaration though no hoisting  is in the scope here.
        let greeting = "Hi";
        console.log(greeting);
    }
}
//What's being indicated is that the greeting variable for that statement is the one from the next line,let greeting = "Hi",
//  rather than from the previous statement var greeting = "Hello".
saySomething();
// ReferenceError: Cannot access 'greeting' before initialization
```
An evidence protests the idea that 'let' does not hoist the declaration.Otherwise, no error would be thrown.
