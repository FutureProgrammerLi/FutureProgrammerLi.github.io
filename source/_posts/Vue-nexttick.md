---
title: Vue-nexttick
date: 2020-05-19 16:42:04
tags: Vue
categories:
- Vue
excerpt: Vue.$nexttick的理解
---

### $nextTick

#### 初步了解顺序：

1.执行主线程同步代码 (==> 2.将异步队列的任务推入栈执行 ==>3.执行nextTick里的异步函数)

​                                    (==> 再查找异步队列并执行 ===> 再执行新的nextTick()                 )

DOM更新可能是在2和3之间,即执行完异步队列的任务后更新DOM,所以nextTick可以对更新后的DOM进行操作.

 Vue的异步渲染怎么理解?

主线程中执行修改DOM的操作=>在2中渲染=>则3为更新完的DOM展示

#### 应用场景（使用原因）：Vue中的DOM是异步渲染的.

1. 在created里对DOM进行操作(???),还没挂载,直接操作是无用的,需要使用Vue.nextTick()里的回调函数.

2. 在mounted里也可能用nextTick, 父组件不确保子组件都已被挂载

   **Question:?????不是子组件都已经mounted才会触发父组件的mounted吗????**

   Answer: 同步加载才是.异步加载子组件时有可能父组件先mounted

3. 需要对数据更新之后的DOM进行操作.

   (默认用的Promise和MessageChannel,如果不支持则转用setTimeout() )

#### 例子

```html
<div class="app">
  <div ref="msgDiv">{{msg}}</div>
  <div v-if="msg1">Message got outside $nextTick: {{msg1}}</div>
  <div v-if="msg2">Message got inside $nextTick: {{msg2}}</div>
  <div v-if="msg3">Message got outside $nextTick: {{msg3}}</div>
  <button @click="changeMsg">
    Change the Message
  </button>
</div>
```



```js
new Vue({
  el: '.app',
  data: {
    msg: 'Hello Vue.',
    msg1: '',
    msg2: '',
    msg3: ''
  },
  methods: {
    changeMsg() {
      this.msg = "Hello world."
      this.msg1 = this.$refs.msgDiv.innerHTML  //DOM还没更新时的msg
      this.$nextTick(() => {
        this.msg2 = this.$refs.msgDiv.innerHTML //DOM更新之后的msg
      })
      this.msg3 = this.$refs.msgDiv.innerHTML
    }
  }
})
```

点击按钮之前:

![img](https://upload-images.jianshu.io/upload_images/3985563-b6bb266285e8d232.png?imageMogr2/auto-orient/strip|imageView2/2/w/152/format/webp)

点击之后:

![img](https://upload-images.jianshu.io/upload_images/3985563-f49bff3190724514.png?imageMogr2/auto-orient/strip|imageView2/2/w/341/format/webp)

体现:在同一个方法里修改某个值,且有多个值依赖于某单一值(即msg1,msg2,msg3依赖于msg)

​         