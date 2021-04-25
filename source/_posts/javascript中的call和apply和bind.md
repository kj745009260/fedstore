---
title: javascript中的call和apply和bind
date: 2021-03-12 16:35:55
tags: call和apply和bind
categories:
  - javascript
---

## __call,apply,bind的基本介绍__

语法:

```javascript
  fun.call(thisArg, param1, param2, ...)
  fun.apply(thisArg, [param1,param2,...])
  fun.bind(thisArg, param1, param2, ...)
```

返回值:

> call/apply：fun执行的结果
> bind：返回fun的拷贝，并拥有指定的this值和初始参数

参数: thisArg(可选)

> + fun的this指向thisArg对象
> + 非严格模式下：thisArg指定为null，undefined，fun中的this指向window对象.
> + 严格模式下：fun的this为undefined
> + 值为原始值(数字，字符串，布尔值)的this会指向该原始值的自动包装对象，如 String、Number、Boolean

param1,param2(可选): 传给fun的参数。

> + 如果param不传或为 null/undefined，则表示不需要传入任何参数.
> + apply第二个参数为数组，数组内的值为传给fun的参数。

调用call/apply/bind的必须是个函数

call、apply和bind是挂在Function对象上的三个方法,只有函数才有这些方法。

只要是函数就可以，比如: Object.prototype.toString就是个函数，我们经常看到这样的用法：Object.prototype.toString.call(data)

区别：

call与apply的唯一区别

传给fun的参数写法不同：

apply是第2个参数，这个参数是一个数组：传给fun参数都写在数组中。

call从第2~n的参数都是传给fun的。

call/apply与bind的区别

执行：

call/apply改变了函数的this上下文后马上执行该函数

bind则是返回改变了上下文后的函数,不执行该函数

返回值:

call/apply 返回fun的执行结果

bind返回fun的拷贝，并指定了fun的this指向，保存了fun的参数。

<span class="c42b983">bind</span> 是返回对应函数，便于稍后调用；<span class="c42b983">apply 、call</span> 则是立即调用 。

```javascript
  Function.prototype.myCall = function(context, ...args) {
    // 指定为 null 和 undefined 的 this 值会自动指向全局对象(浏览器中为window)
    // 值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的实例对象
    context = (context === null || context === undefined) ? window : Object(context);

    // 用于临时储存函数
    const specialPrototype = Symbol('唯一性');
    // 函数的this指向隐式绑定到context上
    context[specialPrototype] = this;
    // 通过隐式绑定执行函数并传递参数
    let res = context[specialPrototype](...args);
    // 删除上下文对象的属性
    delete context[specialPrototype];
    // 返回函数执行结果
    return res;
  }

  Function.prototype.myApply = function(context) {
    context = (context === null || context === undefined) ? window : Object(context)

    function isArrayLike(o) {
      if (o &&  // o不是null、undefined等
          typeof o === 'object' &&  // o是对象
          isFinite(o.length) && // o.length是有限数值
          o.length >= 0 && // o.length为非负值
          o.length === Math.floor(o.length) &&    // o.length是整数
          o.length < 4294967296 // o.length < 2^32
      ) {
        return true;
      }
      return false;
    }

    const specialPrototype = Symbol('唯一性');
    context[specialPrototype] = this;
    let args = arguments[1];
    let res;

    if (args) {
      if (!Array.isArray(args) && !isArrayLike(args)) {
        throw new TypeError('myApply 第二个参数不为数组并且不为类数组对象抛出错误');
      } else {
        // 执行函数并展开数组，传递函数参数
        res = context[specialPrototype](...args);
      }
    } else {
      res = context[specialPrototype]();
    }

    delete context[specialPrototype];

    return res;
  }

  Function.prototype.myBind = function() {
    let thisfn = this;
    let context = [].shift.call(arguments);
    let args = [].slice.call(arguments);

    return function() {
      let _args = [].slice.call(arguments);
      [].push.call(args, ..._args);
      thisfn.call(context, ...args)
    }
  }
```

## __apply、call__

在 javascript 中，<span class="c42b983">call</span> 和 <span class="c42b983">apply</span> 都是为了改变某个函数运行时的上下文（context）而存在的，换句话说，就是为了改变函数体内部 <span class="c42b983">this</span> 的指向。
JavaScript 的一大特点是，函数存在「定义时上下文」和「运行时上下文」以及「上下文是可以改变的」这样的概念。

```javascript
  function fruits() {}
  
  fruits.prototype = {
      color: "red",
      say: function() {
          console.log("My color is " + this.color);
      }
  }
  
  var apple = new fruits;
  apple.say();    //My color is red
```

但是如果我们有一个对象 banana= {color : "yellow"} ,我们不想对它重新定义 say 方法，那么我们可以通过 <span class="c42b983">call</span> 或 <span class="c42b983">apply</span> 用 apple 的 say 方法：

```javascript
  banana = {
      color: "yellow"
  }
  apple.say.call(banana);     //My color is yellow
  apple.say.apply(banana);    //My color is yellow
```

所以，可以看出 <span class="c42b983">call</span> 和 <span class="c42b983">apply</span> 是为了动态改变 <span class="c42b983">this</span> 而出现的，当一个 object 没有某个方法（本栗子中banana没有say方法），但是其他的有（本栗子中apple有say方法），我们可以借助<span class="c42b983">call</span> 或 <span class="c42b983">apply</span>用其它对象的方法来操作。

## __apply、call 区别__

对于 <span class="c42b983">apply</span>、<span class="c42b983">call</span> 二者而言，作用完全一样，只是接受参数的方式不太一样。例如，有一个函数定义如下：

```javascript
  var func = function(arg1, arg2) { };

  // 就可以通过如下方式来调用:

  func.call(this, arg1, arg2);
  func.apply(this, [arg1, arg2])
```

其中 <span class="c42b983">this</span> 是你想指定的上下文，他可以是任何一个 JavaScript 对象(JavaScript 中一切皆对象)，<span class="c42b983">call</span> 需要把参数按顺序传递进去，而 <span class="c42b983">apply</span> 则是把参数放在数组里。　　
为了巩固加深记忆，下面列举一些常用用法：

## __apply、call实例__

### __数组之间追加__

```javascript
  var array1 = [12 , "foo" , {name:"Joe"} , -2458]; 
  var array2 = ["Doe" , 555 , 100];

  Array.prototype.push.apply(array1, array2); 

  // array1 值为  [12 , "foo" , {name:"Joe"} , -2458 , "Doe" , 555 , 100] 
```

### __获取数组中的最大值和最小值__

```javascript
  var  numbers = [5, 458 , 120 , -215 ];

  var maxInNumbers = Math.max.apply(Math, numbers);   //458

  var maxInNumbers = Math.max.call(Math,5, 458 , 120 , -215); //458
```

number 本身没有 max 方法，但是 Math 有，我们就可以借助 call 或者 apply 使用其方法。

### __验证是否是数组（前提是toString()方法没有被重写过）__

```javascript
  functionisArray(obj){ 
    return Object.prototype.toString.call(obj) === '[object Array]' ;
  }
```

### __类（伪）数组使用数组方法__

```javascript
  var domNodes = Array.prototype.slice.call(document.getElementsByTagName("*"));
```

Javascript中存在一种名为伪数组的对象结构。比较特别的是 <span class="c42b983">arguments</span> 对象，还有像调用 <span class="c42b983">getElementsByTagName</span> , <span class="c42b983">document.childNodes</span> 之类的，它们返回NodeList对象都属于伪数组。不能应用 Array下的 push , pop 等方法。
但是我们能通过 <span class="c42b983">Array.prototype.slice.call</span> 转换为真正的数组的带有 length 属性的对象，这样 domNodes 就可以应用 Array 下的所有方法了。

## __bind__

在讨论<span class="c42b983">bind()</span>方法之前我们先来看一道题目：

```javascript
  var altwrite = document.write;
  altwrite("hello");
```

结果：Uncaught TypeError: Illegal invocation
altwrite()函数改变<span class="c42b983">this的指向global或window对象</span>，导致执行时提示非法调用异常，正确的方案就是使用bind()方法：

```javascript
  altwrite.bind(document)("hello")
```

当然也可以使用<span class="c42b983">call()</span>方法：

```javascript
  altwrite.call(document, "hello")
```

### __绑定函数__

<span class="c42b983">bind()</span>最简单的用法是创建一个函数，使这个函数不论怎么调用都有同样的this值。常见的错误就像上面的例子一样，将方法从对象中拿出来，然后调用，并且希望this指向原来的对象。如果不做特殊处理，一般会丢失原来的对象。使用bind()方法能够很漂亮的解决这个问题：

```javascript
  this.num = 9; 
  var mymodule = {
    num: 81,
    getNum: function() { 
      console.log(this.num);
    }
  };

  mymodule.getNum(); // 81

  var getNum = mymodule.getNum;
  getNum(); // 9, 因为在这个例子中，"this"指向全局对象

  var boundGetNum = getNum.bind(mymodule);
  boundGetNum(); // 81
```

<span class="c42b983">bind()</span> 方法与 <span class="c42b983">apply</span> 和 <span class="c42b983">call</span> 很相似，也是可以改变函数体内 <span class="c42b983">this</span> 的指向。

MDN的解释是：<span class="c42b983">bind()</span>方法会创建一个新函数，称为绑定函数，当调用这个绑定函数时，绑定函数会以创建它时传入 <span class="c42b983">bind()</span>方法的第一个参数作为 <span class="c42b983">this</span>，传入 <span class="c42b983">bind()</span> 方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。

直接来看看具体如何使用，在常见的单体模式中，通常我们会使用 <span class="c42b983">_this</span> , <span class="c42b983">that</span> , <span class="c42b983">self</span> 等保存 <span class="c42b983">this</span> ，这样我们可以在改变了上下文之后继续引用到它。 像这样：

```javascript
  var foo = {
    bar : 1,
    eventBind: function(){
      var _this = this;
      $('.someClass').on('click',function(event) {
        /* Act on the event */
        console.log(_this.bar);     //1
      });
    }
  }
```

由于 Javascript 特有的机制，上下文环境在 eventBind:function(){ } 过渡到 $('.someClass').on('click',function(event) { }) 发生了改变，上述使用变量保存 this 这些方式都是有用的，也没有什么问题。当然使用 bind() 可以更加优雅的解决这个问题：

```javascript
  var foo = {
    bar : 1,
    eventBind: function(){
      $('.someClass').on('click',function(event) {
          /* Act on the event */
          console.log(this.bar);      //1
      }.bind(this));
    }
  }
```

在上述代码里，<span class="c42b983">bind()</span> 创建了一个函数，当这个click事件绑定在被调用的时候，它的 <span class="c42b983">this</span> 关键词会被设置成被传入的值（这里指调用<span class="c42b983">bind()</span>时传入的参数）。因此，这里我们传入想要的上下文 <span class="c42b983">this</span>(其实就是 foo )，到 <span class="c42b983">bind()</span> 函数中。然后，当回调函数被执行的时候， <span class="c42b983">this</span> 便指向 foo 对象。再来一个简单的栗子：

```javascript
  var bar = function(){
    console.log(this.x);
  }
  var foo = {
    x:3
  }
  bar(); // undefined
  var func = bar.bind(foo);
  func(); // 3
```

这里我们创建了一个新的函数 func，当使用 <span class="c42b983">bind()</span> 创建一个绑定函数之后，它被执行的时候，它的 <span class="c42b983">this</span> 会被设置成 foo ， 而不是像我们调用 bar() 时的全局作用域。

### __偏函数（Partial Functions）__

这是一个很好的特性，使用<span class="c42b983">bind()</span>我们设定函数的预定义参数，然后调用的时候传入其他参数即可：

```javascript
  function list() {
    return Array.prototype.slice.call(arguments);
  }

  var list1 = list(1, 2, 3); // [1, 2, 3]

  // 预定义参数37
  var leadingThirtysevenList = list.bind(undefined, 37);

  var list2 = leadingThirtysevenList(); // [37]
  var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
```

### __和setTimeout一起使用__

```javascript
  function Bloomer() {
    this.petalCount = Math.ceil(Math.random() * 12) + 1;
  }

  // 1秒后调用declare函数
  Bloomer.prototype.bloom = function() {
    window.setTimeout(this.declare.bind(this), 100);
  };

  Bloomer.prototype.declare = function() {
    console.log('我有 ' + this.petalCount + ' 朵花瓣!');
  };

  var bloo = new Bloomer();
  bloo.bloom(); //我有 5 朵花瓣!
```

注意：对于事件处理函数和setInterval方法也可以使用上面的方法

### __绑定函数作为构造函数__

绑定函数也适用于使用<span class="c42b983">new操作符</span>来构造目标函数的实例。当使用绑定函数来构造实例，注意：<span class="c42b983">this</span>会被忽略，但是传入的参数仍然可用。

```javascript
  function Point(x, y) {
    this.x = x;
    this.y = y;
  }

  Point.prototype.toString = function() { 
    console.log(this.x + ',' + this.y);
  };

  var p = new Point(1, 2);
  p.toString(); // '1,2'


  var emptyObj = {};
  var YAxisPoint = Point.bind(emptyObj, 0/*x*/);
  // 实现中的例子不支持,
  // 原生bind支持:
  var YAxisPoint = Point.bind(null, 0/*x*/);

  var axisPoint = new YAxisPoint(5);
  axisPoint.toString(); // '0,5'

  axisPoint instanceof Point; // true
  axisPoint instanceof YAxisPoint; // true
  new Point(17, 42) instanceof YAxisPoint; // true
```

### __实现__

上面的几个小节可以看出<span class="c42b983">bind()</span>有很多的使用场景，但是<span class="c42b983">bind()</span>函数是在 ECMA-262 第五版才被加入；它可能无法在所有浏览器上运行。这就需要我们自己实现<span class="c42b983">bind()</span>函数了。

首先我们可以通过给目标函数指定作用域来简单实现<span class="c42b983">bind()</span>方法：

```javascript
  Function.prototype.bind = function(context){
    self = this;  //保存this，即调用bind方法的目标函数
    return function(){
        return self.apply(context,arguments);
    };
  };
```

考虑到函数柯里化的情况，我们可以构建一个更加健壮的<span class="c42b983">bind()</span>：

```javascript
  Function.prototype.bind = function(context){
    var args = Array.prototype.slice.call(arguments, 1),
    self = this;
    return function(){
      var innerArgs = Array.prototype.slice.call(arguments);
      var finalArgs = args.concat(innerArgs);
      return self.apply(context,finalArgs);
    };
  };
```

这次的<span class="c42b983">bind()</span>方法可以绑定对象，也支持在绑定的时候传参。

继续，Javascript的函数还可以作为构造函数，那么绑定后的函数用这种方式调用时，情况就比较微妙了，需要涉及到原型链的传递：

```javascript
  Function.prototype.bind = function(context){
    var args = Array.prototype.slice(arguments, 1),
    F = function(){},
    self = this,
    bound = function(){
      var innerArgs = Array.prototype.slice.call(arguments);
      var finalArgs = args.concat(innerArgs);
      return self.apply((this instanceof F ? this : context), finalArgs);
    };

    F.prototype = self.prototype;
    bound.prototype = new F();
    return bound;
  };
```

这是《JavaScript Web Application》一书中对<span class="c42b983">bind()</span>的实现：通过设置一个中转构造函数F，使绑定后的函数与调用<span class="c42b983">bind()</span>的函数处于同一原型链上，用new操作符调用绑定后的函数，返回的对象也能正常使用instanceof，因此这是最严谨的<span class="c42b983">bind()</span>实现。

对于为了在浏览器中能支持<span class="c42b983">bind()</span>函数，只需要对上述函数稍微修改即可：

```javascript
  Function.prototype.bind = function (oThis) {
    if (typeof this !== "function") {
      throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var aArgs = Array.prototype.slice.call(arguments, 1), 
      fToBind = this, 
      fNOP = function () {},
      fBound = function () {
        return fToBind.apply(
            this instanceof fNOP && oThis ? this : oThis || window,
            aArgs.concat(Array.prototype.slice.call(arguments))
        );
      };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
```

有个有趣的问题，如果连续 <span class="c42b983">bind()</span> 两次，亦或者是连续 <span class="c42b983">bind()</span> 三次那么输出的值是什么呢？像这样：

```javascript
  var bar = function(){
      console.log(this.x);
  }
  var foo = {
      x:3
  }
  var sed = {
      x:4
  }
  var func = bar.bind(foo).bind(sed);
  func(); //?
  
  var fiv = {
      x:5
  }
  var func = bar.bind(foo).bind(sed).bind(fiv);
  func(); //?
```

答案是，两次都仍将输出 3 ，而非期待中的 4 和 5 。原因是，在Javascript中，多次 <span class="c42b983">bind()</span> 是无效的。更深层次的原因， <span class="c42b983">bind()</span> 的实现，相当于使用函数在内部包了一个 call / apply ，第二次 <span class="c42b983">bind()</span> 相当于再包住第一次 <span class="c42b983">bind()</span> ,故第二次以后的 bind 是无法生效的。

## __apply、call、bind比较__

那么 <span class="c42b983">apply、call、bind</span> 三者相比较，之间又有什么异同呢？何时使用 <span class="c42b983">apply、call</span>，何时使用 <span class="c42b983">bind</span> 呢。简单的一个栗子：

```javascript
  var obj = {
    x: 81,
  };
  
  var foo = {
    getX: function() {
        return this.x;
    }
  }
  
  console.log(foo.getX.bind(obj)());  //81
  console.log(foo.getX.call(obj));    //81
  console.log(foo.getX.apply(obj));   //81
```

三个输出的都是81，但是注意看使用 <span class="c42b983">bind()</span> 方法的，他后面多了对括号。

也就是说，区别是，当你希望改变上下文环境之后并非立即执行，而是回调执行的时候，使用 <span class="c42b983">bind()</span> 方法。而 <span class="c42b983">apply</span>/<span class="c42b983">call</span> 则会立即执行函数。

再总结一下：

<span class="c42b983">apply 、 call 、bind</span> 三者都是用来改变函数的this对象的指向的；
<span class="c42b983">apply 、 call 、bind</span> 三者第一个参数都是this要指向的对象，也就是想指定的上下文；
<span class="c42b983">apply 、 call 、bind</span> 三者都可以利用后续参数传参；
<span class="c42b983">bind</span> 是返回对应函数，便于稍后调用；<span class="c42b983">apply 、call</span> 则是立即调用 。
