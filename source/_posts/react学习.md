---
title: react学习
date: 2020-07-08 15:14:52
tags: 
- react
---

# Hello World!

```js
const str = "Hello World!";
const ele = <h1>{str}</h1>;  //元素中使用变量，{}
ReactDOM.render(
  ele,            
  document.getElementById('root'))
//类似于new App.mount('#root')
function concat(name){
    return `${name.fsN} + ${name.lsN}`
}
const name = {
    fsN:'Li',
    lsN:'Weijian'
}
const ele2 = <h2>{concat(name)}</h2>       //在{}里面使用函数处理字符串
//建议将JSX拆成多行， 用括号括住， 防止直接return终止了后面
```

JSX中可以用“”表示字符串，也可以用{}表示表达式，但不能同时使用。

**JSX中使用camelCase，函数式组件都必须首字母大写，包括开头**

JSX语法转renderFunctions是通过Babel实现的，

所以html内使用时需要\<script type='text/babel'\>,至少引入react.min.js , react-dom.min.js和babel

# 函数式组件

**函数式组件内不能定义自己的state.除非使用useState???**

**函数组件也不能定义生命周期钩子,因为没有继承React.Component类**

```js
import React, {useState} from 'react'
function HelloWorld(props){
    //定义状态
    const [count,setCount] = useState(0)     //DATA AND METHODS
    //useState接收变量的初始值,数组解构的是变量的名称和设置该变量的,对应的函数.(所以最后有命名规范)
    //在JSX里面 <button onClick={()=>setCount(count+1)}  
    //定义方法 
    //const wrong = ()=>count++  //错误.useState返回的,必须用setXXX修改对应值
    function sayHi(){                       //METHODS
        console.log('Hello!')
    }
    //注意一下区别:
    // <button onClick={sayHi}>Self-defined functions</button>
    // <button onClick={()=>setCount(count+1)} Using setXXX function </button> 
    return (
      //... JSX
    )
}
```

* **useState的变量必须使用返回的setXXX函数修改,且必须是函数式返回,否则报错 Too many assinments**

* **自定义方法不需要返回.直接在{}中输入,会自动被evaluate**

# 类组件

```js
import React from 'react'
class Clock extends React.Component{
    constructor(props){
        super(props);
        this.state = {        //DATA
            date:new Date(),
            count:0,
            sthElse:'blablabla'
        }
    }
    sayHi = ()=>{                 //METHODS
        console.log('Hello')
    }
    render(){
       //JSX
    }
}
ReactDOM.render(element,document.getElementById('root'))
```

* 类组件方法声明有两种:

  ```js
  
  class test extends React.Component{
      //方法1 constructor+bind+definition
      constructor(props){
          super(props);
          this.fn1 = this.fn1.bind(this)
      }
      fn1(){
          //function body
      }
      //方法2, 直接箭头函数. create-react-app 默认支持
      fn2 = () =>{
          //function body
      }
  }
  ```

  

# 声明周期函数(针对类组件)

重点三个:

1. componentDidMount
2. componentDidUpdate    *和6的区别?props和state的问题吗?
3. componentWillUnmount     (前三个可以用useEffect钩子函数代替)
4. componentWillMount (对应1)
5. componentWillUpdate(对应2)
6. componentWillReceiveProps.   组件接收到新的props时调用. 区别于render()的第一次调用
7. shouldComponentUpdate, 返回一个布尔值. 决定是否调用componentWillUpdate钩子函数.(怎么使用?)

### 自我理解

#### 1.对应Vue

WillMount-> DidMount -> WillUpdate -> DidUpdate -> WillUnmount

(beforeMount->mounted-> beforeUpdate-> updated -> beforeDestroyed) //对应Vue

特殊:shouldComponentUpdate 和 componentWillReceiveProps

#### 2.阶段理解

挂载, 渲染,卸载( 初始化和挂载合并了)

顺序, (getDefaultProps-> getInitialState->componentWillMount->render()->componentDidMount)  **挂载阶段**

​         (*State改变, shouldComponentUpdate ->(true) componentWillUpdate -> render()->componentDidUpdate)  **渲染阶段**

​         (*Props改变, componentWillReceiveProps -> shouldComponentUpdate-> (true) componentWillUpdate -> render()->componentDidUpdate)

​         componentWillUnmount        **卸载阶段**

与Vue不同,React并不强调getDefaultProps和getInitialState 这两个数据初始化阶段. (不需要数据劫持?怎么实现的呢?re-render?)

# Slot

**props中有个特殊的属性,children, 指父组件(?) 给子组件的内容分发**

```js
//JSX
<Welcome> Hello World </Welcome>
//Component
function Welcome(props){
    return (
    <p>{props.children}<p>        //实际就是<p>Hello World</p>
    )
}
```

# 状态提升

# State Hook 的第二个参数,空数组,含一个元素.. state对比.

setXXX 在JSX中为什么要以{()=>setXXX( XXX +1)} 的形式? 直接{setXXX(XXX+1)} 就报错Too many re-renders

使用Object.is()判断state是否发生改变

# EffectHook代替了三个声明周期函数

componentDidMount  componentDidUpdate 和 componentWillUnmount

DidMount 是首次渲染, DidUpdate是状态改变, WillUnmount是页面卸载

**useEffect里面return 一个函数,对应函数相当于在WillUnmount里面被调用**

状态改变了之后,在DidUpdate里面实际至少调用了两个函数: 取消对旧值得绑定, 绑定新的state

//..ToBeContinued