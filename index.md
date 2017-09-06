---
title: JavaScript现代化异步
---
# JavaScript现代化异步

## 异步

asynchronous

允许程序在不等待某操作完成时，继续后续的操作

假设有以下代码

```javascript
// code 1
// 这里一定是异步吗？
ajax('http://github.com/good.txt', data => {
  console.log(data)
})
console.log('ajax called')
```

如果以上ajax函数是同步进行，程序需要等到这个函数结束后才能继续运行，页面会出现无响应状态。

问题：为什么有些函数是同步的

通常异步处理一些耗时的操作

## 回调

callback

通过函数参数传递到其他代码的，某一块可执行代码的引用

以下的`cb`都是回调吗？

```javascript
// code 2
const cb = () => console.log('haha')
setTimeout(cb, 1000)
```

```javascript
// code 3
const cb = () => console.log('cb')
const a = () => {
  cb()
}
a()
```

## 回调的问题

### callback hell
### 先来看一段代码

假设一下函数都是异步函数，函数的调用顺序是什么

```javascript
funcA(() => {
  funcB(() => {
    funcC(() => {
      funcD()
    })
    funcE()
  })
})
funcF()

```

#### 回调地狱

```javascript
// code 4
setTimeout(name => {
  show(`welcome ${name}`)
  setTimeout(name => {
    show(`welcome ${name}`)
    setTimeout(name => {
      show(`welcome ${name}`)
      setTimeout(name => {
        show(`welcome ${name}`)
        setTimeout(name => {
          show(`welcome ${name}`)
        }, 1000, 'tom')
      }, 1000, 'hanmeimei')
    }, 1000, 'lilei')
  }, 1000, 'jim')
}, 1000, 'jack')
```

#### “门”
#### 竞态
####