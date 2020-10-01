---
title: 逐行解析vue源码-3
date: 2020-06-05 18:11:53
tags:
- Vue
- 源码
- 生命周期
categories:
- Vue
- 书本
- 整理
excerpt: Vue的生命周期
---

## Vue的生命周期

四个阶段：

1. 初始化阶段 （created之前）
2. 模板编译阶段 （created 到beforeMount）  **created(),中间点**
3. 挂载阶段 （beforeMount到mounted. 包括了beforeUpdate 和updated)
4. 销毁阶段  (beforeDestroy 到destroyed)

### 1.初始化阶段

new Vue 就做了_init及new的提醒.

options merge : 用户传入的options,当前构造函数的options和父级构造函数的options合并.(? 构造函数?)

> options merge的优先级? 

```js
//经典顺序,optionsMerge之后
initLifecycle(vm)  
initEvents(vm)   //初始化实例方法. vm.set,vm.on,vm.delete等等
initRender(vm)
callhooks(vm,'beforeCreated')  //注入vuex
initInjections(vm)
initState(vm)   //!!!!     重中之重*1 !!!  
initProvide(vm)
callhooks(vm,'created')

----
function initState(vm){
    initProps()
    initMethods()
    initData()            //!!!! 重中之重*2 !!!!
    initComputed()
    initWatch()
}
```

不在new Vue里传入el,则不会进入beforeMount,(还是mounted?不知道挂载到哪个元素上).可以自己执行vm.$mount这个方法挂载元素

问:beforeMount和mounted之间做了什么? 

答:$mount挂载啊,挂载前和挂载后你问我做了什么!!?? 挂载中吧..(白眼)

(执行render functions生成VNodes并挂载到元素上)

---

在mergeOptions里面,把一些内置的组件,指令以及过滤器融合到当前组件上.(forEach,extends)

* 这是为什么keep-alive,transition,transition-group组件不需要注册的原因(指令是v-if,v-for这些吗?过滤器又是哪些?)
* 这些操作比initLifecycle还早.

merge顺序,子复制到父上, 父复制到一个空对象上.之后把子有父没有的属性复制到那个空对象中(为什么要复制到一个空对象上?)

> 这策略好吗?不直接父复制到子上,子复制到空?有什么区别?

*子merge到空对象时用的mergeFields有策略,不是单纯的属性复制.*

mergeHook(在mergeFields中)有自己的策略.如果父子对同一个钩子都有定义,则会保存到一个数组中,方便mixin的时候同时调用.

`策略模式`

**钩子函数很可能是数组,(父子),也可能只是父组件定义.callhooks就是遍历这个数组并执行.**

---

1. initLifeHooks()  设置$root,$parent,$children,$refs等,默认属性_isMounted等等

2. initEvents() 浏览器原生事件在父组件中处理,父组件给子组件注册的自定义事件(v-/@)在子组件的initEvents中处理

   ```js
   <child @select="selectHandler" @click.native="clickHandler" />
       //addHandlers:
       el.events = {
       select:{
           value:'selectHandler'
       }
   }
       el.nativeEvents = {
           click:{
               value:'clickHandler'
           }
       }
   //之后的genData生成VNode时,就变成:
   {
       on:{"select":selectHandler},
       nativeOn:{"click",clickHandler}
   }
   ```

   

* 结论是,父组件中, 父给子的原生事件,在父组件中处理; 父给子的自定义事件,在子组件的initEvents中处理.
* 即initEvents处理了父组件传给当前组件的自定义事件. (我被颠覆了,mind explosion...) `高阶函数`
* initEvents: 首先建立_events空对象属性,之后通过调用updateComponentListeners,将父向子组件注册的自定义事件注册到子组件实例中的\_events属性中.

3. initInjections():

   provide可以是`对象或是一个返回对象的函数`;inject可以是`一个数组或一个对象(?)`

   ```js
   provide:{
       foo:'bar'
   }
   provide(){
       return {
           test:'hello'  //可以使用symbol属性  不是响应式的.
           reactiveTest:()=>this.name       //响应式的
       }
   }
   
   data(){
       return {
           name:'haha'
       }
   }
   inject:['foo','reactiveTest'];
   inject:{test} 
   inject:{
       'foo':{
           default:()=>this.name
       }
   }
   //全都会被转化成
   inject:{
       foo:{                     //这foo是在当前组件使用时的名字
           from:'foo',           //这里的foo是祖先里provide的名字
           default:()=>this.name
       }
   }
   //template中
   {{name}}
   computed:{
       name(){
       return reactiveTest()
       }
   }}
   //或者
   {{reativeTest()}}  //未实际用过,好像可以.
   ```

   * **inject的所有东西,都可以在data/methods/computed等options里面使用.(先执行的initInjections()后再执行initState())**

   * provide和inject里的都不是响应式的(!!!???).但传入可观察对象,那么它的属性还是可响应的.

   * provide的值可能被data/methods/computed等处理过.所以initInjections和initProvide之间插了个initState.

     (因果是不是反了?为了在data/...中获取injections,所以要先initInjections,再initState)

   * injections顺序是拿着key不断往父级找,找不到就看自己的default,全都找不到就抛出错误.

     

4. initState()

   ```js
   //主线
   initProps()
   initMethods()
   initData()
   initComputed()
   initWatch()
   ```

   > 在data中可以用props,在watch中可以用data和props.
   >
   > 初始化才能使用的话,methods为什么能用data?定义和实际执行时机不同吗?定义的时候不会undefined吗?

  

* initProps(): 三种写法,数组,对象,对象里面套对象. 前两种在normalize的时候,值统一变成{type:null}

  ```js
  props:['foo'];
  props:{foo};
  props:{
      foo:{
          type:String,
          required:true
      }
  }
  //前两种都变成
  props:{
      foo:{
          type:null
      }
  }
  ```

  vm._provided, vm.\_props

* initMethods():  //有没有? 重没重? 挂载
  1. options里有没有methods?
  2. methods的函数名是否跟props同名?是否跟已有的method重名?//是则警告
  3. 挂载到实例上(bind(methods[key],vm))  //key就是定义在methods里的方法名.

* initData():(最终都要是一个对象?)
  1. options里有没有data?
  2. data中是否跟methods,prop有重名?   
  3. 通过代理,直接通过this.xxx获取,而不是this._data.xxx获取. (proxy(vm,'\_data')) 

* initComputed(): *可以是函数,直接return,直接设置为getter;*或是一个对象,在里面设置getter/setter

  用的大部分是直接函数,即return => getter  | 对象形式的set(){}都比较少见

  const getter = typeof userDef === ' function'?userDef : userDef.getter      //userDef就是计算属性的名字 

  只判定了computed里是否有两个同名的计算属性,没有检验跟data/props的重叠性(?)

  在SSR的情况下,computed**没有缓存作用**. 需要用createComputedGetter劫持一下userDef实现缓存功能.

  也可以手动关闭

  ```js
  computed:{
      add:{
          cache:false,       //重点.函数形式可以实现吗?
          get(){
              return this.sth
          }
      }
      //以下两个等价
      sub(){
          return this.count--
      },
      sub:{
          get(){
              return this.count--
          },
          //set(){}
      }
  }
  sharedPropertyDefinition.get = userDef.get
        ? shouldCache && userDef.cache !== false
          ? createComputedGetter(key)      //缓存.返回结果是一个函数.
          : userDef.get                    //不缓存
        : noop;                            //不操作.no operations
  sharedPropertyDefinition.set = userDef.set
      ? userDef.set
  	: noop
  ```

  主要属性是dirty,为true时通知watcher重新执行get(),并通知其依赖. evaluate() / update()/ getAndInvoke()

  //详细不太懂

* initWatch()

  ```js
  watch:{
      a:{
          handler:'somemethod',
          deep:true,         //深度侦察
          immediate:true     //开始侦听就立即执行handler
      },
      b:[                    //执行多个回调
          'handler1',
          function handler2(){},
          {handler:function handler3(){}}
      ]
  }
  ```

  ---

  ### 2.模板编译阶段

  created到beforeMount是模板编译阶段

  vue有两个版本: runtime-only 和runtime+compiler,前者比后者小大约30%.

  runtime-only要么直接写render functions, 要么借助vue-loader而不是内置的compiler实现编译.  

  (vue-loader比compiler还要小还是什么?有内置的反而要用插件?)

  vue-cli用的是runtime-only(?),借助webpack的vue-loader实现模板预编译,从而减小生产环境代码体积

  **关键是vm.$mount这个方法的差异**

  挂载用的其实都是一个函数,但完整版需要经过模板编译阶段再调用$mount;运行时环境直接调用$mount进行挂载.

> $mount做了什么? (???mount不是挂载而是template-> renderFunctions???)

1. *获取el*.( 不能是html或body)
2. 没写render functions的情况下*获取template*
3. <template\>  -> `compileToFunctions()`  ->render()

template可以是id选择器,也可以是HTML字符串

```js
{ 
    template:'#root'                       //id selector
    //template:'<div>Hello</div>'         //template.innerHTML
}
//不传template,就拿el外部当模板(?)
```

### 3.挂载阶段

>  创建vm.$el并用其代替el

vm.$mount方法会调用callHook(vm,'beforeMount')

vm.$mount -> mountComponent() -> callHook(vm,'beforeMount')

而$mount方法就做了两件事,获取el元素,返回mountComponent().

挂载的细节其实在mountComponent方法中

做了两件事: 

1. updateComponent, 里面实际调用的vm._update(vm.\_render(),hydrating) 就是实际的patch操作.

2. 开启状态观测

   ```js
   new Watcher(
     vm,
     updateComponent,
     noop,
     {
         before(){
             if(vm._isMounted){
                 callHook(vm,'beforeCreate')
             }
         }
     },
     true)
   ```

   