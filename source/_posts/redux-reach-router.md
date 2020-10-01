---
title: redux_reach-router
date: 2020-07-11 00:30:07
tags:
- redux
- reach-router
experpt: redux和reach-router的一些学习整理
---

# Redux

## 核心概念

### Action

一般的action就像新闻的摘要，包括标题（types）以及内容（payload）

```js
//就是一般的对象
//官方demo的一个例子
let nextTodoID = 0
export const addTodo = text=>{
    return {
        type:'ADD_TODO',         //返回的这个对象是修改之后的state
        id:nextTodoID++,
        text
    }
}

```

#### Action Creator

```js
//一般的action返回对象
//在类组件中调用一般的action
import { INCREMENT } from '../actions'
class XXX extends React.Component{
    increment=()=>{
        this.props.dispatch({type:INCREMENT})      //INCREMENT是action.js导出的字符串常量
    }
}

//使用Action Creator
//actions.js
export function increment() {
  return { type: INCREMENT };
}

//组件内
import { increment } from '../actions'
class XXX extends React.Component{
    increment=()=>{
        this.props.dispatch(increment())     
}
//使用了mapDispatchToProps之后, 可以直接this.props.increment() 直接使用store里的dispatch
```

* this.props.dispatch() 本身接受一个action对象,你可以直接在里面写action对象.
* Action Creator 只是将这个action对象以函数执行的形式返回. 实际并没有区别. (?把type属性简化了)

#### Thunk Action

### Reducer

* 接收两个参数,state和action,是一个纯函数,不修改传入的参数本身

```js
const todos = (state=[], action)=>{      //state=[]相当于初始化state
    switch (action.type){
        case 'ADD_TODO':
            return [
                ...state,        //这里是旧状态 , 类似setState,用新状态覆盖(?)旧状态 
                {                //这里是新状态
                    id:action.id,  //旧状态包括在...state中, 新状态会将旧状态覆盖
                    text:action.text,  
                    completed:false
                }
            ]
        case 'TOGGLE_TODO':
            return state.map(todo=>
                (todo.id === action.id)?{...todo,completed:!todo.completed}:todo
                )
        default:
            return state
    }
}

export default todos

// 在reducers/index.js将多个reducers合并起来
import {combineReducers} from 'redux'
import todos from './todos'
import visibilityFilter from './visibilityFilter'

const todoApp = combineReducers({
    todos,
    visibilityFilter
}) //注意这里和connect()的不同. connect接收的参数**不是**对象
```

---

### Store

action描述"发生了什么" , reducer根据action更新了state.

Store就是将它们联系起来的对象(?为什么要联系起来, reducer里不是传了action?)

* 一个Redux应用必须只有一个store,如果需要拆分不同的逻辑处理,使用多个reducer处理

---

### connect

将store和组件连接起来

是个高阶函数,**将传入的组件封装成可以操作store的新组件**

redux并不依赖react, Vue,Express等等的框架都可以使用. 

react-redux才是**将react组件封装成可以操作store的新组件.**

connect可接收四个参数，

1. mapStateToProps
2. mapDispatchToProps
3. mergeProps
4. options

看前两个参数名字就知道,store的state,dispatch操作以props的形式传给对应组件

```js
const mapStateToProps = (state, ownProps) => {    //状态修改
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {   //事件修改
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)             //这是个链式调用connect()(),使用到redux的组件
//这里的Link就是组件名称.
```

因为Link这个组件需要使用redux,所以将其封装起来,装饰成**一个可以操作store的,新的组件**

---

# Reach-Router

## 整个组件的rendering

```js
let Home = ()=><div> Hello Home</div>
let About = ()=><div> Hello About </div>


ReactDOM.render(
       <Router>
         <Home path='/' />
         <About path='/about' />
       </Router>,
  document.getElementById('root')
);
```

## Navigate With Link

```js
import {Router , Link} from '@reach/router'
let Home = ()=>(                //只允许一个根元素
  <div>
    <Link to="/">Home</Link>
    <Link to="about">About</Link>             //这里的to对应<Router>里面的path
  </div>
)
let About = ()=<div>About</div>
ReactDOM.render(
  <Router>
    <Home path="/" />
    <About path="about" />  //这里加不加/不影响。
  </Router>
)
```

