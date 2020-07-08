---
title: 实现一个Event事件机制
date: 2020-07-08 22:37:24
tags: Javascript
categories: Javascript进阶
---


发布订阅模式在JavaScript中非常重要，可以用于模块件的通讯，DOM事件机制就是发布订阅模式的体现。接下来手动实现一个Event模块，实现以下功能

| 方法名称 | 功能 |
| --- | --- |
| on(event, listener) | 为指定事件添加一个监听器 |
| emit(event, [arg1], [arg2], [...]) | 按监听器的顺序执行执行每个监听器，如果事件有注册监听返回 true，否则返回 false |
| once(event, listener) | 为指定事件注册一个单次监听器，即 监听器最多只会触发一次，触发后立刻解除该监听器。 |
| off(event, listener) | 移除指定事件的某个监听器，监听器必须是该事件已经注册过的监听器。 它接受两个参数，第一个是事件名称，第二个是回调函数名称。  |
| listeners(event) | 返回指定事件的监听器数组。 |

```javascript
class EventEmitter {
  constructor() {
    this._eventsMap = {}
  }

  
  /**
   * 添加事件监听
   *
   * @param {*} event
   * @param {*} listener
   * @param {boolean} [once=false]
   * @memberof EventEmitter
   */
  on(event, listener, once = false) {
    if (!this._eventsMap[event]) {
      this._eventsMap[event] = []
    }
    const listeners = this._eventsMap[event]
    if (!this._eventsMap[event].includes(listener)) {
      this._eventsMap[event].push(listener)
      listener.once = once
    }
    
  }


  /**
   * 触发事件
   *
   * @param {*} event
   * @param {*} args
   * @returns
   * @memberof EventEmitter
   */
  emit(event, ...args) {
    const listeners = this._eventsMap[event]
    const _this = this
    if (listeners && listeners.length > 0) {
      listeners.forEach(listener => {
        listener.call(_this, ...args)
        if (listener.once) {
          this.off(event, listener)
        }
      });
      return true
    }
    return false
  }

  
  /**
   * 添加事件监听，并只能触发一次
   *
   * @param {*} event
   * @param {*} listener
   * @memberof EventEmitter
   */
  once(event, listener) {
    this.on(event, listener, true)
  }



  /**
   * 取消事件监听
   *
   * @param {*} event
   * @param {*} listener
   * @memberof EventEmitter
   */
  off(event, listener) {
    if (this._eventsMap[event]) {
      if (!listener) {
        this._eventsMap[event] = []
      } else {
        this._eventsMap[event] = this._eventsMap[event].filter(_listener => _listener !== listener)
      }
    }
  }

}
```
