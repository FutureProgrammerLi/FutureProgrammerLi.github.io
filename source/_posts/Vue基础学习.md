---
title: Vue基础学习
date: 2020-01-01 18:47:01
tags: Vue基础
excerpt: Vue的基本概念
categories: 
- Vue
- 视频学习总结
---
# Vue基础指令
 1. 创建:
```
<div id="app">
    <span>{{mes}}</span>
</div>
<script>
var app = new Vue({
    el:"#app",
    data:{
        mes:"Hello World!"
    }
})
</script>
```
 2. v-if,v-else和v-show
 1 `v-if和v-show`区别:
v-if直接不渲染到界面，如果为false直接没有对应元素
v-show只是给元素添加style="display:none",界面中有对应元素()
 2 `v-if和v-else是怎么绑定的?`
 if开始,else为不满足条件,后面再添加else-if也没用了(true和false的情况都被占用了)

 3. v-bind和v-on 绑定
 用到Vue实例里面的data,就要用v-bind. 简写是:
 用到Vue的方法,就要用v-on 简写是@
 @click(val),可以当按钮被按下后,传递参数进来函数?

 4. v-for循环遍历数组和对象
 ```
 <ul>
	<li v-for="item in people">
	My name is {{item.name}} , I am {{item.age}} years old.
	</li>
</ul>
data:{
    people:[{name:"li",age:22},{name:"zeng",age:33}]
}
```
---
```
 <ul>
	<li v-for="(value,key) in obj">
	Value is {{value}} , Key:{{key}} 
	</li>
</ul>
```
`循环存对象的数组,(item in people), item是对应的每一个对象,item.name获取某个对象的name属性`
`循环对象,(value,key) in obj`
`循环数组,(value,index) in arr`
---
{{index+1}}的应用情况:
*获取的是存对象的数组,index+1就是数组下标加一,就是对应的顺序序号.*

### 5. v-model双向绑定 **获取选中,输入的值的方法 (新1)**
 1 文本框
 2 单选框
 必须要有`value`和`v-model`,name属性和id可有可无,传的是value,绑的是data里的一个字符串
 model一般是空字符串
 可以设置初始值,例如空数组[], 但会输出[]
 3 多选框
 **model绑定的必须是数组,**以数组形式传递,不能是空字符串
 4 下拉选择框
 ```
 <select name="" v-model="selectVal">
	<option value="特殊">A</option>
	<option >B</option>
	<option >C</option>
	<option >D</option>
</select>
	{{selectVal}}
```
不赋值value属性的话,传的就是选择到的内容.(不赋值好像更好?)
默认给值,在selectVal上赋值一个选项的内容或者对应的value//可以是选项label,也可以是option里的value

### Computed计算属性 (新2)

computed有问题,反正就是是个function,return一串对应的处理
*用return代替直接一串?*

`不推荐v-for和v-if达到筛选的结果,更希望使用计算实行达到筛选的效果?`
怎么用computed筛选? return
```
computed:{
    newPerson(){
        return this.people.filter((item)=>{
            return item>30;
        })
    }
}
```
循环的东西可以是个属性,从而筛选people.
people是数组,computed计算这个数组,实现筛选.
v-for实现遍历获取符合条件的东西.
(v-for=("item in newPerson"))
---
### watch监控属性
当某个属性发生改变时,其它属性值也可以跟这变

```
watch:{
    question(){       //为什么是函数?给某个属性添加相应方法?
        this.answer = "No Answer"  
			_this = this
			setTimeout(()=>{       //模拟异步请求
			_this.answer = '404找不到嘻嘻'
			},1000)
    }
}
```

*watch可以有中间状态值,而computed只是一个返回值,return什么,computed就是什么*

---
### class,style绑定

可以是键值对(根据true/false),也可以是数组(绑定多个class). 甚至数组里面有键值对 (又绑定多,又根据判断)

---
## Vue核心原理
1. Object.defineProperty(obj,pro,descriptor)实现数据劫持,
定义get,set,value  // enumerable,configurable等等的属性
**新添加的属性无法实现监控**
官方解释
>受现代 JavaScript 的限制 (以及废弃 Object.observe)，Vue 不能检测到对象属性的添加或删除。由于 Vue 会在初始化实例时对属性执行 getter/setter 转化过程，所以属性必须在 data 对象上存在才能让 Vue 转换它，这样才能让它是响应的。
---
一开始就要在data里创建属性名,空也可以,但必须要有
*可以添加属性给已经存在的对象*
```
//外部
Vue.set(obj,key,value)
vm.$set(this.someObject,'b',2)
//内部
this.$set(this.someObject,'b',2)
```

2. 组件复用
某个input被删除,已经输入的内容是不会变的==>代表已经复用
删了,已经输入的内容也对应没了==>没有复用, 用key标识,也可以直接绑定v-model==>更常见方法
或者用官方文档,toggle的例子

3. 全局注册组件和局部注册组件
```js
//全局注册
Vue.component('component-a',{ / ... / })
//局部注册
var componentb = { ... }
Vue({
	components:{
		'component-b':componentb
	}
})
//或者
Vue({
	components:{
		'componentName':{ ... (definition of the component)}
	}
})
```