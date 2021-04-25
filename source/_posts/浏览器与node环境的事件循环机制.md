---
title: 浏览器与node环境的事件循环机制
date: 2020-10-14 16:50:38
tags: 事件循环
categories:
  - 浏览器特性
---

## __JavaScript中事件循环__

JavaScript中事件循环，主要就在理解宏任务和微任务这两种异步任务。

宏任务（macrotask）

> script (可以理解为外层同步代码)
> setTimeOut / setInterval 
> setImmediate(node环境 或 IE10以上) 
> I/O (node环境)
> 各种callback、 
> UI渲染 / UI事件
> postMessage / messageChannel等

优先级：主代码块 > setImmediate > postMessage > setTimeOut/setInterval

微任务（microtask）

> process.nextTick 
> Promise 
> MutationObserver 
> async(实质上也是promise)

优先级：process.nextTick > Promise > MutationOberser

### __执行分区：__

我们常常吧EventLoop中分为 内存、执行栈、WebApi、异步回调队列(包括微任务队列和宏任务队列)

![事件循环](/images/ssss1.png)

事件处理过程（关于macrotask和microtask的理解）：

![事件循环](/images/ssss2.png)

## __Nodejs中事件循环__

Node.js也是单线程的Event Loop，但是它的运行机制不同于浏览器环境。

![事件循环](/images/ssss3.png)

先看一段代码

```javascript
  setTimeout(()=>{
    console.log('timer1')

    Promise.resolve().then(function() {
      console.log('promise1')
    })
  }, 0)

  setTimeout(()=>{
    console.log('timer2')

    Promise.resolve().then(function() {
      console.log('promise2')
    })
  }, 0)

  //浏览器输出结果
  // timer1
  // promise1
  // timer2
  // promise2

  //Node输出结果
  // timer1
  // timer2
  // promise1
  // promise2
```

### __Node.js的事件循环__

Node.js采用V8作为js的解析引擎，而I/O处理方面使用了自己设计的libuv，libuv是一个基于事件驱动的跨平台抽象层，封装了不同操作系统一些底层特性，对外提供统一的API，事件循环机制也是它里面的实现。

根据Node.js官方介绍，每次事件循环都包含了6个阶段，对应到 libuv 源码中的实现，如下图所示：

![事件循环](/images/ssss4.png)

> 1.timers 阶段：这个阶段执行timer（setTimeout、setInterval）的回调
> 2.I/O callbacks 阶段：执行一些系统调用错误，比如网络通信的错误回调
> 3.idle, prepare 阶段：仅node内部使用
> 4.poll 阶段：获取新的I/O事件, 适当的条件下node将阻塞在这里
> 5.check 阶段：执行 setImmediate() 的回调
> 6.close callbacks 阶段：执行 socket 的 close 事件回调

我们重点看timers、poll、check这3个阶段就好，因为日常开发中的绝大部分异步任务都是在这3个阶段处理的。

timers 阶段

timers 是事件循环的第一个阶段，Node 会去检查有无已过期的timer，如果有则把它的回调压入timer的任务队列中等待执行，

事实上，Node 并不能保证timer在预设时间到了就会立即执行，因为Node对timer的过期检查不一定靠谱，它会受机器上其它运行程序影响，或者那个时间点主线程不空闲。

比如下面的代码，setTimeout() 和 setImmediate() 的执行顺序是不确定的。

```javascript
  setTimeout(() => {
    console.log('timeout')
  }, 0)

  setImmediate(() => {
    console.log('immediate')
  })
```

但是把它们放到一个I/O回调里面，就一定是 setImmediate() 先执行，因为poll阶段后面就是check阶段。

poll 阶段

poll 阶段主要有2个功能：

> 1.处理 poll 队列的事件
> 2.当有已超时的 timer，执行它的回调函数

even loop将同步执行poll队列里的回调，直到队列为空或执行的回调达到系统上限（上限具体多少未详），接下来even loop会去检查有无预设的setImmediate()，分两种情况：

1.若有预设的setImmediate(), event loop将结束poll阶段进入check阶段，并执行check阶段的任务队列
2.若没有预设的setImmediate()，event loop将阻塞在该阶段等待

注意一个细节，没有setImmediate()会导致event loop阻塞在poll阶段，这样之前设置的timer岂不是执行不了了？所以咧，在poll阶段event loop会有一个检查机制，检查timer队列是否为空，如果timer队列非空，event loop就开始下一轮事件循环，即重新进入到timer阶段。

check 阶段

setImmediate()的回调会被加入check队列中， 从event loop的阶段图可以知道，check阶段的执行顺序在poll阶段之后。

小结

> 1.event loop 的每个阶段都有一个任务队列
> 2.当 event loop 到达某个阶段时，将执行该阶段的任务队列，直到队列清空或执行的回调达到系统上限后，才会转入下一个阶段
> 3.当所有阶段被顺序执行一次后，称 event loop 完成了一个 tick

现在，我们再来看Node.js 与浏览器的 Event Loop 差异

回顾上一篇，浏览器环境下，microtask的任务队列是每个macrotask执行完之后执行。

![事件循环](/images/ssss5.png)

而在Node.js中，microtask会在事件循环的各个阶段之间执行，也就是一个阶段执行完毕，就会去执行microtask队列的任务。

![事件循环](/images/ssss6.png)

最初demo回顾

回顾文章最开始的demo，全局脚本（main()）执行，将2个timer依次放入timer队列，main()执行完毕，调用栈空闲，任务队列开始执行；

![事件循环](/images/ssss7.gif)

首先进入timers阶段，执行timer1的回调函数，打印timer1，并将promise1.then回调放入microtask队列，同样的步骤执行timer2，打印timer2；

至此，timer阶段执行结束，event loop进入下一个阶段之前，执行microtask队列的所有任务，依次打印promise1、promise2。

对比浏览器端的处理过程：

![事件循环](/images/ssss8.gif)

process.nextTick() VS setImmediate()

来自官方文档有意思的一句话，从语义角度看，setImmediate() 应该比 process.nextTick() 先执行才对，而事实相反，命名是历史原因也很难再变。

总结

1.Node.js 的事件循环分为6个阶段
2.浏览器和Node 环境下，microtask 任务队列的执行时机不同
Node.js中，microtask 在事件循环的各个阶段之间执行
浏览器端，microtask 在事件循环的 macrotask 执行完之后执行
3.递归的调用process.nextTick()会导致I/O starving，官方推荐使用setImmediate()