---
layout: post
title:  "第10章 改进数组的功能"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 改进数组的功能
mathjax: true
---

* content
{:toc}

<!-- # 第10章 改进数组的功能 -->

## `Array.of()`

ES 5 及以前，创建数组有很多坑：

```js
let items = new Array(2);
console.log(items.length); // 2
console.log(items[0]); // undefined
console.log(items[1]); // undefined

items = new Array('2');
console.log(items.length); // 1
console.log(items[0]); // '2'

items = new Array(1, 2);
console.log(items.length); // 2
console.log(items[0]); // 1
console.log(items[1]); // 2

items = new Array(3, '2');
console.log(items.length); // 2
console.log(items[0]); // 3
console.log(items[1]); // '2'
```

ES 6 引进`Array.of()`解决这个问题：

```js
let items = Array.of(1, 2);
console.log(items.length); // 2
console.log(items[0]); // 1
console.log(items[1]); // 2

items = Array.of(2);
console.log(items.length); // 1
console.log(items[0]); // 2

items = Array.of('2');
console.log(items.length); // 1
console.log(items[0]); // '2'
```

> `Array.of()`不用 `Symbol.species`确定返回值类型，使用`.of()`中的`this`来确定正确的返回值类型。

## `Array.from`

ES 5 中，要用一些手段才能将非数组对象转换为数组：

```js
function makeArray(arrayLike) {
    return Array.prototype.slice.call(arrayLike);
}

function doSomething() {
    var args = makeArray(arguments);
    // ...
}
```

ES 6 则可用`Array.from()`，接受可迭代对象（或所有含`Symbol.iterator`的对象）或类数组对象为第一个参数，返回一个数组：

```js
var args = Array.from(arguments);

// 第二个参数是函数，用于改变元素中的每一个值
// 第三个参数可指定 this 的值
let num = Array.from(arguments, val => val + 1, this);
```

> `Array.from()`也用 `this`确定返回数组的类型。如果对象既是类数组又可迭代，就会根据迭代器决定转换哪个值。

## 数组新方法

- `find(fn[, this])`：返回第一个符合函数判断条件的值。
- `findIndex(fn[, this])`：返回第一个符合函数判断条件的值的索引。
- `fill(val, startIndex, endIndex)`：用指定的值填充一至多个元素。
- `copyWithin()`

```js
// 从数组的索引 2 开始黏贴值
// 从数组的索引 0 开始复制值
// 当位于索引 1 时停止复制值
num.copyWithin(2, 0, 1);
```

## 定型数组（Typed arrays）

定型数组是一种用于处理数值类型数据的专用数组，将任何数字转换为一个包含数字比特的数组，最早在WebGL（OpenGL ES 2.0 移植版）中使用，用`<canvas>`呈现。

js 中用64个比特存储一个浮点形式的数字，并按需转换为32位整数，算术运算非常慢，定型数组可提供高性能运算。

### 数组缓冲区

数组缓冲区是一段可以包含特定数量字节的内存地址：

```js
let buffer = new ArrayBuffer(10); // 分配10字节，大小不可再修改
console.log(buffer.byteLength); // 10
```

### 视图

视图可用来操作数组缓冲区或缓冲区字节的子集。比如`DataView`：

```js
let buffer = new ArrayBuffer(10),
    view = new DataView(buffer, 5, 6); // 包含索引 5 和 6 的字节
// 视图只读属性
console.log(view.buffer === buffer); // true
console.log(view.byteOffset); // 5
console.log(view.byteLength); // 2

// 原始缓冲区内容：0000000000000000
view.setInt8(0, 5); // 0000010100000000
view.setInt8(1, -1); // 0000010111111111
```

特定类型视图（即定型数组）表：

构造函数名称 | 元素字节 | 说明 | 等价C语言类型
-|-|-|-
Int8Array | 1 | 8位二进制补码有符号整数 | 有符号char
Uint8Array | 1 | 8位无符号整数 | 无符号char
Uint8ClampedArray | 1 | 8位无符号整数（强制转换） | 无符号char
Int16Array | 2 | 16位二进制补码有符号整数 | short
Uint16Array | 2 | 16位无符号整数 | 无符号short
Int32Array | 4 | 32位二进制补码有符号整数 | int
Uint32Array | 4 | 32位无符号整数 | int
Float32Array | 4 | 32位IEEE浮点数 | float
Float64Array | 8 | 64位IEEE浮点数 | double

> `Uint8ClampedArray`的缓冲区的值如果小于0或大于255，会分别转换为0或255（如-1变为0）。

```js
// 第一种创建方式
let buffer = new ArrayBuffer(10),
    view = new Int8Array(buffer, 5, 2); // 比特偏移量和长度值

// 第二种创建方式
let floats = new Float32Array(5); // 字节数

// 第三种创建方式
// 传入唯一参数：定型数组、可迭代对象、数组或类数组对象
let ints1 = new Int16Array([25, 50]),
    ints2 = new Int32Array(ints1);

console.log(UInt8Array.BYTES_PER_ELEMENT); // 1
console.log(UInt16Array.BYTES_PER_ELEMENT); // 2
console.log(ints1.BYTES_PER_ELEMENT); // 1

// 与普通数组的差别
console.log(ints1 instanceof Array); // false
console.log(Array.isArray(ints1)); // false
console.log(ints1 instanceof Int16Array); // true

// 非法值会用0代替
let ints = new Int16Array(['hi!']);
console.log(ints[0]); // 0
let mapped = ints.map(v => 'hi!');
console.log(mapped[0]); // 0
```

> 定型数组都有普通数组的属性和方法，有相同的迭代器，但没有会改变数组长度的方法。

### 附加方法

- `set()`：将其它数组复制到已有定型数组上。
- `subarray()`：提取定型数组一部分作为一个新的定型数组。

```js
let ints = new Int16Array(4);

ints.set([25, 50]);
// 参数：数组和偏移量
ints.set([75, 100], 2);

console.log(ints.toString()); // 25, 50, 75, 100

// 参数：开始位置和结束位置
let subints1 = ints.subarray(),
    subints2 = ints.subarray(2),
    subints3 = ints.subarray(1, 3);

console.log(subints1.toString()); // 25, 50, 75, 100
console.log(subints2.toString()); // 75, 100
console.log(subints3.toString()); // 50, 75
```

---

__参考资料__

- 《深入理解ES6》
