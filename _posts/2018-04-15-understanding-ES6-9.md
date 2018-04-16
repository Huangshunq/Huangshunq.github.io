---
layout: post
title:  "第9章 类"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 类的介绍
mathjax: true
---

* content
{:toc}

<!-- # 第9章 类 -->

## ES5 中的类

```js
function PersonType(name) {
    this.name = name;
}

// 共享方法
PersonType.prototype.sayName = function() {
    console.log(this.name);
}

var person = new PersonType('me');
person.sayName(); // 'me'

console.log(person instanceof PersonType); // true
console.log(person instanceof Object); // true
```

## ES6 中的类

### 类声明语法

```js
class PersonClass {
    // 等价于 PersonType 构造函数
    constructor(name) {
        this.name = name;
    }

    // 等价于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
}

let person = new PersonClass('me');
person.sayName(); // 'me'

console.log(person instanceof PersonClass); // true
console.log(person instanceof Object); // true

console.log(typeof PersonClass); // 'function'
console.log(typeof PersonClass.prototype.sayName); // 'function'
```

> 类声明仅是语法糖，实际创建了一个具有构造函数行为的函数。ES6 中类的 prototype 属性不可被赋予新值。

类与以前的构造函数有以下差异：

- 函数声明可以被提升，而类声明与`let`声明类似，不能被提升；真正执行声明语句之前，它们会一直存在于临时死区中。
- 类声明中所有代码将自动运行在严格模式下，且无法强行让代码脱离严格模式执行。
- 在自定义类型中，需用`Object.defineProperty()`手工指定某方法不可枚举；而类所有方法都不可枚举。
- 每个类都有一个名为`[[Construct]]`的内部方法，用`new`调用不含`[[Construct]]`的方法会报错。
- 用`new`以外方式调用类构造函数会报错。
- 在类中修改类名会报错。可在类外修改。

```js
// 与 ES6 中类 等价的语法
let PersonType2 = (function() {
    'use strict'

    const PersonType2 = function(name) {
        // 确保是用 new 调用
        if (typeof new.target === 'undefined') {
            throw new Error('必须用 new 调用');
        }

        this.name = name;
    }

    Object.defineProperty(PersonType2.prototype, 'sayName', {
        value: function() {
            // 确保不用 new 调用
            if (typeof new.target !== 'undefined') {
                throw new Error('不能用 new 调用');
            }
            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
    });

    return PersonType2;
}());
```

### 类表达式

```js
// 等价于前面声明形式
let PersonClass = class {
    constructor(name) {
        this.name = name;
    }

    sayName() {
        console.log(this.name);
    }
};

// 命名类表达式：
// PersonClass2 相当于 前面类等价的语法中 const 的变量名
let PersonClass = class PersonClass2 {
    constructor(name) {
        this.name = name;
    }

    sayName() {
        console.log(this.name);
    }
};

console.log(typeof PersonClass); // 'function'
console.log(typeof PersonClass2); // 'undefined'
```

## 一等公民的类

一等公民：指一个可以传入函数，可以从函数返回，且可赋值给变量的值。

ES 6 将类设为一等公民。

还可用立即调用类构造函数：

```js
let person = new class {
    constructor(name) {
        this.name = name;
    }
    sayName() {
        console.log(this.name);
    }
}('me');
```

类支持直接在原型上定义访问器属性和生成器：

```js
class CustomHTMLElement {
    constructor(element) {
        this.element = element;
    }

    get html() {
        return this.element.innerHTML;
    }

    set html(value) {
        this.element.innerHTML = value;
    }
}

class MyClass {
    *createIterator() {
        yield 1;
        yield 2;
    }
}

// 若用类表示值的集合，定义默认迭代器更好
class Collection {
    constructor() {
        this.items = [];
    }

    *[Symbol.iterator]() {
        yield *this.items.values();
    }
}
```

> 类方法和访问器属性还支持用可计算属性作为名称。

## 静态成员

```js
// ES 5
function PersonType(name) {
    this.name = name;
}
// 静态方法
PersonType.create = function(name) {
    return new PersonType(name);
};
// 实例方法
PersonType.prototype.sayName = function() {
    console.log(this.name);
};

// ES 6
class PersonClass {
    constructor(name) {
        this.name = name;
    }
    sayName() {
        console.log(this.name);
    }
    // 静态方法
    static create(name) {
        return new PersonClass(name);
    }
}
```

> 必须要直接在类中访问静态成员，不能再实例中访问静态成员。

## 继承与派生

ES 5 实现继承与派生：

```js
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
}

function Square(length) {
    Rectangle.call(this, length, length);
}

Square.prototype = Object.create(Rectangle.prototype, {
    constructor: {
        value: Square,
        enumerable: true,
        writable: true,
        configurable: true
    }
});

var square = new Square(3);

console.log(square.getArea()); // 9
console.log(square instanceof Square); // true
console.log(square instanceof Rectangle); // true
```

ES 6 实现继承与派生的等价代码：

```js
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }
}

class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
}

var square = new Square(3);

console.log(square.getArea()); // 9
console.log(square instanceof Square); // true
console.log(square instanceof Rectangle); // true
```

若不用构造函数，则会自动调用`super()`并传入所有参数。若在派生类中指定了构造函数则必须要调用`super()`，不然会报错。

```js
class Square extends Rectangle {
    // 没有构造函数
}

// 等价于

class Square extends Rectangle {
    constructor(...args) {
        super(...args);
    }
}
```

- 只可在派生类的构造函数中使用`super()`，若尝试在不是用`extends`声明的类或函数中使用则会报错。
- 在构造函数中访问`this`之前一定要调用`super()`，它负责初始化`this`，若在调用`super()`之前尝试访问`this`会导致报错。
- 若不想调用`super()`，则唯一方法是让类构造函数返回一个对象。
- 基类方法可以被重写，可以用`super.getArea()`调用基类方法。
- 静态方法在派生类也可用。

## 派生自表达式的类

只要表达式可被解析为一个函数且具有`[[Construct]]`属性和原型，就可用`extends`进行派生：

```js
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

// 形式 1
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
}

// 形式 2
function getBase() {
    return Rectangle;
}
class Square extends getBase() {
    constructor(length) {
        super(length, length);
    }
}

// 形式 3
function mixin(...mixins) {
    var base = function() {};
    Object.assign(base.prototype, ...mixins);
    return base;
}
class Square extends mixin(first, second) {
    constructor(length) {
        super();
        this.length = length;
        this.width = width;
    }
}

var x = new Square(3);
console.log(x.getArea()); // 9
console.log(x instanceof Rectangle); // true
```

> 在 `extends` 后可用任意表达式，但若 `null` 或生成器函数会报错，因为没有 `[[Construct]]`。

ES 6 支持继承内建对象：

```js
class MyArray extends Array {
    // ...
}
```

## `Symbol.species`

以下内建类型均定义`Symbol.species`属性：

- Array
- ArrayBuffer
- Map
- Promise
- RegExp
- Set
- Typed arrays（类型化数组）

该属性返回值为`this`，该属性不能定义`setter`方法，但可用`getter`修改返回值：

```js
class MyClass {
    static get [Symbol.species]() {
        return this;
    }
    constructor(value) {
        this.value = value;
    }
    clone() {
        return new this.constructor[Symbol.species](this.value);
    }
}

class MyDerivedClass1 extends MyClass {
    // ...
}

class MyDerivedClass2 extends MyClass {
    static get [Symbol.species]() {
        return MyClass;
    }
}

let instance1 = new MyDerivedClass1('foo');
let instance2 = new MyDerivedClass2('bar');

let clone1 = instance1.clone(),
    clone2 = instance2.clone();

console.log(clone1 instanceof MyClass); // true
console.log(clone1 instanceof MyDerivedClass1); // true
console.log(clone2 instanceof MyClass); // true
console.log(clone2 instanceof MyDerivedClass2); // false
```

通过调用`this.constructor[Symbol.species]`来实现派生类覆盖该值。这样可决定派生类的方法返回实例时，应该返回的值。

若想在类方法中调用`this.constructor`，就该用`Symbol.species`属性，从而让派生类重写返回类型。

```js
class MyArray extends Array {
    static get [Symbol.species]() {
        return Array;
    }
}

let items = new MyArray(1, 2, 3, 4),
    subitems = items.slice(1, 3);

console.log(items instanceof MyArray); // true
console.log(subitems instanceof Array); // true
console.log(subitems instanceof MyArray); // false
```

## `new.target`

每个构造函数可根据自身被调用的情况改变自己的行为。例如创建抽象基类（不能被直接实例化的类）：

```js
// 抽象类
class Shape {
    constructor() {
        if (new.target === Shape) {
            throw new Error('这个类不能被直接实例化');
        }
    }
}

class Rectangle extends Shape {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
}

var x = new Shape(); // 报错
var y = new Rectangle(3, 4); // 成功
console.log(y instanceof Shape); // true
```

> 因为类必须用 `new` 才能调用，故 `new.target` 不可能是 `undefined`。

---

__参考资料__

- 《深入理解ES6》
