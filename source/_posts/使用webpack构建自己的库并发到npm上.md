---
title: 使用webpack构建自己的库并发到npm上
date: 2019-08-31 15:14:25
tags: 
 - webpack
 - npm
categories: webpack
---

## 闲话
前几天看见大佬公众号推文[Virtual Dom && Diff](https://mp.weixin.qq.com/s/PA-p5GewzMh3esSUDLcOTw),于是mark下来自己实现了一遍（抄了一遍[v-dom](https://github.com/wstreet/v-dom)），又恰好在这期间学习了如何使用webpack打包自己的库并发布到npm上，于是便将二者结合起来学习，本文章将介绍如何使用webpack构建自己的库。

## webpack
webpack打包的原理是从入口处文件开始查找依赖，并通过你的配置规则，对文件进行整合、转换、压缩等操作。具体的底层实现，笔者将在后面继续学习。

## 目录结构

```javascript
│  .gitignore
│  index.js // 从npm上下载后import的文件
│  package.json
│  README.md
│  webpack.config.js // webpack配置文件
│
├─dist // 存放打包后文件的目录
└─src // webpack打包的目录
    constant.js
    diff.js // 入口文件
    element.js
    index.js
    listDiff.js
    patch.js
    utils.js

```
webpack通过从入口文件src/index.js开始查找依赖文件进行打包，输出到dist目录，当我们从npm上下载这个包之后的使用方法一般是`import vdom from 'vdom'`，这里import的便是从根目录下的index.js中引入的，当然在package.json中也要进行相应的配置。最后输出到dist目录中的有两个文件，一个是没有压缩的`vdom.js`在开发环境使用，另一个是经过压缩的`vdom.min.js`在生产环境使用



## webpack配置
先上车再解释
```Javascript
module.exports = {
    entry: {
        'vdom': './src/index.js',
        'vdom.min': './src/index.js',
    },
    output: {
        filename: '[name].js',
        path: __dirname + '/dist',
        library: "vdom",
        libraryExport: 'default',
        libraryTarget: 'umd'
    }
}
```
因为dist之后需要输入两份代码，所以我们需要两个入口，并且输出文件的名称也是`entry`中配的`name`，但是如果直接用这份配置去打包，最后dist中`vdom.js`和`vdom.min.js`都是一样的内容，而且是经过压缩的。

因为上面的配置没有设置`mode`，webpack打包时会以`production`的模式打包


解决方法：在webpack的配置中加入`mode: 'none'`和`optimization`

```Javascript
// 先 npm install terser-webpack-plugin -D

const TerserPlugin = require('terser-webpack-plugin')

module.exports = {
    mode: 'none',
    entry: {
        'vdom': './src/index.js',
        'vdom.min': './src/index.js',
    },
    output: {
        filename: '[name].js',
        path: __dirname + '/dist',
        library: "vdom",
        libraryExport: 'default',
        libraryTarget: 'umd' // umd格式
    },
    optimization: {
        minimize: true,
        minimizer: [
            new TerserPlugin({
                include: /\.min\.js$/ // 只有匹配到.min.js结尾的文件才会压缩
            })
        ]
    }
}
```
```json
// package.json

{
  "name": "v-dom",
  "version": "1.0.0",
  "description": "a demo of v-dom",
  "main": "index.js",
  "scripts": {
    "build": "webpack",
    "prepublish": "npm run build",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "terser-webpack-plugin": "^1.4.1",
    "webpack": "^4.39.2",
    "webpack-cli": "^3.3.7",
    "webpack-dev-server": "^3.8.0"
  }
}
```

通过上面的配置，我们在终端运行`npm run build`便会将我们的库打包好，剩下的就是发到npm上

> uglifyjs-webpack-plugin也能压缩js，但是对于es6并不会转换，而terser-webpack-plugin会将es6语法进行转换

## 发布npm包
首先要有npm账号和密码，如果没有账号先去[npm](https://www.npmjs.com/signup)注册，然后在命令行进行登陆

```
npm login
```

输入账号密码进行登陆然后运行
```
npm publish
```

发布成功