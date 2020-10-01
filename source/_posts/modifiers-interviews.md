---
title: Vue修饰符
date: 2020-05-19 09:03:13
tags: Vue
categories:
- Vue
excerpt: .sync/ .exact / .lazy /.keyCode等
---

### 1.修饰符

1.  .lazy,懒修改(?).

   VueData里的值不立刻随着template里的值改变。失去焦点或按下回车键才更新Data里的值

2. .trim , 删除开头和结尾的空格键,只取有效的输入值部分

3. .number, 第一次输入数字后,后面也只允许输入数字. (若第一次输入非数字,则该修饰符无效(?))

4. .stop, 对应原生的event.stopPropagation事件. 防止冒泡

5. .prevent 对应 event.preventDefault, 阻止默认事件(常见表单提交按钮)

6. .capture 捕获事件.例子:父子都绑定了同样事件,触发子组件事件:不用capture则先触发子再触发父.用capture则先父后子.

7. **.native 子组件触发父组件的事件(!!!????)**  Vue 3子组件直接就能在父组件中触发父组件的事件阿?没区别?

   ```js
   //父组件中
   <my @click="test" />
   <my @click.native="test"      //???没有区别
   ...
   function test(){
       console.log(111)
   }
   components:{        //发现vue3 import Vue from 'vue'
       my              //Vue -> undefined !!! 方法去哪了?用不了Vue.component()
   }
   ..
   ```

8. .left /  .middle / .right     鼠标左中右键时才触发(可能很有用?)

9. .enter / .shift / .ctrl  / .delete / .alt     ....等等  可以用e.keyCode 获取对应键码( 其实与9同理, .keyCode)

10. .exact  当且只有按下该按钮时才触发事件

11. .sync  子组件能修改父组件传进来的prop?  Vue3.0  props are immutable  所以还有什么用? 不会破坏单向流动吗?

    原做法是子组件修改props时通知(:update)父组件更新对应值,.sync是个语法糖?