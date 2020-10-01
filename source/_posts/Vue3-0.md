---
title: Vue3.0
date: 2020-05-01 10:18:35
tags: 
- Vue3.0
categories: 
- Vue
- 视频学习
excerpt: Vue3.0的两种写法:data,method,computed,lifecycle...
---
# 基本改变
CheatSheet from vuemaster 

```js
<template>
  <div ref='aDiv'> </div>
  <component-a />
</template>
//常见写法.第二种在最后.
<sciprt>
import { ref , reactive, computed , onMounted ,watchEffect} from 'vue';
export default{
 props:{                     
   sthFromParent: String              
 },
 components:{
     component-a
     },         //1.组件注册
 setup(props,context){                        //对比Vue2.x
  //2.props.sthFromParent
  //context.attrs/slots/emit
  //可以解构:context=>{attrs , slots , emit}
   const capacity = ref(4)        //3.基本类型 对应{options},data里的基本值
   const attending = reactive(["Tim","Bob","Joe"]) //4.引用类型 data里的对象
   const spacesLeft = computed(()=>{    //5.计算属性 对应computed:{}
      get: ()=>{capacity.value++},     //6.修改计算属性的方法
      set: ()=>{capacity.value--}
      return capacity.value - attending.length
   })
   //
  function inc(){               //对应 7.methods:{}
      capacity.value++
    //   spacesLeft.value++ 会报错,需要设置为computed设置getter/setter属性
   }
  onMounted(){                 //对应8.mounted(){}
      this.inc()
   },
  const aDiv = ref(null)        //9.模板引用,使用同样的函数,但要创建同名ref()
  return {                     //返回给template使用.
      capacity,
      attending,
      spacesLeft,
      inc,
      aDiv
   }
 }
}
</script>
```
* 所有值都可以通过ref创建,变得响应式.*对象*,包括数组,建议使用reactive()方法创建
* ref创建的数组访问: arr.value[0] , 对象访问: obj.value['propsName']
  用reactive创建的数组访问: arr[0] , 访问对象: obj.propsName
* 用ref创建的基本结构: { value:'1' , _isRef:true, get(), set()} //对象类型value为Proxy对象
  所以定义变量时**使用const,返回对象**,对象值可以修改且自身名称不会被修改.
* **删除了beforeCreate和created生命周期,setup()就是这两周期之间被调用的.**(之前在这两周期间的操作可以直接写在这?)
  Vue3.0的生命周期: onBeforeMount=> onMounted => onBeforeUpdate => onUpdated =>onBeforeUnmount => onUnmounted
            另外: onErrorCaptured. 
            NEW HOOKS : onRenderTracked, onRenderTriggered => for debugging use
* 组件通信放在了setup(props,context) 两个参数. 
  (props还是要在外部注册,只是放在setup里面供子组件使用)
  (组件注册还是一样的.与setup()同级,components:{},props:{a:String,b:Number})
  props单独抽出来的原因:(From RFC)
  1.More commonly used than other properties.
  2.Easier to type it individually without messing up the types of other properties;Keep a consistent signature(?)

### 另一种写法

```js
//将变量,计算属性都放在一个对象中
import { reactive , computed , toRefs } from 'vue'
export default{
    setup(props){
        const event = reactive({
            capacity:4,
            attending:["Tim","Bob","Joe"], //区别一
            spaceleft:computed(()=>{return event.capacity - event.attending.length});
        })
        function increment(){
            event.capacity++
        }
        return{
            ...toRefs(event),  //区别二
            increment
        }
    }
}
//好像挺好用?不用区分state是什么值(基本/引用)而用ref还是reactive.
//toRefs(event) 替代了ref,reactive.以及需要展开wrapped object =>event.
```



