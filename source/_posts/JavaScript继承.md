---
title: JavaScript继承
date: 2019-12-22 15:53:39
tags:
categories:
---

## 0、原型和原型链
没个构造函数都会有一个prototype属性，指向它的原型对象，原型对象有一个constructor属性，指向构造函数，实例对象会有一个内部属性__proto__指向构造函数的原型对象，原型链就是通过__proto__一形成的一个链接，链接的顶部是Object.prototype，Object.prototype的__proto__属性指向null
```
function Animal(name) {
  this.name = name
}
Animal.prototype.getName = function() {
    return this.name
}
Animal.prototype.constructor === Animal // true
const cat = new Animal('miao')
cat.__proto__ === Animal.prototype // true
Object.prototype.__proto__ === null // true
```

## 1、原型链继承
我们知道原型链是通过对象的__proto__属性形成的一条链，原型链继承自然是通过__proto__继承

```
function Animal(name) {
  this.name = name
}
Animal.prototype.getName = function() {
    return this.name
}
function Cat(name) {
  if(name) {
    this.name = name
  }
}
// 通过改变Cat的prototype属性，来改变Cat实例的__proto__指向
// 将Animal的实例赋值给Cat的prototype属性
const animal = new Animal('cat')
Cat.prototype = animal
Cat.prototype.constructor = Cat
// 可进一步扩展
// Cat.prototype.getAge = function() {} 
const cat = new Cat()
cat.getName() // cat
const myCat = new Cat('myCat')
myCat.getName() // myCat
```

cat和myCat都是Cat构造函数的实例，我们通过给Cat.prototype重新赋值Animal的实例animal,使得cat和myCat的__proto__都指向animal，而animal具有name属性，animal.__proto__指向Animal.prototype，所以最终才可以使用getName获取name。
> 总结：让子类实例的原型（__proto__）指向父类的一个实例

> 优点：可以继承父类原型对象上的属性和方法

> 缺点：1、父类的引用属性会被所有子类实例共享
          2、实例化子类时不能向父类传参
## 2、构造函数继承
构造函数继承就是在子类构造函数中执行父类构造函数，并绑定this为子类实例（将父类实例的属性复制到在子类上），上代码
```
function Animal(name) {
  this.name = name
  this.footer = 'footer'
  this.getName = function() {}
}
function Cat(name, age) {
    this.age = age
  Animal.call(this, name)
}
const cat = new Cat('miao', 3)

```
如上代码所示，构造函数继承的重点是在子类构造函数中执行父类构造函数Animal.call(this, name)，将父类实例该有的属性都复制过来
> 总结：重点在于Animal.call(this, name)

> 优点：

> 缺点：1、只能继承父类实例上该有的属性，无法继承父类实例原型上的属性和方法
          2、无法复用共有属性和方法，上述例子中，通过实例化Cat创建3个实例，三个实例中都有getName方法，造成内存浪费


## 3、组合继承
组合继承从名字就可以看出来他不是新鲜玩意，他是将上述第一种继承（原型链继承）和第二种继承（构造函数继承）组合起来的一种方法，它既可以继承到父类构造函数原型上的共有属性和方法，也可以继承到父类实例自身该有的属性和方法，不多说上代码

```
function Animal(name) {
  this.name = name
  this.footer = 'footer'
}
Animal.prototype.getName = function() {
    return this.name
}
function Cat(name, age) {
    this.age = age
  // 构造函数继承
  Animal.call(this, name)
}
// 原型链继承
const animal = new Animal('cat')
Cat.prototype = animal
// 重写Cat.prototype.constructor指向
Cat.prototype.constructor = Cat
const cat = new Cat('miao', 4)
```

通过上述代码发现，实例cat和其原型cat.__proto__（animal）上都有父类（Animal）实例该有的属性，只不过cat上的属性会屏蔽其原型上的同名属性和方法

> 总结：将原型链继承和构造函数继承结合起来

> 优点：可以将父类原型对象上的属性、方法和父类实例自身该有的属性、方法都继承过来

> 缺点：子类实例和子类实例原型上存在一部分同名属性和方法，造成内存浪费

## 4、原型式继承
原型式继承和Obje.create()方法原理一致，但是又有点区别

```
function extend(obj) {
  function F() {}
  F.prototype = obj
  return new F()
}
const person = {
  name: 'streetex',
  colors: ['red']
}
const anotherPerson = extend(person) // {name: 'streetex}
anotherPerson.name = '007'
console.log(person.name) // 'streetex'
anotherPerson.colors.push('black')
console.log(person.colors) // ['red', 'black']
```

原型式继承和原型链继承差不多，都是通过修改构造函数的prototype为指定对象，但是原型式继承将这种操作包在了函数里边，另外原型式继承实际上是将参数对象person进行了一次浅复制
> 总结

> 优点：可以复用父类方法

> 缺点：父对象的引用属性会被所有子实例共享；创建子实例时无法传参
上述缺点已被修正，就是Object.create方法

## 5、寄生式继承
寄生式继承是在原型式继承的基础上进行了增强，怎么增强？上代码

```
function extend(obj) {
  function F() {}
  F.prototype = obj
  return new F()
}
function create(obj) {
    const anotherObj = extend(obj)
  anotherObj.getName = function() {
    return this.name
  }
  
  return anotherObj
}
const person = {
  name: 'streetex',
  colors: ['red']
}
const anotherPerson = create(person)
```

可以看出寄生式继承是在函数内通过原型式继承创建一个对象，然后进行增强，最后返回
> 总结：在原型式继承的基础上进行了增强

> 优点：可以复用父类方法，并且可以进行增强

> 缺点：父对象的引用属性会被所有子实例共享；创建子实例时无法传参（和原型式继承一样）

## 6、寄生组合式继承
在第3点介绍的组合继是是原型链继承和构造函数继承的组合，它的缺点有一条是父构造函数调用了两次，造成了浪费，我认为寄生组合式继承是寄生式继承、原型链继承和构造函数继承的组合，父构造函数只会调用一次

```
function inherit(Sub, Sup) {
  // Object.create和原型式继承一样，返回一个参数对象的副本
  // 寄生式继承+原型链继承
    const prototype = Object.create(Sup.prototype)
  
  Sub.prototype = prototype   // 改变子类构造函数的prototype
  Sub.prototype.constructor = Sub
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

> 优点： 

> 缺点：无

## 7、ES6 Class extends继承
JavaScript是基于原型继承的语言，即使ES6新增了class，他依然是基于原型继承的，class只是一种语法糖，es6中的类继承的关键字式extends，并且在子类中通过调用super方法将父类实例对象上的方法和属性加到子类实例上
```
class Animal {
    constructor(name) {
    this.name = name
  }
}
class Cat extends Animal {
    constructor(name) {
    super(name)
  }
}
typeof Cat // function
```