---
title: 逐行解析vue源码-5
date: 2020-06-08 17:52:02
tags:
- Vue
- 源码
categories:
- Vue
- 书本
- 整理
excerpt: 全局API
---

### 1.Vue.extend({options})

扩展Vue实例,以Vue类为父类,自定义创建子类Sub.

参数options就是普通vue文件中,export default的那个对象.包括data/methods/computed等等.

extend里的data必须是一个函数(? 普通实例里不也是?)

```js
Vue.extend({
    data:function (){
        return {
            count:0
        }
    }
})
//vdata生成的.     ???一个es6一个es5????
export default {
    data(){
        return {
            count:1
        }
    }
}
```

### 2.Vue.directive(id,Function/Object)

自定义或获取命令. 只传id不传定义就是返回该指令.两个参数都传就是自定义新的指令.

```js
//第二个参数为对象
Vue.directive(id,{  //挖坑:这些函数是怎么实现的?在哪里注入呢?盲踩beforeCreated
  bind:function(){},
  update:function(){},
  inserted:function(){},
  componentUpdated:function(){},
  unbind:function(){}
})
//第二个参数为函数
Vue.directive(id,function(){
    //...   这里默认会绑定给对象形式的bind和update.
})
Vue.options.directives  //存放指令的位置
```

### 3.Vue.filter(id,Function)

同理于directive. 只传id表示获取过滤器, 两个都传表示定义过滤器.

Vue.options.filters. 存放过滤器的地方

### 4.Vue.component(id[,definition])

全局注册组件,或获取组件. 看传不传入定义.

```js
Vue.component('my-component',function(){})
Vue.component('my-c',Vue.extend({}))
var my = Vue.component('my-component')
//挖坑:和Vue.extend的区别? 局部注册组件? 目前挖了4个坑.还有2个是directive
//注册组件的三种方法
//全局:
Vue.component('my1',Vue.extend({/*definition*/})
Vue.component('my2',{/*definition*/})
//局部
var my3 = {/*PlainObject*/}
export default {
    components:{
        my3
    }
}
Vue.options.components //放组件的地方
```

* 总结component/filter/directive 
  1. 都放在Vue.options里面
  2. 仅传第一个参数,表示获取.两个都传,表示注册.directive特殊,第二个既可是函数也可是对象.

* 实际是写成一起的. 只是为了方便理解(!!!!????)

### 5.Vue.use(plugin)

plugin可以是函数,也可以是对象.

**实际是调用插件提供的install方法,**传入函数,则该方法即视为install方法;传入对象,则必须定义install方法.

插件只能被注册一次.用installedPlugins数组来维护,执行完install之后就push进这个数组里.

1.是否已安装 2.调用函数,或对象的install方法. 3.进入已安装的数组//有坑,传两个参数时???

### 6.Vue.observable(object)

将自定义对象变得可响应式,实际就是调用得observe方法.

等价于vue3 的reactive()?

### Vue.extend和Vue.component的区别

extend返回一个父类是Vue的子类Sub. Vue.component则注册了一个组件

extend的使用:

```js
let Sub = Vue.extend({
    template:'<div>{{count}}</div>',
    data:function(){
        return {
            count:0
        }
    }
})
new Sub().$mount('#root')

//一般的main.js中
new Vue({
    store,
    router
    render:h=>h(App)
}).$mount('#app')
```

Vue.component的坑:第二个参数到底是什么?除了对象,为什么还可以是Vue.extend({})?

```js
Vue.component('my-component',{      //正常对象,说是默认调用了Vue.extend({})
    template:'<div>111</div>'       //将第二个参数对象放进了Vue.extend里
})
Vue.component('my-com2',Vue.extend({  //???
    template:'<div>222</div>'
}))
```



---

### 过滤器

fA,fB => filterA ,filterB 

`两种用法 <input v-bind=" a |filter"/>     或者\{\{a | fA | fB(argument)}}`

`写成函数形式就是 _f("fB")(_f("fA")(message), argument)`

 `简化下来就是 f(B)(f(A)(msg))       f(A)(msg) => {\{msg|fA}}`

expression = msg,

filters = [ 'fA','fB']

filterA(msg)      -> filterB(fA(msg))

`判断 是否遇到| (ASCII码是7C)`

---

### keep-alive内置组件

抽象组件(abstract:true),不出现在实际DOM中,只有export default而没又template,style(内置的函数式组件(?))

接收3种props : include, exclude, max(vue2.5+)

include/exclude又接受三种形式参数：字符串(单个)/数组(多个)/字符串(不定个数)

匹配原则:最先是组件options内的name,之后是局部注册时的名字(components:{'component-a':ComponentA}). 匿名组件不能被缓存

```js
this.cached = {
    'key1':'component1',
    'key2':'component2',
    'key3':'component3'
}
this.keys = ['key1','key2','key3']
//cached是已缓存的组件,include/exclude就是拿着key去校验keys数组中是否已经存在.
//include->match, exclude->!match
//max的原理就是校验数组的长度
```

* keys数组的维护利用Least Recently Used(LRU)策略

1. 新组件进入,数组长度>max,则keys[0]出队//默认keys[0]是最近最少使用的那个组件
2. key在keys数组中匹配,则将数组中的对应key删除,再push一次(放入队列尾部)
3. 数组小于max,未被缓存过,则直接push

* 内部有自己的生命周期(created,mounted,render,detroy)

  重点是render,获取keep-alive内的第一个组件的name/tagName;命中/不命中缓存,实现LRU;打上标志vnode.data.keepAlive=true

  created创建cached对象和keys数组.destroy销毁组件,清空cached对象.mounted观测include和exclude的变化

* 独有的钩子函数activated和deactivated.第一次mounted和activated一起触发.组件切换之后触发deactivated,再切回来触发activated.

---

### 自定义指令

**VDOM渲染时有自己的生命周期,directives就是在其中绑定对应的操作**

VDOM的生命周期:                 

1. init
2. create
3. activate
4. insert
5. prepatch
6. update
7. postpatch
8. destroy
9. remove

指令就是在合适的时期执行定义时所设置的钩子函数.主要是create,update和destroy.

destroy和remove的区别是: (被动移除)父元素被移除导致自己被移除时,destroy会被触发而remove不会.(主动移除)自己被移除的,则两个都会被触发.

```js
Vue.directive('focus',{
    bind:function(el,binding,vnode){},  //el是对应元素,binding是指令的相关信息(name/value/modifier等),vnode就是使用指令对应的元素vnode
    inserted:function(el){
        el.focus()
    },
    update:function(){},         //这个和componentUpdated钩子有独特的第四个参数,oldVnode
    componentUpdated:function(){},
    unbind:function(){}
})
//使用<input v-focus >
```

* 循环遍历oldDir和newDir时,前无后有,则执行bind

* 虚拟DOM的insert和指令的inserted合并触发. postpatch和componentUpdated合并触发

* 同循环oldDir和newDir,前有后无,执行unbind

* 调用update函数时意义未明,保存了旧指令的值和参数(??) 

  ```js
  dir.oldValue = oldDir.value
  dir.oldArg = oldDir.arg
  callHook(dir,'update',vnode,oldVnode)
  ```

  

