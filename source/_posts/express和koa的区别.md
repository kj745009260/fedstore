---
title: express和koa的区别
date: 2021-02-26 15:10:07
tags: express/koa
categories:
  - node.js
---

虽然express.js有着精妙的中间件设计，但是以当前js标准来说，这种精妙的设计在现在可以说是太复杂。里面的层层回调和递归，不花一定的时间还真的很难读懂。

而koa2的代码呢？简直可以用四个字评论：精简彪悍！仅仅几个文件，用上最新的js标准，就很好实现了中间件，代码读起来一目了然。

## __1.express用法和koa用法简单展示__

如果你使用express.js启动一个简单的服务器，那么基本写法应该是这样：

```javascript
  const express = require('express');
  const app = express();
  const router = express.Router();

  app.use(async (req, res, next) => {
    console.log('I am the first middleware');
    next();
    console.log('first middleware end calling');
  })

  app.use(async (req, res, next) => {
    console.log('I am the second middleware');
    next();
    console.log('second middleware end calling');
  })

  router.get('/api/test1', async (req, res, next) => {
    console.log('I am the router middleware => /api/test1');
    res.status(200).send('hello');
  })

  app.use('/', router);

  app.listen(3000);

  console.log('server listening at port 3000')
```

换算成等价的koa2，那么用法是这样的：

```javascript
  const koa = require('koa');
  const Router = require('koa-router');

  const app = new koa();
  const router = Router();

  app.use(async (ctx, next) => {
    console.log('I am the first middleware')
    await next()
    console.log('first middleware end calling')
  })

  app.use(async (ctx, next) => {
    console.log('I am the second middleware')
    await next()
    console.log('second middleware end calling')
  })

  router.get('/api/test1', async(ctx, next) => {
    console.log('I am the router middleware => /api/test1')
    ctx.body = 'hello'
  })

  app.use(router.routes());

  app.listen(3000);

  console.log('server listening at port 3000')
```

二者的使用区别通过表格展示如下: 

![use](/images/nnnkkk.png)

上表展示了二者的使用区别，从初始化就看出koa语法都是用的新标准。在挂载路由中间件上也有一定的差异性，这是因为二者内部实现机制的不同。其他都是大同小异的了。

在理念上，Koa 旨在 “修复和替换节点”，而 Express 旨在 “增加节点”。 Koa 使用Promise(JavaScript一种异步手段)和异步功能来摆脱回调地狱的应用程序，并简化错误处理。 它暴露了自己的 ctx.request 和 ctx.response 对象，而不是 node 的 req 和 res 对象。

另一方面，Express 通过附加的属性和方法增加了 node 的 req 和 res 对象，并且包含许多其他 “框架” 功能，如路由和模板，而 Koa 则没有。

因此，Koa 可被视为 node.js 的 http 模块的抽象，其中 Express 是 node.js 的应用程序框架。

因此，如果您想要更接近 node.js 和传统的 node.js 样式编码，那么您可能希望坚持使用Connect/Express 或类似的框架。 如果你想摆脱回调，请使用 Koa。

由于这种不同的理念，其结果是传统的 node.js “中间件”（即“（req，res，next）”的函数）与Koa不兼容。 你的应用基本上要重新改写了。

Koa 与 Connect/Express 有哪些不同?

### __基于 Promises 的控制流程__

没有回调地狱。

通过 try/catch 更好的处理错误。

无需域。

### __Koa 非常精简__

不同于 Connect 和 Express, Koa 不含任何中间件.

不同于 Express, 不提供路由.

不同于 Express, 不提供许多便捷设施。 例如，发送文件.

Koa 更加模块化.

### __Koa 对中间件的依赖较少__

例如, 不使用 “body parsing” 中间件，而是使用 body 解析函数。

### __Koa 抽象 node 的 request/response__

减少攻击。

更好的用户体验。

恰当的流处理。

### __Koa 路由（第三方库支持）__

由于 Express 带有自己的路由，而 Koa 没有任何内置路由，但是有 koa-router 和 koa-route 第三方库可用。同样的, 就像我们在 Express 中有 helmet 保证安全, 对于 koa 我们有 koa-helmet 和一些列的第三方库可用

## __2.koa2中间件__

![use](/images/k1.gif)

看完这个gif图，也可以思考下如何实现的。根据表现，可以猜测是next是一个函数，而且返回的可能是一个promise，被await调用。

### __2.1阅读koa-compose源码__

```javascript
  function compose(middleware) {
    if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!');
    for (const fn of middleware) {
      if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!');
    }

    /**
     * @param {Object} context
     * @return {Promise}
     * @api public
     */

    return function(context, next) {
      // last called middleware #
      let index = -1;
      // 取出第一个中间件函数执行
      return dispatch(0);

      // 递归函数
      function dispatch(i) {
        if (i <= index) return Promise.reject(new Error('next() called multiple times'));
        index = i;
        let fn = middleware[i];

        // next的值为undefined,当没有中间件的时候直接结束
        // 其实这里可以去掉next参数，直接在下面fn = void 0,和之前的代码效果一样
        // if (i === middleware.length) fn = void 0;
        if (i === middleware.length) fn = next;

        if (!fn) return Promise.resolve();

        try {
          // fn为当前执行的中间件函数
          // 当前中间件函数执行时传入的next参数为下一个中间件
          // 当所有的中间件都执行完毕时, 当前中间件传入的next参数是请求处理的回调
          return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
        } catch (err) {
          return Promise.reject(err);
        }
      }
    };
  }
```

上面的代码等价于

```javascript
  // 这样就可能更好理解了。
  // simpleKoaCompose
  const [fn1, fn2, fn3] = this.middleware;
  const fnMiddleware = function(context){
      return Promise.resolve(
        fn1(context, function next(){
          return Promise.resolve(
            fn2(context, function next(){
                return Promise.resolve(
                    fn3(context, function next(){
                      return Promise.resolve();
                    })
                )
            })
          )
      })
    );
  };


  fnMiddleware(ctx).then(handleResponse).catch(onerror);
```

也就是说koa-compose返回的是一个Promise，Promise中取出第一个函数（app.use添加的中间件），传入context和第一个next函数来执行。

第一个next函数里也是返回的是一个Promise，Promise中取出第二个函数（app.use添加的中间件），传入context和第二个next函数来执行。

第二个next函数里也是返回的是一个Promise，Promise中取出第三个函数（app.use添加的中间件），传入context和第三个next函数来执行。

第三个...

以此类推。最后一个中间件中有调用next函数，则返回Promise.resolve。如果没有，则不执行next函数。
这样就把所有中间件串联起来了。这也就是我们常说的洋葱模型。

## __3.koa2 和 koa1 的简单对比__

koa1中主要是generator函数。koa2中会自动转换generator函数。

app.use时有一层判断，是否是generator函数，如果是则用koa-convert暴露的方法convert来转换重新赋值，再存入middleware，后续再使用。

koa-convert源码挺多，核心代码其实是这样的。

```javascript
  function convert(){
  return function (ctx, next) {
      return co.call(ctx, mw.call(ctx, createGenerator(next)))
    }
    function * createGenerator (next) {
      return yield next()
    }
  }
```

最后还是通过co来转换的。所以接下来看co的源码。

```javascript
  // 写一个请求简版请求
  function request(ms= 1000) {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve({name: '若川'});
      }, ms);
    });
  }

  // 获取generator的值
  function* generatorFunc(){
    const res = yield request();
    console.log(res, 'generatorFunc-res');
  }
  generatorFunc(); // 报告，我不会输出你想要的结果的
```

简单来说co，就是把generator自动执行，再返回一个promise。
generator函数这玩意它不自动执行呀，还要一步步调用next()，也就是叫它走一步才走一步。

所以有了async、await函数。

```javascript
  // await 函数 自动执行
  async function asyncFunc(){
      const res = await request();
      console.log(res, 'asyncFunc-res await 函数 自动执行');
  }
  asyncFunc(); // 输出结果
```

也就是说co需要做的事情，是让generator向async、await函数一样自动执行。

最终来看下co源码

```javascript
  function co(gen) {
    var ctx = this;
    var args = slice.call(arguments, 1)

    // we wrap everything in a promise to avoid promise chaining,
    // which leads to memory leak errors.
    // see https://github.com/tj/co/issues/180
    return new Promise(function(resolve, reject) {
      // 把参数传递给gen函数并执行
      if (typeof gen === 'function') gen = gen.apply(ctx, args);
      // 如果不是函数 直接返回
      if (!gen || typeof gen.next !== 'function') return resolve(gen);

      onFulfilled();

      /**
       * @param {Mixed} res
       * @return {Promise}
       * @api private
       */

      function onFulfilled(res) {
        var ret;
        try {
          ret = gen.next(res);
        } catch (e) {
          return reject(e);
        }
        next(ret);
      }

      /**
       * @param {Error} err
       * @return {Promise}
       * @api private
       */

      function onRejected(err) {
        var ret;
        try {
          ret = gen.throw(err);
        } catch (e) {
          return reject(e);
        }
        next(ret);
      }

      /**
       * Get the next value in the generator,
       * return a promise.
       *
       * @param {Object} ret
       * @return {Promise}
       * @api private
       */

      // 反复执行调用自己
      function next(ret) {
        // 检查当前是否为 Generator 函数的最后一步，如果是就返回
        if (ret.done) return resolve(ret.value);
        // 确保返回值是promise对象。
        var value = toPromise.call(ctx, ret.value);
        // 使用 then 方法，为返回值加上回调函数，然后通过 onFulfilled 函数再次调用 next 函数。
        if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
        // 在参数不符合要求的情况下（参数非 Thunk 函数和 Promise 对象），将 Promise 对象的状态改为 rejected，从而终止执行。
        return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
          + 'but the following object was passed: "' + String(ret.value) + '"'));
      }
    });
  }
```

## 总结

> koa-compose是将app.use添加到middleware数组中的中间件（函数），通过使用Promise串联起来，next()返回的是一个promise。

> koa-convert 判断app.use传入的函数是否是generator函数，如果是则用koa-convert来转换，最终还是调用的co来转换。

> co源码实现原理：其实就是通过不断的调用generator函数的next()函数，来达到自动执行generator函数的效果（类似async、await函数的自动自行）。

> koa框架总结：主要就是四个核心概念，洋葱模型（把中间件串联起来），http请求上下文（context）、http请求对象、http响应对象。

### __koa洋葱模型怎么实现的。__

> app.use() 把中间件函数存储在middleware数组中，最终会调用koa-compose导出的函数compose返回一个promise，中间函数的第一个参数ctx是包含响应和请求的一个对象，会不断传递给下一个中间件。next是一个函数，返回的是一个promise。

### __如果中间件中的next()方法报错了怎么办。__

```javascript
  ctx.onerror = function {
    this.app.emit('error', err, this);
  };

  listen(){
    const  fnMiddleware = compose(this.middleware);
    if (!this.listenerCount('error')) this.on('error', this.onerror);
    const onerror = err => ctx.onerror(err);
    fnMiddleware(ctx).then(handleResponse).catch(onerror);
  }
  onerror(err) {
    // 代码省略
    // ...
  }
```

> 中间件链错误会由ctx.onerror捕获，该函数中会调用this.app.emit('error', err, this)（因为koa继承自events模块，所以有'emit'和on等方法），可以使用app.on('error', (err) => {})，或者app.onerror = (err) => {}进行捕获。

### __co的原理是怎样的。__

> co的原理是通过不断调用generator函数的next方法来达到自动执行generator函数的，类似async、await函数自动执行。