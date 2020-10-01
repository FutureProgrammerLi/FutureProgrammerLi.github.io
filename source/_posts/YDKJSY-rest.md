---
title: YDKJSY-rest
date: 2020-01-08 10:53:10
tags: YDKJSY
excerpt: Iterators / Closures / THIS
categories:
- 书本
---
# Iterators
1. Iterables: Strings, Arrays, Maps, Sets and otherts.
2. Three default iterators
```js
let arr = [2,3,4,5]
//for(i of arr.entries())  Objects/Maps default iterator.Return tuples that contain both keys and values.
//for(i of arr.keys())   iterates the key names. Iterating index for Arrays.
for(i of arr.values()) //by default.Same as (i of arr)
{
    console.log(i)
}
```
3. Beware of for...of /for ... in ...
for...of can only be used on iterables, invoking its iterator repeatedly.
 When for...in is used on an array , *it searches for the index.*If the index exists in the array, `it returns true.`
 When used in an object, `it returns the keys of the object.`

# Closures
Definition:
>Closure is the ability of a function to remember and continue to access variables defined outside its scope, even when that function is executed in a different scope.

Characteristics:
1. Objects do not have closures.
2. To observe a closure, you must `execute a function in a different scope` than where that function was originally defined.
Consider:
```js
function counter(step = 1) {
    var count = 0;
    return function increaseCount(){
        count = count + step;
        return count;
    };
}

var incBy1 = counter(1);
var incBy3 = counter(3);

incBy1();       // 1
incBy1();       // 2

incBy3();       // 3
incBy3();       // 6
incBy3();       // 9
```
*incBy1 and incBy3 are two different instances.They own different inner variables.*

`Take care`: It's not necessary that the outer scope be a function.
```js
for (let [idx,btn] of buttons.entries()) {
    btn.addEventListener("click",function onClick(evt){
       console.log(`Clicked on button (${ idx })!`);   //The Inner function closes over the outer variable idx.
    });
}
```

# this
*Three ways of changin the pointer of `this`*
1. 
```js
function classroom(teacher) {
    return function study() {
        console.log(
            `${ teacher } wants you to study ${ this.topic }`
        );
    };
}
var topic = someSubjects;
var assignment = classroom("Kyle");  //`this` refers to the global object window.
```
2. 
```js
var homework = {
    topic: "JS",
    assignment: assignment
};

homework.assignment();
// Kyle wants you to study JS
//`this` refers to the homework object
```
3. 
```js
var otherHomework = {
    topic: "Math"
};

assignment.call(otherHomework);
// Kyle wants you to study Math
// Only to pass the parameter topic to the assignment function
```
*Benefits of using `this`*compared to the closures(?):
The ability to more flexibly re-use a single function with data from different objects;
Dynamic-context;
A function that closes over a scope can never reference a different scope or set of variables. 
Better re-use in prototype chains.