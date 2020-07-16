---
layout:
title: Event Loop
date: 2020-07-16 22:42:00
tags:
---

JavaScript最初设计的是浏览器脚本语言，用来操作DOM实现交互的，所以他被设计成单线程的。那为什么是单线程的，试想一下，如果是多线程，一个线程修改DOM，一个线程删除DOM，那最后的浏览器也不知道以哪个线程的结果为准，所以决定了JavaScript只能是单线程的，即同一时间只能做一件事。<br />

<a name="Y5lOs"></a>
## 任务队列
在JavaScript中，将任务分为了微任务和宏任务：<br />宏任务：script，setTimeout，setInterval，I/O，UI render<br />微任务：promise.then，process.nextTick， MutationObserver

对于同步任务，直接在主线程中执行，但是遇到异步任务时，并不会立即执行，而是将异步任务添加到对应的任务队列中（宏任务队列 macroTask queue，微任务队列microTask queue）。队列是一种先进先出的线性表结构，队列从队尾插入元素，队首删除元素

每一次事件循环都会经过一下步骤：<br />1、执行一个宏任务（浏览器中第一次执行的时候时script中的代码）<br />2、遇到异步任务时将任务添加到**对应的**任务队列中<br />3、当前宏任务执行完毕后，主线程读取微任务队列中的任务并依次执行<br />4、微任务队列中的任务执行完毕后，读取下一个宏任务又从步骤1开始，一直循环，直到任务队列中没有任务

<a name="nN4nu"></a>
## 题目分析
```javascript
setTimeout(function setTimeout1() {
	console.log('setTimeout1')
}, 0);  //1宏任务
setTimeout(function setTimeout2() {
	  console.log('setTimeout2');
    Promise.resolve().then(() => {
        console.log('promise3');
        Promise.resolve().then(() => {
            console.log('promise4');
        })
        console.log(5)
    })
    setTimeout(() => console.log('setTimeout4'), 0);  //4宏任务
}, 0); // 2宏任务
setTimeout(() => console.log('setTimeout3'), 0);  //3宏任务
Promise.resolve().then(() => {//1微任务
    console.log('promise1');
})
```
结果分析：首先执行这段代码时已经是在运行一个宏任务了，<br />将第1个setTimeout回调添加到宏任务队列，将第2个setTimeout回调添加到宏任务队列，将第3个setTimeout回调添加到宏任务队列，Promise.resolve().then的回调添加到微任务队列，此时这个script的宏任务运行完了，主线程开始读取微任务队列并依次执行，**打印promise1**，微任务队列为空，开始读取宏任务setTimeout1并执行，**打印setTimeout1**，检查微任务队列为空，继续读取宏任务setTimeout2，**打印setTimeout2**，并将promise3的回调添加到微任务队列，再向宏任务队列中添加setTimeout4回调，开始读取微任务列表并执行，**打印promise3**，将promise4回调添加到微任务队列，**打印5**，读取微任务列表并执行，**打印promise4**，微任务列表为空，读取第3个宏任务setTimeout3回调并执行，**打印setTimeout3**，微任务列表为空，读取第4个宏任务setTimeout4回调并执行，**打印setTimeout4**，所以最终结果为：<br />promise1，setTimeout1，setTimeout2，promise3，5，promise4，setTimeout3，setTimeout4<br />

