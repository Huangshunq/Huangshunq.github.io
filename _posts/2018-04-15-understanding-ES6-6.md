---
layout: post
title:  "第6章 `Symbol`和`Symbol`属性"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: Symbol
mathjax: true
---

* content
{:toc}

<!-- # 第6章 `Symbol`和`Symbol`属性 -->

## `Symbol`介绍

之前版本的语言中有字符串型、数字型、布尔型、`null`和`undefined`5种原始数据类型，现在新增一种`Symbol`原始数据类型，用来保证每个属性的名字都是独一无二的。

```js
// 使用方式
let firstName = Symbol();
let person = {};

person[firstName] = 'someone';

// 不能使用 new 创建实例
let name = new Symbol(); // 报错

// 可以传入一个用于描述的参数
let secondName = Symbol('second name');
console.log(secondName); // 'Symbol(second name)'

// 可用 typeof 检测 symbol 类型，推荐
console.log(typeof secondName); // 'symbol'

// 可以用于对象字面量中计算属性
let otherPeople = {
    [secondName]: 'second one'
};

// 将属性设置为只读：
Object.defineProperty(person, firstName, { writable: false });

Object.defineProperties(person, {
    [secondName]: {
        value: 'second one',
        writable: false
    }
});
```

`Symbol`用于描述的参数存储在`[[Description]]`内部属性中，调用`Symbol.toString()`才能读取该属性，`console.log()`隐式调用该方法并输出。

## 全局`Symbol`

若想在不同地方共享同一个`Symbol`，可以利用新增的全局`Symbol`注册表。使用`Symbol.for()`创建获取一个共享的`Symbol`；`Symbol.keyFor()`在全局注册表上检索有关的键。

```js
// 传入一个作为键的参数，也用于描述
let uid = Symbol.for('uid');

let object = {};

object[uid] = '123';

console.log(uid); // 'Symbol(uid)'

// 先在全局 Symbol 注册表上找键为'uid'的Symbol，找到则返回该值，否则新建一个返回
let uid2 = Symbol.for('uid');

console.log(uid === uid2);
console.log(uid2); // 'Symbol(uid)'

// 用值检索对应键
console.log(Symbol.keyFor(uid)); // 'uid'
console.log(Symbol.keyFor(uid2)); // 'uid'

// 没注册
let uid3 = Symbol('uid');
console.log(Symbol.keyFor(uid3)); // undefined
```

## `Symbol`类型转换

`Symbol`类型不能强制转换成字符串（字符拼接会报错）和数字类型。使用逻辑运算符时，`Symbol`等价为`true`。

## `Symbol`属性检索

`Symbol` 作为属性名，该属性不会出现在`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`返回。但是，它也不是私有属性，`Object.getOwnProperty-Symbols()`可以检索对象中`Symbol`属性，返回包含所有拥有的`Symbol`的数组。

## well-known Symbol

除了定义自己使用的 `Symbol` 值以外，ES6 还提供了 11 个内置的`Symbol`值，指向语言内部使用的方法。重写这些方法，会改变对象的内部默认行为，变成一个奇异（exotic）对象。

### `Symbol.hasInstance`

用于确定对象是否为函数实例，只接受一个参数，即要检查的值，若值为函数实例，则返回`true`。定义在`Function.prototype`中，不可写、不可配置、不可枚举。

```js
obj instanceof Array;

// 等价于
Array[Symbol.hasInstance](obj);
```

因为其不可写，只能用`Object.defineProperty()`改写。建议只在确实有需要才修改，因为会让运行结果不合常态：

```js
function MyObject() {
    // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(v) {
        return false;
    }
});

let obj = new MyObject();

console.log(obj instanceof MyObject); // false
```

### `Symbol.isConcatSpreadable`

该属性是一个布尔值，标准对象没有这个属性，用于增强对象类型的`concat()`功能：

```js
let collection = {
    0: 'hello',
    1: 'world',
    length: 2,
    [Symbol.isConcatSpreadable]: true
};

let msg = ['Hi'].concat(collection);

console.log(msg.length); // 3
console.log(msg); // ['Hi', 'hello', 'world']
```

该值为`true`表明属性值应当作为独立元素添加到数组中。

### 正则的`Symbol`属性

- `Symbol.match`
- `Symbol.replace`
- `Symbol.search`
- `Symbol.split`

以上属性分别表示`match()`、`replace()`、`search()`、`split()`第一个参数应该调用正则表达式的方法，定义在`RegExp.prototype`中。对象的这些属性，指向一个函数。当执行对应方法时，如果该属性存在，会调用它，返回该方法的返回值。

```js
// 等价于 /^.{10}$/
// val 参数为字符串
let hasLengthOf10 = {
    [Symbol.match]: function(val) {
        return val.length === 10 ? [val] : null;
    },
    // 接受两个参数，第二个为替换用的字符串
    [Symbol.replace]: function(val, rep) {
        return val.length === 10 ? rep : val;
    },
    [Symbol.search]: function(val) {
        return val.length === 10 ? 0 : -1;
    },
    [Symbol.split]: function(val) {
        return val.length === 10 ? [, ] : [val]
    }
};

let msg1 = 'hello world'; // 11
let msg2 = 'hello john'; // 10

let match1 = msg1.match(hasLengthOf10), // null
    match2 = msg2.match(hasLengthOf10); // ['hello john']

let replace1 = msg1.replace(hasLengthOf10), // 'hello world'
    replace2 = msg2.replace(hasLengthOf10); // 'hello john'

let search1 = msg1.search(hasLengthOf10), // -1
    search2 = msg2.search(hasLengthOf10); // 0

let split1 = msg1.split(hasLengthOf10), // ['hello world']
    split2 = msg2.split(hasLengthOf10); // ['', '']
```

### `Symbol.toPrimitive`

该属性指向的方法定义在每一个标准类型的原型上，规定了对象被转换为原始值时应当执行的操作。当对象需要转换为原始值时，总会调用该属性的方法并传入一个参数，该参数叫类型提示（hint）。类型提示参数的值只有三种：`'number'`、`'string'`、`'default'`，对应返回值通常分别是：数字、字符串、无类型偏好的值。

方法内部有三种动作：

1.调用`valueOf()`，若结果为原始值，则返回

2.调用`toString()`，若结果为原始值，则返回

3.抛出错误

数字模式下，会按 1，2，3 的顺序运行；

字符串模式下，会按 2，1，3 的顺序运行；

默认模式下，大多数对象按数字模式处理（`Date`对象会按字符串模式处理）。该模式只用于`==`运算，`+`运算及给`Date`构造函数传递一个参数。

```js
function Tem(deg) {
    this.deg = deg;
}

// 覆写
Tem.prototype[Symbol.toPrimitive] = function(hint) {
    switch (hint) {
        case 'string':
            return this.deg + '\u00b0'; // 度数符号
        case 'number':
            return this.deg;
        case 'default':
            return this.deg + ' deg';
    }
};

var freezing = new Tem(32);

console.log(freezing + '!'); // '32 degrees!'
console.log(freezing / 2); // 16
console.log(String(freezing)); // '32\u00b0'
```

### `Symbol.toStringTag`

ECMAScript 6 重新定义了原生对象过去的状态，通过该属性改变了调用`Object.prototype.toString()`时返回的值，每个对象都有该属性：

```js
// ES 5
function isArray(val) {
    return Object.prototype.toString.call(val) === '[object Array]';
}

// ES 6 可以自定义该属性
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = 'Person';

var me = new Person('me');

console.log(me.toString()); // '[object Person]'
console.log(Object.prototype.toString.call(me)); // '[object Person]'
```

> 所有对象的`Symbol.toStringTag`默认都为'object'

该属性可以随意修改，但最好不要修改属性。

### `Symbol.unscopables`

该属性通常用于`Array.prototype`，用来标示在`with`语句中不创建绑定的属性名。

```js
var values = [1, 2, 3],
    colors = ['red', 'green', 'blue'];

// ES 6 中，语句内 values 将会被绑定为 Array.values，与原意不符。
with(colors) {
    push(...values);
}

console.log(colors);

// 该属性指向对象，键为在 with 语句中忽略的标识符，值必须为 true
// with 语句将不再创建这些方法的绑定，以支持老代码运行
Array.prototype[Symbol.unscopables] = Object.assign(Object.create(null), {
    copyWithin: true,
    entries: true,
    fill: true,
    find: true,
    findIndex: true,
    keys: true,
    values: true
})
```

---

__参考资料__

- 《深入理解ES6》
- [《ECMAScript 6 入门》](http://es6.ruanyifeng.com/#docs/symbol)