---
layout: post
title:  "第3章 函数"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 函数
mathjax: true
---

* content
{:toc}

# 第3章 函数

## 默认参数

ECMAScript 5及以前，为函数参数赋予默认值会这样做：

```js
// 方式一
function makeReq(url, timeout, callback) {
    timeout = timeout || 2000;
    callback = callback || function() {};
    // ...
}

// 方式二
function makeReq(url, timeout, callback) {
    timeout = (typeof timeout !== "undefined") ? timeout : 2000;
    callback = (typeof callback !== "undefined") ? callback : function() {};
    // ...
}
```

方式一是有缺陷的，如果我们想给该函数第二个形参`timeout`传入值0，也会被视为一个假值。方式二是更安全的方法。

ECMAScript 6提供了更简便的写法：

```js
function makeReq(url, timeout = 2000, callback) {
    // ...
}
```

当不为第二个参数传入值或主动为第二个参数传入`undefined`时才会使用默认值。

ECMAScript 5下，默认参数对`arguments`对象的影响如下：

```js
function mixArgs(first, second) {
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = 'c';
    second = 'd';
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs('a', 'b');

// 非严格模式下，arguments会一起被修改，输出：
// true
// true
// true
// true

// 严格模式下，arguments不会被改变，输出：
// true
// true
// false
// false
```

而在ECMAScript 6中，使用默认参数值时，`arguments`对象行为都与ECMAScript 5严格模式下一致。

其它用法：

```js
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

// 不能这么做，second尚未初始化。
function add(first = second, second) {
    return first + second;
}
```

函数参数有自己的作用域和 __临时死区__ ，其与函数体的作用域是各自独立的，参数的默认值不可访问函数体内声明的变量。

## 不定参数

JavaScript函数语法规定，无论函数已定义的命名参数有多少，调用时可以传入任意数量的参数。因此在ECMAScript 5中，就会出现没有命名的参数，只能通过`arguments`访问，只查看函数参数定义不能保证掌握了所需参数的所有信息。

ECMAScript 6中，在函数命名参数前添加三个点`...`就表明这是一个不定参数，该参数为一个数组，包含自它之后传入的所有参数。

```js
function checkArgs(first, ...args) {
    console.log(args.length);
    console.log(arguments.length);
    console.log(args[0], arguments[0]);
    console.log(args[1], arguments[1]);
}

checkArgs('a', 'b', 'c');

console.log(checkArgs.length);

// 输出：
// 2
// 3
// b a
// c b
// 1
```

> 函数的`length`属性统计的是函数命名参数的数量，不包括不定参数和有默认值的参数。

不定参数的使用限制：

- 每个函数最多只能声明一个不定参数，而且一定要放在所有参数的末尾。
- 不定参数不能用于对象字面量`setter`中。`setter`的参数有且只能有一个。

```js
let obj = {
    // 语法错误
    set name(...value) {
        // ...
    }
}
```

## 增强Function构造函数

ECMAScript 6支持在创建函数时定义默认参数和不定参数，只需要在传入参数名时给相应的形式：

```js
var add = new Function('first', 'second = first', '...args', 'return first + second + args[0]');
```

## 展开运算符

使用新增的展开运算符可以简化使用数组给函数传参的编码过程：

```js
let values = [25, 50, 75, 100];

// 打印数组中最大的值
// ES5
console.log(Math.max.apply(Math, values));

// ES6
console.log(Math.max(...values));
```

## name属性

ECMAScript 6中为所有函数新增了`name`属性，每种情况都有合适的值，用于 __协助调试__：

```js
function doSomething() {
    // ...
}

var doAnotherThing = function() {
    // ...
}

var doSomething2 = function doSomethingElse() {
    // ...
}

var person = {
    get firstName() {
        return 'aaa';
    },
    sayName: function() {
        console.log(this.name);
    }
}

console.log(doSomething.name);
// 'doSomething'

console.log(doAnotherThing.name);
// 'doAnotherThing'

console.log(doSomething2.name);
// 'doSomethingElse'

console.log(person.sayName.name);
// 'sayName'

console.log(person.firstName.name);
// 'get firstName'
// setter函数的name也有'set'前缀

console.log(doSomething.bind().name);
// 'bound doSomething'（带有bound前缀）

console.log((new Function()).name);
// 'anonymous'
```

## 判断函数是否通过`new`调用：`new.target`

JavaScript函数有两个不同的内部方法：`[[Call]]`和`[[Construct]]`。当用`new`关键字调用函数时，执行的是`[[Construct]]`，它会创建一个实例，然后执行函数体，绑定实例；当不用`new`时，则执行`[[Call]]`，直接执行函数体。有`[[Construct]]`的函数叫构造函数。不是所有方法都有`[[Construct]]`，都可以用`new`调用，如前头函数。

判断函数是否通过`new`调用，以前的做法如下：

```js
function Person(name) {
    if (this instanceof Person) {
        this.name = name;
    } else {
        throw new Error('必须用new调用Person');
    }
}

var person = new Person('me');

// 有效，不报错
var notAPerson = Person.call(person, 'cat');
```

这种做法也无法区分是通过`Person.call`还是`new`调用得到实例。

所有，ECMAScript 6引入`new.target`元属性。元属性指非对象的属性，可提供非对象目标的补充信息。当调用了`[[Construct]]`，`new.target`的值就是要为其创建对象的类；若调用了`[[Call]]`，`new.target`的值就`undefined`。

```js
function Person(name) {
    if (new.target === Person) {
        this.name = name;
    } else {
        throw new Error('必须用new调用Person');
    }
}

var person = new Person('me');

// 报错
var notAPerson = Person.call(person, 'cat');

function AnotherPerson(name) {
    Person.call(this, name);
}
// 报错
var anotherPerson = new AnotherPerson('dog');
```

> 在函数外使用`new.target`是一个语法错误。

## 块级函数

```js
'use strict'

if (true) {
    // ES6中打印'function'
    console.log(typeof doA);
    
    // 报错
    console.log(typeof doB);

    // 在ES5中报错，在ES6中不报错
    function doA() {
        //...
    }
    
    let doB = function() {
        //...
    }
}
```

严格模式下，上面的代码在ES6中，会将`doA()`视作一个块级声明，被提升至顶部。然而 用`let`声明的函数表达式`doB()`不会被提升，会先放入临时死区中。

在ECMAScript 6中， __非严格模式__ 下，块级函数会提升到外围的函数或全局的作用域中。

## 箭头函数

箭头函数与传统函数有以下不同：

- 没有`this`，`super`，`arguments`，`new.target`绑定，这些值由外围的非箭头函数决定。
- 不能用`new`调用，没有`[[Construct]]`。
- 没有原型`prototype`。
- 不能改变`this`绑定。
- 不支持`arguments`对象。
- 不支持重复的命名参数（传统的可以）。

> 箭头函数也有`name`属性。

语法：

```js
// 单参数
let fa = a => a;

// 多参数
let fb = (a, b) => a + b;

// 没有参数
let fc = () => 'fc';

// 多表达式的函数体
let fd = (a, b) => {
    let m = a * b;
    return a + b + m;
}

// 空函数
let f = () => {};

// 返回对象
let fe = e => ({ val: e });

// 立即执行，小括号只包裹函数定义
let person = ((name) => {
    return {
        getName: function() {
            return name;
        }
    };
})('me');
```

> 箭头函数没有`this`, 它里面的`this`会按照变量查找的形式查找。

## 尾调用优化

尾调用指函数作为另一个函数的最后一条语句被调用。

ECMAScript 5引擎中，尾调用的实现与其它情况一样：创建一个新的栈帧（stack frame），将其推入调用栈来表示函数调用。

ECMAScript 6 则在 __严格模式__ 下优化了尾调用栈，以下情况，尾调用不再创建新栈帧，而是清除并重用当前栈帧：

- 尾调用不访问当前栈帧的变量（也不是闭包）。
- 尾调用是函数内部最后一条语句。
- 尾调用的结果 _直接_ 作为函数值返回。

---

__参考资料__

- 《深入理解ES6》