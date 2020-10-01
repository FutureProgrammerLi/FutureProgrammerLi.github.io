---
title: 重读vue3-RFC
date: 2020-06-29 17:21:35
tags:
- vue3.0
- 基本配置
excerpt: 读vue3RFC后的笔记以及使用vue3的一些简单webpack配置(单文件或多文件都有)
---

# RFC笔记

## 1.Why composition API?

1. Large scale of components are hard to read as they scale up. 
2. Lack of logic extraction and reuse.
3. Commons for better Typescirpt support.

## 2.Notes

1. `reactive()` API is equivalent to the Vue.observable() API in Vue 2.x

2. `reactive` states can be used during render.

3. > To apply and automatically re-apply a side effect, we can use the watchEffect API

     ```js
let price = 5, count = 2,total = price*count
console.log(total)     //10
count = 4
console.log(total)     //still 10, you have to re-run the total = price * count to update
//Here , total = price * count is the side effect(?)
     ```

4. The object returned from data() is like wrapped by reactive().

5. watchEffect() doesn't require separating the watched data source and the side effect callback

   ```js
   watch:{
       watchedData(newVal,oldVal){
           //..side effect callback
       }
   }
   watchEffect(()=>/**side effect callback**/console.log(watchedData))
   ```

6. The computed method with a function inside can be rewritten as follows:

   ```js
   let count = ref(1)
   let double = computed(()=>count.value++)
   //the same as follows:
   function computed(getter){
       let value
       watchEffect(()=>{
           value = getter()
       })
       return value  //Is this a completely new copy?
   }
   /**PROBLEM: If the value is a primitive type, it will lose reactivity because JS primitive types are passed by value.**/
   //Therefore, change it:
   function computed(getter){
       const ref = {
           value:null
       }
       watchEffect(()=>{
           ref.value = getter()
       })
       return ref  //And this is a memory pointer?
   }
   ```

   **watchEffect() &&  reactive()**

7. A SFC can be rewritten as followed for understanding case:

   ```js
   function setup(){     //Do not have to be a function exported as default, for better logic reuse.
       const state = reactive({
           count:1,
           double:computed(()=>state.count*2)
       })
       return {
           state          //why not ...toRefs(state)?
       }
   }
   const renderContext = setup()
   watchEffect(()=>{
       renderTemplate(
       `
        <div> {{count}}  {{double}} </div>
       `,
       renderContext)
   })
   ```

8. setup() is a replacement hook for beforeCreate and created, 

   but what does it mean by using hooks functions in the setup() function?

   ```js
   import {onMounted , onUnmounted} from 'vue'
   export default{
       setup(){
           onMounted(){
               console.log('The component is mounted') //?
           }
       }
   }
   //Lifehook functions will be put in an array , and ran one by one.
   //>To reduce friction when extracting logic into external functions.
   ```

9. > Does "Organized code "  just mean that we know what is contained in options?
   >
   > We do care more about what a component is trying to do (features) instead of what options the component happens to use.

10. Drawbacks of the past three patterns that achieve logic reuse( mixins / high-level components / renderless components via scoped slots)
    1. Unclear source of mixin property. Hard to tell some properties are mixined by which mixin.
    
    2. Namespace collision.
    
    3. HOCs(High-Order Components) and renderless components require extra stateful instances which come at a performance cost.
    
       Solutions:
    
       1. Names are returned from functions , which is explicitly imported. 
    
       2. Names can be more arbitrary using destructuring aliases.

11. No `this` keyword. Plugins like vuex, vue-router , using this.$store /this.$router , will replaced by methods

    ```js
    import {useStore} from 'vuex'
    import {computed} from 'vue'
    const store = useStore()
    const count = computed(()=>store.state.count)
    function test(){
        store.commit('SET_COUNT',1)
        //store.dispatch("ASYN_METHOD")
    }
    ```

**12. Instead of importing useStore() in every component, provide the store in the root component. **

```js
const StoreSymbol = Symbol()
export function provideStore(store){
    provide(StoreSymbol,store)
}
export function useStore(){
    const store = inject(StoreSymbol)
    return store
}
//in root component: App.vue
const App = {
    setup(){
        provideStore(store)
    }
}
//child components:
const Child = {
    setup(){
        const store = useStore()
    }
}
```

13. ref and reactive

    Potential hazards are using reactive may lose reactivity by being destrutured or spreaded.

    ```js
    function useMousePos(){
        const pos = reactive({
             x:0,
             y:0
        })
        return pos
    }
    
    
    //consuming component
    //case 1
    const {x,y} = useMousePos()
    return {
        x,y           //lose reactivity!
    }
    //case 2
    return{
        ...useMousePos()     //lose reactivity!
    }
    
    return {
        pos:useMousePos()   //reactivity RETAINED!
    }
    
    //the other solution:
    function useMousePos(){
        //...
        return toRefs(pos)
    }
    ```

14. Will the return statement be too verbose?  //suggestions, returned automatically by IDE or Babel Plugin.

# 使用vue3

## 1.单html文件

```html
<script src="http://unpkg.com/vue@next"></script>
<div id="app">
    
</div>
<script>
const {createApp} = Vue
const App = {
    //.. Write components description here.
}
create(App).mount("#app")
</script>
```



## 2.多文件配置,使用webpack

1. package.json里的依赖项：

```js
"devDependencies": {
    "@vue/compiler-sfc": "^3.0.0-beta.14",
    "css-loader": "^3.5.3",
    "html-webpack-plugin": "^4.3.0",
    "mini-css-extract-plugin": "^0.9.0",
    "vue": "^3.0.0-beta.14",
    "vue-loader": "^16.0.0-beta.3",
    "vue-router": "^4.0.0-alpha.12",
    "vue-template-compiler": "^2.6.11",
    "vuex": "^4.0.0-beta.2",
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.11",
    "webpack-dev-server": "^3.11.0"
  }
```

2. webpack.config.js里的配置项

   ```js
   const path = require('path')
   const { VueLoaderPlugin } = require('vue-loader')
   const MiniCssExtractPlugin = require('mini-css-extract-plugin') 
   const HtmlWebpackPlugin = require('html-webpack-plugin')
   const webpack = require('webpack')
   // import webpack from 'webpack'
   
   // /**       //这里可以用来提示,但最后必须注释掉,否则不支持
   //  * @type {webpack.Configuration}
   //  */
   const config = {
       mode:'development',
       entry:'./src/main.js',
       output:{
           filename:'bundle.js',
           path:path.join(__dirname,'dist')
       },
       module:{
           rules:[
               {
                   test:/\.vue$/,
                   use:'vue-loader'
               },
               {
                   test:/\.css$/,
                   use:[MiniCssExtractPlugin.loader,'css-loader']
               }
           ]
       },
       plugins:[
           new VueLoaderPlugin(),
           new MiniCssExtractPlugin(),
           new HtmlWebpackPlugin({
               template:'./src/index.html'
           }),
           new webpack.HotModuleReplacementPlugin()
       ]
   }
   
   module.exports = config
   ```

3. main.js

   ```js
   import {createApp} from 'vue'
   import App from './App.vue'
   //import store from './store'  //可选,使用vuex和vue-router
   //import router from './router'
   
   const app = createApp(App)
   //app.use(store)
   //app.use(router)
   app.mount("#app")
   //index.html
   <div id="app">
       
   </div>
   //App.vue 里直接vbase
   ```

   