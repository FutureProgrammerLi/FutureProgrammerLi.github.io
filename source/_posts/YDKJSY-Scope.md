---
title: YDKJSY-Scope
date: 2020-02-11 22:29:54
tags: YDKJSY
excerpt: Global scope / Variable hoisting / Loop 
categories:
- 书本
---

## Global Scope
1. Three ways of browser-executed Global Scope
 1.1 ES Module. Use import s references to every .js files.They each acts separate module to cooperate
 1.2 Using a bundler to compact as a bundle.
 Consider:
 ```js
 (function outerScope(){
     var moduleOne = (function(){

     })();
     var moduleTwo = (function(){

     })();
 })()
 ```
 1.3 The combination of the two ways above.So the global object becomes the only way for them to cooperate
 Like:(The difference is the outerScope)
 ```js
 var moduleOne = (function(){

     })();
 var moduleTwo = (function(){

     })();
```
Behavior from global object browser 'window'

```js
var name = 42;  //A default property of window
console.log(typeof name)   //string. Type is changed by 'window' default behavior
```

ES Modules treated modules in the second way above(with outerScope function scope)

## Global scope in Node 
**Node treats every .js file fairly. Even the main Node program is the same as other modules**
Node supports for both ES modules (import ... from ... ) form and Common.JS form (module.exports.xxx= xxx)
The way to reach global scope in Node
```js
global.studentName = 'xxx' 
//Then Node global object will obtain a property no matter what module this line is in.
```

### A trick to get reference to "globalThis"
```
const globalThis = (new Function("return this"))()
```

## Hoisting of Functions and Variables
```js
greeting()  //Hello undefined
student = 'Suzy'
function greeting(){
    console.log(`Hello ${student}`)
}
var student
```
The above is "re-written" like the following:
```js
function greeting(){                  //Auto-registration
    console.log(`Hello ${student}`)
}
var student                          //Auto-initialization?(not exactly)

greeting()
student = 'Suzy'                  //Initialization
```
**Function hoisting only applies to formal `function` declarations , not to `function` expressions**
```js
greeting() //TypeError for greeting was found but doesn't hold a function reference at that moment.
           //Like , greeting is not a function temporally?
var greeting = function(){}
```

**Shared names of functions and variables**
```js
console.log(greeting)   // ==> f greeting(){ consle.log('Hello')}
var greeting = 'hello'
function greeting(){
console.log('Hello')
}
```
**Functions names are prior to variables name**

## Re-declarations 
'let', 'const' are not allowed to be re-declared both with functions name and variables name.
They cannot be used on the same name in any order
```js
var greeting = 'hello'
function greeting(){
  console.log('Hello')
}
let greeting = 'hello'
console.log(greeting) //'greeting' has already been declared
```

## Loops scope
### No 're-declarations'
**'let'-created variables are applied per scope instance**
**'var'-created variables will be lifted to the global object as a property**
```js
var keep = true
while(keep){
    let value = Math.random() //
    if(value>0.5){
        keep = false
        console.log(value)
    }
}
```
#### for-loops
```js
for( let i = 0;i<3; i++){

}
//for observable reasons
//Same as the following:
{
    let $$i = 0
    for(;$$i <3;$$i++){
        let i = $$i
    }
}
```
**for-loops have their own scopes**

`Syntax Error stops it from starting excutions , and Type Errors arise during program execution`
Temporal Dead Zone(Prior to the declaration cannot used the variables in TPZ)
We cannot use the variable at any point prior to that initialization occuring.(Only auto-registration but no initialization(?))
--The term coined by TC39 to refer to this period of time from the entering of a scope to where the auto-initialization of the variable occurs, is: Temporal Dead Zone (TDZ).`
