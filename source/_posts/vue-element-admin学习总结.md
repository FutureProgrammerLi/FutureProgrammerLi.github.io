---
title: vue-element-admin学习总结
date: 2020-05-30 11:01:28
tags:
- Vue
categories:
- 整理
- Vue
excerpt: vue-element-admin 学到的东西
---

### Vue-router

模块化： 将router按模块分开，导出。之后再index里面的route结合起来

```js
//router/modules/table.js
const tableRouter ={
    path:'/table',
    component:Layout,
    //...
}
export default tableRouter

//router/index.js
import tableRouter from './modules/table.js'
export asyncRoutes = [
    tableRouter
]
//怎么整合的asyncRoutes和constantRoutes?
```

* 给router-view 添加一个key，可以在每次切换组件的时候都触发生命周期函数。解决路径不同，组件相同而不触发生命周期的问题

  ```js
  <router-view :key="key" />
  computed(){
      key(){
          return this.$route.fullPath  //唯一即可
      }
  }
  ```

* 路由里的redirect:'noDirect'  表示面包屑的导航不可用

* 路由中的children大于一个，自动嵌套

* 基本上index里每个一级path的component都是Layout.(必须的吧?)

  用的meta:{title:"Table",icon:'table}来控制菜单标题和图标