---
title: YDKJSY-ch2/ 类型转换
date: 2020-01-07 15:12:05
tags: You Don't Know JS Yet
excerpt: 值的类型 / 类型转换 
categories:
- 书本
- important
---
# Type of Value
1. Primitive value
Number,String,Boolean,Undefined,Null,Symbol
Exclusively Symbol:
You won't encounter direct usage of symbols very often in typical JS programs.
They're mostly used in low-level code such as in libraries and frameworks.
2. Object
Array , Object

*Problems: typeof Array , typeof null returns object (JS bug)*
## Declaring and using values
Variables are just containers of values.
Declarations(5 ways in total)
var,let,const
function hello(name){} // try{something()}catch(err){console.log(err)} //variable `name` is the same as `err`
"const"s cannot be re-assigned
If you stick to using 'const' only with primitive values,you'll never have the confusion of the difference between re-assignment (not allowed) and mutation (allowed)!

# Functions
## Function Declaration Statements && Function Expressions
Different from the function declaration form, a function expression `is not associated with its identifier until that statement during runtime`.
JS treats functions as values
Since functions are values, they can be assigned as properties on objects:
whatToSay.greeting();

# Comparisons
equality comparsion ===  "strictly equal"  disallow any “coercion”
equavalence comparsion == "broderer-ly equal"  `loose equality` `coercive equality`
Similiarity: Both check values' type and the value
Differences: "==" allows coercions while "===" doesn't.
"==": If types are the same , no coercions will be done(?)
*Typical Cases*
```js
NaN === NaN;   //false
0 === -0;      //true
//loose equality 'bugs'
"" == 0;       //true
0 == false;    //true
```
Solutions:
1. Object.is() *suitable for both cases*
2. Number.isNaN() for NaN
Humorous understanding: Object.is() ==> "quadruple-equals" ====, the really-really-strict comparison!

## Object Values Comparisons
JS doesn't provide structural equality comparison because it's almost intractable to handle all the corner cases!
Consider the following case:
```js
var x = [ 1, 2, 3 ];

// assignment is by reference-copy, so
// y references the *same* array as x,
// not another copy of it.
var y = x;

y === x;              // true
y === [ 1, 2, 3 ];    // false ,a new Array
x === [ 1, 2, 3 ];    // false
[1,2,3] === [1,2,3]   //false . two different Arrays. 
```
Functions and objects share the same mechanism of arrays.

## Coercion order:
valueOf()==> toString() => Number()
```js
var x = "10";
var y = "9"; //converted into ASCII? 
x < y;      // true, watch out!
//Coercions
(undefined == null)  //true
Number(undefined); // NaN
Number(null); //  0
Number(""); // 0
//anything plus NaN equals NaN
var a={
    i:0,
    valueOf(){         //Key ,override
        return ++a.i
    }
}
if(a==1&&a==2&&a==3){      //Problem
    console.log(1)
}
```
Empty arrays
```js
let a =[]
console.log(a.valueOf()) //[]
console.log(a.toString()) //空
console.log(Number(a)) //0 a.valueOf().toString()has already been invoked secretly
console.log(Number(a.valueOf().toString()))  //0 
console.log(Boolean(a))     //true
console.log(![]==0)  //true.Memory have already been distributed to the Array ,so Arrays always equals true(?)
                     //Mechanism is the same as empty Objects
let b={}
console.log(b.valueOf())  //{}
console.log(b.toString())  //[object Object]
console.log(Number(b))   //NaN
```
Summary
```js 
console.log([]==0)  //toString()
console.log(![]==0)  //Number(false)

console.log([]==![])  //Number("") ,Number(false)
console.log([]==[])  

console.log({}==!{})
console.log({}=={})
```

