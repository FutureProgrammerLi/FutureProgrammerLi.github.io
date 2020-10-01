---
title: 逐行解析vue源码-4
date: 2020-06-08 00:52:59
tags:
- Vue
- 源码
categories:
- Vue
- 书本
- 整理
excerpt: Vue的实例方法($mount,$set,$delete,$emit,$forceUpdate等)
---

### 1.数据相关

1. Vue.watch(target,callback,options)

   target为数组或对象,不能是原始值、vue实例或vue根数据(?)

   callback和options可以整合成对象

   ```js
   {
       handler:function test(){},
       immediate:true,
       deep:true
   }
   //内部会自动将其抽离(可以同时传两个option吗?)
   ```

   immediate实现就是立即利用callback,call(vm,watcher.value)

   deep深度遍历每个属性值,触发每个属性值的get(),从而将这个watcher添加到每个值的dep里面.某个值变化之后就会通知这个watcher.

2. Vue.set(target,propertyName/index,value)

   判断:target为原生值/vue实例/vue根数据? 警告.返回

   ​        target为数组?

   ```js
   target.length=Math.max(target.length,key) //用户传进来的index小于数组原长度,则数组长度+1.index大于原长度,则修改总长度为key.
   target.splice(key,1,val)              //在index处插入一个val项
   //splice方法已被截取,所以它是可被观测的.
   ```

   ​      target为对象?

   ​      该属性已存在? 直接修改值

   ​      该属性不存在?对象是响应式的吗?是,defineReactive; 不是,直接修改值

3. Vue.delete(target,propertyName/index)

   与set相似.为数组且索引有效? target.splice(index,1)

   为对象? delete target[key] 可观察对象,(有\_\_ob\_\_属性??target.dep.notify()    : 不是?直接return.

   原生的delete用在数组上,为将该项设置为empty. 而Vue.delete用的是splice,被劫持过的方法,所以结果比较符合预期.

   ```js
   let a = [1,2,3]
   let b = [4,5,6]
   delete a[1]   //[1,empty,3] 实际只有两项,但length仍为3.且a[2]仍然为3
   Vue.delete(b,1)   //[4,6]   b.length为2
   ```

//TODO 事件相关方法和生命周期相关方法

on/emit/off/once

$mount/$forceUpdate/$nextTick/$destroy

### 2.事件相关方法

1. vm.$on()

   接受两个参数,事件名称,回调函数.

   事件名称可是字符串,可是字符串数组,表示多个事件触发同一个回调函数.

   vm._events管理,on就是往里添加

   ```js
   vm.events={
       eventName:[cb1,cb2,cb3]  //...
   }
   ```

2. vm.$emit(eventName:string , [...args]) 

   第一个参数是事件名,第二个开始是传给该事件的参数.

   坑:$emit传输3/4/5个参数给事件池.官方介绍是封装成对象.

   ```js
   const args = toArray(arguments,1)
   cbs[i].apply(vm,args)
   ```

   ```js
   //子组件
   this.$emit('test',this.param1,this.param2,this.param3)
   //父组件
   <child @test="test($event,userDefined)"
   test(e,userDefined){
       console.log(userDefined[0],userDefined[1],userDefined[2])  //对应param1/2/3
   }
   ```

3. vm.$off([event,callback])

   ```js
   vm._events ={       //举例,可能不准确
       event1:['f1','f2']
       event2:'f3'
   }
   ```

   三种传参形式:

   不传,清空_events.全部删除.

   仅传event,删除对应属性,即删event1或event2这些属性名.对应删除.

   两个都传,删除对应event下的callback.精准删除.

4. vm.$once(event,callback)

   $on的一次性版本.

   原理:劫持了原来的callback.once内部使用了$on, 但这个$on方法是在内部重新定义过的,跟全局的on不一样.

   ```js
   Vue.prototype.$once = function(event,callback){
       function on(){
           vm.$off(event,callback)   //用一次就删.还先删后用(!??)
           callback.apply(vm,arguments)
       }
       on.fn = fn
       vm.$on(event,on)
       return vm
   }
   ```

### 3.生命周期相关方法

1. vm.$forceUpdate()

   迫使当前实例更新.仅影响当前实例及有slot的子组件.并不是所有子组件都会被更新.

   原理: 一般的数据变化导致实例刷新过程: 1. 收集依赖 2.触发watcher的update方法.

   $forceUpdate就是直接拿第二个步骤来用.直接触发watcher的update方法.(vm._watcher.update())

2. $mount,$nextTick,$destroy 见其它章节.