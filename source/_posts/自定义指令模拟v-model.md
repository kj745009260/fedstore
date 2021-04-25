---
title: 自定义指令模拟v-model
date: 2021-03-23 17:50:07
tags: 指令
categories:
  - vue.js
---

## __钩子函数__

![指令](/images/gzhs.png)

## __钩子函数的参数__

![指令](/images/gzcs.png)

## __模拟实现__

```javascript
  Vue.directive('mymodel', {
    bind: function(el, binding, vnode) {
      const vm = vnode.context
      const { value, expression } = binding
      el.value = value
      el.addEventListener('input', e => {
        vm[expression] = e.target.value
      })
    },
    // 当数据被更新的时候，需要更新指令绑定的元素
    update: function(el, binding) {
      const { value } = binding
      el.value = value
    }
  })
```
