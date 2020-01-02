---
title: new的模拟实现
date: 2020-01-02 21:22:35
tags:
categories:
---

```

/**
 * new 一个构造函数之后，在构造函数内部做了如下几件事
 * 1、创建一个Object实例
 * 2、改变原型链
 * 3、调用构造函数，并将它的this指向第一步创建的实例
 * 4、返回结果
 */


function myNew() {
    const angs = [...arguments]


    // 1、创建一个实例
    const obj = new Object()


    // 2、改变实例原型链，实现继承
    const Con = angs.shift()
    obj.__proto__ = Con.prototype


    // 调用构造函数，并将内部this指向实例
    const ret = Con.aplly(obj, args)


    // 如果构造函数返回的结果是对象，则返回这个对象
    // 否则返回第一步创建的实例对象
    if (typeof ret === 'object') {
        return ret
    }

    return obj
}

```