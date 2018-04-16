---
layout: post
title:  "第5章 解构：使数据访问更便捷"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 了解下解构
mathjax: true
---

* content
{:toc}

# 第5章 解构：使数据访问更便捷

> 解构，是一种打开数据结构，将其拆分为更小部分的过程。

## 对象解构

对象解构形式如下：

```js
let { type, name } = node;
// 相当于下面的简写形式：
let { type: type, name: name } = node;
// node.type的值存到type中，node.name的值存到name中，声明的 type，name 都是局部变量。
```

一旦使用解构功能，必须提供等式右侧的用来初始化的值。

```js
// 以下都是语法错误
var { type, name };
let { type, name };
const { type, name };
```

使用解构给已存在的变量赋值，**必须加括号**，因为语法规定，代码块语句不允许出现在赋值语句的左侧：

```js
let type = 'type', name = 5;

({ type, name } = node);
```

解构赋值表达式的值与表达式右侧的值相等，如果右侧的值为`null`或`undefined`，会报错。

```js
let node = {};

function fn(val) {
    console.log(val === node); // true
}

fn({ type, name } = node);
```

如果指定的局部变量不存在，会赋值为`undefined`，这种情况下，若有默认值，则局部变量值为默认值。

```js
let node = {
    type: 1,
    name: 2
};
// 此时 value 的值为 undefined
let { value } = node; 
// 此时 value 的值为 true
let { type, name, value = true } = node;
```

如果希望使用不同名的局部变量来存储对象属性的值，使用如下形式：

```js
let node = {
    type: 1,
    name: 2
};

// 读取 type 的值并存到 localType 中，即变量名在冒号右边：
let { type: localType } = node;

console.log(localType); // 1

// 添加默认值的形式：
let { value: localValue = true } = node;

console.log(localValue); // true
```

还可以解构对象中的对象：

```js
let node = {
    loc: {
        start: 1
    }
};

let { loc: { start } } = node;
let { loc: { start: localStart } } = node;

console.log(start); // 1
console.log(localStart); // 1

// 未声明任何变量，合法但什么都没做
let { loc: {} } = node;
```

## 数组解构

数组解构形式如下：

```js
let colors = [ 'red', ['green', 'lightgreen'], 'blue' ],
    firstColor = '?';

// 已声明和未声明的变量都可赋值，支持嵌套赋值
let [ firstColor, [ secondColor ] ] = colors;

// 可以直接忽略元素
let [ , , thirdColor ] = colors;

// 可以有默认值，与对象解构规则相同
let [ , , , forthColor = 'yellow' ] = colors;

console.log(firstColor); // 'red'
console.log(secondColor); // 'green'
console.log(thirdColor); // 'blue'
console.log(forthColor); // 'yellow'
```

> 当通过 `var`，`let`，`const`声明数组解构的绑定时必须初始化（与对象解构相同）。

可以用解构交换变量的值：

```js
let a = 1, b = 2;

[ a, b ] = [ b, a ];

console.log(a); // 2
console.log(b); // 1
```

可以用`...`将数组中其余元素赋值给一个特定的变量：

```js
let colors = [ 'red', 'green', 'lightgreen', 'blue' ];

let [ firstColor, ...restColors ] = colors;

console.log(firstColor); // 'red'
console.log(restColor.length); // 3
console.log(thirdColor[0]); // 'green'
console.log(forthColor[1]); // 'lightgreen'

// 可以用这语法实现相同的目标：
let [ ...clonedColors ] = colors;
```

> 不定元素必须为解构数组中最后一个元素，不然会报错。

## 解构参数

可以解构函数参数中的对象和数组，支持以上所有特性。但是不提供被解构的参数会导致程序错误。如果希望被解构的参数是可选的，必须提供默认值（规则和上述相同）：

```js
// 解构参数与默认值
function setCookie(name, value, { path, domian } = {}) {
    // ...
}

let obj = {};

function setCookie(name, value, { path, domian } = obj) {
    // ...
}
```

---

__参考资料__

- 《深入理解ES6》
- [《ECMAScript 6 入门》](http://es6.ruanyifeng.com/#docs/string#%E6%A8%A1%E6%9D%BF%E5%AD%97%E7%AC%A6%E4%B8%B2)