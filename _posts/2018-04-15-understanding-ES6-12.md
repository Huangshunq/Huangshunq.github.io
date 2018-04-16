---
layout: post
title:  "第12章 代理（Proxy）和反射（Reflection）API"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 了解下代理和反射
mathjax: true
---

* content
{:toc}

<!-- # 第12章 代理（Proxy）和反射（Reflection）API -->

代理（Proxy）是一种可以拦截并改变底层 JavaScript 引擎操作的包装器。反射（Reflection）的设计目的有：

1. 将`Object`对象的一些明显属于语言内部的方法（如`Object.defineProperty`），放到`Reflect`对象上，从该对象能拿到语言内部的方法。
1. 修改某些`Object`方法的返回结果，让其变得更合理。
1. 让`Object`操作都变成函数行为。如`name in obj`变成`Reflect.has(obj, name)`。
1. `Reflect`的方法与`Proxy`的方法一一对应。

ECMAScript 6 之前，开发者不能通过自己定义的对象模仿像 JavaScript 数组对象的行为类似的默认行为，如当给数组的特定元素赋值时，影响到该数组的`length`属性，也可以通过`length`属性修改数组元素。现在可以用代理解决问题。

## 代理对应的反射


代理陷阱 | 被覆写的特性 | 默认特性
---|---|---
get | 读取一个属性值 | Reflect.get
set | 写入一个属性 | Reflect.set
has | in 操作符 | Reflect.has
deleteProperty | delete 操作符 | Reflect.deleteProperty
getPrototypeOf | Object.getPrototypeOf | Reflect.getPrototypeOf
setPrototypeOf | Object.setPrototypeOf | Reflect.setPrototypeOf
isExtensible | Object.isExtensible | Reflect.isExtensible
preventExtensions | Object.preventExtensions | Reflect.preventExtensions
getOwnPropertyDescriptor | Object.getOwnPropertyDescriptor | Reflect.getOwnPropertyDescriptor
defineProperty | Object.defineProperty | Reflect.defineProperty
ownKeys | Object.keys | Reflect.ownKeys
apply | Object.getOwnPropertyNames 或 Object.getOwnPropertySymbols | Reflect.apply
construct | new 调用一个函数 | Reflect.construct

每个代理陷阱覆写对象一些内部特性，可以使用相应的反射方法来使用原本对象的默认特性行为。

## 创建代理

```js
let target = {};
// 代理将所有操作直接转发到目标
let proxy = new Proxy(target, {});

proxy.name = 'proxy';
console.log(proxy.name); // 'proxy'
console.log(target.name); // 'proxy'
// proxy.name 和 target.name 引用的都是 target.name
target.name = 'target';
console.log(proxy.name); // 'target'
console.log(target.name); // 'target'
```

## 用`set`陷阱验证属性

```js
let target = {
    name: 'target'
};

// 验证 target 属性值是否为数字，不是则报错
let proxy = new Proxy(target, {
    // trapTarget 代理的目标
    // key 要写入的属性键
    // value 被写入的属性值
    // receiver 操作发生的对象（通常是代理）
    set(trapTarget, key, value, receiver) {
        // 忽略不希望受到影响的已有属性
        if (!trapTarget.hasOwnProperty(key)) {
            if (isNaN(value)) {
                throw new TypeError('属性必须是数字！');
            }
        }
        // 添加属性，参数同上注释
        return Reflect.set(trapTarget, key, value, receiver);
    }
});
// 添加属性
proxy.count = 1;
console.log(proxy.count); // 1
console.log(target.count); // 1

// 已有属性赋值
proxy.name = 'proxy';
console.log(proxy.name); // 'proxy'
console.log(target.name); // 'proxy'

// 报错
proxy.anotherName = 'proxy';
```

## 用`get`陷阱验证对象结构（Object Shape）

```js
// 验证对象某属性是否存在，不存在则报错
let proxy = new Proxy({}, {
    // 同上
    get(trapTarget, key, value, receiver) {
        if (!(key in receiver)) {
            throw new TypeError('属性' + key + '不存在');
        }
        return Reflect.get(trapTarget, key, receiver);
    }
});

// 添加属性，正常
proxy.name = 'proxy';
console.log(proxy.name); // 'proxy'

// 若属性不存在则报错。
console.log(proxy.name); // 报错
```

## 用`has`陷阱隐藏已有属性

```js
let target = {
    name: 'target',
    value: 42
};

// 检测 value 属性时返回 false
let proxy = new Proxy(target, {
    // trapTarget 代理目标
    // key 要检查的键（字符串或 Symbol）
    has(trapTarget, key) {
        if (key === 'value') {
            return false;
        } else {
            return Reflect.has(trapTarget, key);
        }
    }
});

console.log('value' in proxy); // false
console.log('name' in proxy); // true
console.log('toString' in proxy); // true
```

## 用`deleteProperty`陷阱防止删除属性

```js
let target = {
    name: 'target',
    value: 42
};

Object.defineProperty(target, 'name', { configurable: false });

console.log('value' in target); // true

let result1 = delete target.value;
console.log(result1); // true

console.log('value' in target); // false

// 严格模式下会报错
let result2 = delete target.name;
console.log(result2); // false

console.log('name' in target); // true

// 现在希望保护属性不被删除的同时在严格模式下不报错
let proxy = new Proxy(target, {
    // 同上
    deleteProperty(trapTarget, key) {
        // 保护 value 不被删除
        if (key === 'value') {
            return false;
        } else {
            return Reflect.deleteProperty(trapTarget, key);
        }
    }
});

// 删除 value
console.log('value' in proxy); // true

let result1 = delete proxy.value;
console.log(result1); // false

console.log('value' in proxy); // true

// 删除 name
console.log('name' in proxy); // true

let result2 = delete proxy.name;
console.log(result2); // true

console.log('name' in proxy); // false
```

## 原型代理陷阱

```js
let target = {};

// 以下操作隐藏了代理的原型
let proxy = new Proxy(target, {
    // 必须返回对象或 null（无原型时），否则该函数报错
    getPrototypeOf(trapTarget) {
        return null;
    },
    // 返回 false 表示操作失败，此时该函数会报错
    // 任何不是 false 的值会被假设操作成功
    setPrototypeof(trapTarget, proto) {
        return false;
    }
});

let targetProto = Object.getPrototypeOf(target);
let proxyProto = Object.getPrototypeOf(proxy);

console.log(targetProto === Object.prototype); // true
console.log(proxyProto === Object.prototype); // false
console.log(proxyProto); // null

// 成功
Object.setPrototypeOf(target, {});

// 报错
Object.setPrototypeOf(proxy, {});

// 使用默认行为
let proxy = new Proxy(target, {
    getPrototypeOf(trapTarget) {
        return Reflect.getPrototypeOf(trapTarget);
    },
    setPrototypeOf(trapTarget, proto) {
        return Reflect.setPrototypeOf(trapTarget, proto);
    }
});
```

> `Object.getPrototypeOf()`、`Object.setPrototypeOf()` 是高级操作，在进行底层操作之前做一些额外操作； `Reflect.getPrototypeOf()`、`Reflect.setPrototypeOf()` 是底层操作，分别是 `[[GetPrototypeOf]]` 或  `[[SetPrototypeOf]]` 内部操作的包裹器。

以下显示了它们的差异：

```js
// 执行前会将参数强制转换为一个对象
let result1 = Object.getPrototypeOf(1);
console.log(result1 === Number.prototype); // true

// 不会转换，报错
Reflect.getPrototype(1);

let target2 = {};
// 成功则返回第一个参数，失败会报错
let result2 = Object.setPrototypeOf(target2, {});
console.log(result2 === target2); // true

// 成功返回 true ， 失败返回 false
let result3 = Reflect.setPrototypeOf(target2, {});
console.log(result3 === target2); // false
console.log(result3); // true
```

## 对象可拓展性陷阱

```js
let target = {};

// 防止 proxy 变为不可拓展
let proxy = new Proxy(target, {
    isExtensible(trapTarget) {
        return Reflect.isExtensible(trapTarget);
    },
    preventExtensions(trapTarget) {
        return false;
    }
});

console.log(Object.isExtensible(target)); // true
console.log(Object.isExtensible(proxy)); // true

Object.preventExtensions(proxy); // 无效

console.log(Object.isExtensible(target)); // true
console.log(Object.isExtensible(proxy)); // true
```

`Object.isExtensible()`、`Object.preventExtensions()` 与 `Reflect.isExtensible()`、`Reflect.preventExtensions()` 有所差异：

```js
// 传入非对象值时返回 false
let result1 = Object.isExtensible(2);
console.log(result1); // false

// 传入非对象值时，报错
let result2 = Reflect.isExtensible(2);

// 无论参数是否为一个对象，都返回该对象
let result3 = Object.preventExtensions(2);
console.log(result3); // 2

// 参数为对象时，成功返回 true，失败返回 false
let target = {};
let result4 = Reflect.preventExtensions(target);
console.log(result4); // true

// 参数不为对象时，报错
let result5 = Reflect.preventExtensions(2);
```

## 属性描述符陷阱

### 限制`Object.defineProperty()`

```js
// 阻止定义 Symbol 属性
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {
        if (typeof key === 'symbol') {
            return false;
        }
        return Reflect.defineProperty(trapTarget, key, descriptor);
    }
});

Object.defineProperty(proxy, 'name', {
    value: 'proxy'
});
console.log(proxy.name); // 'proxy'

let nameSymbol = Symbol('name');

// 报错
Object.defineProperty(proxy, nameSymbol, {
    value: 'proxy'
});
```

### 描述符（descriptor）限制

```js
// descriptor 不是实际传入 Object.defineProperty() 的第三个引用
let proxy = new Proxy({}, {
    defineProperty(trapTarget, key, descriptor) {
        console.log(descriptor.value); // 'proxy'
        console.log(descriptor.name); // undefined

        return Reflect.defineProperty(trapTarget, key, descriptor);
    }
});

Object.defineProperty(proxy, 'name', {
    value: 'proxy',
    name: 'custom'
});



let proxy = new Proxy({}, {
    // 返回值必须是 null, undefined, 一个对象
    // 若返回对象，属性只能是 enumerable、configurable、value、writable、get、set，
    // 有其他属性会报错
    getOwnPropertyDescriptor(trapTarget, key) {
        return {
            name: 'proxy'
        };
    }
});

// 报错
let descriptor = Object.getOwnPropertyDescriptor(proxy, 'name');
```

### 差异

```js
let target = {};

// 返回 target
let result1 = Object.defineProperty(target, 'name', { value: 'target' });

console.log(target === result); // true

// 成功返回 true，失败返回 false
let result2 = Reflect.defineProperty(target, 'name', { value: 'reflect' });

console.log(result2); // true


// 传入原始值为第一个参数，会被强制转换为一个对象
let descriptor1 = Object.getOwnPropertyDescriptor(2, 'name');
console.log(descriptor1); // undefined

// 传入原始值会报错
let descriptor2 = Reflect.getOwnPropertyDescriptor(2, 'name');
```

## `ownKeys`陷阱

`ownKeys` 拦截内部方法 `[[OwnPropertyKeys]]`，通过返回一个数组以覆写其行为。

```js
// 过滤不想引入的属性键
let proxy = new Proxy({}, {
    // 返回值必须是一个数组或类数组对象，否则报错
    ownKeys(trapTarget) {
        return Reflect.ownKeys(trapTarget).filter(key => {
            return typeof key !== 'string' || key[0] !== '_';
        })
    }
});

let nameSymbol = Symbol('name');

proxy.name = 'proxy';
proxy._name = 'private';
proxy[nameSymbol] = 'symbol';

let names = Object.getOwnPropertyNames(proxy),
    keys = Object.keys(proxy),
    symbols = Object.getOwnPropertySymbols(proxy);

console.log(names.length); // 1
console.log(names[0]); // name

console.log(keys.length); // 1
console.log(keys[0]); // name

console.log(symbols.length); // 1
console.log(symbols[0]); // 'Symbol(name)'
```

> `ownKeys` 也影响 `for-in` 循环（用于确定内部使用的键）。

## 函数代理

代理陷阱中，只有 apply 和 construct 的代理目标是一个函数，分别对应 `[[Construct]]`（使用 `new` 时调用） 和 `[[Call]]`（不使用 `new` 时调用），相应的陷阱覆写对应的内部方法。

```js
// proxy 模拟函数默认的行为
let target = function() { return 42; },
    proxy = new Proxy(target, {
        apply: function(trapTarget, thisArg, argumentList) {
            return Reflect.apply(trapTarget, thisArg, argumentList);
        },
        construct: function(trapTarget, argumentList) {
            return Reflect.construct(trapTarget, argumentList);
        }
    });

console.log(typeof proxy); // 'function'

console.log(proxy()); // 42

var instance = new Proxy();
console.log(instance instanceof proxy); // true
console.log(instance instanceof target); // true
```

### 验证函数的参数属于特定类型

```js
// 将所有参数相加
function sum(...values) {
    return values.reduce((previous, current) => previous + current, 0);
}

// 验证参数是否为数字，并使之不能用 new 调用
let sumProxy = new Proxy(sum, {
    apply: function(trapTarget, thisArg, argumentList) {
        argumentList.forEach((arg) => {
            if (typeof arg !== 'number') {
                throw new TypeError('所有参数必须是数字');
            }
        });
        
        return Reflect.apply(tarpTarget, thisArg, argumentList);
    },
    construct: function(trapTarget, argumentList) {
        throw new TypeError('该函数不可用 new 来调用');
    }
});

console.log(sumProxy(1, 2, 3, 4)); // 10

// 报错
console.log(sumProxy(1, '2', 3, 4));

// 报错
let result = new sumProxy();
```

### 不用 `new` 调用构造函数

使用代理，可以在不能改动原函数的情况下，完成一些拓展任务。

比如：不使用 `new` 调用构造函数

```js
function Numbers(...values) {
    if (typeof new.target === 'undefined') {
        throw new Error('该函数必须通过 new 来调用');
    }
    
    this.values = values;
}

// 报错
Numbers(1, 2, 3, 4);

let NumbersProxy = new Proxy(Numbers, {
    apply: function(trapTarget, thisArg, argumentList) {
        // 调用 默认构造函数行为
        return Reflect.construct(trapTarget, argumentList);
    }
});

let instance = NumbersProxy(1, 2, 3, 4);
console.log(instance.values); // [1, 2, 3, 4]
```

### 覆写抽象基类构造函数

可以通过修改`Reflect.construct()`的第三个参数来指定 `new.target` 的值，绕过本来设定不继承就报错的抽象基类：

```js
class AbstractNumbers {
    constructor(...values) {
        if (new.target === AbstractNumbers) {
            throw new TypeError('此函数必须被继承');
        }
        this.values = values;
    }
}

// 报错
new AbstractNumbers(1, 2, 3, 4);

let AbstractNumbersProxy = new Proxy(AbstractNumbers, {
    construct: function(trapTarget, argumentList) {
        return Reflect.construct(trapTarget, argumentList, function() {});
    }
});

let instance = new AbstractNumbersProxy(1, 2, 3, 4);
console.log(instance.values); // [1, 2, 3, 4]
```

### 可直接调用的类构造函数

如果希望类构造函数不用 `new` 就可以执行，可用代理实现：

```js
class Person {
    constructor(name) {
        this.name = name;
    }
}

let PersonProxy = new Proxy(Person, {
    apply: function(trapTarget, thisArg, argumentList) {
        return new trapTarget(...argumentList);
    }
});

let me = PersonProxy('me');
console.log(me.name); // 'me'
console.log(me instanceof Person); // true
console.log(me instanceof PersonProxy); // true
```

## 可撤销代理

使用 `Proxy.revocable()` 创建可被撤销的代理，用 `Proxy` 相同的参数：目标对象和代理处理程序。返回值有：`proxy`，可被撤销的代理；`revoke`，撤销代理要调用的函数。

```js
let target = {
    name: 'target'
};

let { proxy, revoke } = Proxy.revocable(target, {});

console.log(proxy.name); // 'target'

// 撤销代理
revoke();

// 报错
console.log(proxy.name);
```

## 解决数组问题

要完全重建数组行为，需要模拟以下行为：

### 检测数组索引

要判断一个属性是否是一个索引，可参考 ECMAScript 6 规范的说明：

> 当且仅当 ToString(ToUint32(P)) 等于 P，且 ToUint32(P) 不等于 2^32 - 1 时，字符串属性名称 P 才是一个数组索引。

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}
```

### 添加新元素时增加 `length` 的值

实现添加新元素时自动增加 `length` 的值的行为：

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

function createMyArray(length = 0) {
    return new Proxy({ length }, {
        set(trapTarget, key, value) {
            let currentLength = Reflect.get(trapTarget, 'length');
            
            // 特殊情况
            if (isArrayIndex(key)) {
                let numericKey = Number(key);
                
                if (numericKey >= currentLength) {
                    Reflect.set(trapTarget, 'length', numericKey + 1);
                }
            }
            
            // 无论 key 是什么类型总是执行该语句
            return Reflect.set(trapTarget, key, value);
        }
    });
}

let colors = createMyArray(3);
console.log(colors.length); // 3

colors[0] = 'red';
colors[1] = 'green';
colors[2] = 'blue';

console.log(colors.length); // 3

colors[3] = 'black';

console.log(colors.length); // 4

console.log(colors[3]); // 'black'
```

### 减少 `length` 的值来删除元素

将上例扩展，再实现减少 `length` 的值时自动删除元素的行为：

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

function createMyArray(length = 0) {
    return new Proxy({ length }, {
        set(trapTarget, key, value) {
            let currentLength = Reflect.get(trapTarget, 'length');
            
            // 特殊情况
            if (isArrayIndex(key)) {
                let numericKey = Number(key);
                
                if (numericKey >= currentLength) {
                    Reflect.set(trapTarget, 'length', numericKey + 1);
                }
            } else if (key === 'length') {
                if (value < currentLength) {
                    for (let index = currentLength - 1; index >= value; index--) {
                        Reflect.deleteProperty(trapTarget, index);
                    }
                }
            }
            
            // 无论 key 是什么类型总是执行该语句
            return Reflect.set(trapTarget, key, value);
        }
    });
}

let colors = createMyArray(3);
console.log(colors.length); // 3

colors[0] = 'red';
colors[1] = 'green';
colors[2] = 'blue';
colors[3] = 'black';

console.log(colors.length); // 4

colors.length = 2;

console.log(colors.length); // 2
console.log(colors[3]); // undefined
console.log(colors[2]); // undefined
console.log(colors[1]); // 'green'
console.log(colors[0]); // 'red'
```

### 综合实现解决数组问题

使用类，在构造函数中返回一个代理：

```js
class Thing {
    constructor() {
        return new Proxy(this, {});
    }
}

let myThing = new Thing();
console.log(myThing instanceof Thing); // true
```

```js
function toUint32(value) {
    return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32);
}

function isArrayIndex(key) {
    let numericKey = toUint32(key);
    return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1);
}

class MyArray {
    constructor(length = 0) {
        this.length = length;
        
        return new Proxy(this, {
            set(trapTarget, key, value) {
                let currentLength = Reflect.get(trapTarget, 'length');
                
                if (isArrayIndex(key)) {
                    let numericKey = Number(key);
                    
                    if (numericKey >= currentLength) {
                        Reflect.set(trapTarget, 'length', numericKey + 1);
                    }
                } else if (key === 'length') {
                    if (value < currentLength) {
                        for (let index = currentLength - 1; index >= value; index--) {
                            Reflect.deleteProperty(trapTarget, index);
                        }
                    }
                }
                
                // 无论 key 是什么类型总是执行该语句
                return Reflect.set(trapTarget, key, value);
            }
        });
    }
}

let colors = new MyArray(3);
console.log(colors instanceof MyArray); // true

console.log(colors.length); // 3

colors[0] = 'red';
colors[1] = 'green';
colors[2] = 'blue';
colors[3] = 'black';

console.log(colors.length); // 4

colors.length = 2;

console.log(colors.length); // 2
console.log(colors[3]); // undefined
console.log(colors[2]); // undefined
console.log(colors[1]); // 'green'
console.log(colors[0]); // 'red'
```

## 将代理用作原型

### 原型上使用 `get` 陷阱

原型上使用 `get` 陷阱时，读取属性（[[Get]]）时先查找自有属性，若未找到指定名称的自有属性，则继续到原型中查找，此时会调用原型上的陷阱。

```js
let target = {},
    thing = Object.create(new Proxy(target, {
        get(trapTarget, key, receiver) {
            throw new ReferenceError(`${key} doesn't exist.`);
        }
    }));

thing.name = 'thing';

consolg.log(thing.name); // 'thing'

// 报错
let unknown = thing.unknown;
```

### 原型上使用 `set` 陷阱

原型上使用 `set` 陷阱，在设置属性时（[[Set]]），先检查目标对象中是否含有某个目标属性，若不存在则继续查找原型。赋值时默认一定是在实例中创建属性。

```js
let target = {},
    thing = Object.create(new Proxy(target, {
        set(trapTarget, key, value, receiver) {
            return Reflect.set(trapTarget, key, value, receiver);
        }
    }));

console.log(thing.hasOwnProperty('name')); // false

// 触发 set 代理陷阱
thing.name = 'thing';

console.log(thing.name);
console.log(thing.hasOwnProperty('name')); // true

// 不触发
thing.name = 'boo';

console.log(thing.name); // 'boo'
```

### 原型上使用 `has` 陷阱

原型上使用 `has` 陷阱，只有当指定名称没有对应的自有属性时才会调用 `has` 陷阱。

```js
let target = {},
    thing = Object.create(new Proxy(target, {
        has(trapTarget, key) {
            return Reflect.has(trapTarget, key);
        }
    }));

// 触发 has 代理陷阱
console.log('name' in thing); // false

thing.name = 'thing';

// 不触发
console.log('name' in thing); // false
```

### 代理用作类的原型

ES 6 的类的`prototype`是不可写的，所以不能直接修改类的原型为代理。

```js
function NoSuchProperty() {
    // empty
}

let proxy = new Proxy({}, {
    get(trapTarget, key, receiver) {
        throw new ReferenceError(`${key} doesn't exist.`);
    }
});

// 作为原型
NoSuchProperty.prototype = proxy;

class Square extends NoSuchProperty {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
    
    getArea() {
        return this.length * this.width;
    }
}

let shape = new Square(2, 6);

// shape 的原型 Square.prototype 不是一个代理，
// 但 Shape.prototype 的原型是继承自 NoSuchProperty 的代理
// 因为类的方法如 getArea，都会放到类的原型上，
// 如果直接继承代理 proxy，则当调用 shape.getArea 时，
// 会先在实例搜索方法，然后在原型上找到该方法。
// 若原型是代理 proxy，则会直接报错，就找不到 getArea了。
let shapeProto = Object.getPrototypeOf(shape);
console.log(shapeProto === proxy); // false
let secondLevelProto = Object.getPrototypeOf(shapeProto);
console.log(secondLevelProto === proxy); // true

let area1 = shape.length * shape.width;
console.log(area1); // 12

// 'wdth' 不存在，报错
let area2 = shape.length * shape.wdth;
```

---

__参考资料__

- 《深入理解ES6》
- [《ECMAScript 6 入门》](http://es6.ruanyifeng.com/#docs/symbol) 