---
title: jquery
date: 2020-05-27 19:27:08
tags: 
- jQuery
categories:
- jQuery
excerpt: jQuery学习笔记
---

## jquery

1. $符号相当于jQuery对象, $.get / $.ajax 等相当于jquery.get / jquery.ajax 

就是调用jquery这个对象的方法.

$('h1'),将选择器传给jquery.  相当于将被选择的元素(?)加工成jquery对象

2. $(document).ready(fn)和$(window).on("load",fn)

   $(document).ready(fn)可以缩写成(fn)

   先执行ready,再执行onload

   ready对应原生DOMContentLoaded事件. onload对应load事件.

   **ready代表仅DOM渲染完成,load代表DOM,CSS,JS都加载完成.**

3. Alias for jQuery to avoid  same meanings for $

   ```js
   //将$让给其它框架
   var j = jQuery.noConflict();
   $j(document).ready(fn)
   //强行要用
   jQuery.noConflict();
   jQuery(document).ready(function($){
       //这里可以用$代表jQuery,出去了就不行了.
   })
   (function($){
       //...       同理
   })(jQuery)
   //或者
   jQuery(function($){
       //... $代表jQuery,外面可能不是
   })
   ```

4. attr()既是getter也是setter,一个值为getter, 属性名+值或者对象则为setter(可以同时获取多个属性吗?)  **仅针对attr()这个方法,区别于其它方法**

5. 选择器

   ID,类,属性,复合CSS选择器(用CSS选择器的写法传给$(''))

   CSS里面能用的,jq基本都能

   判断选择器是否包含元素

   ```js
   //错误
   if($('div.foo')){
       //..           //无论如何都会被执行,$返回一个对象.对象转Boolean类型不为空.
   }
   //正确
   if($('div.foo').length){
       //...          //正确的判断
   }
   var divs = $("div")     //获取执行这条语句时页面上的所有div,
                           //在这句之后新增的div不包含在divs里面(doesn't magically update.)
   ```

   获取表单元素:

   $('form :checked') ==>获取被选取的项

   Works for checked checkboxes /checked radios / \<select> element

​              $('fom :disabled/:enabled')=>获取表单不能填写的/能填写的项

​              $('form :input') =>获取所有input,如text,textarea,select(?),button

​              $('form :selected') =>获取\<select>里被选择的option

​             也可以按类型获取$('form :password/:file/:image...')

6. .html()方法不传值就是getter获取值,传字符串就是设置选择的所有的元素InnerHTMLText为该值

   getter时,只返回选择到的第一个元素;setter时,设置所有选到的元素

   * get就get第一个的值,set就set所有

     类似还有.width/.height()/.position()  *getterOnly*    /.val()

7. 移动元素:insertAfter() / insertBefore() / appendTo() / prependTo()  去掉介词也有对应方法,主宾不同而已.

   复制元素: .clone()       //.clone(true)深复制? 数据和对应事件也会被复制

   ```js
   // Copy the first list item to the end of the list:
   $( "#myList li:first" ).clone().appendTo( "#myList" );
   ```

​      删除元素: .remove() / .detach()      保不保留数据和事件的区别,前者不会,后者会.用于restore.

​                      (empty()方法将元素InnerHTML内容清空.)

​      创建元素:

```js
// Creating a new element with an attribute object.
$( "<a/>", {
    html: "This is a <strong>new</strong> link",
    "class": "new",
    href: "foo.html"
});
//创建了不代表在页面上就能看见 还要append/insert等操作dom
var myNewElement = $( "<p>New element</p>" );
myNewElement.appendTo( "#content" );
myNewElement.insertAfter( "ul:last" );
```

* 不要多次适用append,insert等操作,性能极差.

做法:

```js
var myItems = [];
var myList = $( "#myList" );
for ( var i = 0; i < 100; i++ ) {
    myItems.push( "<li>item " + i + "</li>" );
}//Concatenate all the HTML into a single string, and then append that string to the container
myList.append( myItems.join( "" ) );
```

**TODO: The jQuery Object**