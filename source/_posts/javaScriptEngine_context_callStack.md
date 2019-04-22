---
title: Javascript引擎、执行上下文和调用栈
date: 2019-04-22 01:34:00
tags: JavaScript
categories: JavaScript进阶
---

## 概览
本文旨在记录自己学习V8引擎、执行上下文和调用堆栈是什么，以及他们之间的关系，通过阅读本文，你将会了解JavaScript是如何运行的，在项目中也会编写更好的程序。如有错误请在评论中指出。

## V8
由 Google 构建的 V8 引擎是开源的，用 C ++ 编写。 此引擎被用在 Google Chrome 中。 与其他引擎不同的是，V8 也被用于流行的 Node.js 中。

V8引擎是一个比较流行的JavaScript解释器，它主要包含两个重要的组件：
* Memory Heap （内存堆），内存分配的地方
* Call Stack （调用堆栈），码执行时栈帧存放的位置

![V8](http://ww1.sinaimg.cn/large/006NiFm7ly1g2askmgg4bj30sg0lcq3z.jpg)

## 执行上下文（Execution Context）
引擎在加载完代码后会解析代码，创建一个执行环境，形成一个作用域，这个执行环境便是执行上下文。
执行上下文分为3中：
* 全局执行上下文（全局作用域）
* 函数内部执行上下文（局部作用域）
* eval,不推荐使用

## call stack（调用堆栈）
在当前执行环境中遇到函数时，会将函数压入一个栈（一种数据结构）中并执行函数，若函数中还有函数调用，会继续将遇到的函数压入栈中并执行，函数执行完后会将其弹出栈

## 图解三者关系
> 本文只介绍引擎、执行上下文、调用堆栈之间的关系，关于Event Loop会在后续文章中介绍

如上图所展示一样:
![](http://ww1.sinaimg.cn/large/006NiFm7ly1g2btjlesiuj30nh0budfz.jpg)
1. 引擎在加载完代码后，创建全局执行上下文
2. 解析代码，在代码中找到函数申明和变量定义语句，将变量和函数存储在堆内存中
3. 执行代码，如果遇到函数将其压入call stack中并执行函数first()
4. 每当函数执行时，又会创建一个局部执行上下文（局部作用域）
5. 如果再次遇到函数，重复2-4步骤
6. 函数执行完后，此函数将被从调用栈中弹出

## 代码示例
```
console.log(a)

function say() {
  console.log('say')
}

var a = 'aa';
var b = 'bb';

say()
```

* js引擎加载完这段代码，创建一个全局执行上下文
* 解析代码，找到定义的变量和申明的函数，将其存入堆内存中
```
// Memory Heap

a: undefined
b: undefined
say: function() {}

```
* 执行代码，首先将`console.log(a)`压入栈中并执行，结果为`undefined`。
![](http://ww1.sinaimg.cn/large/006NiFm7ly1g2btw1wq9gj306q074dfm.jpg)

* 执行完后将其弹出栈
![](http://ww1.sinaimg.cn/large/006NiFm7ly1g2bu05gy5mj30ai07hdfn.jpg)

* 函数say引用和变量a,b赋值
```
// Memory Heap

a: 'aa'
b: 'bb'
say: function() {console.log('say')}

```

* 将`say()`压入call stack中并执行，这个过程中又会创建一个局部作用域
![](http://ww1.sinaimg.cn/large/006NiFm7ly1g2bub8sgc0j30dj06p0sk.jpg)

* 执行`say()`中的代码时遇到`console.log('say')`，将其压入call stack中
![](http://ww1.sinaimg.cn/large/006NiFm7ly1g2bue4eaz8j30il07xjr9.jpg)

* 执行完`console.log('say')`将其弹出栈，继续执行`say()`中剩余的代码
![](http://ww1.sinaimg.cn/large/006NiFm7ly1g2bug49bhej30mm07t747.jpg)

* 执行完`say()`将其弹出栈，继续执行全局执行上下文中剩余的代码，直至执行完

![](http://ww1.sinaimg.cn/large/006NiFm7ly1g2buka7hz5j30ps08mdfr.jpg)

* 关闭浏览器，销毁全局执行上下文

本文结束，希望内容对正在浏览的你有所帮助，如有错误，希望指出。

参考文章：
* [js的基础(平民理解的执行上下文/调用堆栈/内存栈/值类型/引用类型)](https://www.cnblogs.com/jiebba/p/9897287.html)
* [解读 JavaScript 之引擎、运行时和堆栈调用](https://www.oschina.net/translate/how-does-javascript-actually-work-part-1)