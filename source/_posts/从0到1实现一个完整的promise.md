---
title: 从0到1实现一个完整的promise
date: 2021-01-04 17:48:31
tags: promise
categories:
  - javascript
---

## __基础框架__

new Promise()时接收一个executor函数作为参数，该函数会立即执行，函数中有两个参数，它们也是函数，分别是resolve和reject，函数同步执行一定要放在try...catch中，否则无法进行错误捕获。

```javascript
function MyPromise(executor) {
  try {
    // 这里要保证executor中传入的resolve和reject的this指向一致
    executor(this.resolve.bind(this), this.reject.bind(this))
  } catch (reason) {
    this.reject(reason)
  }
}
MyPromise.prototype.resolve = function(value) { /* ...some code */ }
MyPromise.prototype.reject = function(reason) { /* ...some code */ }
```

## __添加状态机__

> Promise是一个状态机的机制，初始状态为 pending，成功状态为 fulfilled，失败状态为 rejected。只能从 pending -> fulfilled，或者从 pending -> rejected，并且状态一旦转变，就永远不会再变了。

```javascript
function MyPromise(executor) {
  this.state = 'pending'
  try {
    // 这里要保证executor中传入的resolve和reject的this指向一致
    executor(this.resolve.bind(this), this.reject.bind(this))
  } catch (reason) {
    this.reject(reason)
  }
}
MyPromise.prototype.resolve = function(value) { 
  if (this.state === 'pending') {
    this.state = 'fulfilled'
    /* ...some code */
  }
}
MyPromise.prototype.reject = function(reason) {
  if (this.state === 'pending') {
    this.state = 'rejected'
    /* ...some code */
  }
}
```

## __添加then方法__

> + Promise拥有一个then方法，接收两个函数 onFulfilled 和 onRejected，分别作为Promise成功和失败的回调。所以，在then方法中我们需要对状态state进行判断，如果是fulfilled，则执行onFulfilled(value)方法，如果是rejected，则执行onRejected(reason)方法。
> + 由于成功值value和失败原因reason是由用户在executor中通过resolve(value) 和 reject(reason)传入的，所以我们需要有一个全局的value和reason供后续方法获取。

```javascript
function MyPromise(executor) {
  this.state = 'pending'
  this.value = null
  this.reason = null
  try {
    // 这里要保证executor中传入的resolve和reject的this指向一致
    executor(this.resolve.bind(this), this.reject.bind(this))
  } catch (reason) {
    this.reject(reason)
  }
}
MyPromise.prototype.resolve = function(value) { 
  if (this.state === 'pending') {
    this.state = 'fulfilled'
    this.value = value
    /* ...some code */
  }
}
MyPromise.prototype.reject = function(reason) {
  if (this.state === 'pending') {
    this.state = 'rejected'
    this.reason = reason
    /* ...some code */
  }
}
MyPromise.prototype.then = function(onFulfilled, onRejected) {
  if (this.state === 'fulfilled') {
    onFulfilled(this.value)
  }

  if (this.state === 'rejected') {
    onRejected(this.reason)
  }
}
```

## __实现异步调用resolve__

> 同步调用resolve()没有问题，但如果是异步调用，比如放到setTimeout中，因为目前的代码在调用then()方法时，state仍是pending状态，当timer到时候调用resolve()把state修改为fulfilled状态，但是onFulfilled()函数已经没有时机调用了。

```javascript
function MyPromise(executor) {
  this.state = 'pending'
  this.value = null
  this.reason = null
  this.fulfillCallbacks = []
  this.rejectedCallbacks = []
  try {
    // 这里要保证executor中传入的resolve和reject的this指向一致
    executor(this.resolve.bind(this), this.reject.bind(this))
  } catch (reason) {
    this.reject(reason)
  }
}
MyPromise.prototype.resolve = function(value) { 
  if (this.state === 'pending') {
    this.state = 'fulfilled'
    this.value = value

    this.fulfillCallbacks.forEach(fulfillCallback => {
      fulfillCallback()
    })
  }
}
MyPromise.prototype.reject = function(reason) {
  if (this.state === 'pending') {
    this.state = 'rejected'
    this.reason = reason
    this.rejectedCallbacks.forEach(rejectedCallback => {
      rejectedCallback()
    })
  }
}
MyPromise.prototype.then = function(onFulfilled, onRejected) {
  if (this.state === 'pending') {
    this.fulfillCallbacks.push(() => {
      onFulfilled(this.value)
    })
    this.rejectedCallbacks.push(() => {
      onRejected(this.reason)
    })
  }
  if (this.state === 'fulfilled') {
    onFulfilled(this.value)
  }

  if (this.state === 'rejected') {
    onRejected(this.reason)
  }
}
```
我们添加了两个回调函数数组onFulfilledCallbacks和onRejectedCallbacks，用来存储then()方法中传入的成功和失败回调。然后，当用户调用resolve()或reject()的时候，修改state状态，并从相应的回调数组中依次取出回调函数执行。

同时，通过这种方式我们也实现了可以注册多个then()函数，并且在成功或者失败时按照注册顺序依次执行。

## __then返回的仍是Promise__

> 读过PromiseA+规范的同学肯定知道，then()方法返回的仍是一个Promise，并且返回Promise的resolve的值是上一个Promise的onFulfilled()函数或onRejected()函数的返回值。如果在上一个Promise的then()方法回调函数的执行过程中发生了错误，那么会将其捕获到，并作为返回的Promise的onRejected函数的参数传入。

```javascript
MyPromise.prototype.then = function(onFulfilled, onRejected) {
  let promise2 = null

  // 创建一个onCall统一处理
  let onCall = (promise2, onCallFn, v, resolve, reject) => {
    try {
      let x = onCallFn(v)
      this.resolvePromise(promise2, x, resolve, reject)
    } catch (reason) {
      reject(reason)
    }
  }
  promise2 = new MyPromise((resolve, reject) => {
    if (this.state === 'pending') {
      this.fulfillCallbacks.push(() => {
        onCall(promise2, onFulfilled, this.value, resolve, reject)
      })
      this.rejectedCallbacks.push(() => {
        onCall(promise2, onRejected, this.reason, resolve, reject)
      })
    }

    if (this.state === 'fulfilled') {
      onCall(promise2, onFulfilled, this.value, resolve, reject)
    }

    if (this.state === 'rejected') {
      onCall(promise2, onRejected, this.reason, resolve, reject)
    }
  })

  return promise2
}
```

resolvePromise()是用来解析then()回调函数中返回的仍是一个Promise，这个Promise有可能是我们自己的，有可能是别的库实现的，也有可能是一个具有then()方法的对象，所以这里靠resolvePromise()来实现统一处理。

```javascript
MyPromise.prototype.resolvePromise = function(promise2, x, resolve, reject) {
  let called = false // called 防止多次调用

  if (promise2 === x) {
    return reject(new TypeError('循环引用'))
  }

  let typeX = Object.prototype.toString.call(x)
  if (typeX !== null && (typeX === '[object Object]' || typeX === '[object Function]')) {
    try {
      let then = x.then
      if (typeof then === 'function') {
        then.call(x, v => {
          if (called) return
          called = true

          this.resolvePromise(promise2, v, resolve, reject)
        }, reason => {
          if (called) return
          called = true
          reject(reason)
        })
      } else {
        if (called) return
        called = true
        resolve(x)
      }
    } catch (reason) {
      if (called) return
      called = true
      reject(reason)
    }
  } else {
    // x是普通值，直接resolve
    resolve(x)
  }
}
```

下面是翻译自PromiseA+规范关于resolvePromise()的要求：

Promise 解决过程

> Promise 解决过程是一个抽象的操作，其需输入一个 promise 和一个值，我们表示为 [[Resolve]](promise, x)，如果 x 有 then 方法且看上去像一个 Promise ，解决程序即尝试使 promise 接受 x 的状态；否则其用 x 的值来执行 promise 。

这种 thenable 的特性使得 Promise 的实现更具有通用性：只要其暴露出一个遵循 Promise/A+ 协议的 then 方法即可；这同时也使遵循 Promise/A+ 规范的实现可以与那些不太规范但可用的实现能良好共存。

运行 [[Resolve]](promise, x) 需遵循以下步骤：
> + x 与 promise 相等
>   如果 promise 和 x 指向同一对象，以 TypeError 为据因拒绝执行 promise
> + x 为 Promise
>   如果 x 为 Promise ，则使 promise 接受 x 的状态:
>     + 如果 x 处于等待态， promise 需保持为等待态直至 x 被执行或拒绝
>     + 如果 x 处于执行态，用相同的值执行 promise
>     + 如果 x 处于拒绝态，用相同的据因拒绝 promise
> + x 为对象或函数
>   如果 x 为对象或者函数：
>     + 把 x.then 赋值给 then
>     + 如果取 x.then 的值时抛出错误 e ，则以 e 为据因拒绝 promise
>     + 如果 then 是函数，将 x 作为函数的作用域 this 调用之。传递两个回调函数作为参数，第一个参数叫做 resolvePromise ，第二个参数叫做 rejectPromise:
>         + 如果 resolvePromise 以值 y 为参数被调用，则运行 [[Resolve]](promise, y)
>         + 如果 rejectPromise 以据因 r 为参数被调用，则以据因 r 拒绝 promise
>         + 如果 resolvePromise 和 rejectPromise 均被调用，或者被同一参数调用了多次，则优先采用首次调用并忽略剩下的调用
>         + 如果调用 then 方法抛出了异常 e：
>             + 如果 resolvePromise 或 rejectPromise 已经被调用，则忽略之
>             + 否则以 e 为据因拒绝 promise
>         + 如果 then 不是函数，以 x 为参数执行 promise
>     + 如果 x 不为对象或者函数，以 x 为参数执行 promise

如果一个 promise 被一个循环的 thenable 链中的对象解决，而 [[Resolve]](promise, thenable) 的递归性质又使得其被再次调用，根据上述的算法将会陷入无限递归之中。算法虽不强制要求，但也鼓励施者检测这样的递归是否存在，若检测到存在则以一个可识别的 TypeError 为据因来拒绝 promise。

参考上述规范，结合代码中的注释，相信大家可以理解resolvePromise()的作用了。

## __实现catch()方法__

then()方法的onFulfilled和onRejected回调函数都不是必传项，如果不传，那么我们就无法接收reject(reason)中的错误，这时我们可以通过链式调用catch()方法用来接收错误。

不仅如此，catch()可以作为Promise链式调用的最后一步，前面Promise发生的错误会冒泡到最后一个catch()中，从而捕获异常。

那么catch()方法到底是如何实现的呢？

答案就是在Promise的实现中，onFulfilled和onRejected函数是有默认值的：

```javascript
MyPromise.prototype.then = function(onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => { return value }
  onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }
}

MyPromise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected)
}
```

可以看到，onRejected的默认值是把错误reason通过throw抛出去。由于我们对于同步代码的执行都是在try...catch中的，所以如果Promise发生了错误，如果没传onRejected，默认的函数会把错误reason抛出，然后会被promise2捕捉到，作为reject(reason)决议。

catch()实现就是调用this.then(null, onRejected)，由于promise2被reject，所以会执行onRejected回调，于是就捕捉到了第一个promise的错误。

总结来说，then()方法中不传onRejected回调，Promise内部会默认帮你写一个函数作为回调，作用就是throw抛出reject或者try...catch到的错误，然后错误reason会被promise2作为reject(reason)进行决议，于是会被下一个then()方法的onRejected回调函数调用，而catch只是写了一个特殊的then(null, onRejected)而已。

所以，我们在写Promise的链式调用的时候，在then()中可以不传onRejected回调，只需要在链式调用的最末尾加一个catch()就可以了，这样在该链条中的Promise发生的错误都会被最后的catch捕获到。

## __实现finally方法__

finally是某些库对Promise实现的一个扩展方法，无论是resolve还是reject，都会走finally方法。

```javascript
MyPromise.prototype.finally = function(finallyCallback) {
  return this.then(value => {
    finallyCallback()
    return value
  }, reason => {
    finallyCallback()
    throw reason
  })
}
```

## __实现done方法__

done方法作为Promise链式调用的最后一步，用来向全局抛出没有被Promise内部捕获的错误，并且不再返回一个Promise。一般用来结束一个Promise链。

```javascript
MyPromise.prototype.done = function() {
  return this.catch(reason => {
    throw reason
  })
}
```

## __实现Promise.all方法__

Promise.all()接收一个包含多个Promise的数组，当所有Promise均为fulfilled状态时，返回一个结果数组，数组中结果的顺序和传入的Promise顺序一一对应。如果有一个Promise为rejected状态，则整个Promise.all为rejected。

```javascript
MyPromise.all = function(promiseList) {
  return new MyPromise((resolve, reject) => {
    let result = []
    promiseList.forEach((promise, index) => {
      promise.then(value => {
        result[index] = value

        if (result.length === promiseList.length) {
          resolve(result)
        }
      }, reject)
    })
  })
}
```

## __实现Promise.race方法__

Promise.race()接收一个包含多个Promise的数组，当有一个Promise为fulfilled状态时，整个大的Promise为onfulfilled，并执行onFulfilled回调函数。如果有一个Promise为rejected状态，则整个Promise.race为rejected。

```javascript
MyPromise.race = function(promiseList) {
  return new MyPromise((resolve, reject) => {
    promiseList.forEach(promise => {
      promise.then(value => {
        resolve(value)
      }, reject)
    })
  })
}
```

## __实现Promise.resolve方法__

Promise.resolve用来生成一个fulfilled完成态的Promise，一般放在整个Promise链的开头，用来开始一个Promise链。

```javascript
MyPromise.resolve = function(value) {
  let promise = null

  promise = new MyPromise((resolve, reject) => {
    this.prototype.resolvePromise(promise, value, resolve, reject)
  })

  return promise
}
```

由于传入的value有可能是普通值，有可能是thenable，也有可能是另一个Promise，所以调用resolvePromise进行解析。

## __实现Promise.reject方法__

Promise.reject用来生成一个rejected失败态的Promise。

```javascript
MyPromise.reject = function(reason) {
  return new MyPromise((resolve, reject) => {
    reject(reason)
  })
}
```

## __如何停止一个Promise链__

假设这样一个场景，我们有一个很长的Promise链式调用，这些Promise是依次依赖的关系，如果链条中的某个Promise出错了，就不需要再向下执行了，默认情况下，我们是无法实现这个需求的，因为Promise无论是then还是catch都会返回一个Promise，都会继续向下执行then或catch。举例：

```javascript
new Promise(function(resolve, reject) {
  resolve(1111)
}).then(function(value) {
  // "ERROR!!!"
}).catch()
  .then()
  .then()
  .catch()
  .then()
```

有没有办法让这个链式调用在ERROR!!!的后面就停掉，完全不去执行链式调用后面所有回调函数呢？

我们自己封装一个Promise.stop方法。

```javascript
MyPromise.stop = function() {
  return new MyPromise(function() {});
};
```

stop中返回一个永远不执行resolve或者reject的Promise，那么这个Promise永远处于pending状态，所以永远也不会向下执行then或catch了。这样我们就停止了一个Promise链。

但是这样会有一个缺点，就是链式调用后面的所有回调函数都无法被垃圾回收器回收。

## __完整代码__

```javascript
function MyPromise(executor) {
  this.state = 'pending'
  this.value = null
  this.reason = null
  this.fulfillCallbacks = []
  this.rejectedCallbacks = []

  try {
    executor(this.resolve.bind(this), this.reject.bind(this))
  } catch (reason) {
    this.reject(reason)
  }
}
MyPromise.prototype.resolve = function(value) {
  if (this.state === 'pending') {
    this.state = 'fulfilled'
    this.value = value
    this.fulfillCallbacks.forEach(fulfillCallback => {
      fulfillCallback()
    })
  }
}
MyPromise.prototype.reject = function(reason) {
  if (this.state = 'pending') {
    this.state = 'rejected'
    this.reason = reason
    this.rejectedCallbacks.forEach(rejectedCallback => {
      rejectedCallback()
    })
  }
}
MyPromise.prototype.then = function(onFilfilled, onRejected) {
  onFilfilled = typeof onFilfilled === 'function' ? onFilfilled : value => { return value }
  onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }
  let promise2 = null

  let callOn = (promise2, callOnFn, v, resolve, reject) => {
    try {
      let x = callOnFn(v)
      this.resolvePromise(promise2, x, resolve, reject)
    } catch (reason) {
      reject(reason)
    }
  }
  promise2 = new MyPromise((resolve, reject) => {
    if (this.state === 'pending') {
      this.fulfillCallbacks.push(() => {
        callOn(promise2, onFilfilled, this.value, resolve, reject)
      })
      this.rejectedCallbacks.push(() => {
        callOn(promise2, onRejected,  this.reason, resolve, reject)
      })
    }

    if (this.state === 'fulfilled') {
      callOn(promise2, onFilfilled, this.value, resolve, reject)
    }

    if (this.state === 'rejected') {
      callOn(promise2, onRejected, this.reason, resolve, reject)
    }
  })

  return promise2
}
MyPromise.prototype.resolvePromise = function(promise2, x, resolve, reject) {
  let called = false

  if (promise2 === x) {
    return reject(new TypeError('循环引用'))
  }

  let typeX = Object.prototype.toString(x)
  if (typeX !== null && (typeX === '[object Object]' || typeX === '[object Function]')) {
    try {
      let then = x.then
      if (typeof then === 'function') {
        then.call(x, v => {
          if (called) return
          called = true

          this.resolvePromise(promise2, v, resolve, reject)
        }, reason => {
          if (called) return
          called = true
          reject(reason)
        })
      } else {
        if (called) return
        called = true
        resolve(x)
      }
    } catch (reason) {
      if (called) return
      called = true
      reject(reason)
    }
  } else {
    resolve(x)
  }
}
MyPromise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected)
}
MyPromise.prototype.finally = function(fn) {
  return this.then(value => {
    fn()
    return value
  }, reason => {
    fn()
    throw reason
  })
}
MyPromise.prototype.done = function() {
  this.catch(reason => {
    throw reason
  })
}
MyPromise.all = function(promiseList) {
  return new MyPromise((resolve, reject) => {
    let result = []

    promiseList.forEach((promise, index) => {
      promise.then(value => {
        result[index] = value

        if (result.length === promiseList.length) {
          resolve(result)
        }
      }, reject)
    })
  })
}
MyPromise.race = function(promiseList) {
  return new MyPromise((resolve, reject) => {
    promiseList.forEach(promise => {
      promise.then(value => {
        resolve(value)
      }, reject)
    })
  })
}
MyPromise.resolve = function(value) {
  let promise;
  promise = new MyPromise((resolve, reject) => {
    this.prototype.resolvePromise(promise, value, resolve, reject)
  })
  return promise;
}
```