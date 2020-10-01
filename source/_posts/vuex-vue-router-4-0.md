---
title: vuex_vue-router-4.0
date: 2020-06-01 17:08:07
tags:
- Vue
categories:
- Vue
- Vuex
- Vue-router
excerpt: vue3.0+vuex4.x+vue-router4.x以及如何去中心化
---

## 使用vue 3.0 语法需要的插件

vue@next 

vue-loader@next  @vue/compiler-sfc  (vue-template-loader这个用以提升性能(?)) vue的加载器

html-webpack-plugin   mini-css-extract-plugin   css-loader(html和css的加载器)

webpack webpack-dev-server   打包模块

vuex@next vue-router@next

全部通过npm i -D .........       安装

---

### vuex 4.x 使用

npm i -D vuex@next

```js
//创建 store/index.js
import {createStore} from 'vuex'
const store = {
    state(){
        return {
            count:0      //vuexRFC里的原文
        }
    },
    mutations:{},
    //...
}
export default store

//main.js
import {createApp} from 'vue'
import store from './store'
import App from './App.vue'

const app = createApp(App)
app.use(store)
app.mount('#root')
```

### 去中心化

* 避免所有state,mutations等都放在index里面.划分模块

* 实现参考vue-element-admin

  ```js
  // store/index.js
  //在这里汇总其它文件的state，mutations等。
  const modulesFiles = require.context('./module', true, /\.js$/)
  
  const modules = modulesFiles.keys().reduce((modules, modulePath) => {
    const moduleName = modulePath.replace(/^\.\/(.*)\.\w+$/, '$1')
    const value = modulesFiles(modulePath)
    modules[moduleName] = value.default
    return modules
  }, {})
  //console.log(modules)  可以看到module文件夹下的多个js文件仓库
  const store = createStore({
      modules  //可以扩充
  })
  export default store
  
  //在组件中使用
  import {useStore} from 'vuex'
  import { computed } from 'vue'
  setup(){
      const store = useStore();
      const count = computed(()=>store.state.otherStore.count) 
      //otherStore是一个独自的仓库
      function test(){
          //触发otherStore里的SET_INFO这个mutation
          store.commit('otherStore/SET_INFO') 
      }
      return {
          count
      }
  }
  ```

  **使用模块中的mutations**
  
  ```js
  //  store/module/otherStore.js
  const state = {
      count:0
  }
  const mutations = {
      INCREMENT(state){
          state.count++
      }
  }
  export default{
      namespaced:true,
      state,
      mutations
  }
  
  //组件里面
  const store = useStore()
  function test(){
  //*************************************************************
      store.commit('otherStore/INCREMENT')  //!!!关键点!!而且要开namespaced:true
  //*************************************************************  
  }
  ```
  
  

### vue-router 4.x使用

npm i -D vue-router@next

```js
// 创建router文件夹
// router/index.js
import { createRouter , createWebHashHistory } from 'vue-router'

const routes = [
    {
        path:'/home',
        component:()=>import('../views/Home.vue')
    }
]
export default createRouter({
    history:createWebHashHistroy(),
    routes
})

// main.js
import router from './router'
app.use(router)
```

### 去中心化

思路: 将其它文件导出的路由**对象或数组** ，在index.js里面整合起来.

```js
//1.导出对象
// router/modules/table.js
const tableRouter = {
    path:'/table',
    component:()=>import('@/views/table')
}
export default tableRouter

// router/index.js
import tableRouter from './modules/table.js'
const routes = [
    tableRouter
]          
export default createRouter({
    history:createWebHashHistroy(),
    routes
})

//routes必须是一个数组.当其他文件导出的是数组的时候,直接将它们连接起来就行.

//2.导出数组
export const otherRoutes = [
    {path:'/somewhere',component:Layout}
]

// router/index.js
const routes = [].concat(...otherRoutes)
export default createRouter({
    history:createWebHashHistroy(),
    routes
})
//导出对象的方法比较好

//3.require.context()     优化的导出数组
let routes = []
let matches = require.context('./',true,/^\.\/[^/]+\/.+\.js$/)  //这正则???
matches.keys().forEach(key=>{
    routes = routes.concat(matches(key).default)
})
export default createRouter({
    history:createWebHashHistroy(),
    routes
})
```

