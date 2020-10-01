---
title: Vuex-intro
date: 2020-01-11 23:27:54
tags: Vuex
excerpt: Basic knowledge of Vuex
categories:
- Vue
---
# 数据共享 Vuex
### 组件间传值
父传子 v-bind
子传父 v-on
兄弟组件,或没相连的,用Eventbus?
- $on (subscriber)
- $emit (publisher)
*小范围使用,大范围关系复杂 --- Vuex状态管理*
---
### 好处:
1. 集中管理(store)
2. 共享(shareable)
3. 同步更新,不用逐个改(updated)
### 场景
需要共享的数据,存Vuex;私有数据,存data里(具体看情况,可以全部放vuex里)
<figure>
    <img src="fig1.png" width="650" alt="Interpreting a script to execute it" align="center">
</figure>

# 基本使用
## vue add vuex完成了以下四个步骤
1. npm install vuex --save
2. import Vuex from 'vuex'     //store.js
3. const store = new Vuex.Store({     //store.js
    state:{},
    mutations:{},
    actions:{},
    modules:{}
})
4. import store from './store'       //main.js
new Vue({          //
    store,
    render: h => h(app)
    /*...*/
})

# State Mutations Actions
1. 定义和调用state里面的值的两种方法
  1.1 $store.state.count(变量名)
  1.2 import mapState from 'vuex'
     computed里面,
     ...mapState([count])  
     template里面
     {{count}}   
2. 怎么使用Mutations
  2.1 为什么要用Mutations
  方便集中管理,如果直接在vue里面修改,容易找不到哪个页面修改了store里的值
  2.2 Mutations里面函数的参数?
  第一个必须为state对象,之后可以传入自定义参数. 
  mutations:{
    add(state,a,b,c){}
  }
  2.3 触发Mutations的两种方法  //hint:同理于state,为什么解构之后还要this.sub()?
  2.3.1 this.$store.commit('functionName',para1,para2,para3)
  2.3.2 import mapMutations from 'vuex'
        methods:{
          ...mapMutations(['function1'],['function2'])
          handle(){
            this.function1(a,b,c)     //传参举例
          }
        }
        
  2.4 Mutations 里用第二种方法使用,需要包装.即解构来的不能直接用,要用methods包括它在内,在里面传递参数
  2.5 Mutations里不能用异步函数
3. Action
  3.1为什么要用Action?
  3.2 action里函数的参数?
  3.3 怎么触发actions里的函数?  hint:commit->dispatch->mutations->state
  3.4 actions里面为什么不能直接访问state而要用mutations来修改?
  只有mutations里的函数有权利修改state.
  3.5 Actions里的函数可以修改state吗? hint:已有答案
  3.6用Action的两种方法

4. 实例方法：
store/index.js里面
  1.在actions里
  getList(){
    axios.get().then(({data})=>{
    context.commit('getList',data)  //getList是mutations的方法,注意context,是Actions的.data是给Mutations的
  })
  }
  2.mutations里getList(state,data){
    state.list = data
  }
  3.state里 { list:[]}
vue界面:
  1. created(){
    this.$store.dispatch('getList')   //触发actions
  }
  2. this.list = this.$store.state.list
  或者 import { mapState} from 'vuex'
      computed:{
        ...mapState(['list'])
      }

** Antd 的list 数据源用 :dataSource**
  <a-list-item slot="renderItem" slot-scope="item">
  意义不明,用于循环? renderItem是什么

5. 文本框和state里的数据实现双向绑定
 1. input显示store里的值
  <input :value="$store.state.inputValue">    //此时输入框无法被修改
 2. 允许修改:
 vue里面:
 input绑定onchange="handleChange"
 handleChange(e){
   this.$store.commit('setInputVal',e.target.value)       //setInputVal是mutations里的函数,*e.target.value*是输入框的内容!!可变!!
 }
 vuex里面:
 mutations:{
   setInputVal(state,val){
     state.inputValue = val;
   }
 },
 state:{
   inputValue:'aaa'
 }
 ===
 DONE
 重点是e.target.value动态获取输入框内容

6. 添加item
@click ,commit触发mutations
mutations方法里面定义
const obj={
        id:state.nextId,
        info:state.inputValue,
        done:"false"
      }
      state.list.push(obj)       //**重点**
      state.nextId++         //模拟id加一
      state.inputValue=''    //添加后清空输入框
    }
}

7. 删除item ,传id,找,触发删除splice
@click="delItem(item.id)"      //将对应的id传给函数,注意是item.id
delItem(id){
  this.$store.commit('remove',id)
}
mutations里:
remove(state,id){
  const i = state.list.findIndex(x=>x.id === id)
  if( i !== -1){
    state.list.splice(i,1)
  }
}
<a-list-item slot="renderItem" slot-scope="item">
:checked="item.done"  //renderItem是什么?
**改变选中状态**
@change="(e)=>{handle(e)}"     //?????
**传两个参就不行，整合成一个对象就可以？**
为什么不可以changeState(state,bool,id){
const i =state.list.findIndex(x=>x.id === id)
      if(i !==-1){
        state.list[i].done = bool;
    }
}
handle(e,id){
      const obj ={
        id:id,
        status: e.target.checked
      }
      this.$store.commit('changeState',obj)
    }




