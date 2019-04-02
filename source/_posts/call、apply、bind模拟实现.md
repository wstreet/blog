---
title: call、apply、bind模拟实现
date: 2019-04-01 19:06:47
tags:
  - JavaScript
categories: JavaScript
---
### call、apply、bind
#### 区别：
call和apply都是为了解决this的指向，作用相同，只是传参方式不同。除了第一个参数外，call可以接受一个参数列表，apply只接收一个参数数组。

```
let a = {
    value: 1
}

function getValue(anme, age) {
    console.log(name)
    console.log(age)
    console.log(this.value)
}
getValue.call(a, 'md', 24)
getValue.apply(a, ['md', 24])
```

#### 模拟实现call和apply
参考以下几点考虑如何实现：
* 不传入第一个参数，默认this指向window
* 改变this指向后，让新的对象执行该函数，那么思路是否可以变成给新的对象添加一个函数，然后在执行后删除。

```
// call的实现思路
Function.prototype.myCall = function(context, ...rest) {
    const context = context || window
    
    // 给context添加一个新属性
    // getValue.call(a, 'yck', '24') => a.fn = getValue
    context.fn = this

    const result = context.fn(...rest)
    // 删除fn
    delete context.fn
    return result
}
```
```
// apply的实现思路
Function.prototype.myApply = function(context) {
    const context = context || window
    context.fn = this
    
    let result
    // 判断是否存在第二个参数
    // 如果存在，就将第二个参数展开
    if(arguments[1]) {
        result = context.fn(...arguments[1])
    } else {
        result = context.fn()
    }
    
    delete context.fn
    
    return result
}
```

bind和其他两个方法作用是一样的，只是该方法会返回一个函数。并且我们可以通过bind实现柯里化。

```

var value = 0;
var obj = {
   value : 1,
}
function show(name){
    console.log(this.value);
    console.log(name);
}

show('007') 
// 0
// 007
show.bind(obj, '步行街')
// 1
// 步行街
show.bind(null, '步行街')
// 0
// 步行街

```
#### bind模拟实现


##### 返回函数的模拟实现
```
Function.prototype.myBind = function(context) {
    const context = context || window
    return function() {
        this.apply(context)
    }
}

```
##### 传参模拟实现
```
Function.prototype.myBind = function(context) {
    const context = context || window
    
    var self = this
    
    var args = [...arguments].splice(1)
    
    
    
    return function() {
        return self.apply(context, args.concat(...arguments))
    }
}

```

##### 构造函数效果的模拟实现
>模拟完以上功能还有一个最难的部分没有实现：一个绑定函数也能使用`new`操作符创建对象

```
Function.prototype.myBind = function(context) {
    if (typeof this !== 'function') {
        throw new Error('Error')
    }
    const context = context || window
    
    var self = this
    
    var args = [...arguments].splice(1)
    
    
    
    return function F() {
    if (this instanceof F) {
        return new self(...args, ...arguments)
    }
        return self.apply(context, args.concat(...arguments))
    }
}
```