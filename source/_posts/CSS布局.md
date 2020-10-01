---
title: CSS布局
date: 2020-05-04 16:00:28
tags:
- CSS
categories:
- 整理
excerpt: 两列布局&三列布局(圣杯布局.双飞翼还没写。)
---

## 1.两列布局

### 方法一、左浮动+右margin-left:

```css
.left, .right{
    height:600px;
}
.left{
    width:400px;
    background-color:aqua;
    float:left;      /*左浮动*/
}
.right{
    margin-left:400px; /*左边距,必须知道左边宽度*/
    background-color:red;
}
```

### 方法二、左浮动+右overflow:hidden

```css
.left{
    width:400px;
    background-color:aqua;
    float:left;
}
.right{
    overflow:hidden;
    background-color:hotpink;
}
```

### 方法三、flex布局（比较好,半自适应,右边不会被隐藏)

```css
/*添加父容器parent*/
.parent{
    display:flex;
    height:100%;
}
.left{
    width:400px;
    background-color:aqua;
}
.right{
    flex:1;
    background-color:hotpink;
}
```



## 2.三列布局

### 方法一、 float+overflow：hidden

* margin当然也可以,但需要知道左边两个的宽度和

```css
/*同两列布局的第二种方法类似,float+overflow:hidden*/
.left{
    width:400px;
    float:left;
    background-color:red;
}
.center{
    width:200px;
    float:left;
    background-color:yellow;
}
.right{
    overflow:hidden;
    background-color:green;
}
```

### 方法二、flex布局

```css
.parent{
    display:flex;
    height:600px;
}
/*三列自适应,可以为某一部分添加width,其它flex:1*/
.left{
    background-color:aqua;
    /*width:400px;*/
}
.center{
    background-color:green;
}
.right{
    background-color:red;
}
.left, .center, .right{
    flex:1;
    height:600px;
}
```

### (1)圣杯布局

* 两侧固定宽度,中间宽度自适应

### 方法一、 flex+两侧等宽(是吗?)

```css
.p{
    display:flex;
    height:100%;
}
.l{
    width:200px;
    background-color:red;
}
.r{
    width:300px;
    background-color:green;
}
.c{
    flex:1;
    background-color:yellow;
}
```

### 方法二、浮动+margin

```css
/*HTML中.right元素需要放在.center元素之前*/
/*如果右侧元素在中间元素后面，由于浮动元素位置上不能高于（或平级）前面的非浮动元素，导致右侧元素会下沉。中间比较核心，放在比较后的位置，不利于SEO(?)*/
/*左侧宽度大于中间时,中间元素会被下移*/
.l, .r, .c{
    height:100%;
}
.l{
    width:200px;
    float:left;
    background-color:aqua;
}
.r{
    width:300px;
    float:right;
    background-color:aqua;
}
.c{
    margin:0 auto; /*知道左右宽时可以用margin-left/right*/
}
```



### 方法三、父容器用margin,左右均浮动,利用定位和margin移动到正确位置(巨麻烦)

```css
.wrap{
    margin:0 auto;
}
.left, .center, .right{
    height:600px;
    float:left;
}
.left{
    width:400px;
    background-color:aqua;
    position:relative;
    margin-left:-100%;
    left:-400px;
}
.center{
    width:100%;
    background-color:blueviolet;
}
.right{
   width:400px;
    background-color:brown;
    position:brown;
    position:relative;
    margin-left:-400px;
    right:-400px;
}
```





