---
title: webpack-notes
date: 2020-05-28 16:08:37
tags:
- webpack
categories:
- webpack
- 整理
---

## 1.html和压缩

###  html-webpack-plugin

基本配置:

```js
//webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports={
   plugins:[
       new HtmlWebpackPlugin({
           template:'./src/index.html',
           filename:'outputFileName',
           hash:true,      //添加到html文件的js/css文件会有哈希值结尾，防止缓存？
           minify:{     //pass boolean to enable or disable 
              collapseWhiteSpace :true, //pass object to specify configurations
              removeComments:true,
              removeAttributeQuotes:true,
              minifyCSS:true,
              minifyJS:true,     //压缩<style>和<script> 没看到区别!?
              removeReduncdantAttributes:true,
              useShrotDoctype:true,
              removeScriptTypeAttributes:true,
              removeStyleLinkTypeAttributes:true
           }   //前三个可能更常用       
       })
   ]
}
```

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
plugins:[
    new CleanWebapckPlugin()  //不用传参,默认清除的是output里面的path
]
```

## CSS加载和压缩

### 加载: style-loader,css-loader

```js
module.exports={
    module:{
        rules:[
            {
                test:/\.css$/,
                loader:['style-loader','css-loader']
            }
        ]
    }
}
//问题:用styleloader会覆盖自己写的,在head标签的style(style-loader加在了head最后)
```

### 处理url:url-loader

```js
module:{
    rules:[
        {
            test:/\.(png|jpg|jpeg|gif)$/,
            loader:'url-loader'
        }
    ]
}
```



### 行内CSS的提取:mini-css-extract-plugin

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module.exports={
    module:{
        rules:[
            {
                test:/\.css/,
                use:[
                    MiniCssExtractPlugin.loader,
                    'css-loader'
                ]
            }
        ]
    },
    plugin:[
        new MiniCssExtractPlugin({
            filename:'main.css'     //指定抽取后的css文件名称
        })
    ]
}
```

* 跟style-loader的区别是这个变成了link使用,而style-loader以Style标签添加到了html文件中.

* 既要在module.rules里使用,也要在plugin里面new一个实例.

  ???打包一次出一个新的css文件!?

### CSS样式添加前缀以适应浏览器 postcss-loader,autoprefixer

* 仅用于低版本使用CSS3特性. 就是加了个前缀
* package.json里面用browserslist指明浏览器版本号, 或者 postcss.config.js里面require('autoprefixer')({"browsers":[/** 浏览器 **/]}) //都是数组

```js
//webpack.config.js
module:{
    rules:[
        {
            test:/\.(css|scss)/,
            loader:[
                'style-loader',
                'css-loader',
                'sass-loader',
                'postcss-loader'    //注意顺序post->pre->css
            ]
        }
    ]
}

//postcss.config.js      与webpack.config.js同级
module.exports={
    plugins:[
        require('autoprefixer')
    ]
}

//package.json
"browserslist": [
    "defaults",
    "not ie < 11",
    "last 2 versions",
    "> 1%",
    "iOS 7",
    "last 3 iOS versions"
  ],     //...  browserslist 在最后说明
//设计2-3个文件
```

## CSS文件压缩:optimize-css-assets-webpack-plugin

```js
const OpitimizeCss = require('optimize-css-assets-webpack-plugin')
module.exports={
    plugins:[
        new OptimizeCss({ /**options **/})      //官方推荐用法.也可以配合uglifyjs使用
    ]
}
```

## ES678910语法转ES5 babel

```js
//在webpack中使用
npm i -D @babel/cli @babel/core @babel/preset-env babel-loader
```

配置:

```js
//另起文件 .babelrc
{
    "presets":[
        "@babel/preset-env"
    ]
}
//或者在package.json里面
"babel":{
    "presets":[
        "@babel/preset-env"
    ]
},     //之后的html文件好像有点问题?collapseWhiteSpace呢?
```

```js
//webpack.config.js
module:{
    rules:[
        {
            test:/\.js$/,
            exclude:/node_modules/,
            loader:'babel-loader'
        }
    ]
}
```



//TODO 图片处理？ base64？ img-loader？ 

## browserslist说明

例子                       说明  

 `> 1%` 全球超过1%人使用的浏览器  

`> 5% in US` 指定国家使用率覆盖  

`last 2 versions` 所有浏览器兼容到最后两个版本根据CanIUse.com追踪的版本 

 `Firefox ESR` 火狐最新版本  

`Firefox > 20` 指定浏览器的版本范围 

 `not ie <=8` 方向排除部分版本  

`Firefox 12.1` 指定浏览器的兼容到指定版本  

`unreleased versions` 所有浏览器的beta测试版本  

`unreleased Chrome versions` 指定浏览器的测试版本  

`since 2013` 2013年之后发布的所有版本

