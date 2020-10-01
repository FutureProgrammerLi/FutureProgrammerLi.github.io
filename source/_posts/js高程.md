---
title: BOM
date: 2020-04-28 22:12:16
tags: Javascript高级程序设计
excerpt: BOM中的location&history对象
categories: 
- 书本
---
## location对象
* 提供与当前窗口加载的文档有关的信息
* URL解析, 导航功能
* window.location , document.location 引用的是同一个对象

属性列举(由长到短,由左到右)
1. href: 返回`完整URL`,包括哈希值. location.toString方法也返回这个值
2. origin: 协议名和主机名 (https://www.baidu.com)
3. protocol: 页面使用的协议, 一般为http / https
4. host: 主机名和端口号(如果有的话,否则仅返回主机名)
5. hostname: 主机名
6. port: 端口号
7. pathname: 请求路径 /user/xxx
8. search: 请求参数, 包括问号
9. hash: 哈希值, 包括井号

**修改任意一个值都可以实际操作URL,发出HTTP请求**

方法列举:
1. assign([URL])  **必须有http, www可以没有**
- location.assign("http://baidu.com")
等价于: window.location = "http://baidu.com"
       location.href = "http://baidu.com"

2. replace([URL])
位置改变而不生成新纪录,无法后退到被替代的页面.

3. reload() 接收true/false为参数,默认为false
重新加载当前页面. 设置为true,则强制从服务器重新加载.
                设置为false,则可能会从缓存中重新加载(没有变化的情况下)

## History对象
属性: 
* length: 历史记录的数量,是个number.

方法:
1.go([String/Number]): 
- Number: 正数是向前,负数是向后退
- String: **跳转到历史记录中包含该字符串的第一个位置** 
*Chrome里面字符串无效?(用的H5 history)*
2.forward() / back() 向前进或后退. 接受负数,但自动转为正数(?)
