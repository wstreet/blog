---
title: 如何在canvas内部元素上监听事件
date: 2020-06-16 23:08:23
tags: canvas
categories: canvas
---

完整代码：[链接](https://github.com/wstreet/canvas-event)<br />Demo：[链接](https://wstreet.github.io/canvas-event/)<br />
<br />首先canvas绘图是要有一个 `<canvas></canvas>` 标签的，然后使用脚本取绘制。`<canvas></canvas>`  就是一个dom节点，所以我们可以在这个节点上监听一些事件，比如click。但是这样存在问题，点击画布中的任意一个地方，都会触发click事件，现在我只想点击画布中的rect时才触发click事件，该怎么做呢？其实很简单，通过以下几个步骤就可以实现。

> 这里只是用click事件和rect图形举例，其他事件和shape可以用同样的方法实现。

- 实现一个带有自定义监听(on)、触发(emit)功能的Rect类
- 用Rect类创建一个实例rect，并在rect上用on方法监听click事件
- 在canvas上用addEventListener监听click事件，handler中获取当前触发事件的点的坐标，判断是否在rect内部，如果在，用emit方法触发rect上绑定的事件处理函数



![image.png](https://cdn.nlark.com/yuque/0/2020/png/211977/1591531886533-52fa5131-0b3f-435e-967c-6c3c71f4cfa3.png#align=left&display=inline&height=504&margin=%5Bobject%20Object%5D&name=image.png&originHeight=720&originWidth=1065&size=55729&status=done&style=none&width=746)

一、Rect<br />事件机制Event
```javascript
class Event {
  constructor() {
    this._listener = {}
  }


  /**
   * 监听
   * @param {string} type
   * @param {function} handler
   * @memberof Event
   */
  on(type, handler) {
    if (!this._listener[type]) {
      this._listener[type] = []
    }
    
    this._listener[type].push(handler)
  }



  /**
   *触发
   *
   * @param {*} type
   * @param {*} event
   * @returns
   * @memberof Event
   */
  emit(type, event) {
    if (event == null || event.type == null) {
        return;
    }
    const typeListeners = this._listener[type]
    if (!typeListeners) return
    for (let index = 0; index < typeListeners.length; index++) {
      const handler = typeListeners[index];
      
      handler(event)
    }
  }


  /**
   * 删除
   *
   * @param {*} type
   * @param {*} handler
   * @memberof Event
   */
  remove(type, handler) {
    if(!handler) {
      this._listener[type] = []
      return
    }

    if (this._listener[type]) {
      const listeners = this._listeners[type];
      for (let i = 0, len = listeners.length; i < len; i++) {
          if (listeners[i] === listener) {
              listeners.splice(i, 1);
          }
      }
    }
  }
}

export default Event
```
绘制矩形Rect继承Event
```javascript
import Event from './Event.js'

class Rect extends Event {
    constructor(opts, canvas) {
        super()
        this.canvas = canvas
        this.config = opts
    }

    draw() {
        const ctx = this.canvas.ctx
        const { x, y, width, height, fillStyle } = this.config
        ctx.fillStyle = fillStyle
        ctx.fillRect(x, y, width, height)
    }

    isEventInRegion(clientX, clientY) {
        const point = this.getEventPosition(clientX, clientY); // 计算基于canvas坐标系的坐标值
        const { x, y, width, height } = this.config
        if (
            x < point.x 
            && point.x < x + width 
            && y < point.y 
            && point.y < y + height
            ) {
            return true
        }
        return false
    }

    getEventPosition(clientX, clientY) {
        const bbox = this.canvas.canvas.getBoundingClientRect();
        return {
            x: clientX - bbox.left,
            y: clientY - bbox.top
        }
    }

}

export default Rect
```
我们实现一个Rect类，将ctx.fillRect方法封装在draw方法中，在需要的时候去调用<br />
<br />二、创建Canvas类
```javascript
import Event from './Event.js'
import Rect from './Rect.js'

const eventList = [
  'click',
  'mousemove',
  // ...
]


class Canvas extends Event {
  defaultOpts = {}
  constructor(c) {
    super()
    this.canvas = c
    this.ctx = c.getContext('2d')
     
    this.children = []
    
  }

  addChild(shape) {
    this.children.push(shape)
  }

  draw() {
    this.children.forEach(shape => shape.draw())
  }
  
  rect(config) {
    const rect = new Rect(config, this)
    this.addChild(rect)
    return rect
  }
}

export default Canvas
```
首先使用Canvas类创建一个实例，参数是canvas节点
```javascript
const canvas = new Canvas(document.querySelector('#canvas'))
```
然后使用canvas.rect(config)创建一个rect，并监听click事件
```javascript
const rect = canvas.rect({
        x: 0,
        y: 0,
        width: 150,
        height: 50,
        fillStyle: '#ccc'
    })

rect.on('click', (event) => {
        console.log('rect1 click')
    })

```
现在已经在rect上监听了click事件，接下来就要考虑，在什么时候emit。答案很简单，click事件触发点在rect内部的时候。我们在Rect类中添加两个方法，getEventPosition方法将point的坐标从基于浏览器窗口转换成基于canvas，isEventInRegion方法判断当前事件触发点是否在rect内部。

转换后的坐标point(px, py)，px = clientX - bbox.left，py = clientY - bbox.top
```javascript
import Event from './Event.js'

class Rect extends Event {
    constructor(opts, canvas) {
        super()
        this.canvas = canvas
        this.config = opts
    }

    draw() {
        const ctx = this.canvas.ctx
        const { x, y, width, height, fillStyle } = this.config
        ctx.fillStyle = fillStyle
        ctx.fillRect(x, y, width, height)
    }

    isEventInRegion(clientX, clientY) {
        const point = this.getEventPosition(clientX, clientY); // 计算基于canvas坐标系的坐标值
        const { x, y, width, height } = this.config
        if (
            x < point.x 
            && point.x < x + width 
            && y < point.y 
            && point.y < y + height
            ) {
            return true
        }
        return false
    }

    getEventPosition(clientX, clientY) {
        const bbox = this.canvas.canvas.getBoundingClientRect();
        return {
            x: clientX - bbox.left,
            y: clientY - bbox.top
        }
    }

}

export default Rect
```
三、事件派发<br />在canvas上用addEventListener监听click事件，事件触发后使用rect.isEventInRegion()方法判断是否去做事件派发，如果事件触发点在rect内部，使用rect.emit(event)触发执行handler。实现一个initEvent方法，在实例化canvas的时候去执行他
```javascript
import Event from './Event.js'
import Rect from './Rect.js'

const eventList = [
  'click',
  'mousemove',
  // ...
]


class Canvas extends Event {
  defaultOpts = {}
  constructor(c) {
    super()
    this.canvas = c
    this.ctx = c.getContext('2d')
     
    this.children = []
    this.initEvent()
    
  }

  initEvent() {
    eventList.forEach(eventName => {
      this.canvas.addEventListener(eventName, this.handleEvent)
    })
  }

  handleEvent = (event) => {
    this.children
    .filter(shape => shape.isEventInRegion(event.x, event.y))
    .forEach(shape => shape.emit(event.type, event))
  }

  addChild(shape) {
    this.children.push(shape)
  }

  draw() {
    this.children.forEach(shape => shape.draw())
  }
  
  rect(config) {
    const rect = new Rect(config, this)
    this.addChild(rect)
    return rect
  }
}

export default Canvas
```

<br />总结：平时在开发中一定会出现一种场景，就是只想在某个shape上触发某个事件，而不是在canvas上任意一个地方都能触发。canvas本身的机制是只能在canvas上监听事件，子元素没法监听，我们将子元素进行封装一下，自定义一个事件机制，然后在监听canvas的handler中去做处理，符合条件其触发子元素的事件监听。
