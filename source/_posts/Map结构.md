---
title: Map结构
date: 2021-02-23 11:39:46
tags: map
categories:
  - javascript
---

## __map相关方法__

<b class="c42b983">map.set(key, val) </b>
> 1.如果对同一个键多次赋值，后面的值将覆盖前面的值。
> 2.注意，只有对同一个对象的引用，Map 结构才将其视为同一个键。
> 3.Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。
> 4.如果 Map 的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map 将其视为一个键，比如0和-0就是一个键，布尔值true和字符串true则是两个不同的键。另外，undefined和null也是两个不同的键。虽然NaN不严格相等于自身，但 Map 将其视为同一个键。

<b class="c42b983">map.get(key)</b>
<b class="c42b983">map.size</b>, size属性返回 Map 结构的成员总数。 
<b class="c42b983">map.has(key)</b>, has方法返回一个布尔值，表示某个键是否在当前 Map 对象之中。
<b class="c42b983">map.delete(key)</b>, delete方法删除某个键，返回true。如果删除失败，返回false。
<b class="c42b983">map.clear()</b>, clear方法清除所有成员，没有返回值。

遍历方法

Map 结构原生提供三个遍历器生成函数和一个遍历方法。

> Map.prototype.keys()：返回键名的遍历器。
> Map.prototype.values()：返回键值的遍历器。
> Map.prototype.entries()：返回所有成员的遍历器。
> Map.prototype.forEach()：遍历 Map 的所有成员。

## __与其他数据结构的互相转换__

### __1.Map 转为数组__

Map 转为数组最方便的方法，就是使用扩展运算符（...）

```javascript
  const myMap = new Map()
  .set(true, 7)
  .set({foo: 3}, ['abc']);

  [...myMap]
  // [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
```

### __2.数组 转为 Map__

将数组传入 Map 构造函数，就可以转为 Map。

```javascript
  new Map([
    [true, 7],
    [{foo: 3}, ['abc']]
  ])
  // Map {
  //   true => 7,
  //   Object {foo: 3} => ['abc']
  // }
```

### __3.Map 转为对象__

如果所有 Map 的键都是字符串，它可以无损地转为对象。

```javascript
  function strMapToObj(strMap) {
    let obj = Object.create(null);
    for (let [k,v] of strMap) {
      obj[k] = v;
    }
    return obj;
  }

  const myMap = new Map()
    .set('yes', true)
    .set('no', false);
  strMapToObj(myMap)
  // { yes: true, no: false }
```

如果有非字符串的键名，那么这个键名会被转成字符串，再作为对象的键名。

### __4.对象转为 Map__

对象转为 Map 可以通过Object.entries()。

```javascript
  let obj = {"a":1, "b":2};
  let map = new Map(Object.entries(obj));

  // 此外，也可以自己实现一个转换函数。
  function objToStrMap(obj) {
    let strMap = new Map();
    for (let k of Object.keys(obj)) {
      strMap.set(k, obj[k]);
    }
    return strMap;
  }

  objToStrMap({yes: true, no: false})
  // Map {"yes" => true, "no" => false}
```

### __5.Map 转为 JSON__

Map 转为 JSON 要区分两种情况。一种情况是，Map 的键名都是字符串，这时可以选择转为对象 JSON。

```javascript
  function strMapToJson(strMap) {
    return JSON.stringify(strMapToObj(strMap));
  }

  let myMap = new Map().set('yes', true).set('no', false);
  strMapToJson(myMap)
  // '{"yes":true,"no":false}'
```

另一种情况是，Map 的键名有非字符串，这时可以选择转为数组 JSON。

```javascript
  function mapToArrayJson(map) {
    return JSON.stringify([...map]);
  }

  let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
  mapToArrayJson(myMap)
  // '[[true,7],[{"foo":3},["abc"]]]'
```

### __6.JSON 转为 Map__

JSON 转为 Map，正常情况下，所有键名都是字符串。

```javascript
  function jsonToStrMap(jsonStr) {
    return objToStrMap(JSON.parse(jsonStr));
  }

  jsonToStrMap('{"yes": true, "no": false}')
  // Map {'yes' => true, 'no' => false}
```

但是，有一种特殊情况，整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。这时，它可以一一对应地转为 Map。这往往是 Map 转为数组 JSON 的逆操作。


```javascript
  function jsonToMap(jsonStr) {
    return new Map(JSON.parse(jsonStr));
  }

  jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
  // Map {true => 7, Object {foo: 3} => ['abc']}
```

