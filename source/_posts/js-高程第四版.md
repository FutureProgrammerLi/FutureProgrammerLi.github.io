---
title: js-高程第四版
date: 2020-07-03 09:24:23
tags: 
- Javascript
categories:
- 书本
excerpt: 高程4读书笔记/未完成
---

# What is JavaScript?

A round-up trip to server was needed for just simple input validations.

Brendan Eich, 10days of development , 25years of impression.

Slow speed (Network), large size (webpage)

Netscape->Brendan Eich -> Mocha -> LiveScript -> JavaScript

NetScape Navigator   < vs >    Internet Explorer 3 August,1996  "INFAMY" for NetScapce

 JavaScript                   <  vs >              JScript

**TC39**(Technical Committee #39) was assigned to standardize the syntax in JavaScript, (DEFINITION)

ECMAScript , a new scripting language. (Relations between TC39 and ECMA?) 

**Javascript is more than what just defined in ECMA-262. It is made up of three parts**

* The Core (ECMAScript)   //BASE
* The Document Object Model (DOM)   //EXTENSION
* The Browser Object Model (BOM)       //EXTENSION

ES1,ES3,ES5,ES6.  (ES2无改变,所以不常见; ES4变化过大,被改为ES3.1. **ES3.1最终被变为ES5;**)

ES5: December,2009 ; Strongly typed variables,classes/new statements and data structure ...

ES6:June 2015, ES2015,ES Harmony: class/modules/iterators/generators/arrow functions/promises/reflection/proxies/new data types

---

Dynamic HTML,allows developers to alter content of page without reloading it.

*BUT* NetScape and IE implemented it in different ways.

**W3C** ,the one who rein in to stop web developing into two different factions.(in DHTML)

---

BOM,  HTML5 made it standardized.Different browsers implement it differently.

Summary: ECMAScript, core functionalities.

DOM, methods and interfaces for working with the content of web page.

BOM, methods and interfaces for working with the browser

# How to make JavaScript coexist in HTML?

script attributes:

1. `async` , immediately download scripts but not prevent other actions on the page(?)
2. `defer`, IE7 or earlier allow inline scripts.Defered until DOM content was completely parsed and displayed

3. `src`,`type`(replaces the `language` option,type/javascript or module)

4. `charset`,`crossorigin`,(anonymous/use-credentials)`integrity`(to ensure CDN,like a key to use CDN)

**TWO WAYS TO USE \<script> ELEMENT**

1. Inline scripting \<script> function hi(){console.log("hi")}\</script> (avod using \</script>in code)

2. From external JS files \<script src="sth.js" >\</script>

   If both provided , inline scripts will be ignored and external files will be downloaded.

Powerful but controversal: src attribute can be set to a URL , which is not subject to cross-origin restrictions, but those JS returned and executed will be. 

Use attr `integrity` to limit a bit of the capability of \<script> (Acts like a key)

**Rendering begins when browser parses the tag \<body>, DON'T place script tags in \<head>,DOM will be displayed only after all scripts are excuted.DO put them before the end tag of body \</body>.

**Defer scripts**

Begin downloading immediately , but execution should be deferred.

1. Scripts WILL NOT CHANGE THE STRUCTURE of the page as it executes.

2. Executed in defer order, before `DOMContentLoaded` event.

**Asynchronorous scripts**

1. Not guaranteed to executed in scripts order.

2. NO SCRIPTS DEPENDENCIES
3. Guaranteed to be executed before `load` event and before/after `DOMContentLoaded`

**Dynamic scripts**

1. ```js
   let script = document.createElement('script');
   script.src = 'sth.js';
   script.async = false;  //Set async to false.
   document.head.appendChild(script)
   ```

   Scripts created in this way is `async` by default.You can set it to false explicitly.

2. Severely damage performance.(Not informing the preloaders.)

   Try replace dynamic scripting as followed:

   ```js
   <link ref="subresource" href="sth.js" >
   ```

**Linking external JS files is considered the best pratice compared to inline scripts.**

1. Maintainability(Put all JS files in a directory for maintainability)

2. Caching(Files downloaded will be cached by browser)
3. Future-proof(Same syntax to include files for XHTML and HTML)

**Document Modes**

Quirk Mode(Omit the doctype) and Standards Mode(Use the doctype exclamation)

**Use \<noscript> element to provide content for browsers that do not support or disable JS.**

It can contain any HTML element aside from \<script> tag

# Basic syntax of ECMAScript

typeof cannot be a variable name , but can be a function name.!!!??

Identifiers can only be started with **letter , underscore(\_),or dollar($) sign**

### Three ways to declare variables: var / let /const 

1. var (function scoped)

```js
function test(){
    var msg = 'hi'  //a local variable to the function scope.
}
test()             //after this , msg is destroyed.
console.log(msg)   //error
function test2(){
    msg2 = 'what' //create a global variable
                  //Reference Error will be thrown in strict mode.
                  //hard maintainability
}
test2()
console.log(msg2)
```

2. let (block scoped)

```js
if(true){
    var name = 'hi'
    console.log(name)  //'hi'
}
console.log(name)     //'hi'
if(true){
    let age = '26'
    console.log(age)  //26
}
console.log(age)      //error
```

#### Differences between var and let

1. **BLOCK SCOPED VS FUNCTION SCOPED**

   The former is strictly a subset of the latter one.

2. **Redundant Declaration**

   `let` is not allowed to declare variables repeatly while using `var` can .(except for nested cases)

3. No coexistence

   ```js
   let a
   var a
   
   var b
   let b //Both throw error "xxx has been declared"
   ```

4. Temporal Dead Zone: 

   The segment of execution that occurs before the declaration is referred to.

   ```js
   console.log(1)
   console.log(name)              //TDZ
   let name = 'hh'
   ```

5. Variables declared with 'let' will not attached to window, while those with 'var' will.

   ```js
   let a = 1;
   var b = 2;
   console.log(window.a)  //undefined
   console.log(window.b)  // 2
   ```

6. let variables redeclarations will throw errors even they were declared in different \<script> tags.

   ```js
   <scirpt>
       let a = 1
   </scirpt>
   <script>
       let a = 2           //Throw error
   </script>
   ```

   **let variables disadvantage(Conditional Declaration)**

   Use try/catch block scoped to deal with the problem.

7. `const` ,acts identically with that of `let`,

   But it has to be initialized when declared , and cannot be redefined anywhere else.

   ```js
   const a = 2
   //const a
   //const a = 3  
   //a = 4
   //will both throw errors.
   ```

### Data Types

Six simple data types: null , undefined , number , string ,boolean, symbol , (bigint)

One complex data type : Object.

#### typeof operator

Function can be detected by typeof operator

```js
function test(){}
console.log(typeof test)    //"function"
console.log(typeof NaN)     //"number"
console.log(typeof Symbol('1')) //"symbol"

//confusing cases
console.log(typeof null)  //"object"
```

#### Undefined type

The value of undefined is falsy.

**Undeclared vs Uninitialized**

```js
//operator typeof can be used on an undeclared variable,
//which can be confusing, unable to differentiate from those uninitialized.
let msg                 //uninitialized variable
console.log(typeof msg)   //"undefined"
console.log(typeof age)   //"undefined"   undeclared variable
```

#### Null type

It is advisable to initialize a value of null for variables that later hold an object.

#### Differences between null and undefined

1. null == undefined     //true , undefined is a derivative of null.
2. Do not explictly assign undefined to a variable ,but you can do with null for different purpose.

#### Boolean type

Literal values: true  false

Automatic conversions

​                  true                                                                                   false

String      nonempty string                                                     ""(empty string)

Number      nonezero number(including Infinity)                      0,NaN

Object          any object                                                                    null

Undefined        n/a                                                                          undefined

#### Number type

Octal literals are invalid in strict mode.

Octal/Hexadecimal numbers will treated as decimal numbers in arithmetic operations.

0.05+0.25 == 0.15+0.15 == 0.3      //true

0.1+0.2 == 0.3       //false

Biggest num / Smallest num      Number.MAX_VALUE/Number.MIN_VALUE

Infinity cannot be calculated later in the programer, therefore, use isFinite() to test whether to continue.

#### NaN

Any operation involves NaN returns NaN, AND NaN is not equal to any value,including NaN 

Use isNaN() to test an identifier whether equals to NaN.

#### Three ways to convert variables into number

1. Number(),     null->0 undefined->NaN      Strings=>("-011")->-11

   When used on an object , the order is valueOf() , toString() , Number()

2. parseInt() ,always use the second param, which means the radix(0 or default,takes the number as a decimal.)

3. parseFloat()

   ```js
   //Differences 
   Number('1234blue') //NaN
   parseInt('1234blue')  //1234
   parseInt(011,0)  //11
   parseInt(111,2)  //7
   parseInt('AF')   //NaN
   parseInt('AF',16)  //175  this is why you should use the radix parameter.
   parseFloat('22.34.5')  //22.34 ,see the second decimal point as an invalid character
   ```

   

#### String type

Strings are immutable. 

```js
let lang = 'Java'
lang = lang + 'Script'
```

Behind the scenes , a new space to stored 10 characters is created for lang .And the old lang, with the value of 'Java' , and the string,"Script" are detroyed after the concatenation. //The reason why old browsers may run slow doing strings concatenation.

**Two ways to convert to a String**

1. toString() , when used on a number , a param meaning raidx is accepted , which turns the value to the radix base.

   ```js
   let a = 10
   a.toString(2)  //3
   a.toString(8)  //12
   ```

2. String() .For cases like `undefined` and `null`. If the value has the method toString(), call it; Otherwise ,return 'undefined' or 'null'

   ```js
   String(undefined)  //undefined
   undefined.toString()   //cannot read property 'toString' of undefined
   ```

#### Template literals

Interpolations will be coerced into a string using the toString() method, which can be overwrited.

```js
let foo = {
    toString(){
    return 'World'
}}
console.log(`Hello , ${foo}!`)  //Hello , World!
function capitalize(word){
    return `${ word[0].toUpperCase() }${word.slice(1)}`
}
console.log(`${capitalize('hello')}, ${capitalize('world')`)
```

Tag function    P52 **NEW**

parmas: (strings , ...expressions) // strings is an array consisted of those not expressions.expressions are corresponding *values*

```js
let a = 1 , b =2;
function simpleTag(strings,...expressions){
    console.log(strings)
    console.log(...expressions)
}
let result = simpleTag`${a}+${b}=${a+b}`
//["","+","=",""]     //why 2 empty items?
//1,2
```

```js
String.raw`\u00A9`      //\u00A9   ???What is this??
```

#### Symbol Type

Usage: For not property collision in an object. Every symbol instances is `unique and immutable`

Two ways to create Symbol,

```js
let basic = Symbol('desc')
let globalSymbol = Symbol.for('desc')  
typeof basic      //symbol
typeof globalSymbol //symbol
```

**Two totally different symbols**

The second way registration is for reuse, and the description parameter is required.

*Symbol.for('desc')*will create a symbol and register it on a global 'list', and *Symbol.keyFor('desc')* will return the symbol which description is desc.

```js
Object.getOwnPropertyNames(o)    //return regular property names in an array
Object.getOwnPropertySymbols(o)  //return symbol properties
Object.getOwnPropertyDescriptors(o)
Reflect.ownKeys(o)             //These two are the same , both will return regular and symbol properties descriptors.
```

#### Object Type

```js
let obj = new Object
let obj2 = new Object()  //valid in both ways
```

Properties in Object

* constructor / isPrototypeOf(object) / hasOwnProperty /propertyIsEnumerable / toLocaleString  / toString  / valueOf

  toLocaleString will return a string that is appropriate for the locale of execution environment while toString will not.

DOM / BOM's behavior are considered `host objects` ,which are not governed by ECMAscript.

### Operators

#### Unary operators

```js
 //        ++ /  --
let a = 20
let b = a-- + 1;
let c = ++a + 2;
//the same as following
let b = a + 1
a = a -1
//
a = a + 1
c = a + 2
//Therefore , let b = a-- + 1     --> b = 21 , a = 19
//let c = ++a + 2 --> a = 19 + 1 ; c = 20 + 2 = 22
//****PRE AND POST ****
```

### Bitwise operators

Values operated by bitwise operators are stored in 64-bits format , converted into 32-bits to calculate, and converted back into 64-bits.

31bits represent the value and the 32 th bit represents the type of the value.`0 for Positive and 1 for Negative  `

**Only for storage view**

Positive 18: 00000000000000000000000000010010 -> 10010

Negative 18 : 1111111111111111111111101110    ->Two's complement.

**Get the binary representation of its absolute value . Convert 0 to 1 , 1 to 0 , plus 1 **

#### Bitwise NOT / AND / OR / XOR

~ / & / | / ^

XOR's value is one less than OR's value(? really?)

```js
let num = 25  //11001
let num2 = ~num //111111111111 00110        //not 00111 ,no plus 1, not the two complement?
//equals to let num2 = -num - 1
//but the Bitwise operator is faster.
```

#### Signed Shift

\>>  / \<<  , ignores the 32th bit ,filled the moved bits with 0.

#### Unsigned Shift

\>>> / <<< ,using on negative numbers may turn it into a positive one.

```js
let a = -64
a >>> 5          //1342177226
```

#### Boolean operators

**!!sth is equal to Boolean(sth) , ++string is equal to Number(sth)**

Logical AND (Short-circuited.)

* If the first operand is an object , return the second operand
* **Return the second operand if the first operand evaluates to true**
* If both are objects , return the second object.
* If **either(one or two** operand is null/NaN/undefined,    return null/NaN/undefined.

```js
//Consider the case
let result = (true && somethingUndeclared)
console.log(result)       //Error occurs
let result2 = (false && somethingError)
console.log(result2)     //This will be executed
```

Logical OR

* If the first operand evaluates to false , then the second operand is returned.
* If both are objects, return the first object.
* If both are null/NaN/undefined , return null/NaN/undefined

```js
let result = (true || somethingUndeclared)
console.log(result)    //no error
//Frequently used case:
let myObject = preferredObject || backupObject
//assign the former one if available, otherwise , assign the backupObject to myObject
```

### Multiplicative operators

#### Multiply *

* Either operand is NaN , returns NaN
* Infinity * 0     -> NaN , Infinity / 0     -> Infinity
* Infinity * any other value except 0 , Infinity or -Infinity , depending on the sign of second operand
* Infinity * Infinity = Infinity

#### Divide / 

* Either operand is NaN, returns NaN
* 0/0 -> NaN ,     finite Number / 0 -> Infinity or -Infinity
* Infinity / Infinity -> NaN , Infinity / any number -> Infinity or -Infinity (Infinity / 0 => Infinity)

#### Remainder(Modulus) %

```js
console.log(26 % 5 ) //1
Infinity % 2    //NaN
2 % 0           //NaN
Infinity % Infinity //NaN
2 % Infinity    // 2
0 % 2           // 0 
```

#### Exponential **

```js
Math.pow(3,2)
let a = 3
a **2        //both are 9
a **= 2      //a = a ** 2 = 9
let b = 16
b **= 0.5   // b = b ** 0.5 = 4
```

#### Additive Operators

```js
Infinity + -Infinity  //NaN   SPECIAL CASE
Infinity + Infinity  //Inifnity
-Infinity + -Infinity //-Infinity
+0 + -0            //+0
let result = 5 + "5"
//result -> 55
```

**If only one operand is a string, the other operand is converted to a string** and the result is 

the concatenation of the two strings.

```js
//Common Error,String(the sum is) convert num1 to a string , and the string (the sum is 5) convert num2 into a string too
//Each addition is done separately.
let num1 = 5 , num2 = 10
console.log("The sum is " + num1 + num2)    //The sum is 510
//Solution
console.log("The sum is " + (num1 + num2))
```

#### Relational Compare

```js
let result = "Brick" < "alphabet"     //true,True answer but wrong process
"Brick".toLowerCase() < "alphabet"    //false. Same start up.  b compares with a , which means 98 < 97 is false.
//All the same with strings
let result2 = "23" < "3"        //true. Because 3 is greater than 2.
let result3 = "23" < 3          //false,which is the right answer.
let result4 = "a" < 3           //NaN, "a" can't be converted into a number.
```

**Whenever a number is compared with a string , the string will be converted into number.**

#### Equality Operators

Equal and not equal , perform type conversion before doing comparisons

Identically equal and identically not equal, no types conversions performed.

Boolean / String  ->converted into a number 

Object, valueOf() / toString() / Numebr()

`If both operands are objects , check if they point to the same object (same space storing the object)`

```js
let num = (1,2,3,4,5,8,4)    //num =4 , always return the last value in the set
```

### Statements

do-while , post-test loop , the body of loop is at least executed once.

while, pre-test loop. The body of loop may never be executed.

for , combination of pre and post - test loop.

for(initialization ; expression ; `post-loop-expression`) { statement }

Question: Why you have to use variable declaration keyword inside the for loop,even it's not useful outside the loop?

Answer:To limit the iterator to only 'for' loop scope itself.

```js
for(i=0;i<5;i++){}  //this is valid even i is not declared
for(let j = 0;j<5;j++){}      //This is recommended.
```

```js
//for in statement: to enumerate non-symbol keyed properties of an object.Orders are unpredictable.
for(const property in someObject){console.log(property)}
//for of statement:to enumerate values of sth.
for(const values of someArray){console.log(values)}
//in for index
//of for values
```

**for in enumerate properties/index,types are string** 

**for of enumerate values**

*for of cannot be used on an object, objects' values cannot be iterated directly.*

#### Label statements

```js
start: for(let i = 0 ;i<5;i++){}
//start can be refered later by break or continue (??)
//typically used on nested loops.
```

#### Loops operators.

break : exit the whole loop.

continue: exit the loop immediately , but will continue executing the loop from top

```js
let num1 = 0;
for(let i = 1;i<10;i++){
    if(i % 5 == 0){
        break
    }
    num1++
}  //num1=4
let num2 = 0;
for(let i = 1;i<10;i++){
    if(i % 5 == 0){
        continue
    }
    num2++
} //num2=8
let num3 = 0;
outermost:
for(let i =0 ; i<10;i++){
    for(let j = 0;j<10;j++){
        if(i==5&&j==5){
            break outermost
        }
        num3++
    }
} //55
let num4 = 0;
outermost:
for(let i =0 ; i<10;i++){
    for(let j = 0;j<10;j++){
        if(i==5&&j==5){
            continue outermost
        }
        num4++
    }
} //95
```

#### with statement

```js
with(location){
    var qs = search.substring(1);
    var hostName = hostname;
    var url = href;
}//let is not recommended using here
//same as the following
let qs = location.search.substring(1);
let hostName = location.hostname;
let url = location.href;
```

#### switch statement

* no value constrains for cases.

* Using identically equal , meaning that 

  ```js
  switch("55") {
      case 55: 
          console.log('matched!')
          break;
      default:
          console.log('not matched!')
  }
  //this will output not matched, for "55" === 55 returns false.
  ```


### Primitive and reference values / Execution context / Garbage collection

**A variable is literally just a name for a particular value at a particular time**

```js
let a = "This is a String"
a.name = "test" 
console.log(a.name)       //no error, but this will output undefined

let b = new String("A second string")
b.name = "test2"
console.log(b.name)  //test2
```

**When an argument is passed by reference, the location of the value in memory is stored into a local variable, which means that changes to the local variable are reflected outside the function**

local variable is also called "named parameters"

```js
function setName(obj){
    obj.name = "Nicholas"
}
let person = new Object()
setName(person)
console.log(person.name) //"Nicholas"
```

When person is passed into the function , it may look the same like the following:

```js
obj = person  //person is the global object outside the function scope.For understanding reason.
obj.name = "Nicholas"
```

Passing person as a parameter means obj and person point to the same object that exists globally on the heap.

To prove objects are passed by value, consider the case:

```js
function setName(obj){
    obj.name = "Nicholas"
    obj = new Object()   // Relations between obj and person  are broken in this line.
    obj.name = "Greg"
}
setName(person)
console.log(person.name)  //Output Nicholas but not Greg.
//obj is reassigned to the other object in the funcion.
```

#### Determining types

Use `typeof` to differentiate primitive types, and use `instanceof` to concretely differ types of objects.

typeof -> "string" , "number" , "boolean" ,"undefined" , "function" , "object"

Using typeof on null will return "object" , and RegExp objs / arrays / Dates ... will all return "object"

```js
colors instanceof Array
pattern instanceof RegExp     //Get concrete types of object
//Other ways to dectect data types:
//Object.prototype.toString.call()
//sth.constructor === Array
```

#### Execution Context

> The execution context of a variable or function defines what other data it has access to, as well as how it should behave.

* Variable Object:Scope Chain , Activation Object

* Global Context

* Functions Context  , using stack for maintenance

* Scope Chain. This is a variable object.

* Activation Object, This is a variable object too, if the context is a function.This starts with a single defined variable `arguments`

  > The next variable object in the chain is from the containing context, and the next after that is from the next containing context(??)

#### Scope Chain Aumentation

Using catch and with statements. 

`catch` will create a new variable object to the front of the scope chain.

`with` will add the specified variable object to the front of the scope chain.

```js
function buildUrl(){
    let qs = "?debug=true";
    with(location){
        var url = href + qs  //'let' is used in the book, which outputs an error,cannot get reference to the "url"
    }
    return url
}
//Here, specified variable object location is added to the scope chain.
```

#### Garbage Collection

**Two ways:**

1. "mark-and sweep": mark the in-scope variables , sweep them once the function is finished

2. "Reference Counting":Keep track of how many references are made to the value, once count is zero , collect it.

   **Problem: Circular Reference**

   ```js
   function problem(){
       let a = new Object()
       let b = new Object()
       a.sth = b
       b.sth = a
   }
   //This will never be collected once ran, and memory may be filled up if run it for countless times
   //Solution: clean the circular reference:
   a.sth = null;
   b.sth = null;
   ```

   

### Three ways to use undeclared variables

```js
typeof sthUndeclared     //undefined
let result = (false && sthUndecalred) //no error, short-circuited.
function test(){
    sthUndeclared = 2        //Not in strict mode , this will be automatically added to window, window.sthUndeclared.
}                       //?undefined?
```

