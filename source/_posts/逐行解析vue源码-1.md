---
title: 逐行解析vue源码-1
date: 2020-06-02 23:35:53
tags:
- Vue
- 源码
categories:
- Vue
- 书本
- 整理
excerpt: Vue2.x 的响应式原理
---

### 1.变化侦测篇

UI = render(state)

用户所见 = 视图 | 网页数据 = state | Vue所扮演的角色 = render()

当state/UI发生变化时,通知Vue去更改视图或状态.

#### 1.1对象劫持

>  问题引入, 怎么通知Vue去修改UI/state呢?

vue 2.x Object.defineProperty 遍历劫持data数据.添加访问器属性

```js
Object.defineProperty(car,'price',[
    get(val){
      console.log('Get price')
      return val
    },
    set(newVal){
        console.log(`price is set to ${newVal}`)
        val = newVal  //哪来的val?
    }   
])
```

```js
export class Observer{
    constructor(val){
        this.val = val;
        def(val,'__ob__',this); //给val添加一个标记,代表该值已经被观测(observable)
        if(Array.isArray(val)){
            //...数组操作
        }else{
            this.walk(val) //只有object类型的才会walk(?原始值呢?直接defineReactive?)
        }
    }
    walk(obj:Object){
        const keys = Object.keys(obj);
        for(let i =0 ; i<keys.length;i++){
            defineReactive(obj,key[i])
        }
    }
}
//walk函数 ->负责遍历data对象.(为什么不用traverse表示?)
//walk里面又调用defineReactive.  //一个用于遍历,一个用于实际设置
function defineReactive(obj,key,val){
    if(arguments.length == 2){
        val = obj[key]
    }
    if(typeof val == 'object'){
        new Observer(val)
    }
    Object.defineProperty(obj,key,{
        get(){
            console.log('Get hacked')
            return val
        },
        set(newVal){
            if(val == newVal) return;
            console.log('Set hacked')
            val = newVal;
        }
    }
}
```

```js
//使对象变得响应式 vue2.x
let car = new Observer({
    price:20000,
    brand:'bmw'
})
//vue3.0
const car = reactive({  
    price:30000,
    brand:'benz'
})
```

---

> 知道数据变化了,哪些依赖需要变化呢? [(谁依赖了这变化了的数据) ? ]

依赖收集:  获取的时候添加依赖,设置新的值的时候通知依赖

vue 2.x 里 dep()添加依赖, notify()通知依赖更新.

vue3.x 里 track()添加依赖, trigger()通知依赖

Q:vue2用的什么结构存储依赖?vue3用的WeakMap,Map和Set.

A:数组.

```js
class Dep{
    constructor(){
        this.subs = []
    }
    //实际的添加依赖操作
    addSub(sub){
        this.subs.push(sub)
    }
    //解除依赖.remove好像有别的用处所以抽离出来?
    removeSub(sub){
        remove(this.subs,sub)
    }
    depend(){
        if(window.target){
            this.addSub(window.target)  //这是什么意思?
        }
    }
    notify(){
        const subs = this.subs.slice();  //为什么要修改拷贝的那一份?slice劫持?
        for(let i =0 ; i<subs.length; i++){
**************************************************************************
            subs[i].update()  //这里的subs是否被劫持过?都是Wathcer实例?
**************************************************************************
        }
    }
}
//depend和notify好像都不是实际操作,实际操作似乎是addSub()和update()
```

```js
//优化上面的Object.defineProperty.不单是console.log
const dep = new Dep()      //"依赖管理器"
Object.defineProperty(obj,key,{
    get(){
        dep.depend();
        return val
    },
    set(newVal){
        if(val == newVal) return;
        val = newVal
        dep.notify()
    }
})
//好像比vue3管理方式容易一点? 还是还未完全?
//如果我只是读取这个数据,就不用depend.但还是触发了get(),怎么处理?
```

---

> 依赖是什么东西? 上面的this.subs.push(sub) 这里的sub到底是什么?即数组的项是什么?

Watcher类      -->对应vue3.0的effect(? 存疑)

subs数组里放的是Watcher实例.一个依赖一个实例(?),由这个watcher实例通知具体依赖实现更新.

watcher类有很多不懂.主要是window.target的问题.

思路是watcher获取被依赖的数据,从而触发被依赖数据的get,以及之后的dep.depend从而添加到依赖.

```js
class Watcher{
    constructor(vm,expOrFn,cb){
        //...
        this.getter = parsePath(expOrFn)  //获取对象的值
        this.value = this.get()
    }
    get(){
        window.target = this;  //意义不明."把实例自身赋给了全局的一个唯一对象上"
        const vm = this.vm;
        let value = this.getter.call(vm,vm) //获取被依赖的值,触发getter
        window.target = undefined;  
        return value
    }
    update () {
    const oldValue = this.value
    this.value = this.get()
    this.cb.call(this.vm, this.value, oldValue)
  }
}
```

####  缺点

1. 需要遍历data对象并对数据进行劫持.当数据量庞大时效率不高.
2. 直接给vue实例添加的属性不是响应式的,需要使用Vue.set,Vue.delete方法使新添加的数据变得响应式.

# 挖坑,Vue.set和Vue.delete的原理

---

#### 1.2数组的劫持

思路: 新建一套数组操作,在新函数(newPush)里调用原生的操作(push等)

```js
const arrayProto = Array.prototype;
export const arrayMethods = Object.create(arrayProto)  //为什么是一个对象?

const methodsToPatch = [  //劫持可以修改原数组的操作
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse
]

methodsToPatch.forEach(method=>{
    const original = arrayMethods[method];  //orginal就是push/pop/shift等单个方法
    Obeject.defineProperty(arrayMethods,method,{
        // ...descriptors
        value: function mutator(...args){
            const result = original.apply(this,args)
            return result
        }
    })
})
//为什么调用push的时候是调用arrayMethods这一套?
// arr.__proto__ = arrayMethods    (???哪里有这个)
//目前还是单纯的劫持,还未进行depend/notify操作(?)
```

* 修改数组的原型方法:

  ```js
  class Observer{
      constructor(val){
          this.val = val;
  ****************************************************************************
          if(Array.isArray(val)){
              //检测是否支持__proto__属性
              const augment = hasProto?protoAugment : copyAugment 
              augment(val,arrayMethods,arrayKeys)
              //相当于arr.__proto__ = arrayMethods 
              //√ 劫持成功!?
              this.observeArray(val) //深度侦察,解决数组项是对象的情况
          }   ****************************************************************************
      }
  }
  function observeArray(items){
      for(let i = 0;i<items.length;i++){
          observe(items[i])    //递归变成响应式
      }
  }
  ```

> 通知数组依赖更新

```js
methodsToPatch.forEach(method=>{
    const original = arrayMethods[method]
    def(arrayMethods,method,function mutator(...args){
        const result = original.apply(this,args)
        const ob = this.__ob__
        ob.dep.notify()
        return result
    })
    //def是劫持getter,setter的一个专门函数(?)
})
```

## 不熟悉: 对象的添加依赖和通知依赖.不太明白怎么收集依赖和通知更新依赖

### 缺点:

直接通过下标修改或直接修改数组长度,Vue是侦测不到的

```js
let arr = [1,2,3]
//下标操作及属性操作并没有拦截.所以(?操作无效还是操作有效但丧失响应性?)
arr[0] = 5;
arr.length = 2     //正常情况下arr会变为[1,2]
```

push/unshift/splice 操作会变响应式.但以上两个操作需要通过Vue.set和Vue.delete来实现





> 原文章 https://vue-js.com/learn-vue/reactive/ 