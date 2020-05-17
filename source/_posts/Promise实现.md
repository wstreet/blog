---
title: Promise实现
date: 2020-05-16 22:02:22
tags: JavaScript
categories: JavaScript进阶
---

```javascript

/**
*  工具方法
*/ 

 const getType = value => {
  return Object.prototype.toString.call(value).match(/^\[object (.*)\]$/)[1];
}

const isFuntion = value => {
  return getType(value) === 'Function'
} 



/**
*  Promise开始
*/ 
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'



class MyPromise {
  constructor(handler) {
    this.state = PENDING
    this.value = undefined
    this.callbacks = []

    try {
      handler(this.resolve, this.reject)
    } catch (error) {
      this.reject(error)
    }
  }

  resolve = (value) => {
    if (this.state !== PENDING) return

    this.state = FULFILLED
    this.value = value

    this.excuteCallBacks()
  }

  reject = (reason) => {
    if (this.state !== PENDING) return

    this.state = REJECTED
    this.value = reason

    this.excuteCallBacks()

  }


  handleCallback = (callback) => {
    const { onFulfilled, onRejected, resolve, reject } = callback
    if (this.state === FULFILLED) {
      isFuntion(onFulfilled) ? resolve(onFulfilled(this.value)) : resolve(this.value)
    }
    if (this.state === REJECTED) {
      isFuntion(onRejected) ? reject(onRejected(this.value)) : reject(this.value)
    }
  }

  excuteCallBacks = () => {
    this.callbacks.forEach(callback => {
      setTimeout(() => {
        this.handleCallback(callback)
      }, 0)
    })
  }


  then = (onFulfilled, onRejected) => {
    return new MyPromise((resolve, reject) => {
      const callback = { onFulfilled, onRejected, resolve, reject }
      if (this.state === PENDING) {
        this.callbacks.push(callback)
      } else {
        setTimeout(() => {
          this.handleCallback(callback)
        }, 0)
      }
    })
  }

  catch = (onRejected) => {
    const onFulfilled = () => {}
    return new MyPromise((resolve, reject) => {
      const callback = { onFulfilled, onRejected, resolve, reject }
      if (this.state === PENDING) {
        this.callbacks.push(callback)
      } else {
        setTimeout(() => {
          this.handleCallback(callback)
        }, 0)
      }
    })
  }

  finally = (callback) => {
    return this.then(
      value => MyPromise.resolve(callback()).then(() => value),
      reason => MyPromise.resolve(callback()).then(() => { throw reason })
    )
  }

  static resolve = (value) => {
    return new MyPromise((resolve, reject) => resolve(value))
  }

  static reject = (reason) => {
    return new MyPromise((resolve, reject) => reject(reason))
  }

  // 所有结果fulfilled
  static all = (promiseList) => {
    const result = []
    let count = 0
    return new MyPromise((resolve, reject) => {
      for (let index = 0; index < promiseList.length; index++) {
        const promise = promiseList[index];
        promise.then(
          res => {
            result.push(res)
            count =+ 1

            if (count === promiseList.length) {
              resolve(result)
            }
          },
          reason => {
            reject(reason)
          }
        )
      }
    })
  }

  // 返回先有结果的promise的值，不管是fulfilled还是rejected
  static race = () => {
    return new MyPromise((resolve, reject) => {
      for (let index = 0; index < promiseList.length; index++) {
        const promise = promiseList[index];
        promise.then(
          res => {
            resolve(res)
          },
          reason => {
            reject(reason)
          }
        )
      }
    })
  }

  // 任意一个变成fulfilled，结果是fulfilled
  static any = () => {
    const result = []
    let count = 0
    return new MyPromise((resolve, reject) => {
      for (let index = 0; index < promiseList.length; index++) {
        const promise = promiseList[index];
        promise.then(
          res => {
            resolve(res)
          },
          reason => {
            result.push(res)
            count =+ 1

            if (count === promiseList.length) {
              reject(reason)
            }
            
          }
        )
      }
    })
  }
}

```
