---
title: redux中compose详解
date: 2019-04-02 19:39:28
tags:
  - Redux
  - React
categories: React
---
redux中`compose`的代码一共9行，最核心代码只有1行

```
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

#### Array.prototype.reduce
Array.prototype.reduce是一个神奇的方法，为了搞清楚redux中的compose方法，我们研究一下reduce先，

##### 定义和用法
reduce() 方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值。

reduce() 可以作为一个高阶函数，用于函数的 compose。

注意: reduce() 对于空数组是不会执行回调函数的。

##### 语法
> array.reduce(function(total, currentValue, currentIndex, arr), initialValue)

##### 参数说明
* function  必需。累加器函数
  * total  必需。初始值, 或者计算结束后的返回值
  * currentValue  必需。当前元素
  * currentIndex  可选。当前元素的索引
  * arr  可选。当前元素所属的数组对象
* initialValue  可选。传递给函数的初始值

> ##### 注意
> 回调函数第一次执行时，total  和 currentValue 的取值有两种情况：

> 1. 调用 reduce 时提供initialValue，total  取值为 initialValue ，currentValue 取数组中的第一个值；
>2. 没有提供 initialValue ，total  取数组中的第一个值，currentValue 取数组中的第二个值。

#### compose
现在我们回过头来看一下compose函数
```
// redux中compose方法
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```
我们让`let funcs = [fn1, fn2, fn3, fn4]`；

累加函数`(a, b) => (...args) => a(b(...args))`第一次执行的时候，传入fn1和fn2，紧接着又返回一个函数` (...args) => fn1(fn2(...args)`作为下一次执行时的total，也就是这里的a;

累加函数第二次执行的时候也是返回一个函数` (...args) => fn1(fn2(fn3(...args)))`，以此类推，最后的结果是`(...args) => fn1(fn2(fn3((fn4(...args))))`。