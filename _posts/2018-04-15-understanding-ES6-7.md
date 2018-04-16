---
layout: post
title:  "第7章 Set集合与Map集合"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: Set集合与Map集合
mathjax: true
---

* content
{:toc}

<!-- # 第7章 Set集合与Map集合 -->

## ECMAScript 5 的集合

`Set`集合常用于检查是否存在某键名，`Map`集合常用于获取已存的信息。以前通过以下方式模拟`Set`与`Map`集合：

```js
// Set
var set = Object.create(null);

set.foo = true;

if (set.foo) {
    // ...
}

// Map
var map = Object.create(null);

map.foo = 'bar';

var val = map.foo;
console.log(val);

// 然而该实现有不足之处：
// 想用数值型作为键名，会很麻烦
map[5] = 'foo';
// 数字被自动转换为字符串，用对象作为键名也有问题，也会被自动转换
console.log(map['5']); // 'foo'

map.count = 1;
// 本意是检查'count'是否存在，实际运行时是检查该值是否非零
if (map.count) {
    // ...
}
```

> 若使用 `in`运算符判断属性是否存在，会检索对象原型，对象原型为`null`时才比较稳妥。

## ECMAScript 6 的 `Set`

`Set`是一种有序列表，含相互独立的非重复值。

```js
// 构造函数接受所有可迭代对象作为参数
let set = new Set([1, 2, 3, 4, 4]);

// 不会自动转换成字符串
set.add(5);
set.add('5');

let key1 = {};
let key2 = {};

set.add(key1);
set.add(key2);

console.log(set.size); // 4

// 重复的值会被忽略
// 内部使用 Object.is()检测是否重复，
// 但 +0 与 -0 被认为相等
set.add(5);

console.log(set.size); // 4

// 检测是否存在某个值
set.has(5); // true
set.has(6); // false

// 移除元素
set.delete(5);

// 移除所有元素
set.clear(5);

// 遍历集合
// value与key相等，因为键值相同
set.forEach(function(value, key, ownerSet) {
    // ...
}, this); // 第二个参数是 this 引用
```

可以用来创建一个无重复元素的数组：

```js
let arr = [...new Set(items)];
```

## `WeakSet`

```js
let set = new Set(),
    key = {};

set.add(key);
console.log(set.size); // 1

// 移除原始引用
key = null;

console.log(set.size); // 1

// 重新取回原始引用
key = [...set][0];
```

若想让被引用的对象被回收，但是上述情况因为有引用，阻碍了回收（内存泄漏）。此时可以换成`WeakSet`集合：只储存对象弱引用，只有该引用的对象会被回收并释放内存。

```js
// 构造函数不接受任何原始值元素，不然会报错
let key1 = {},
    set = new WeakSet([ key1 ]),
    key2 = {};

// 添加
set.add(key);

// 判断是否存在
console.log(set.has(key)); // true

// 删除
set.delete(key);

// WeakSet 中弱引用也会自动移除
key1 = null;
```

`WeakSet`的规则：

- 向`add()`传入非对象参数会报错，而`has()`、`delete()`只会返回`false`。
- 不可迭代
- 没有迭代器
- 不支持`forEach()`
- 不支持`size`属性

## ECMAScript 6 的 `Map`

`Map`是一种存储着许多键值对的有序列表，其中键名和对应值支持所有数据类型，键名等价判断用`Object.is()`实现。

```js
let map = new Map();

// 添加元素
map.set('name', 'me');
map.set(1, 2);

// 获取值
console.log(map.get('name')); // 'me'
console.log(map.get(1)); // 2
console.log(map.get(2)); // undefined

console.log(map.size); // 2

// 检测值是否存在
console.log(map.has('name')); // true

// 删除值
map.delete(1);

// 清除所有值
map.clear();

// 按元素索引顺序依次传入回调函数
map.forEach(function(value, key, ownerMap) {
    // ...
}, this); // this
```

`Map`构造函数传入数组来初始化一个`Map`集合，每个子数组包含一个键值对。这种方式保证键值在传入之前不会被强制转换。

```js
let map = new Map([['name', 'me'], ['age', 22], ['gender', 'male']]);
```

## `WeakMap`

`WeakMap`是一种存储键值对的无序列表，是弱引用的`Map`，键名必须是一个非`null`的对象，对应值可以是任意类型。使用非对象键名会报错，如果在弱引用外不存在其它强引用，引擎的垃圾回收机制会自动回收这个对象，同时移除`WeakMap`的键值对。但键名对应的值如果是对象，则保存的是强引用，不会触发垃圾回收机制。

```js
let map = new WeakMap(),
    element = document.querySelector('.element');

// 添加数据
map.set(element, 'original');

// 获取数据
map.get(element);

// 移除数据，WeakMap变为空
element.parentNode.removeChild(element);
```

初始化过程与`Map`类似，但如果传入非对象的键，会报错。

```js
let key1 = {},
    key2 = {},
    map = new WeakMap([[key1, 'hello'], [key2, 42]]);

// 检测键是否存在
console.log(map.has(key1)); // true
console.log(map.has(key2)); // true

// 移除对应键值对
map.delete(key1); 

console.log(map.has(key1)); // false
console.log(map.get(key1)); // undefined
```

> WeakMap 也没有`size`属性，`forEach()`和`clear()`

可以利用`WeakMap`创建更容易被回收的私有属性：

```js
// ES 5
var Person = (function() {
    var privateData = {},
        privateId = 0;
    
    function Person(name) {
        Object.defineProperty(this, '_id', { value: privateId++ });
        
        privateData[this._id] = {
            name: name
        };
    }

    Person.prototype.getName = function() {
        return privateData[this._id].name;
    };
    
    return Person;
})();
```

上述做法可以创建不能被改变的私有属性（`privateData`不能访问，而`_id`不能被修改，只能通过`getName()`获取值）。但是**如果不主动管理，由于无法获知对象实例何时被销毁，因此`privateData`中的数据就永远不会消失。**

```js
// ES 6
let Person = (function() {
    let privateData = new WeakMap();
    
    function Person(name) {
        privateData.set(this, { name: name });
    }
    
    Person.prototype.getName = function() {
        return privateData.get(this).name;
    };
    
    return Person;
})();
```

而使用`WeakMap`的做法，用`Person`对象实例作集合的键，只要对象被销毁，相关信息也会被及时销毁。

## 使用建议

如果只需要跟踪对象引用，则选`WeakSet`更好。

如果只是用对象作为键名，则选`WeakMap`更好，因为能有效避免内存泄漏。但如果要更多的特性使用集合，推荐`Map`。

---

__参考资料__

- 《深入理解ES6》
- [理解Object.defineProperty的作用](https://segmentfault.com/a/1190000007434923)
- [Object.defineProperty() - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)