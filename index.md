---
title: JavaScript现代化异步
---
# JavaScript现代化异步

## 异步

asynchronous

允许程序在不等待某操作完成时，继续后续的操作

假设有以下代码

```javascript
// code 1
// 这里一定是异步吗？
ajax('http://github.com/good.txt', data => {
  console.log(data)
})
console.log('ajax called')
```

如果以上ajax函数是同步进行，程序需要等到这个函数结束后才能继续运行，页面会出现无响应状态。

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

### 信任问题

```javascript
function myTimeout(sec, cb) {
  setTimeout(() => {
    doSth()
    cb()
  }, sec * 1000)
}

myTimeout(1000, () => {
  console.log('done')
})
```

`cb`作为我们自己写的函数，被传进myTimeout之后，就不受我们自己控制了，而是myTimeout的实现者来控制调用，那就会出现信任问题，如果在myTimeout里，我写出了bug，调用了`cb`多次，那么最后的输出结果为多个`done`，同理没有被调用的话，则没有输出

### 同步or异步？

以下函数实现如果已经获取到值，则立即返回，如果没有获取到，则先获取再通知

```javascript
let num = null
let a = 0

function getA(cb) {
  if (num) {
    a = num / 2
    return cb(a)
  }
  ajax('http://geta.com/', n => {
    num = n
    a = num / 2
    cb(a)
  })
}
```

上面的函数，有时候是同步的，有时候是异步的，一般来说不会出问题，但如果我有以下代码

```javascript
let num = null
let a = 0

getA(cb)

console.log(a)
```

上面代码，num的初始值决定了第三句`console.log(a)`的输出，如果初始值不为`null`，则输出`num/2`，如果为`null`，则输出0

### 不直观

先来看一段代码

假设以下函数都是异步函数，函数的调用顺序是什么

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

### 回调地狱

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

### 门

门：所有人都到齐了，才能进门

也就是所有异步操作都完成了，才能进行下一步操作

在下面的函数异步取两个值，最后对这两个值进行运算后输出

```javascript
let a = null
let b = null

function aGetted(num) {
  a = num / 2
  if (a && b) {
    cacl(a, b)
  }
}

function bGetted(num) {
  b = num * 2
  if (a && b) {
    calc(a, b)
  }
}

function calc(a, b) {
  console.log(a + b)
}

ajax('http://geta.com/', aGetted)
ajax('http://getb.com/', bGetted)
```

### 竞态

竞态：竞争一方赢了，则结束

只处理第一个完成的异步操作

```javascript
let getted = false

function aGetted(num) {
  if (getted === false) {
    console.log(num / 2)
    getted = true
  }
}

function bGetted(num) {
  if (getted === false) {
    console.log(num * 2)
    getted = true
  }
}

ajax('http://geta.com/', aGetted)
ajax('http://getb.com/', bGetted)
```

## Promise

先来看一下Promise的典型使用方法

```javascript
const promise = new Promise((resolve, reject) => {
  ajax('http://example.com/', data => {
    if (data.status === 0) {
      resolve(data.payload)
    } else {
      reject(data.status)
    }
  })
})

promise.then(data => {
  console.log(data)
}).catch(error => {
  console.error(error)
})
```

!! 仍然使用回调技术，只不过规定了回调形式，传递必要数据给我们自己处理

### 三种状态

- Pending：进行中
- Fulfilled：成功
- Rejected：拒绝

对象的状态不受外部改变

一旦状态改变，就不会再变

### 强制回调

Promise的构造函数传入一个回调，该回调规定接受两个回调函数，`resolve`和`reject`，分别表示操作成功和失败，如果调用了resolve，状态从Peding变为Fulfilled，如果调用了reject，状态从Pending变为Rejected，如果没有调用，则一直是Pending

resolve函数决定promise.then函数被调用，reject函数决定promise.catch函数被调用，如果一起为Pending状态，则不会结束，没有函数被调用

所以机制上，不会造成我们传进去的回调由于实现者BUG的原因，造成没有被调用或者被多调用的情况

### 强制异步

Promise内部的状态只会变化一次，如果第一次从Pending变成了Fulfilled，那么后续的.then，也依然是异步调用

### 链式调用

我们可以像以下调用一样，对Promise做链式调用

```javascript
Promise.then((a) => {
  // print name a
}).then((b) => {
  // print name b
})
```

对应上面的回调地狱，我们可以这样写

```javascript
function welcome(time, name) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      show(`welcome ${name}`)
      resolve()
    })
  })
}

welcome(1000, 'jack').then(() => {
  return welcome(1000, 'jim')
}).then(() => {
  return welcome(1000, 'lilei')
}).then(() => {
  return welcome(1000, 'hanmeimei')
}).then(() => {
  return welcome(1000, 'tom')
})
```

这样从人类可角度看，是非常易读的

### Promise.all

我们来看一下Promise如何解决门问题

```javascript
function ajaxPromise(url) {
  return new Promise((resolve, reject) => {
    ajax(url, num => {
      resolve(num)
    })
  })
}

const promiseA = ajaxPromise('http://geta.com')
const promiseB = ajaxPromise('http://getb.com')

Promise.all([promiseA, promiseB]).then(result => {
  const sum = result.reduce((a, b) => {
    return a + b
  })
  console.log(sum)
})
```

### Promise.race

```javascript
function ajaxPromise(url) {
  return new Promise((resolve, reject) => {
    ajax(url, num => {
      resolve(num)
    })
  })
}

const promiseA = ajaxPromise('http://geta.com')
const promiseB = ajaxPromise('http://getb.com')

Promise.race([promiseA, promiseB]).then(result => {
  console.log(result)
})
```
