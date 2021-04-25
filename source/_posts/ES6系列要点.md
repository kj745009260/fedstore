---
title: ES6系列要点
date: 2021-03-30 09:36:32
tags: es6
categories:
  - javascript
---

## __1.说说var、let、const之间的区别__

### __1.1 var__

在ES5中，顶层对象的属性和全局变量是等价的，用var声明的变量既是全局变量，也是顶层变量

注意：顶层对象，在浏览器环境指的是window对象，在 Node 指的是global对象

```javascript
  var a = 10;
  console.log(window.a) // 10
```

使用var声明的变量存在变量提升的情况

```javascript
  console.log(a) // undefined
  var a = 20
```

在编译阶段，编译器会将其变成以下执行

```javascript
  var a
  console.log(a)
  a = 20
```

使用var，我们能够对一个变量进行多次声明，后面声明的变量会覆盖前面的变量声明

```javascript
  var a = 20
  var a = 30
  console.log(a) // 30
```

在函数中使用使用var声明变量时候，该变量是局部的

```javascript
  var a = 20
  function change(){
      var a = 30
  }
  change()
  console.log(a) // 20 
```

而如果在函数内不使用var，该变量是全局的

```javascript
  var a = 20
  function change(){
    a = 30
  }
  change()
  console.log(a) // 30
```

### __1.2 let__

let是ES6新增的命令，用来声明变量

用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效

```javascript
  {
    let a = 20
  }
  console.log(a) // ReferenceError: a is not defined.
```
不存在变量提升

```javascript
  console.log(a) // 报错ReferenceError

  let a = 2
```

这表示在声明它之前，变量a是不存在的，这时如果用到它，就会抛出一个错误

只要块级作用域内存在let命令，这个区域就不再受外部影响

```javascript
  var a = 123
  if (true) {
    a = 'abc' // ReferenceError
    let a;
  }
```

使用let声明变量前，该变量都不可用，也就是大家常说的“暂时性死区”

最后，let不允许在相同作用域中重复声明

```javascript
  let a = 20
  let a = 30
  // Uncaught SyntaxError: Identifier 'a' has already been declared
```

注意的是相同作用域，下面这种情况是不会报错的

```javascript
  let a = 20
  {
    let a = 30
  }
```

因此，我们不能在函数内部重新声明参数

```javascript
  function func(arg) {
    let arg;
  }
  func()
  // Uncaught SyntaxError: Identifier 'arg' has already been declared
```

### __1.3 const__

const声明一个只读的常量，一旦声明，常量的值就不能改变

```javascript
  const a = 1
  a = 3
  // TypeError: Assignment to constant variable.
```

这意味着，const一旦声明变量，就必须立即初始化，不能留到以后赋值

```javascript
  const a;
  // SyntaxError: Missing initializer in const declaration
```

如果之前用var或let声明过变量，再用const声明同样会报错

```javascript
  var a = 20
  let b = 20
  const a = 30
  const b = 30
  // 都会报错
```
const实际上保证的并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动

对于简单类型的数据，值就保存在变量指向的那个内存地址，因此等同于常量

对于复杂类型的数据，变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的，并不能确保改变量的结构不变

```javascript
  const foo = {};

  // 为 foo 添加一个属性，可以成功
  foo.prop = 123;
  foo.prop // 123

  // 将 foo 指向另一个对象，就会报错
  foo = {}; // TypeError: "foo" is read-only
```

其它情况，const与let一致

### __1.4 区别__

var、let、const三者区别可以围绕下面五点展开：

> + 变量提升
> + 暂时性死区
> + 块级作用域
> + 重复声明
> + 修改声明的变量
> + 使用

<b class="c42b983">变量提升</b>

var声明的变量存在变量提升，即变量可以在声明之前调用，值为undefined

let和const不存在变量提升，即它们所声明的变量一定要在声明后使用，否则报错

```javascript
// var
  console.log(a)  // undefined
  var a = 10

  // let 
  console.log(b)  // Cannot access 'b' before initialization
  let b = 10

  // const
  console.log(c)  // Cannot access 'c' before initialization
  const c = 10
```

<b class="c42b983">暂时性死区</b>

var不存在暂时性死区

let和const存在暂时性死区，只有等到声明变量的那一行代码出现，才可以获取和使用该变量

```javascript
  // var
  console.log(a)  // undefined
  var a = 10

  // let
  console.log(b)  // Cannot access 'b' before initialization
  let b = 10

  // const
  console.log(c)  // Cannot access 'c' before initialization
  const c = 10
```

<b class="c42b983">块级作用域</b>

var不存在块级作用域

let和const存在块级作用域

```javascript
  // var
  {
      var a = 20
  }
  console.log(a)  // 20

  // let
  {
      let b = 20
  }
  console.log(b)  // Uncaught ReferenceError: b is not defined

  // const
  {
      const c = 20
  }
  console.log(c)  // Uncaught ReferenceError: c is not defined
```

<b class="c42b983">重复声明</b>

var允许重复声明变量

let和const在同一作用域不允许重复声明变量

```javascript
// var
  var a = 10
  var a = 20 // 20

  // let
  let b = 10
  let b = 20 // Identifier 'b' has already been declared

  // const
  const c = 10
  const c = 20 // Identifier 'c' has already been declared
```

<b class="c42b983">修改声明的变量</b>

var和let可以

const声明一个只读的常量。一旦声明，常量的值就不能改变

```javascript
  // var
  var a = 10
  a = 20
  console.log(a)  // 20

  //let
  let b = 10
  b = 20
  console.log(b)  // 20

  // const
  const c = 10
  c = 20
  console.log(c) // Uncaught TypeError: Assignment to constant variable
```

<b class="c42b983">使用</b>

> 能用const的情况尽量使用const，其他情况下大多数使用let，避免使用var

## __2.ES6中数组新增了哪些扩展?__

### __2.1 扩展运算符的应用__

ES6通过扩展元素符...，好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列

```javascript
  console.log(...[1, 2, 3])
  // 1 2 3

  console.log(1, ...[2, 3, 4], 5)
  // 1 2 3 4 5

  [...document.querySelectorAll('div')]
  // [<div>, <div>, <div>]
```

主要用于函数调用的时候，将一个数组变为参数序列

```javascript
  function push(array, ...items) {
    array.push(...items);
  }

  function add(x, y) {
    return x + y;
  }

  const numbers = [4, 38];
  add(...numbers) // 42
```

可以将某些数据结构转为数组

```javascript
  [...document.querySelectorAll('div')]
```

能够更简单实现数组复制

```javascript
  const a1 = [1, 2];
  const [...a2] = a1;
  // [1,2]
```

数组的合并也更为简洁了

```javascript
  const arr1 = ['a', 'b'];
  const arr2 = ['c'];
  const arr3 = ['d', 'e'];
  [...arr1, ...arr2, ...arr3]
  // [ 'a', 'b', 'c', 'd', 'e' ]
```

注意：通过扩展运算符实现的是浅拷贝，修改了引用指向的值，会同步反映到新数组

下面看个例子就清楚多了

```javascript
  const arr1 = ['a', 'b',[1,2]];
  const arr2 = ['c'];
  const arr3  = [...arr1,...arr2]
  arr[1][0] = 9999 // 修改arr1里面数组成员值
  console.log(arr[3]) // 影响到arr3,['a','b',[9999,2],'c']
```

扩展运算符可以与解构赋值结合起来，用于生成数组

```javascript
  const [first, ...rest] = [1, 2, 3, 4, 5];
  first // 1
  rest  // [2, 3, 4, 5]

  const [first, ...rest] = [];
  first // undefined
  rest  // []

  const [first, ...rest] = ["foo"];
  first  // "foo"
  rest   // []
```

如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错

```javascript
  const [...butLast, last] = [1, 2, 3, 4, 5];
  // 报错

  const [first, ...middle, last] = [1, 2, 3, 4, 5];
  // 报错
```

可以将字符串转为真正的数组

```javascript
  [...'hello']
  // [ "h", "e", "l", "l", "o" ]
```

定义了遍历器（Iterator）接口的对象，都可以用扩展运算符转为真正的数组

```javascript
  let nodeList = document.querySelectorAll('div');
  let array = [...nodeList];

  let map = new Map([
    [1, 'one'],
    [2, 'two'],
    [3, 'three'],
  ]);

  let arr = [...map.keys()]; // [1, 2, 3]
```

如果对没有 Iterator 接口的对象，使用扩展运算符，将会报错

```javascript
  const obj = {a: 1, b: 2};
  let arr = [...obj]; // TypeError: Cannot spread non-iterable object
```

### __2.2 构造函数新增的方法__

关于构造函数，数组新增的方法有如下：

> + Array.from()
> + Array.of()

<b class="c42b983">Array.from()</b>

将两类对象转为真正的数组：类似数组的对象和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）

```javascript
  let arrayLike = {
      '0': 'a',
      '1': 'b',
      '2': 'c',
      length: 3
  };
  let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```

还可以接受第二个参数，用来对每个元素进行处理，将处理后的值放入返回的数组

```javascript
  Array.from([1, 2, 3], (x) => x * x)
  // [1, 4, 9]
```

<b class="c42b983">Array.of()</b>

用于将一组值，转换为数组

```javascript
  Array.of(3, 11, 8) // [3,11,8]
```

没有参数的时候，返回一个空数组

当参数只有一个的时候，实际上是指定数组的长度

参数个数不少于 2 个时，Array()才会返回由参数组成的新数组

```javascript
  Array() // []
  Array(3) // [, , ,]
  Array(3, 11, 8) // [3, 11, 8]
```

### __2.3 实例对象新增的方法__

关于数组实例对象新增的方法有如下：

> + copyWithin()
> + find()、findIndex()
> + fill()
> + entries()，keys()，values()
> + includes()
> + flat()，flatMap()

<b class="c42b983">copyWithin()</b>

将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组

参数如下：

> target（必需）：从该位置开始替换数据。如果为负值，表示倒数。
> start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示从末尾开始计算。
> end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示从末尾开始计算。

```javascript
  [1, 2, 3, 4, 5].copyWithin(0, 3) // 将从 3 号位直到数组结束的成员（4 和 5），复制到从 0 号位开始的位置，结果覆盖了原来的 1 和 2
  // [4, 5, 3, 4, 5] 
```

<b class="c42b983">find()、findIndex()</b>

find()用于找出第一个符合条件的数组成员

参数是一个回调函数，接受三个参数依次为当前的值、当前的位置和原数组

```javascript
  [1, 5, 10, 15].find(function(value, index, arr) {
    return value > 9;
  }) // 10
```
findIndex返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1

```javascript
  [1, 5, 10, 15].findIndex(function(value, index, arr) {
    return value > 9;
  }) // 2
```

这两个方法都可以接受第二个参数，用来绑定回调函数的this对象。

```javascript
  function f(v){
    return v > this.age;
  }
  let person = {name: 'John', age: 20};
  [10, 12, 26, 15].find(f, person);    // 26
```

<b class="c42b983">fill()</b>

使用给定值，填充一个数组

```javascript
  ['a', 'b', 'c'].fill(7)
  // [7, 7, 7]

  new Array(3).fill(7)
  // [7, 7, 7]
```

还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置

```javascript
  ['a', 'b', 'c'].fill(7, 1, 2)
  // ['a', 7, 'c']
```
注意，如果填充的类型为对象，则是浅拷贝

<b class="c42b983">entries()，keys()，values()</b>

keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历

```javascript
  for (let index of ['a', 'b'].keys()) {
    console.log(index);
  }
  // 0
  // 1

  for (let elem of ['a', 'b'].values()) {
    console.log(elem);
  }
  // 'a'
  // 'b'

  for (let [index, elem] of ['a', 'b'].entries()) {
    console.log(index, elem);
  }
  // 0 "a"
```

<b class="c42b983">includes()</b>

用于判断数组是否包含给定的值

```javascript
  [1, 2, 3].includes(2)     // true
  [1, 2, 3].includes(4)     // false
  [1, 2, NaN].includes(NaN) // true
```

方法的第二个参数表示搜索的起始位置，默认为0

参数为负数则表示倒数的位置

```javascript
  [1, 2, 3].includes(3, 3);  // false
  [1, 2, 3].includes(3, -1); // true
```

<b class="c42b983">flat()，flatMap()</b>

将数组扁平化处理，返回一个新数组，对原数据没有影响

```javascript
  [1, 2, [3, 4]].flat()
  // [1, 2, 3, 4]
```
flat()默认只会“拉平”一层，如果想要“拉平”多层的嵌套数组，可以将flat()方法的参数写成一个整数，表示想要拉平的层数，默认为1

```javascript
  [1, 2, [3, [4, 5]]].flat()
  // [1, 2, 3, [4, 5]]

  [1, 2, [3, [4, 5]]].flat(2)
  // [1, 2, 3, 4, 5]
```

flatMap()方法对原数组的每个成员执行一个函数相当于执行Array.prototype.map()，然后对返回值组成的数组执行flat()方法。该方法返回一个新数组，不改变原数组

```javascript
  // 相当于 [[2, 4], [3, 6], [4, 8]].flat()
  [2, 3, 4].flatMap((x) => [x, x * 2])
  // [2, 4, 3, 6, 4, 8]
```

flatMap()方法还可以有第二个参数，用来绑定遍历函数里面的this

### __2.4 数组的空位__

数组的空位指，数组的某一个位置没有任何值

ES6 则是明确将空位转为undefined，包括Array.from、扩展运算符、copyWithin()、fill()、entries()、keys()、values()、find()和findIndex()

建议大家在日常书写中，避免出现空位

### __2.5 排序稳定性__

将sort()默认设置为稳定的排序算法

```javascript
  const arr = [
    'peach',
    'straw',
    'apple',
    'spork'
  ];

  const stableSorting = (s1, s2) => {
    if (s1[0] < s2[0]) return -1;
    return 1;
  };

  arr.sort(stableSorting)
  // ["apple", "peach", "straw", "spork"]
```

排序结果中，straw在spork的前面，跟原始顺序一致

## __3.ES6中对象新增了哪些扩展?__

### __3.1 属性的简写__

ES6中，当对象键名与对应值名相等的时候，可以进行简写

```javascript
  const baz = {foo:foo}

  // 等同于
  const baz = {foo}
```

方法也能够进行简写

```javascript
  const o = {
    method() {
      return "Hello!";
    }
  };

  // 等同于

  const o = {
    method: function() {
      return "Hello!";
    }
  }
```

在函数内作为返回值，也会变得方便很多

```javascript
  function getPoint() {
    const x = 1;
    const y = 10;
    return {x, y};
  }

  getPoint()
  // {x:1, y:10}
```

注意：简写的对象方法不能用作构造函数，否则会报错

```javascript
  const obj = {
    f() {
      this.foo = 'bar';
    }
  };

  new obj.f() // 报错
```

### __3.2 属性名表达式__

ES6 允许字面量定义对象时，将表达式放在括号内

```javascript
  let lastWord = 'last word';

  const a = {
    'first word': 'hello',
    [lastWord]: 'world'
  };

  a['first word'] // "hello"
  a[lastWord] // "world"
  a['last word'] // "world"
```

表达式还可以用于定义方法名

```javascript
  let obj = {
    ['h' + 'ello']() {
      return 'hi';
    }
  };

  obj.hello() // hi
```

注意，属性名表达式与简洁表示法，不能同时使用，会报错

```javascript
  // 报错
  const foo = 'bar';
  const bar = 'abc';
  const baz = { [foo] };

  // 正确
  const foo = 'bar';
  const baz = { [foo]: 'abc'};
```

注意，属性名表达式如果是一个对象，默认情况下会自动将对象转为字符串[object Object]

```javascript
  const keyA = {a: 1};
  const keyB = {b: 2};

  const myObject = {
    [keyA]: 'valueA',
    [keyB]: 'valueB'
  };

  myObject // Object {[object Object]: "valueB"}
```

### __3.3 super关键字__

this关键字总是指向函数所在的当前对象，ES6 又新增了另一个类似的关键字super，指向当前对象的原型对象

```javascript
  const proto = {
    foo: 'hello'
  };

  const obj = {
    foo: 'world',
    find() {
      return super.foo;
    }
  };

  Object.setPrototypeOf(obj, proto); // 为obj设置原型对象
  obj.find() // "hello"
```

### __3.4 扩展运算符的应用__

在解构赋值中，未被读取的可遍历的属性，分配到指定的对象上面

```javascript
  let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
  x // 1
  y // 2
  z // { a: 3, b: 4 }
```

注意：解构赋值必须是最后一个参数，否则会报错

解构赋值是浅拷贝

```javascript
  let obj = { a: { b: 1 } };
  let { ...x } = obj;
  obj.a.b = 2; // 修改obj里面a属性中键值
  x.a.b // 2，影响到了结构出来x的值
```

对象的扩展运算符等同于使用Object.assign()方法

### __3.5 属性的遍历__

ES6 一共有 5 种方法可以遍历对象的属性。

> + for...in：循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）

> + Object.keys(obj)：返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名

> + Object.getOwnPropertyNames(obj)：回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名

> + Object.getOwnPropertySymbols(obj)：返回一个数组，包含对象自身的所有 Symbol 属性的键名

> + Reflect.ownKeys(obj)：返回一个数组，包含对象自身的（不含继承的）所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举

上述遍历，都遵守同样的属性遍历的次序规则：

> + 首先遍历所有数值键，按照数值升序排列
> + 其次遍历所有字符串键，按照加入时间升序排列
> + 最后遍历所有 Symbol 键，按照加入时间升序排

```javascript
  Reflect.ownKeys({ [Symbol()]:0, b:0, 10:0, 2:0, a:0 })
  // ['2', '10', 'b', 'a', Symbol()]
```

### __3.6 对象新增的方法__

关于对象新增的方法，分别有以下：

> + Object.is()
> + Object.assign()
> + Object.getOwnPropertyDescriptors()
> + Object.setPrototypeOf()，Object.getPrototypeOf()
> + Object.keys()，Object.values()，Object.entries()
> + Object.fromEntries()

<b class="c42b983">Object.is()</b>

严格判断两个值是否相等，与严格比较运算符（===）的行为基本一致，不同之处只有两个：一是+0不等于-0，二是NaN等于自身

```javascript
  +0 === -0 //true
  NaN === NaN // false

  Object.is(+0, -0) // false
  Object.is(NaN, NaN) // true
```

<b class="c42b983">Object.assign()</b>

Object.assign()方法用于对象的合并，将源对象source的所有可枚举属性，复制到目标对象target

Object.assign()方法的第一个参数是目标对象，后面的参数都是源对象

```javascript
  const target = { a: 1, b: 1 };

  const source1 = { b: 2, c: 2 };
  const source2 = { c: 3 };

  Object.assign(target, source1, source2);
  target // {a:1, b:2, c:3}
```

注意：Object.assign()方法是浅拷贝，遇到同名属性会进行替换

<b class="c42b983">Object.getOwnPropertyDescriptors()</b>

返回指定对象所有自身属性（非继承属性）的描述对象

```javascript
  const obj = {
    foo: 123,
    get bar() { return 'abc' }
  };

  Object.getOwnPropertyDescriptors(obj)
  // { foo:
  //    { value: 123,
  //      writable: true,
  //      enumerable: true,
  //      configurable: true },
  //   bar:
  //    { get: [Function: get bar],
  //      set: undefined,
  //      enumerable: true,
  //      configurable: true } }
```

<b class="c42b983">Object.setPrototypeOf()</b>

Object.setPrototypeOf方法用来设置一个对象的原型对象

```javascript
  Object.setPrototypeOf(object, prototype)

  // 用法
  const o = Object.setPrototypeOf({}, null);
```

<b class="c42b983">Object.getPrototypeOf()</b>

用于读取一个对象的原型对象

```javascript
  Object.getPrototypeOf(obj);
```

<b class="c42b983">Object.keys()</b>

返回自身的（不含继承的）所有可遍历（enumerable）属性的键名的数组

```javascript
  var obj = { foo: 'bar', baz: 42 };
  Object.keys(obj)
  // ["foo", "baz"]
```

<b class="c42b983">Object.values()</b>

返回自身的（不含继承的）所有可遍历（enumerable）属性的键对应值的数组

```javascript
  const obj = { foo: 'bar', baz: 42 };
  Object.values(obj)
  // ["bar", 42]
```

<b class="c42b983">Object.entries()</b>

返回一个对象自身的（不含继承的）所有可遍历（enumerable）属性的键值对的数组

```javascript
  const obj = { foo: 'bar', baz: 42 };
  Object.entries(obj)
  // [ ["foo", "bar"], ["baz", 42] ]
  Object.fromEntries()
```
用于将一个键值对数组转为对象

```javascript
  Object.fromEntries([
    ['foo', 'bar'],
    ['baz', 42]
  ])
  // { foo: "bar", baz: 42 }
```

## __4.ES6中函数新增了哪些扩展?__

### __4.1 参数__

ES6允许为函数的参数设置默认值

```javascript
  function log(x, y = 'World') {
    console.log(x, y);
  }

  console.log('Hello') // Hello World
  console.log('Hello', 'China') // Hello China
  console.log('Hello', '') // Hello
```

函数的形参是默认声明的，不能使用let或const再次声明

```javascript
  function foo(x = 5) {
      let x = 1; // error
      const x = 2; // error
  }
```

参数默认值可以与解构赋值的默认值结合起来使用

```javascript
  function foo({x, y = 5}) {
    console.log(x, y);
  }

  foo({}) // undefined 5
  foo({x: 1}) // 1 5
  foo({x: 1, y: 2}) // 1 2
  foo() // TypeError: Cannot read property 'x' of undefined
```

上面的foo函数，当参数为对象的时候才能进行解构，如果没有提供参数的时候，变量x和y就不会生成，从而报错，这里设置默认值避免

```javascript
  function foo({x, y = 5} = {}) {
    console.log(x, y);
  }

  foo() // undefined 5
```

参数默认值应该是函数的尾参数，如果不是非尾部的参数设置默认值，实际上这个参数是没发省略的

```javascript
  function f(x = 1, y) {
    return [x, y];
  }

  f() // [1, undefined]
  f(2) // [2, undefined]
  f(, 1) // 报错
  f(undefined, 1) // [1, 1]
```

### __4.2 属性__

函数的length属性

length将返回没有指定默认值的参数个数

```javascript
  (function (a) {}).length // 1
  (function (a = 5) {}).length // 0
  (function (a, b, c = 5) {}).length // 2
```

rest 参数也不会计入length属性

```javascript
  (function(...args) {}).length // 0
```

如果设置了默认值的参数不是尾参数，那么length属性也不再计入后面的参数了

```javascript
  (function (a = 0, b, c) {}).length // 0
  (function (a, b = 1, c) {}).length // 1
```

name属性

返回该函数的函数名

```javascript
  var f = function () {};

  // ES5
  f.name // ""

  // ES6
  f.name // "f"
```

如果将一个具名函数赋值给一个变量，则 name属性都返回这个具名函数原本的名字

```javascript
  const bar = function baz() {};
  bar.name // "baz"
```

Function构造函数返回的函数实例，name属性的值为anonymous

```javascript
  (new Function).name // "anonymous"
```
bind返回的函数，name属性值会加上bound前缀

```javascript
  function foo() {};
  foo.bind({}).name // "bound foo"

  (function(){}).bind({}).name // "bound "
```

### __4.3 作用域__

一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域

等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的

下面例子中，y=x会形成一个单独作用域，x没有被定义，所以指向全局变量x

```javascript
  let x = 1;

  function f(y = x) { 
    // 等同于 let y = x  
    let x = 2; 
    console.log(y);
  }

  f() // 1
```

### __4.4 严格模式__

只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数内部就不能显式设定为严格模式，否则会报错

```javascript
  // 报错
  function doSomething(a, b = a) {
    'use strict';
    // code
  }

  // 报错
  const doSomething = function ({a, b}) {
    'use strict';
    // code
  };

  // 报错
  const doSomething = (...a) => {
    'use strict';
    // code
  };

  const obj = {
    // 报错
    doSomething({a, b}) {
      'use strict';
      // code
    }
  };
```

### __4.5 箭头函数__

使用“箭头”（=>）定义函数

```javascript
  var f = v => v;

  // 等同于
  var f = function (v) {
    return v;
  };
```

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分

```javascript
  var f = () => 5;
  // 等同于
  var f = function () { return 5 };

  var sum = (num1, num2) => num1 + num2;
  // 等同于
  var sum = function(num1, num2) {
    return num1 + num2;
  };
```

如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用return语句返回

```javascript
  var sum = (num1, num2) => { return num1 + num2; }
```

如果返回对象，需要加括号将对象包裹

```javascript
  let getTempItem = id => ({ id: id, name: "Temp" });
```

注意点：

> + 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象
> + 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误
> + 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替
> + 不可以使用yield命令，因此箭头函数不能用作 Generator 函数

## __5.Set、Map两种数据结构怎么理解?__

> Set是一种叫做集合的数据结构，Map是一种叫做字典的数据结构
> 集合: 是由一堆无序的、相关联的，且不重复的内存结构【数学中称为元素】组成的组合
> 字典: 是一些元素的集合。每个元素有一个称作key 的域，不同元素的key 各不相同

> 共同点: 集合、字典都可以存储不重复的值
> 不同点: 集合是以<b class="c42b983">[值，值]</b>的形式存储元素，字典是以<b class="c42b983">[键，值]</a>的形式存储

### __5.1 Set__

Set是es6新增的数据结构，类似于数组，但是成员的值都是唯一的，没有重复的值，我们一般称为集合

Set本身是一个构造函数，用来生成 Set 数据结构

```javascript
  const s = new Set();
```

增删改查

Set的实例关于增删改查的方法：

> + add()
> + delete()
> + has()
> + clear()
> + add()

添加某个值，返回 Set 结构本身

当添加实例中已经存在的元素，set不会进行处理添加

```javascript
  s.add(1).add(2).add(2); // 2只被添加了一次
```

<b class="c42b983">delete()</b>

删除某个值，返回一个布尔值，表示删除是否成功

```javascript
  s.delete(1)
```

<b class="c42b983">has()</b>

返回一个布尔值，判断该值是否为Set的成员

```javascript
  s.has(2)
```

<b class="c42b983">clear()</b>

清除所有成员，没有返回值

```javascript
  s.clear()
```

遍历

Set实例遍历的方法有如下：

关于遍历的方法，有如下：

> + keys()：返回键名的遍历器
> + values()：返回键值的遍历器
> + entries()：返回键值对的遍历器
> + forEach()：使用回调函数遍历每个成员

Set的遍历顺序就是插入顺序

keys方法、values方法、entries方法返回的都是遍历器对象

```javascript
  let set = new Set(['red', 'green', 'blue']);

  for (let item of set.keys()) {
    console.log(item);
  }
  // red
  // green
  // blue

  for (let item of set.values()) {
    console.log(item);
  }
  // red
  // green
  // blue

  for (let item of set.entries()) {
    console.log(item);
  }
  // ["red", "red"]
  // ["green", "green"]
  // ["blue", "blue"]
```

forEach()用于对每个成员执行某种操作，没有返回值，键值、键名都相等，同样的forEach方法有第二个参数，用于绑定处理函数的this

```javascript
  let set = new Set([1, 4, 9]);
  set.forEach((value, key) => console.log(key + ' : ' + value))
  // 1 : 1
  // 4 : 4
  // 9 : 9
```

扩展运算符和Set 结构相结合实现数组或字符串去重

```javascript
  // 数组
  let arr = [3, 5, 2, 2, 5, 5];
  let unique = [...new Set(arr)]; // [3, 5, 2]

  // 字符串
  let str = "352255";
  let unique = [...new Set(str)].join(""); // ""
```

实现并集、交集、和差集

```javascript
  let a = new Set([1, 2, 3]);
  let b = new Set([4, 3, 2]);

  // 并集
  let union = new Set([...a, ...b]);
  // Set {1, 2, 3, 4}

  // 交集
  let intersect = new Set([...a].filter(x => b.has(x)));
  // set {2, 3}

  // （a 相对于 b 的）差集
  let difference = new Set([...a].filter(x => !b.has(x)));
  // Set {1}
```

### __5.2 Map__

Map类型是键值对的有序列表，而键和值都可以是任意类型

Map本身是一个构造函数，用来生成 Map 数据结构

```javascript
  const m = new Map()
```

增删改查
Map 结构的实例针对增删改查有以下属性和操作方法：

> + size 属性
> + set()
> + get()
> + has()
> + delete()
> + clear()

<b class="c42b983">size</b>

size属性返回 Map 结构的成员总数。

```javascript
  const map = new Map();
  map.set('foo', true);
  map.set('bar', false);

  map.size // 2
```

<b class="c42b983">set()</b>

设置键名key对应的键值为value，然后返回整个 Map 结构

如果key已经有值，则键值会被更新，否则就新生成该键

同时返回的是当前Map对象，可采用链式写法

```javascript
  const m = new Map();

  m.set('edition', 6)        // 键是字符串
  m.set(262, 'standard')     // 键是数值
  m.set(undefined, 'nah')    // 键是 undefined
  m.set(1, 'a').set(2, 'b').set(3, 'c') // 链式操作
```

<b class="c42b983">get()</b>

get方法读取key对应的键值，如果找不到key，返回undefined

```javascript
  const m = new Map();

  const hello = function() {console.log('hello');};
  m.set(hello, 'Hello ES6!') // 键是函数

  m.get(hello)  // Hello ES6!
```

<b class="c42b983">has()</b>

has方法返回一个布尔值，表示某个键是否在当前 Map 对象之中

```javascript
  const m = new Map();

  m.set('edition', 6);
  m.set(262, 'standard');
  m.set(undefined, 'nah');

  m.has('edition')     // true
  m.has('years')       // false
  m.has(262)           // true
  m.has(undefined)     // true
```

<b class="c42b983">delete()</b>

delete方法删除某个键，返回true。如果删除失败，返回false

```javascript
  const m = new Map();
  m.set(undefined, 'nah');
  m.has(undefined)     // true

  m.delete(undefined)
  m.has(undefined)       // false
```

<b class="c42b983">clear()</b>

clear方法清除所有成员，没有返回值

```javascript
  let map = new Map();
  map.set('foo', true);
  map.set('bar', false);

  map.size // 2
  map.clear()
  map.size // 0
```

遍历

Map结构原生提供三个遍历器生成函数和一个遍历方法：

> + keys()：返回键名的遍历器
> + values()：返回键值的遍历器
> + entries()：返回所有成员的遍历器
> + forEach()：遍历 Map 的所有成员

遍历顺序就是插入顺序

```javascript
  const map = new Map([
    ['F', 'no'],
    ['T',  'yes'],
  ]);

  for (let key of map.keys()) {
    console.log(key);
  }
  // "F"
  // "T"

  for (let value of map.values()) {
    console.log(value);
  }
  // "no"
  // "yes"

  for (let item of map.entries()) {
    console.log(item[0], item[1]);
  }
  // "F" "no"
  // "T" "yes"

  // 或者
  for (let [key, value] of map.entries()) {
    console.log(key, value);
  }
  // "F" "no"
  // "T" "yes"

  // 等同于使用map.entries()
  for (let [key, value] of map) {
    console.log(key, value);
  }
  // "F" "no"
  // "T" "yes"

  map.forEach(function(value, key, map) {
    console.log("Key: %s, Value: %s", key, value);
  });
```

### __5.3 WeakSet 和 WeakMap__

WeakSet

创建WeakSet实例

```javascript
  const ws = new WeakSet();
```

WeakSet可以接受一个具有 Iterable接口的对象作为参数

```javascript
  const a = [[1, 2], [3, 4]];
  const ws = new WeakSet(a);
  // WeakSet {[1, 2], [3, 4]}
```

在API中WeakSet与Set有两个区别：

> 没有遍历操作的API
> 没有size属性

WeackSet只能成员只能是引用类型，而不能是其他类型的值

```javascript
  let ws=new WeakSet();

  // 成员不是引用类型
  let weakSet=new WeakSet([2,3]);
  console.log(weakSet) // 报错

  // 成员为引用类型
  let obj1={name:1}
  let obj2={name:1}
  let ws=new WeakSet([obj1,obj2]); 
  console.log(ws) //WeakSet {{…}, {…}}
```

WeakSet里面的引用只要在外部消失，它在 WeakSet里面的引用就会自动消失

WeakMap

WeakMap结构与Map结构类似，也是用于生成键值对的集合

在API中WeakMap与Map有两个区别：

> 没有遍历操作的API
> 没有clear清空方法

```javascript
  // WeakMap 可以使用 set 方法添加成员
  const wm1 = new WeakMap();
  const key = {foo: 1};
  wm1.set(key, 2);
  wm1.get(key) // 2

  // WeakMap 也可以接受一个数组，
  // 作为构造函数的参数
  const k1 = [1, 2, 3];
  const k2 = [4, 5, 6];
  const wm2 = new WeakMap([[k1, 'foo'], [k2, 'bar']]);
  wm2.get(k2) // "bar"
```

WeakMap只接受对象作为键名（null除外），不接受其他类型的值作为键名

```javascript
  const map = new WeakMap();
  map.set(1, 2)
  // TypeError: 1 is not an object!
  map.set(Symbol(), 2)
  // TypeError: Invalid value used as weak map key
  map.set(null, 2)
  // TypeError: Invalid value used as weak map key
```

WeakMap的键名所指向的对象，一旦不再需要，里面的键名对象和所对应的键值对会自动消失，不用手动删除引用

举个场景例子：

在网页的 DOM 元素上添加数据，就可以使用WeakMap结构，当该 DOM 元素被清除，其所对应的WeakMap记录就会自动被移除

```javascript
  const wm = new WeakMap();

  const element = document.getElementById('example');

  wm.set(element, 'some information');
  wm.get(element) // "some information"
```

注意：WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用

下面代码中，键值obj会在WeakMap产生新的引用，当你修改obj不会影响到内部

```javascript
  const wm = new WeakMap();
  let key = {};
  let obj = {foo: 1};

  wm.set(key, obj);
  obj = null;
  wm.get(key)
  // Object {foo: 1}
```

## __6.怎么理解ES6中 Decorator 的？使用场景？__

Docorator修饰对象为下面两种：

> 类的装饰
> 类属性的装饰

### __6.1 类的装饰__

当对类本身进行装饰的时候，能够接受一个参数，即类本身

将装饰器行为进行分解，大家能够有个更深入的了解

```javascript
  @decorator
  class A {}

  // 等同于

  class A {}
  A = decorator(A) || A;
```

下面@testable就是一个装饰器，target就是传入的类，即MyTestableClass，实现了为类添加静态属性

```javascript
  @testable
  class MyTestableClass {
    // ...
  }

  function testable(target) {
    target.isTestable = true;
  }

  MyTestableClass.isTestable // true
```

如果想要传递参数，可以在装饰器外层再封装一层函数

```javascript
  function testable(isTestable) {
    return function(target) {
      target.isTestable = isTestable;
    }
  }

  @testable(true)
  class MyTestableClass {}
  MyTestableClass.isTestable // true

  @testable(false)
  class MyClass {}
  MyClass.isTestable // false
```

### __6.2 类属性的装饰__

当对类属性进行装饰的时候，能够接受三个参数：

> 类的原型对象
> 需要装饰的属性名
> 装饰属性名的描述对象

首先定义一个readonly装饰器

```javascript
  function readonly(target, name, descriptor){
    descriptor.writable = false; // 将可写属性设为false
    return descriptor;
  }
```

使用readonly装饰类的name方法

```javascript
  class Person {
    @readonly
    name() { return `${this.first} ${this.last}` }
  }
```
相当于以下调用

```javascript
  readonly(Person.prototype, 'name', descriptor);
```
如果一个方法有多个装饰器，就像洋葱一样，先从外到内进入，再由内到外执行

```javascript
  function dec(id){
    console.log('evaluated', id);
    return (target, property, descriptor) =>console.log('executed', id);
  }

  class Example {
      @dec(1)
      @dec(2)
      method(){}
  }
  // evaluated 1
  // evaluated 2
  // executed 2
  // executed 1
```

外层装饰器@dec(1)先进入，但是内层装饰器@dec(2)先执行
