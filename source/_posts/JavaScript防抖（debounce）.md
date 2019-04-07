---
title: JavaScript防抖（debounce）
date: 2019-04-07 12:28:44
tags: JavaScript
categories: JavaScript专题系列
---
## 前言
在日常的开发中，经常会遇到事件频繁触发，比如：
* 表单元素的`change`
* `window`的`scroll`、`resize`
* `mousemove`等

举例：现在有一个搜索框，但是没有搜索按钮，搜索触发点是在输入关键字的时候

```
// React写法
// index.jsx

class Index extends Component {
    constructor(props) {
        super(props)
        state = {
            value: ''
        }
    }
    onChange(e) {
        search(e.target.value) // 假设search是有个已经实现的搜索函数
    }
    render() {
        return (
            <div>
                <form>
                    <input value={this.state.value} onChange={this.onChange.bind(this)} />
                </form>
            </div>
        )
    }
}
```
上面的例子每次输入值都会调用this.onChange函数，我们想要的效果是主动触发搜索，但是没有必要每次输入都去执行，解决这个问题有两种方法：

1. 防抖
2. 节流

## 防抖
### 原理
> 你尽管出发触发事件，但是我一定在事件触发n秒后才执行，如果你在一个事件触发的 n 秒内又触发了这个事件，那我就以新的事件的时间为准，n 秒后才执行，总之，就是要等你触发完事件 n 秒内不再触发事件，我才执行，真是任性呐!
----- [冴羽](https://github.com/mqyqingfeng/Blog/issues/22)

所以针对上面的例子，只需要在所有内容输入完后再去执行this.onChange，条件是输入时停顿的时间不能大于设置的wait时间，否则就认为你已经输入完成

### 实现

#### 第一版
```
function debounce(func, wait) {
    let timer;
    return function (...args) {
        if (timer) clearTimeout(timer)
        
        timer = setTimeout(() => {
            func.apply(this, args)
        }, wait);
    }
}
```
调用：
```
onChange={debounce(this.onChange)}
```

第一版已经实现了防抖功能，但是还存在缺陷，这个防抖只能适用于输入这种后触发，有些事件可能需要立即触发，比如resize、mousemove等，所以我们还需要再加一个immediate选项，表示是否立即调用

#### 第二版

```
function debounce(func, wait, immediate) {

    let timer;

    return function (...args) {
        const context = this;

        if (timer) clearTimeout(timer);
        if (immediate) {
            // 如果已经执行过，不再执行
            let callNow = !timer;
            timer = setTimeout(function(){
                timer = null;
            }, wait) // 触发wait时间后，timer为null，才可以重新触发
            if (callNow) func.apply(context, args)
        }
        else {
            timer = setTimeout(function(){
                func.apply(context, args)
            }, wait);
        }
    }
}
```
两种效果：
* 立即执行：immediate为真时，立即执行，停止触发n秒后才能再次执行
* 延迟执行：immediate为假时，事件停止触发n秒后执行