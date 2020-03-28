---
title: 模拟class继承
date: 2019-12-28 17:39:53
tags: JavaScript
categories: JavaScript进阶
---


在模拟实现class继承之前，如果还不熟悉JavaScript的继承，可以看一下这篇文章JavaScript继承
首先看一下ES6的继承
```
// ES6继承
class Animal {
  static footer = 'footer'
  constructor(opt) {
    this.name = opt.name
  }
  getName() {
    return this.name
  }
}
class Cat extends Animal {
  constructor(opt) {
    super(opt)
    this.age = opt.age
  }
  getAge() {
    return this.age
  }
}
const cat = new Cat({ name: 'miao', age: 5 })
```

在上面代码中，Cat继承Animal，通过使用extends继承Animal原型上的方法，通过super调用父构造函数
> 关于super：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/super

> super([arguments]); 
// 调用 父对象/父类 的构造函数

> super.functionOnParent([arguments]); 
// 调用 父对象/父类 上的方法

我们要想模拟实现ES6继承，关键看ES6 class extends最终转成ES6是什么样，这里急用Babel工具：在线转换，
摘出最主要的代码
```
function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function");
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: { value: subClass, writable: true, configurable: true }
  });
  if (superClass) _setPrototypeOf(subClass, superClass);
}
// ES6继承
var Animal =
  /*#__PURE__*/
  (function() {
    function Animal(opt) {
      _classCallCheck(this, Animal);
      this.name = opt.name;
    }
    _createClass(Animal, [
      {
        key: "getName",
        value: function getName() {
          return this.name;
        }
      }
    ]);
    return Animal;
  })();
_defineProperty(Animal, "footer", "footer");
var Cat =
  /*#__PURE__*/
  (function(_Animal) {
    _inherits(Cat, _Animal);
    function Cat(opt) {
      var _this;
      _classCallCheck(this, Cat);
      _this = _possibleConstructorReturn(
        this,
        _getPrototypeOf(Cat).call(this, opt)
      );
      _this.age = opt.age;
      return _this;
    }
    _createClass(Cat, [
      {
        key: "getAge",
        value: function getAge() {
          return this.age;
        }
      }
    ]);
    return Cat;
  })(Animal);
var cat = new Cat({
  name: "miao",
  age: 5
});
```
其中最主要的是 _inherits(Cat, _Animal); 和 _getPrototypeOf(Cat).call(this, opt) ，看到这里我们发现有点像寄生组合式继承。别急，还有跟他不一样的地方.

我们运行转换后的代码，看一下Cat.__proto__
![](https://cdn.nlark.com/yuque/0/2019/png/211977/1577524705049-4c0a8f74-23de-4670-89e3-95639def1ac0.png)

发现Cat.__proto__是指向Animal的，在转换后的代码中也能找到对应的地方，就是在_inherits函数最后一句

所以es6 class的继承实现就是在寄生组合式继承的基础上将子类的__proto__属性指向父类，为什么要这样做？

因为寄生组合式继承没法继承父类的静态方法

 模拟实现：
 ```
function inherit(Sub, Sup) {
  // Object.create和原型式继承一样，返回一个参数对象的副本
  // 寄生式继承+原型链继承
  const prototype = Object.create(Sup.prototype)
  
  Sub.prototype = prototype   // 改变子类构造函数的prototype
  Sub.prototype.constructor = Sub
  
  Sub.__proto__ = Sup
}
function SupType(name) {
  this.name = name
  this.colors = ['red']
}
SupType.prototype.getName = function(){
  return this.name
};
function SubType(name,age) {
  this.age = age
  // 构造函数继承
  SupType.call(this, name)
}
inherit(SubType, SupType)
SubType.prototype.getAge = function() {
  return this.age
}
const sub = new SubType('streetex', 25)
```