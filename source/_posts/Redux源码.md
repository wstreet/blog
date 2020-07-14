---
title: Redux源码
date: 2020-07-14 22:58:02
tags: 
  - React
  - Redux
categories:
  - React
  - Redux
---

React更偏向于构建UI，虽然也能进行状态管理，但是面对大型复杂的项目，往往会面临跨组件通信，这时候就可以使用Redux。Redux是一种状态管理容器，他可以将组件用到的所有数据都保存在store中。<br />

<a name="sJbhM"></a>
## createStore
createStore(reducer, initialState, enhancer)用于创建一个store，第一个参数reducer用来修改store中的state，initialState为初始的state，enhancer是一个增强器，如果项目使用middleware的话，enhancer就是applyMiddleware的返回值，如果没有用middleware，则enhancer不传，

```javascript
export default function createStore(reducer, initialState, enhancer) {
 	// ...
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }
		// 如果存在enhancer：enhancer(createStore)返回一个增强的（使用了middleware）createStore
    return enhancer(createStore)(reducer, initialState)
  }
	// ...
  function dispatch(action) {
    

    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    return action
  }
  
  // State 是只读的，唯一改变 state 的方法就是触发 action，
  // reducer根据action返回一个新的state
  function getState() {
    return currentState
  }
  //如果不存在enhancer，则返回的store是这样的
  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```
<a name="07H7Y"></a>
## applyMiddleware
我们在上面说过了，applyMiddleware创建一个store的增强器，之所以增强，是因为dispatch被增强了，他是在原来的dispatch上附加了很多功能，具体是怎么实现的的，我们一起往下看<br />
<br />先给出中间件的写法
```javascript
const middleware1 = middlewareAPI => next => action => {
  console.log('m1 start')
  const result =  next(action)
  console.log('m1 end')
  
  return result
}

const middleware2 = middlewareAPI => next => action => {
  console.log('m2 start')
  const result =  next(action)
  console.log('m2 end')
  
  return result
}
```


```javascript

export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    // 将上面两个middleware传入applyMiddleware，得到
    // chain = [dispatch1, dispatch2]
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}

// chain = [chain1, chain2]
const chain1 = next => action => {
  console.log('m1 start')
  const result =  next(action)
  console.log('m1 end')
  
  return result
}

const chain2 = next => action => {
  console.log('m2 start')
  const result =  next(action)
  console.log('m2 end')
  
  return result
}

```
<a name="zqUKF"></a>
## compose
compose方法最核心的就是使用Array.prototype.reduce，具体使用方法可以查看[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)，经过reduce之后得到的结果
```javascript
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}

 // compose(f, g, h) ====> (...args) => f(g(h(...args)))
 // composeResult = compose(...chain) = (...args) => chain1(chain2(...args))
// dispatch = chain1(chain2(defaultDispatch))
```
<a name="PnIaj"></a>
## 新dispatch
我们用defaultDispatch表示最开始创建的dispatch(createStore中的dispatch)，来看一下最后经过middleware包装过的dispatch长啥样
```javascript
const chain1 = next => action => {
  console.log('m1 start')
  const result =  next(action)
  console.log('m1 end')
  
  return result
}

const chain2 = next => action => {
  console.log('m2 start')
  const result =  next(action)
  console.log('m2 end')
  
  return result
}

dispatch = chain1(chain2(defaultDispatch))

dispatch2 = action => {
  console.log('m2 start')
  const result =  defaultDispatch(action)
  console.log('m2 end')
  
  return result
}

dispatch1 = action => {
  console.log('m1 start')
  const result =  dispatch2(action)
  console.log('m1 end')
  
  return result
}

dispatch = action => {
  console.log('m1 start')
  // const result =  dispatch2(action)
  // ----------------------------------
  console.log('m2 start')
  const result =  defaultDispatch(action)
  console.log('m2 end')
 // ----------------------------------
  console.log('m1 end')
  
  return result
}
```
这里也展示了middleware是洋葱模型，中间件的访问顺序是外 => 内 => 外，最内部其实是调用最原生的(createStore中定义的)dispatch，来调用reducer生成最新的state<br />
<br />另外可以看到middlewareAPI中的dispatch是
```javascript
const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
```
而不是
```javascript
const middlewareAPI = {
      getState: store.getState,
      dispatch: dispatch
    }
```
是因为这里使用的是闭包，这样可以将经过中间件生成新的dispatch传给每一个中间件。
