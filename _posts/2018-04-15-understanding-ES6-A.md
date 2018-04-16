---
layout: post
title:  "附录 A 较小变动"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 其它变动
mathjax: true
---

* content
{:toc}

<!-- # 附录 A 较小变动 -->

## 识别整数

JavsScript 使用 IEEE 754 编码系统表示整数和浮点数。浮点数和整数的存储方式不同，有可能需要判断：

```js
// 传入一个值，引擎会查看该值的底层表示方式确定该值是否为整数：
console.log(Number.isInteger(25)); // true
console.log(Number.isInteger(25.0)); // true
console.log(Number.isInteger(25.1)); // false
```

> 只给数字添加小数点不会被存储为浮点数，如上

## 安全整数

```js
console.log(Math.pow(2, 53)); // 9007199254740992
console.log(Math.pow(2, 53) + 1); // 9007199254740992
```

IEEE 754 只能准确表示 -2^53 ~ 2^53 之间的整数。需要检测变量是否是准确范围内的整数：

```js
var inside = Number.MAX_SAFE_INTEGER, // ES 6 引入的常量，表示范围的最大值
    outside = inside + 1;

console.log(Number.isInteger(inside)); // true
console.log(Number.isSafeInteger(inside)); // true

console.log(Number.isInteger(outside)); // true
console.log(Number.isSafeInteger(outside)); // false
```

> `Number.MIN_SAFE_INTEGER` 常量表示准确范围内的最小值。

## 新的 `Math` 方法

## Unicode 标识符

以前可用Unicode转义字符作为标识符，现在可用码位转义字符作标识符：

```js
// 合法
var \u{61} = 'abc';

console.log(\u{61}); // 'abc'

console.log(a); // 'abc'
```

[正式指定的有效标识符](http://unicode.org/reports/tr31) 中，包括以下规则：

- 第一个字符必须是$、_ 或任何带有 ID\_Start 的派生核心属性的 Unicode 符号
- 后续的每个字符必须是$、\_、\u200c（零宽度不连字，zero-width non-joiner）、\u200d（零宽度连字、zero-width joiner）或具有 ID\_Continue 的派生核心属性的任何 Unicode 符号。

ID\_Start、ID\_Continue 派生的核心属性，用于标识适用于标识符（变量名和符号）的符号，非 js 特有。

## `__proto__`属性被加入标准（不建议使用）
