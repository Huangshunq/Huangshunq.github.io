---
layout: post
title:  "第4章 拓展对象的功能性"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 拓展对象的功能性
mathjax: true
---

* content
{:toc}

# 第4章 拓展对象的功能性

## 对象类别

- 普通（Ordinary）对象：具有JavaScript对象所有的默认内部行为。
- 特异（Exotic）对象：具有某些与默认行为不符的内部行为。
- 标准（Standard）对象：ECMAScript 6 规范中定义的对象如`Date`，可以是普通对象，也可以是特异对象。
- 内建对象：脚本开始执行时存在于JavaScript执行环境中的对象，所有标准对象都是内建对象。

## 对象字面量语法拓展

新增了下面几种语法：

```js
// ES5
function createPerson(name, age) {
    return {
        name: name,
        age: age,
        sayName: function() {
            console.log(this.name);
        }
    };
}

// ES6
function createPerson(name, age) {
    return {
        // 属性初始化简写
        name,
        age,
        // 方法简写
        sayName() {
            console.log(this.name);
        }
    };
}
```

> 简写方法具有`name`属性，值为小括号前的名称，还可以使用`super`关键字，而非简写则不能。

### 计算属性（Computed Property Name）

对象字面量中，想通过计算获得属性名，需要用方括号代替点记法，有特定字符的字符串字面量作为标识符会出错，但可直接作为属性名，也可用方括号访问：

```js
var person = {
    'first name': 'Tim' // 字符串作为属性名
},
    lastName = 'last name';
    
person['first name'] = 'Baker'; // 方括号访问
person[lastName] = 'Zakas'; // 计算属性

console.log(person['first name']);
console.log(person[lastName]);
```

ECMAScript 6中，对象字面量也可用计算属性作为属性名称：

```js
var lastName = 'last name';
var person = {
    'first name': 'Baker',
    [lastName]: 'Tim' // 定义计算属性
};

console.log(person['first name']);
console.log(person[lastName]);
```

## 新增对象方法

### `Object.is()`

```js
console.log(+0 == -0);              // true
console.log(+0 === -0);             // true
console.log(Object.is(+0, -0));     // false

console.log(NaN == NaN);            // false
console.log(NaN === NaN);           // false
console.log(Object.is(NaN, NaN));   // true

console.log(5 == 5);                // true
console.log(5 == '5');              // true
console.log(5 === 5);               // true
console.log(5 === '5');             // false

console.log(Object.is(5, 5));       // true
console.log(Object.is(5, '5'));     // false
```

`Object.is()`运行结果大部分与`===`运算符相同，唯一区别是`Object.is()`会将`+0`和`-0`识别为不相等并且`NaN`与`NaN`等价。

### `Object.assign()`

混合（Mixin）是让一个对象不用继承而获得其它对象属性的模式（合并对象）。

`Object.assign()`实现相同的功能，实行的是 __浅拷贝__（如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用）：

```js
Object.assign(target, source1, source2);
// 第一个参数是目标对象，后面的参数都是源对象
```

> - 如果多个源对象具有同名属性，则后面会覆盖前面的。
> - `Object.assign`只能进行值的复制，如果要复制的值是一个取值函数`get`，那么将求值后再复制。

```js
const source = {
  get foo() { return 1 }
};
const target = {};

Object.assign(target, source)
// { foo: 1 }
```

## 重复字面量属性

```js
'use strict'

var person = {
    name: 'Bob',
    name: 'Tim'
}
```

ES5下，重复属性会报错，但是ECMAScript 6下不报错，值会取后一个。

## 自有属性枚举顺序

以前未定义对象属性的枚举顺序，ECMAScript 6规定了自有属性枚举的顺序规则：

1. 所有数字键按升序排序，优先排在前面
1. 所有字符串按照它们被加入对象的顺序排序，排在数字键后面。
1. 所有`symbol`键按照它们被加入对象顺序排序。
 
> `for-in`循环、`Object.keys()`、`JSON.stringify()`枚举顺序尚不明确。

## 对象原型

### 改变对象的原型

`Object.setPrototypeOf()`可以改变任意指定对象的原型，第一个参数是被改变原型的对象，第二个参数是替代第一个参数原型的对象。

对象原型的真实值被储存在内部专门属性`[[Prototype]]`中，调用`Object.getPrototypeOf()`会返回储存在其中的值，调用`Object.setPrototypeOf()`会改变其中的值。

### `super`访问引用

如果你想重写对象实例的方法，又要调用与它同名的原型方法，可以用`super`：

```js
let dog = {
    getGreeting() {
        return 'Woof';
    }
};

// ES6
let friend = {
    getGreeting() {
        // 相当于ES5中：
        // Object.getPrototypeOf(this).getGreeting.call(this) + ', hi!';
        return super.getGreeting() + ', hi!';
    }
};

Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting()); // 'Woof, hi!'

// 语法错误：必须使用简写方法才能用 super
let anotherFriend = {
    getGreeting: function() {
        return super.getGreeting() + ', haha!';
    }
};
```

> `super`引用可以解决以前多重继承的问题。

## 将方法定义为一个函数

ECMAScript 6中将方法定义为一个函数，它会有一个内部的`[[HomeObject]]`属性储存方法从属的对象。

`super`会在`[[HomeObject]]`属性上调用`Object.getPrototypeOf()`得到原型的引用，然后搜寻原型找到同名函数，最后设置`this`绑定并且调用相应的方法。

```js
let person = {
    // 是方法
    getGreeting() {
        return 'hi!';
    }
};

// 不是方法
function shareGreeting() {
    return 'hi!';
}
```

---

__参考资料__

- 《深入理解ES6》
- [《ECMAScript 6 入门》](http://es6.ruanyifeng.com/#docs/string#%E6%A8%A1%E6%9D%BF%E5%AD%97%E7%AC%A6%E4%B8%B2)