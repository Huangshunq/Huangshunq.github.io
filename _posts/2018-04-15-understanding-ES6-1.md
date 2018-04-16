---
layout: post
title:  "第1章 块级作用域绑定"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 块级作用域绑定
mathjax: true
---

* content
{:toc}

<!-- # 第1章 块级作用域绑定 -->

## `var`与变量提升（Hoisting）机制

通过`var`声明的变量，都会被当作当前作用域顶部声明的变量。在函数体内任意一处用`var`定义的变量，都会提升到函数的顶部。

```js
function func(arg) {
    if (arg) {
        var value = 'blue';
    } else {
    // 此处可访问 value不会报错，值为undefined。
        return null;
    }
    // 此处可访问 value不会报错，值为undefined。
}

// 上述函数会解析为:

function func(arg) {
    var value;
    if (arg) {
        value = 'blue';
    } else {
        return null;
    }
}
```

## 块级声明

块级声明用于声明在指定的作用域之外无法访问的变量。块级作用域（词法作用域）存在于：

- 函数内部
- 块中（字符`{`和`}`之间的区域）

## `let`声明与`const`声明

`let`声明和`const`声明是块级声明的方式，与var的变量提升不同。

不能在同一作用域中用`let`或`const`重复声明同名的变量（标识符），不同作用域中则允许。

用`let`声明的变量的值可以再更改。

`const`声明的是常量，其值一旦被设定后不可更改，声明时必须进行初始化。但是如果声明的是对象，则可以修改该对象的属性的值。

## 临时死区（Temporal Dead Zone，TZD）

JavaScript 引擎在扫描代码发现变量声明时，要么将它们提升至作用域顶部（遇到`var`声明），要么将声明放到TDZ中（遇到`let`和`const`声明）。访问TDZ中的变量会触发运行时错误。只有执行过变量声明语句后，变量才会从TDZ中移除，然后才能正常访问。

临时死区是 js 块级绑定的特色。

## 改进循环

为了输出 0 - 9，需要在循环中使用立即调用函数表达式（IIFE），以强制生成计数器变量的副本：

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push((function(value) {
        return function() {
            console.log(value);
        };
    })(i));
}

funcs.forEach(function(func) {
    func();  // 输出0，1，2，...，9
});
```

但是现在使用`let`，可以简化循环过程：

```js
var funcs = [];

for (let i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}

funcs.forEach(function(func) {
    func();  // 输出0，1，2，...，9
});
```

`let`在循环中的表现是：在每次迭代循环都会创建一个新变量，并用之前迭代中同名变量的值初始化这个新变量。在for-in和for-of循环的表现行为也一样。

`const`在for-in和for-of循环的表现行为也和`let`一样，但是在普通for循环初始化变量时使用，会报错：

```js
// 完成第一次迭代后报错
for (const i = 0; i < 10; i++) {
    console.log(i);
}

// 不会报错
for (const key in obj) {
    console.log(key);
}
```

`const`之所以可以运用在for-in和for-of循环中，是因为每次迭代时不会修改已有绑定的值，而只是创建一个新绑定。

## 全局块作用域绑定

当在全局作用域中使用`var`时，它会创建一个新的全局变量作为全局对象（浏览器环境中的`window`对象）的属性。这可能会造成无意中覆盖一个已存在的全局属性。

```js
// 在浏览器中
var RegExp = 'Hello';
console.log(window.RegExp); // 'Hello'
```

但是在全局作用域中使用`let`或`const`时，该变量不会添加为全局变量的属性。即用`let`或`const`不能覆盖全局变量，只能屏蔽它。

```js
// 同样是在浏览器中
let RegExp = 'haha';
console.log(RegExp); // 'haha'
console.log(window.RegExp === RegExp); // false
```

> 如果希望在全局对象下定义变量，仍然可以使用`var`。这种情况常见于在浏览器中跨frame或跨window访问代码。

## `let`与`const`实践总结

默认使用`const`，只有确实要改变变量值时才使用`let`。

---

__参考资料__

- 《深入理解ES6》
