---
layout: post
title:  "附录 B ES2016（ES7）"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 关于 ES7
mathjax: true
---

* content
{:toc}

<!-- # 附录 B ES2016（ES7） -->

## 指数运算符 `**`

```js
let result = 5 ** 2;

console.log(result); // 25
console.log(result === Math.pow(5, 2)); // true

// 具有最高的优先级（一元运算符优先级高于 ** ）
let result2 = 2 * 5 ** 2;
console.log(result); // 50

// 报错
let result3 = -5 ** 2;

// 合法
let result4 = -(5 ** 2); // -25
let result5 = (-5) ** 2; // 25
```

> 无需括号可用 ++ 和 --，因为没有歧义。

## `Array.prototype.includes`

```js
let values = [1, NaN, 2];

console.log(values.includes(1)); // true
console.log(values.includes(0)); // false

// 从索引 2 开始搜索
console.log(values.includes(1, 2)); // false

// index 使用 === 比较
console.log(values.indexOf(NaN)); // -1
console.log(values.includes(NaN)); // true
// 但是 +0 与-0 都被认为相等
// Object.is() 认为 +0 与 -0 不相等
```

## 严格模式的改动

只有参数为不包含解构或默认值非法的简单参数列表时才能在函数体使用 `'use strict'`。

```js
// 此时，ES 6 标准会让引擎用严格模式处理参数，
// 然而实际引擎实现该特性非常困难，所以没实现
function doSomething(first = this) {
   'use strict';
   return first;
}

// ES 7

// 合法
function okay(first, second) {
    'use strict';
    return first;
}

// 报错
function notOkay1(first, second = first) {
    'use strict';
    return first;
}

// 报错
function notOkay2({ first, second }) {
    'use strict';
    return first;
}
```