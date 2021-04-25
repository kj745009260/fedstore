---
title: javascript继承总结
date: 2020-10-13 16:41:21
tags: 继承
categories:
  - javascript
---

## __es6中的继承__

es6中的继承很简单，使用extends关键字就可以实现

```javascript
  // 父类
  class Parent {
    constructor(name) {
      this.name = name
    }

    sayName() {
      console.log(this.name)
    }
  }
  
  // 子类继承
  class Children extends Parent { }

  let creatChildren = new Children('kite')

  creatChildren.sayName() 会输出 kite
```

## __es5中的继承__

### __1.原型继承。继承父类模版和原型对象__

```javascript
  function Parent(name) {
    this.name = name
  }

  Parent.prototype = {
    constructor: Parent,
    sayName: function () {
      console.log(this.name)
    }
  }

  function Children () {}
  
  // 把父类的实例复制给子类的原型
  Children.prototype = new Parent('kite')

  var createChildren = new Children()

  createChildren.sayName() 会输出 kite
```

### __2.类继承。只继承父类模版,不继承父类原型对象__

```javascript
  function Parent(name) {
    this.name = name
  }

  Parent.prototype = {
    constructor: Parent,
    sayName: function () {
      console.log(this.name)
    }
  }

  function Children(name) {
    Parent.call(this, name)
  }

  var createChildren = new Children('kite')

  createChildren.sayName() 会输出 kite
```

### __3.混合继承,推荐使用.缺点：子类继承了2次父类的模版__

```javascript
  function Parent(name) {
    this.name = name
  }

  Parent.prototype = {
    constructor: Parent,
    sayName: function () {
      console.log(this.name)
    }
  }

  function Children(name) {
    Parent.call(this, name)
  }

  Children.prototype = new Parent();

  var createChildren = new Children('kite')

  createChildren.sayName() 会输出 kite
```

### __4.最佳继承。只继承一次父类模版和原型对象__

```javascript
  function Parent(name) {
    this.name = name
  }

  Parent.prototype = {
    constructor: Parent,
    sayName: function () {
      console.log(this.name)
    }
  }

  function Children(name) {
    Parent.call(this, name)
  }

  function extendParent(Parent, Children) {
    var F = new Function()
    F.prototype = Parent.prototype
    Children.prototype = new F() // 继承父类原型
    Children.prototype.constructor = Children // 还原子类构造器
    if (Parent.prototype.constructor === Object.prototype.constructor) {
      Parent.prototype.constructor = Parent // 还原父类构造器
    }
  }

  extendParent(Parent, Children)

  var createChildren = new Children('kite')

  createChildren.sayName() 会输出 kite
```

## __es5中new关键字干了啥__

> 1.创建一个新对象，新对象的原型指向构造函数的原型

> 2.将新对象赋值给构造函数的this, 并且执行构造函数内部的代码

> 3.如果构造函数返回的是非空对象，则返回这个对象；否则返回新对象

```javascript
  /* @method new操作的实现 new命令的原理
   * @param constructor 构造函数
   * @param params 参数
   */
  function _new(constructor, params) {
    // 取出入参中的constructor
    var constructor = [].shift.call(arguments)

    // 创建一个新对象，新对象的原型指向构造函数的原型
    var context = Object.create(constructor.prototype)
    
    // 取出入参中的params
    var args = [].slice.call(arguments)
    
    // 将新对象赋值给构造函数的this, 并且执行构造函数内部的代码
    var res = constructor.apply(context, args)

    // 如果构造函数返回的是非空对象，则返回这个对象；否则返回新对象
    return (typeof res === 'object' && res !== null) ? res : context
  }
```