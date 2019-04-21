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

## call stack（调用堆栈）
在当前执行环境中遇到函数时，会将函数压入一个栈（一种数据结构）中并执行函数，若函数中还有函数调用，会继续将遇到的函数压入栈中并执行，函数执行完后会将其弹出栈

## 图解三者关系

## 代码示例
```
console.log(a)

function say() {
  console.log('say')
}

var a = 'aa';
var b = 'bb';

console.log(b)
```