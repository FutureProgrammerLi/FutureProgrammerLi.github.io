---
title: 逐行解析vue源码-2
date: 2020-06-04 10:02:51
tags:
- Vue
- 源码
categories:
- Vue
- 书本
- 整理
excerpt: Virtual DOM和模板编译
---

### 看完书后自己理解的内容记忆:

#### 虚拟DOM篇

> 为什么需要Virtual DOM？

1. 真实DOM属性十分之多,抽象成Virtual DOM,取一些必要的属性能简化操作,以JS计算性能换取DOM操作性能

2. VNode实际渲染需要对应环境操作,为此可以针对不同环境设置一套真实挂载的操作,从而提高代码的兼容性

3. 渲染优化,标记静态节点,在下次遍历时直接跳过静态节点(isStatic)

   ```js
   //如在H5中创建元素
   let VNode = nodeOps.createEle(...);
   //这里的nodeOps对应的就是document.createElement()
   //更换环境就可以引另一套nodeOps, 编程wx/uni....createElement()
   ```

> VNode种类有哪些?

1. 文本节点(只有text?)
2. 注释节点(isComment+text)
3. 元素节点(tag)
4. 组件节点
5. 函数组件节点
6. 克隆节点(isCloned)

各种节点的特有属性:

文本节点只有text属性, 注释节点特有的(isComment),元素节点特有的tag.

组件节点componentOptions/componentInstance.  

函数式组件节点fnOptions/fnContext(组件节点有的它也有,但这两个属性是函数式组件节点特有的)

克隆节点:isCloned

---

**主要渲染的节点只有三种: 文本节点,注释节点和元素节点. (绝大多数是元素节点)**

> DIF算法及优化(?)

理解上可以有3种DOM树.  oldTree,newTree,以及真实的DOMTree.

(oldTree和真实DOM区别是节点的真假,结构一样吧?)

* 先修改oldTree,之后一顿操作,变成newTree结构之后,实际渲染到DOMTree.       

* oldTree和newTree的作用是找出区别,DIF完成后,newTree和DOMTree结构一样.
* 节点操作主要有4种:创建新节点,删除旧节点,更新节点,移动节点位置(后两种好像一样?)
* **在所有未处理的节点前插入/创建新节点,而不是在所有已经处理的节点后插入/创建新节点**

> 以下是子节点更新操作

优化之前,两重循环: newChildren外层循环,oldChildren内层循环.效率明显不够.

优化策略:用四个关键位置,newStartIndex,newEndIndex  ||  oldStartIndex,           oldEndIndex.

​            分别表示 新树开始索引(新开)   新树结束索引(新尾)   旧树开始索引(旧开)    旧树结束索引(旧尾)

对比策略: 先新开和旧开对比,之后新尾和旧尾,新尾和旧开,新开和旧尾按顺序来.最后都找不到,就双重循环.

(建议复习,自己都不知道在说什么)

**总之就是两边向中间靠拢,开始的只会往后,结尾的只会往前.**

StartIndex>EndIndex 表明遍历完毕.

新树先遍历完毕,则旧树oldStartIndex到oldEndIndex的节点都删除.

旧树先遍历完,则新树newStartIndex到newEndIndex的节点都添加.

----

#### 模板编译篇

1. 生成语法抽象树,优化编译,生成render函数.

   ```js
   const ast = parse(template,options)  //生成抽象语法树
   if(!options.optimize){               //以下两个的操作对象都是AST
       optimize(ast,options)            //节点优化
   }
   const code = generate(ast,options)   //生成渲染函数
   return{
      ast, //抽象树
      render:code.render,
      staticRenderFns:code.staticRenderFns
   }
   ```

   ?三种解析器,HTML parser>文本解析器(parseText)> 过滤器解析器(parseFilter)

   HTML中有文本         ->               文本中有过滤器      ->  

   (真的吗? 源码里parseText和parseFilter好像没关系呀?)

HTML解析模板时,怎么区分标签是真正的开始,而不是嵌套的开始?结尾怎么区分是真正的结尾,而不是子元素结尾? 怎么分辨是不是用户语法出错? (以<template> 和</template> 作为最后底线?)

```js
parseHTML(template,{
    // ... static options
    //解析到开始标签时执行start
    start(tag,attrs,unary){  //tag标签名称start[1],attrs标签属性,如class/id/src等
                             //unary是否为自闭和标签,<div></div> | <img src="..."/>
        let element = createASTElement(tag,attrs,currentParent) //?
    },
    //解析到结束标签
    end(){
        
    },
    //解析到文本
    chars(text){
        //分动态文本和静态文本.动态文本就是包含变量的文本内容. 
        //  动态文本:Hello, {{name}} |静态文本:Hello
        //动态文本的NodeType为2.  静态纯文本的type为3 (元素节点的type为1?)
        if(动态文本){
*******************************************************************************
            let element = {   //动态文本的结构.记住有助于后续理解
                type:2,
                expression:'我的年龄是'+_s(...) +'我的姓名是:' + _s(...),
                tokens:['我的年龄是','@binding=age','我的姓名是:','@binding=name'],
                text
            }   *******************************************************************************
        }else{
            let element = {
                type:3,
                text
            }
        }
    },
    //解析到注释
    comment(text){
        let element = {
            type:3,
            text,
            isComment:true    //与文本节点的差别就是isComment.参考VNode种类
        }
    }
})
//四个钩子用于生成具体的AST(start,end,chars,comment)
```

2. 实际parse的过程:

   主要解析的内容主要有三类:

   (1) \<!--   开头的: 

   自定义注释(<!--这是注释-->,   \<!--开头,-->结尾

   条件注释<!--[if !IE]> <h1>这不是IE浏览器</h1>  <![endif]--> \<!--[,   ]-->结尾

   //自定义注释和条件注释是否有关系?判断自定义注释的时候不就包括了条件注释了吗?

   (2)DOCTYPE,  <!DOCTYPE html>

   (3)开始和结束标签..

   (4)文本.(绝大多数)

* 上面(1)所述操作有优化点,options.shouldKeepComment,为false的话则不为注释创建节点,直接advance(commentEnd +3).

* \<template comments> \</template>可以设置是否保留注释.默认true还是false?

  ```js
  //(1)自定义注释的正则
  const comment = /^<!\--/ //   <!--我是注释-->
  //(1)条件注释的正则
  const conditionalComment = /^<!\[/    //??? <!--[if IE] ...  <![endif]-->
  //(2)DOCTYPE的正则
  const doctype = /^<!DOCTYPE [^>]+>/i
  ----
  //(3)解析开始
  const ncname = '[a-zA-Z_][\\w\\-\\.]*'
  const qnameCapture = `((?:${ncname}\\:)?${ncname})`
  const startTagOpen = new RegExp(`^<${qnameCapture}`)
  //返回结果的样子,符合就从下标1取tagName,不符合就返回null           //第一个参数,是字符串
  //<div></div> => ['<div','div',index:0,input:'<div></div>']
  //sth</p>     => null
  
  //除了tagName,start函数还需要attrs和unary参数.需要分别解析,判断
  ```

  ```js
  // 属性是一个个被截取出来放到attrs数组里的.有多少个属性就要匹配多少次.
  const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/
  while (!(end = html.match(startTagClose)) && (attr = html.match(attribute))) {
   advance(attr[0].length) 
   match.attrs.push(attr)          //第二个参数,是数组
   //attr长这样:
   //["class="a"", "class", "=", "a", undefined, undefined, index: 0, input: "class="a" id="b"></div>", groups: undefined]
   //遇到动态绑定的需要传给parseText吧?
  }
  ```
  
  ```js
  //判断是否为自闭和
  const startTagClose = /^\s*(\/?)>/
  let end = html.match(startTagClose)
  if (end) {            //第三个参数,boolean?
   match.unarySlash = end[1]  
   advance(end[0].length)
   match.end = index
   return match
  }
  //示例
  '></div>'.match(startTagClose) // [">", "", index: 0, input: "></div>", groups: undefined]
  '/>'.match(startTagClose) // ["/>", "/", index: 0, input: "/><div></div>", groups: undefined]
  //区别是end[1],为空则非自闭合,为/则是自闭合
  ```
  
  * 解析结束标签和解析开始标签类似,是正则表达式startTagOpen变化成另一个正则endTag.截到则返回整个结束标签,截不到就返回null
  * 都不是直接调取start和end钩子函数,调用的是handleStartTag和parseEndTag,在里面再调取start和end函数
  * parseEndTag(tagName,start,end)  //三个均为可选参数,三个都传->正常处理|只传tagname | 三个都不传->处理栈中剩余的标签.

(4) 文本的截取:判断'<' 的位置,0则为之前3(5)种,大于0则将0到<的位置之间的东西截取出来,(必是文字?),之后再对<到剩余(?)的文字进行5种匹配,均匹配不到说明整段都是纯文本.



> 怎么截掉已经解析了的字符串?

使用advance(start[0].length)就相当于把tagName截掉了.使用游标(index)检索template字符串.

---

> AST的层级关系

* 用栈来维护,有两个作用: 1)匹配到正确的startTag和endTag之后才建立AST节点.2)匹配不正确的时候,给出提示(tag has no matching end tag)