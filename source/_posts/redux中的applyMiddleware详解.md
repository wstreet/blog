---
title: redux中的applyMiddleware详解
date: 2019-04-03 20:21:10
tags:
  - Redux
  - React
categories: React
---
``` JavaScript
// applyMiddleware.js
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    // fn1 = next => action => {}
    
    // fn2 = next => action => {}
    // fn3 = next => action => {}
    // chain = [fn1, fn2, fn3]
    dispatch = compose(...chain)(store.dispatch)
    // compose(...chain) ========> (...args) => fn1(fn2(fn3((...args)))
    // dispatch = compose(...chain)(store.dispatch) ========> (...args) => fn1(fn2(fn3(...args)))(store.dispatch)    =====>   fn1(fn2(fn3(store.dispatch)))

    return {
      ...store,
      dispatch
    }
  }
}
```

> `applyMiddleware`返回了一个`createStore`方法，这个方法和`createStore.js`中的createStore方法比起来到底强在哪里？

> 答案 ： 增强的dispatch方法！

最终返回的`store`中的`dispatch`方法是将原始的裸体`store.dispatch`经过每一层中间件包裹形成的，就像穿衣服一样，下面将详细介绍

## 中间件
中间件的样子就像下面这样，传入store，返回一个dispatch方法

```
const middlewareName = store => next => action => {}
```


## applyMiddleware
核心代码如下：
```
// 创建一个store
const store = createStore(...args)

// 构造一个像中间件中传递的store
const middlewareAPI = {
    getState: store.getState,
    dispatch: (...args) => dispatch(...args)
}

// const m1 = store => next => action => {}
// const m2 = store => next => action => {}
// const m3 = store => next => action => {}
// const middlewares = [m1, m2, m3]
const chain = middlewares.map(middleware => middleware(middlewareAPI))
// 返回一个列表， 内容如下
// 假设:
// dis1 = next => action => {}
// dis2 = next => action => {}
// dis3 = next => action => {}

// chain = [dis1, dis2, dis3]


// 关于compose可在另一篇文章中查看
// https://wstreet.github.io/blog/2019/04/02/redux%E4%B8%ADcompose%E8%AF%A6%E8%A7%A3/
dispatch = compose(...chain)(store.dispatch)

//            compose(...chain)                 ===> (...args) => dis1(dis2(dis3(...args)))
// dispatch = compose(...chain)(store.dispatch) ===> (...args) => dis1(dis2(dis3(...args)))(store.dispatch) ===> dis1(dis2(dis3(store.dispatch)))
```
代码分析到这里一目了然，最后的dispatch是执行中间件列表生成的复杂dispatch