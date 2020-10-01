---
title: freecodecamp
date: 2020-06-25 16:22:01
tags:
- js
- 算法
excerpt: FreeCodeCamp 算法题
---

## 1.sumAll

给一个含两项的数组,计算这个闭区间的和

1. for循环

   ```js
   function sumAll(arr){
       let res = 0;
       let min = Math.min(...arr);
       let max = Math.max(...arr);
       for (let i = min;i<= max;i++){
           res += i
       }
       return res
   }
   ```

2. 高斯公式, 等差数列 (a0+an)*n/2 = sum

   ```js
   function sumAll(arr){
       return (arr[0]+arr[1])*(Math.abs(arr[0]-arr[1])+1) / 2
   }
   ```

## 2.diffArray

给两个数组,取差集

利用filter实现数组去重:

arr1.filter((item,index,arr)=>{

 arr.indexOf(item) === index

})

//利用indexOf找到第一个就会停止并返回该索引的特性

1. 抽离for循环

   ```js
   function diffArray(arr1, arr2) {
     return getDiffOf(arr1,arr2).concat(getDiffOf(arr2,arr1))
   }
   
   function getDiffOf(res,target){
     let result = []
     for(let i = 0;i<res.length;i++){
       if(!target.includes(res[i])){
         result.push(res[i])
       }
     }
     return result
   }
   ```

2. 一行搞定(思路:先连接数组,再找. 或者先找区别,再连接起来)

   ```js
   //先找区别再连接
   function diffArray(arr1,arr2){
       return arr1.filter(i=>!arr2.includes(i)).concat(arr2.filter(i=>!arr1.includes(i)))
   }
   //用filter代替了for
   //先连接再找区别
   function diffArr(arr1,arr2){
       return arr1.concat(arr2).filter(i=>!arr1.includes(i) || !arr2.includes(i))
   }
   function diffArr2(arr1,arr2){
       return [...new Set(arr1.concat(arr2))].filter(i=>!arr1.includes(i) || !arr2.includes(i))
   }
   //去重,影响可能不大.针对单个数组内的重复元素
   ```

## 3.Aim and Destroy

给一个数组和不定个要删除的数, 返回已经删掉要求数的数组

destroyer(arr,arg1,arg2,arg3){}

* 取到要删除的元素,arguments.
* for-if 循环判断(!?) 或者filter

```js
//朴素解法
function destroyer(arr){
    let result = []
    let toDelete = Array.from(arguments).slice(1)  
    //或者[...new Set(argument)].splice(1)或者Array.prototype.splice.call(arguments).splice(1)
    //或者[].splice.call(arguments,1)
    for(let i=0;i<arr.length;i++){
        if(!toDelete.includes(arr[i])){
            result.push(arr[i])
        }
    }
    return result
}
//简单解法,思路是一样的
function destroyer2(arr){
    let toDelete = Array.from(arguments).splice(1)
    return arr.filter(item=>!toDelete.includes(item))
}
```

## WhatIsInName

给个对象数组,和一个对象x作为参数, 找出数组内包含对象x所有属性和值的对象

例如:[{"name":"what","age":"22"},

​          {"name":"the","age":"30"},

​          {"name":"hell","age":"40"}]

找出{"age":"40"}的对应对象

whatIsInName(collection,source){//...}

思路:1.循环collection

2. 根据source的键,去对比collection每个对象和source的值 //这里自动去除了collection对象有而source对象没有的属性.因为key都是source的

3. **设置一个flag,判断是否全部键/值完全相等, 一遇到值不相等的就将它设置为false**

4. 第二层循环完了,shouldPush没被设置为false,则将该对象放进结果数组

   ```js
   function whatIsInName(collection,source){
       var arr = [];
       for(let i = 0;i<collection.length;i++){
           let shouldPush = true;
           for(const key in source){        //可以抽出来, return false, true
               if(collection[i][key] !== source[key]){
                   shouldPush = false
               }
           }
           if(shouldPush){
               arr.push(collection[i])
           }
       }
       return arr
   }
   ```

   ```js
   function whatIsInAName(collection,source){
       return collection.filter(item=>Object.keys(source).every(key=>item[key] === source[key]))
   }
   //绝了
   ```

   