---
title: Vue组件间通信方式
date: 2021-03-24 16:28:36
tags: 组件间通信
categories:
  - vue.js
---

## __组件间通信的分类__

> + 父子组件之间的通信
> + 兄弟组件之间的通信
> + 祖孙与后代组件之间的通信
> + 非关系组件间之间的通信

## __组件间通信的方案__

整理vue中8种常规的通信方案

> 1. 通过 props 传递
> 2. 通过 $emit 触发自定义事件
> 3. 使用 ref
> 4. EventBus
> 5. parent 或 root
> 6. attrs 与 listeners
> 7. Provide 与 Inject
> 8. Vuex

### __1.props传递数据__

> + 适用场景：父组件传递数据给子组件
> + 子组件设置props属性，定义接收父组件传递过来的参数
> + 父组件在使用子组件标签中通过字面量来传递值

Children.vue组件

```javascript
props:{
  // 字符串形式
  name:String // 接收的类型参数

  // 对象形式
  age:{  
    type:Number, // 接收的类型为数值
    defaule:18,  // 默认值为18
    require:true // age属性必须传递
  }
}
```

Father.vue组件

```html
  <Children name="jack" age=18 />
```

### __2.$emit 触发自定义事件__

> + 适用场景：子组件传递数据给父组件
> + 子组件通过$emit触发自定义事件，$emit第二个参数为传递的数值
> + 父组件绑定监听器获取到子组件传递过来的参数

Children.vue组件

```javascript
  this.$emit('add', good)
```

Father.vue组件

```html
  <Children @add="cartAdd($event)" />
```

### __3.ref__

> + 父组件在使用子组件的时候设置ref
> + 父组件通过设置子组件ref来获取数据

父组件

```javascript
  <Children ref="foo" />

  this.$refs.foo  // 获取子组件实例，通过子组件实例我们就能拿到对应的数据
```

### __4.EventBus__

> + 使用场景：兄弟组件传值
> + 创建一个中央时间总线EventBus
> + 兄弟组件通过$emit触发自定义事件，$emit第二个参数为传递的数值
> + 另一个兄弟组件通过$on监听自定义事件

Bus.js

```javascript
  const vm = new Vue()
  vm.$on('listener', callback)
  vm.$emit('listener', data)
  vm.$off('listener', callback)
```

自定义Bus.js

```javascript
  class Bus {
    constructor() {
      // 存放事件的名字
      this.callbacks = {}
    }

    $on(name, fn) {
      this.callbacks[name] = this.callbacks[name] || []
      this.callbacks[name].push(fn)
    }

    $emit(name, args) {
      if (this.callbacks[name]) {
        this.callbacks[name].forEach(cb => cb(args));
      }
    }

    $off(name, fn) {
      if (this.callbacks[name]) {
        let delIndex = this.callbacks[name].indexOf(fn)
        this.callbacks[name].splice(delIndex, 1)
      }
    }
  }
```

### __5.parent或root__

通过共同祖辈$parent或者$root搭建通信侨联

```javascript
  // 兄弟组件
  this.$parent.on('add',this.add)
  // 另一个兄弟组件
  this.$parent.emit('add')

  // 或者直接通过$parent.$refs访问另一个兄弟组件
  this.$parent.$refs.foo
```


### __6.attrs与listeners__

> + 适用场景：祖先传递数据给子孙
> + 设置批量向下传属性$attrs和 $listeners
> + 包含了父级作用域中不作为 prop 被识别 (且获取) 的特性绑定 ( class 和 style 除外)。
> + 可以通过 v-bind="$attrs" 传⼊内部组件

```html
  <!-- child组件：并未在props中声明foo -->
  <p>{{$attrs.foo}}</p>

  <!-- parent组件 -->
  <HelloWorld foo="foo"/>
```

```html
  <!-- 给Grandson隔代传值 -->
  <Child msg="lalala" @some-event="onSomeEvent"></Child>

  <!-- 在Child组件中引用Grandson组件时直接做展开 -->
  <Grandson v-bind="$attrs" v-on="$listeners"></Grandson>

  <!-- Grandson使⽤ -->
  <div @click="$emit('some-event', 'msg from grandson')">
  {{msg}}
  </div>
```

### __7.provide 与 inject__

> + 在祖先组件定义provide属性，返回传递的值
> + 在后代组件通过inject接收组件传递过来的值

祖先组件

```javascript
  provide(){
    return {
      foo:'foo'
    }
  }
```

后代组件

```javascript
  inject:['foo'] // 获取到祖先组件传递过来的值
```

## __小结__

> + 父子关系的组件数据传递选择 props  与 $emit进行传递，也可选择ref
> + 兄弟关系的组件数据传递可选择$bus，其次可以选择$parent进行传递
> + 祖先与后代组件数据传递可选择attrs与listeners或者 Provide与 Inject
> + 复杂关系的组件数据传递可以通过vuex存放共享的变量
