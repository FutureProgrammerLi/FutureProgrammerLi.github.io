---
title: react-hooks
date: 2020-07-12 16:33:45
tags:
- react
category: React Hooks
---

# Hooks的用途

react 16.8的新特性。

允许函数式(无状态)组件使用state，生命周期等功能。

## 1. useState(initialValue)

自我理解: (可以实现类似Vue中的watch, setXXX中接受参数,就是对应的更新前的XXX

const [count, setCount] = useState(initialValue)

[变量名,设置变量的方法] = useState(变量初始值)

> 惰性更新

**useState传个函数进去, 可以对initialValue进行一些运算,返回值就是initialValue**

```js
const [state,setState] = useState(()=>{
    const initialValue = ExpensiveComputation()
    return initialValue
})
```

> 函数式更新

**setXXX传一个参数进去, 就是之前的对应值oldValue,类似watch(oldValue,newValue)**

````html
<button onClick={()=>setCount(prevCount=>prevCount+1)}
````

```js
//不同于类组件的setState,useState不会自动合并更新对象(!?)
//使用setXXX的参数类似实现对象合并
setCount(prevState=>{
    return {...prevState,...updatedState}
}
```

* 如果初始值是一个对象,则不能局部更新
* useState传个函数进去是什么意思?Initial State Computations.函数返回什么,什么就当初始值.

## 2. useEffect(didUpdate)

接受一个函数作为参数,可以实现类似Vue中的computed

* didUpdate代替了类组件的componentDidMount和componentDidUpdate两个生命周期函数

* return 一个函数,该函数相当于componentWillUnmount内调用

* 可传第二个参数,是个数组.表示第一个函数中的依赖项. 

  1. **传个空数组,函数仅在渲染完成后执行,相当于仅在componentDidMount中调用**

     > effect的条件执行:

  2. 传非空数组,表示第一个函数中的依赖项, 当依赖项发生改变时重新执行第一个参数函数.(**类似于computed?**)

     会对比函数依赖项和数组的项, 如果全都没有改变才不执行第一个参数的函数.

* useEffect顺序执行. 执行下一个effect之前,上一个(如果有return)effect已经被清除

* **useEffect在浏览器完成布局和渲染之后,延时调用传入的函数(!!!???)**,想在渲染前调用,则用useLayoutEffect

**使用useEffect请求数据**

```js
import React , { useEffect , useState} from 'react'
export const userList = ()=>{
    const [list , setList ] = useState(null);
    useEffect(()=>{
        ajax('/list').then(list=>{
            setList(list)
        })
    },[])       //传个空数组确保仅在componentDidMount调用一次
    return {
        list,
        setList
    }
}

//在组件内调用获取数据hook
import {userList} from 'user.js'
function App(){
    const {list , setList} = userList();
    return {
        <div>
        {list?(
        <ol>
        {list.map(item=>(
               <li key={item.id}>{item.name}</li>
                        )
               )}
        </ol>
        ):("加载中...")}
        </div>
    }
}
```



## 3.useContext

接受一个context对象(React.createContext的返回值)

```js
const value = useContext(MyContext)
const MyContext = React.createContext(defaultValue)
```

