---
title: Debug下查看webpack编译过程
date: 2020-06-23 23:08:29
tags: webpack
categories: webpack
---


我们知道在浏览器的source面板中可以进行断点调试，但是webpack编译过程是在Nodejs环境中进行的，怎么才能打断点查看编译过程呢？<br />
<br />vscode提供了断点调试功能，包括Chrome和Nodejs，下面介绍一下怎么去配置<br />1、首先用vscode打开你要调试的项目<br />2、键盘F5键或者点击菜单（Run->Start Debugging），会出现下图<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/211977/1591275442418-1dc8ea4a-493b-4c08-ab6b-b4ad1966cddc.png#align=left&display=inline&height=188&margin=%5Bobject%20Object%5D&name=image.png&originHeight=376&originWidth=508&size=25660&status=done&style=none&width=254)<br />然后点击创建一个launch.js文件，内容如下：
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node", // node环境
            "request": "launch",
            "name": "Launch Program",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "program": "${workspaceFolder}\\index.js" // 开始执行的文件
        }
    ]
}
```
然后检查type对应的环境和program对应的开始执行文件，在你的index.js文件相应的位置打上debugger，再次按F5，就会从index.js开始执行到debugger的地方<br />
<br />下面用webpack编译举个例子<br />1、创建一个文件夹 `webpack-debug` ,    运行 `npm init -y` ,然后 `npm i webpack -D` <br />2、在根目录下创建一个app.js文件，简单添加点内容
```javascript
// app.js
const app = 'app'

export default app
```

<br />3、配置webpack.config.js
```javascript
module.exports = {
    mode: 'none',
    entry: './app.js',
    output: {
        filename: '[name].js',
        path: __dirname + '/lib',
    }
}
```
4、添加启动脚本文件start.js
```javascript
const webpack = require('webpack')
const config = require('./webpack.config')

debugger

const compiler = webpack(config)

debugger

compiler.run(err => {
    console.log(err)
})
```
5、根据前面介绍的添加launch.json文件
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "program": "${workspaceFolder}\\start.js" // 从start.js开始启动
        }
    ]
}
```
6、F5进行调试<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/211977/1591276773092-bbe6bda2-491c-4db8-bff0-a8799e2e6371.png#align=left&display=inline&height=422&margin=%5Bobject%20Object%5D&name=image.png&originHeight=843&originWidth=1259&size=190426&status=done&style=none&width=629.5)<br />
<br />可以使用step into ![image.png](https://cdn.nlark.com/yuque/0/2020/png/211977/1591276862852-b9dd8b5f-bcad-4fba-ac09-c7ce6ee27d21.png#align=left&display=inline&height=44&margin=%5Bobject%20Object%5D&name=image.png&originHeight=87&originWidth=339&size=5696&status=done&style=none&width=169.5)查看每一步调用

本节代码：[链接](https://github.com/wstreet/webpack-debug)

webpack基本原理可查看这篇：[webpack基本原理](https://www.yuque.com/streetex/msp6tb/dgx90d)
