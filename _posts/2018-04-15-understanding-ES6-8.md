---
layout: post
title:  "第8章 Iterator和Generator"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: Iterator和Generator
mathjax: true
---

* content
{:toc}

<!-- # 第8章 Iterator和Generator -->

## 迭代器 Iterator

迭代器是一种特殊对象，它具有一些专门为迭代过程设计的专有接口，所有的迭代器对象都有一个`next()`方法，返回值是一个有两个属性的对象，一个是`value`，表示下一个要返回的值，另一个是`done`，值为布尔型，在没有下一个可返回的值时返回`true`，否则为`false`。

除此之外，迭代器还会保存一个内部指针来指向当前集合中值的位置。

```js
// ES5 模拟实现
function createIterator(items) {
    var i = 0;

    return {
        next: function() {
            var done = (i >= items.length);
            var value = !done ? items[i++] : undefined;

            return {
                done: done,
                value: value
            };
        }
    };
}

var iterator = createIterator([1, 2, 3]);

console.log(iterator.next()); // '{ value: 1, done: false }'
console.log(iterator.next()); // '{ value: 2, done: false }'
console.log(iterator.next()); // '{ value: 3, done: false }'
console.log(iterator.next()); // '{ value: undefined, done: true }'
// 之后结果不变
console.log(iterator.next()); // '{ value: undefined, done: true }'
```

## 生成器 Generator

生成器是一种返回迭代器的函数，形式如下：

```js
// ES6 生成器
function *createIterator() {
    yield 1;
    yield 2;
    yield 3;
}

// 会返回一个迭代器
let iterator = createIterator();

console.log(iterator.next().value); // 1
console.log(iterator.next().value); // 2
console.log(iterator.next().value); // 3
```

在生成器函数中，每当执行完一条`yield`语句后函数会自动停止执行，等待下一条`next()`才继续执行。

> `yield`只能在生成器函数内部使用，在内部嵌套的普通函数内使用也不行。

```js
// 函数表达式创建生成器
let createIterator = function *(items) {
    for (let i = 0; i < items.length; i++) {
        yield items[i];
    }
}

// 对象简写创建方式
let o = {
    *createIterator(items) {
        for (let i = 0; i < items.length; i++) {
            yield items[i];
        }
    }
}
```

> 不能用箭头函数创建生成器。

## 可迭代对象

可迭代对象具有`Symbol.iterator`属性，是一种与迭代器密切相关的对象。`Symbol.iterator`可返回迭代器。在ES6中，所有集合对象（数组、`Set`和`Map`）和字符串都是可迭代对象，都有默认迭代器。

### 1. `for-of` 循环

`for-of`运行时，首先会调用可迭代对象的`Symbol.iterator`方法获取迭代器，然后每次循环都会调用迭代器的`next()`，并将迭代器返回对象的`value`存在指定变量中，直到`done`为`true`时循环退出。

```js
let values = [1, 2, 3];

for (let num of values) {
    console.log(num);
}
// 1
// 2
// 3
```

> `for-of`用于不可迭代对象、`null`、`undefined`会报错。

### 2. 访问默认迭代器

可用`Symbol.iterator`访问对象默认迭代器：

```js
let iterator = values[Symbol.iterator]();

// 检测对象是否为可迭代对象
function isIterable(object) {
    return typeof object[Symbol.iterator] === 'function';
}
```

### 3. 创建可迭代对象

开发者自定义的对象都是不可迭代对象，但给`Symbol.iterator`属性添加生成器，可变为可迭代对象。

```js
let collection = {
    items: [],
    *[Symbol.iterator]() {
        for (let item of this.items) {
            yield item;
        }
    }
};
```

## 内建迭代器

一般来说，内建迭代器已经可解决日常的问题，最常使用的是集合的迭代器。数组、`Map`、`Set`都内建了以下迭代器：

- entries() 返回一个迭代器，值为多个键值对
- values() 返回一个迭代器，值为集合的值
- keys() 返回一个迭代器，值为集合中所有键名

```js
let colors = ['red', 'green', 'blue'];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set('title', 'abcde');
data.set('format', 'book');

for (let entry of colors.entries()) {
    console.log(entry);
}
// [0, 'red']
// [1, 'green']
// [2, 'blue']

for (let entry of tracking.entries()) {
    console.log(entry);
}
// [1234, 1234]
// [5678, 5678]
// [9012, 9012]

for (let entry of data.entries()) {
    console.log(entry);
}
// ['title', 'abcde']
// ['format', 'book']

for (let value of colors.values()) {
    console.log(entry);
}
// 'red'
// 'green'
// 'blue'

for (let entry of tracking.values()) {
    console.log(entry);
}
// 1234
// 5678
// 9012

for (let entry of data.values()) {
    console.log(entry);
}
// 'abcde'
// 'book'

for (let entry of colors.keys()) {
    console.log(entry);
}
// 0
// 1
// 2

for (let entry of tracking.keys()) {
    console.log(entry);
}
// 1234
// 5678
// 9012

for (let entry of data.keys()) {
    console.log(entry);
}
// 'title'
// 'format'

// 默认情况调用 colors.values()
for (let entry of colors) {
    console.log(entry);
}
// 'red'
// 'green'
// 'blue'

// 默认情况调用 tracking.values()
for (let entry of tracking) {
    console.log(entry);
}
// 1234
// 5678
// 9012

// 默认情况调用 data.entries()
for (let entry of data) {
    console.log(entry);
}
// ['title', 'abcde']
// ['format', 'book']

// 可在循环中用解构语法
for (let [key, value] of data) {
    console.log(`${key} = ${value}`);
}
```

ES5 不支持双字节字符的问题，ES6 解决了：

```js
var message = 'A 𠮷 B';

// ES5
for (let i = 0; i < message.length; i++) {
    console.log(message[i]); // 错误输出
}

// ES6
for (let c of message) {
    console.log(c); // 正确输出
}
```

## `NodeList`迭代器

DOM标准中`NodeList`类型代表页面文档所有元素的集合，现在也有默认迭代器，行为与数组默认迭代器一致。

## 展开运算符

展开运算符可以操作所有可迭代对象。

## 更多功能

### 给迭代器传参

```js
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2; // 4 + 2
    yield second + 3; // 5 + 3
}

let iterator = createIterator();

console.log(iterator.next()); // '{ value: 1, done: false }'
console.log(iterator.next(4)); // '{ value: 6, done: false }'
console.log(iterator.next(5)); // '{ value: 8, done: false }'
console.log(iterator.next()); // '{ value: undefined, done: true }'
```

上例中，第一次调用`next()`无论传什么参数都没用，由于传给`next()`的参数是替代上一次`yield`返回值，而第一次调用`next()`前不会执行任何`yield`语句。

第二次调用的参数会成为 `yield 1` 的值，赋予`first`变量，同理，`second`被赋予`5`。

### 迭代器抛出错误

```js
function *createIterator() {
    let first = yield 1;
    let second;

    try{
        second = yield first + 2;
    } catch(e) {
        second = 6;
    }
    yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next()); // '{ value: 1, done: false }'
console.log(iterator.next(4)); // '{ value: 6, done: false }'
console.log(iterator.throw(new Error('Boom'))); // 在生成器中抛出错误，返回 '{ value: 9, done: false }'
console.log(iterator.next()); // '{ value: undefined, done: true }'
```

### 生成器返回语句

```js
function *createIterator() {
    yield 1;
    return 42;
    yield 2;
    yield 3;
}

let iterator = createIterator();

console.log(iterator.next()); // '{ value: 1, done: false }'
console.log(iterator.next()); // '{ value: 42, done: true }'
console.log(iterator.next()); // '{ value: undefined, done: true }'
```

> 展开运算符与`for-of`循环只要检测到`done`变为`true`就立刻停止读取其他值。

### 委托生成器

若要将两个迭代器合二为一，可以这样：

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createRepeatingIterator(count) {
    for (let i = 0; i < count; i++) {
        yield 'repeat';
    }
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield *createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next()); // '{ value: 1, done: false }'
console.log(iterator.next()); // '{ value: 2, done: false }'
console.log(iterator.next()); // '{ value: 'repeat', done: false }'
console.log(iterator.next()); // '{ value: 'repeat', done: false }'
console.log(iterator.next()); // '{ value: 'repeat', done: false }'
console.log(iterator.next()); // '{ value: undefined, done: true }'
```

> `yield 'hello'`语句将使用字符串的默认迭代器。

### 实例：异步任务执行器

```js
function run(taskDef) {
    // 创建一个无使用限制的迭代器
    let task = taskDef();

    // 开始执行任务
    let result = task.next();

    // 循环调用 next() 函数
    function step() {
        // 如果任务未完成，则继续执行
        if (!result.done) {
            if (typeof result.value === 'function') {
                result.value(function(err, data) {
                    if (err) {
                        result = task.throw(err);
                        return;
                    }

                    result = task.next(data);
                    step();
                });
            } else {
                result = task.next(result.value);
                step();
            }
        }
    }

    // 开始迭代执行
    step();
}

// 若要读取文件，可创建包装器（wrapper）
let fs = require('fs');
function readFile(filename) {
    return function(callback) {
        fs.readFile(filename, callback);
    };
}

// 使用：
run(function*() {
    let contents = yield readFile('config.json');
    doSomethingWith(contents);
    console.log('Done');
});
```

---

__参考资料__

- 《深入理解ES6》
