---
title: javascript中判断数据类型的几种方法
date: 2020-10-14 13:36:33
tags: 类型判断
categories:
  - javascript
---

## __javascript中判断数据类型的几种方法__

在 ECMAScript 规范中，共定义了 7 种数据类型，分为 基本类型 和 引用类型 两大类，如下所示：

> 基本类型：String、Number、Boolean、Symbol、Undefined、Null

> 引用类型：Object

基本类型也称为简单类型，由于其占据空间固定，是简单的数据段，为了便于提升变量查询速度，将其存储在栈中，即按值访问。

引用类型也称为复杂类型，由于其值的大小会改变，所以不能将其存放在栈中，否则会降低变量查询速度，因此，其值存储在堆(heap)中，而存储在变量处的值，是一个指针，指向存储对象的内存处，即按址访问。引用类型除 Object 外，还包括 Function 、Array、RegExp、Date 等等。

鉴于 ECMAScript 是松散类型的，因此需要有一种手段来检测给定变量的数据类型。对于这个问题，JavaScript 也提供了多种方法，但遗憾的是，不同的方法得到的结果参差不齐。

### __1.typeof操作符__

typeof 是一个操作符，其右侧跟一个一元表达式，并返回这个表达式的数据类型。返回的结果用该类型的字符串(全小写字母)形式表示，包括以下 7 种：number、boolean、symbol、string、object、undefined、function 等。

> typeof ''; // string 有效
> typeof 1; // number 有效
> typeof Symbol(); // symbol 有效
> typeof true; //boolean 有效
> typeof undefined; //undefined 有效
> typeof null; //object <span class="c42b983">无效</span>
> typeof [] ; //object <span class="c42b983">无效</span>
> typeof new Function(); // function 有效
> typeof new Date(); //object <span class="c42b983">无效</span>
> typeof new RegExp(); //object <span class="c42b983">无效</span>

有些时候，typeof 操作符会返回一些令人迷惑但技术上却正确的值：

> + 对于基本类型，除 null 以外，均可以返回正确的结果。
> + 对于引用类型，除 function 以外，一律返回 object 类型。
> + 对于 null ，返回 object 类型。
> + 对于 function 返回  function 类型。

其中，null 有属于自己的数据类型 Null ， 引用类型中的 数组、日期、正则 也都有属于自己的具体类型，而 typeof 对于这些类型的处理，只返回了处于其原型链最顶端的 Object 类型，没有错，但不是我们想要的结果。

### __2.instanceof操作符__

instanceof 是用来判断 A 是否为 B 的实例，表达式为：A instanceof B，如果 A 是 B 的实例，则返回 true,否则返回 false。 在这里需要特别注意的是：instanceof 检测的是原型，我们用一段伪代码来模拟其内部执行过程：

```javascript
  instanceof (A,B) = {
      varL = A.__proto__;
      varR = B.prototype;
      if(L === R) {
          // A的内部属性 __proto__ 指向 B 的原型对象
          returntrue;
      }
      returnfalse;
  }
```

从上述过程可以看出，当 A 的 __proto__ 指向 B 的 prototype 时，就认为 A 就是 B 的实例，我们再来看几个例子：

```javascript
  [] instanceof Array; // true
  {} instanceof Object; // true
  new Date() instanceof Date; // true
  
  function Person(){};
  new Person() instanceof Person; // true
  
  [] instanceof Object; // true
  new Date() instanceof Object;// true
  new Person() instanceof Object;// true
```

我们发现，虽然 instanceof 能够判断出 [ ] 是Array的实例，但它认为 [ ] 也是Object的实例，为什么呢？

我们来分析一下 [ ]、Array、Object 三者之间的关系：

从 instanceof 能够判断出 [ ].__proto__  指向 Array.prototype，而 Array.prototype.__proto__ 又指向了Object.prototype，最终 Object.prototype.__proto__ 指向了null，标志着原型链的结束。因此，[]、Array、Object 就在内部形成了一条原型链：

![arratProto](/images/arrayproto.png)

从原型链可以看出，[] 的 __proto__  直接指向Array.prototype，间接指向 Object.prototype，所以按照 instanceof 的判断规则，[] 就是Object的实例。依次类推，类似的 new Date()、new Person() 也会形成一条对应的原型链 。因此，<span class="c42b983">instanceof 只能用来判断两个对象是否属于实例关系， 而不能判断一个对象实例具体属于哪种类型。</span>

instanceof 操作符的问题在于，它假定只有一个全局执行环境。如果网页中包含多个框架，那实际上就存在两个以上不同的全局执行环境，从而存在两个以上不同版本的构造函数。如果你从一个框架向另一个框架传入一个数组，那么传入的数组与在第二个框架中原生创建的数组分别具有各自不同的构造函数。

```javascript
  var iframe = document.createElement('iframe');
  document.body.appendChild(iframe);
  var xArray = window.frames[0].Array;
  var arr = newxArray(1,2,3); // [1,2,3]
  arr instanceof Array; // false
```

针对数组的这个问题，ES5 提供了 Array.isArray() 方法 。该方法用以确认某个对象本身是否为 Array 类型，而不区分该对象在哪个环境中创建。

```bash
  if(Array.isArray(value)){
    //对数组执行某些操作
  }
```

Array.isArray() 本质上检测的是对象的 [[Class]] 值，[[Class]] 是对象的一个内部属性，里面包含了对象的类型信息，其格式为 [object Xxx] ，Xxx 就是对应的具体类型 。对于数组而言，[[Class]] 的值就是 [object Array] 。

### __3.constructor__

当一个函数 F被定义时，JS引擎会为F添加 prototype 原型，然后再在 prototype上添加一个 constructor 属性，并让其指向 F 的引用。如下所示：

![constructor](/images/constructor.png)

当执行 var f = new F() 时，F 被当成了构造函数，f 是F的实例对象，此时 F 原型上的 constructor 传递到了 f 上，因此 f.constructor == F

![constructor](/images/constructorf.png)

可以看出，F 利用原型对象上的 constructor 引用了自身，当 F 作为构造函数来创建对象时，原型上的 constructor 就被遗传到了新创建的对象上， 从原型链角度讲，构造函数 F 就是新对象的类型。这样做的意义是，让新对象在诞生以后，就具有可追溯的数据类型。

同样，JavaScript 中的内置对象在内部构建时也是这样做的:

```javascript
  ''.constructor === String  // true

  new Number(1).constructor === Number  // true

  true.constructor === Boolean  // true

  (function() {}).constructor === Function  // true

  new Date().constructor === Date  // true

  new Error().constructor === Error  // true

  [].constructor === Array  // true

  document.constructor === HTMLDocument  // true

  window.constructor === Window  // true
```

细节问题:
> 1. null 和 undefined 是无效的对象，因此是不会有 constructor 存在的，这两种类型的数据需要通过其他方式来判断。

> 2. 函数的 constructor 是不稳定的，这个主要体现在自定义对象上，当开发者重写 prototype 后，原有的 constructor 引用会丢失，constructor 会默认为 Object

![constructor](/images/constructorn.png)

为什么变成了 Object？

因为 prototype 被重新赋值的是一个 { }， { } 是 new Object() 的字面量，因此 new Object() 会将 Object 原型上的 constructor 传递给 { }，也就是 Object 本身。

因此，为了规范开发，在重写对象原型时一般都需要重新给 constructor 赋值，以保证对象实例的类型不被篡改。

### __4.toString方法__

toString() 是 Object 的原型方法，调用该方法，默认返回当前对象的 [[Class]] 。这是一个内部属性，其格式为 [object Xxx] ，其中 Xxx 就是对象的类型。

对于 Object 对象，直接调用 toString()  就能返回 [object Object] 。而对于其他对象，则需要通过 call / apply 来调用才能返回正确的类型信息。

```javascript
  Object.prototype.toString.call('')  // [object String]
  Object.prototype.toString.call(1)  // [object Number]
  Object.prototype.toString.call(true)  // [object Boolean]
  Object.prototype.toString.call(Symbol())  //[object Symbol]
  Object.prototype.toString.call(undefined)  // [object Undefined]
  Object.prototype.toString.call(null)  // [object Null]
  Object.prototype.toString.call(function() {})  // [object Function]
  Object.prototype.toString.call(new Date())  // [object Date]
  Object.prototype.toString.call([])  // [object Array]
  Object.prototype.toString.call(new RegExp())  // [object RegExp]
  Object.prototype.toString.call(new Error())  // [object Error]
  Object.prototype.toString.call(document)  // [object HTMLDocument]
  Object.prototype.toString.call(window)  // [object Window]
```