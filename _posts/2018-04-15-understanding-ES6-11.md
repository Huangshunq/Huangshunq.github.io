---
layout: post
title:  "第11章 Promise与异步编程"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: Promise与异步编程
mathjax: true
---

* content
{:toc}

<!-- # 第11章 Promise与异步编程 -->

## 基于事件模型的异步编程

JavaScript引擎同一时刻只能执行一个代码块，当一段代码准备执行时，就会被添加到任务队列。每当引擎中的一段代码结束执行，事件循环（event loop，负责监控代码执行并管理任务队列的程序）会执行队列中下一个任务。

用户通过触发事件的形式向任务队列添加一个新任务的形式是事件模型。

## `Promise`

每个`Promise`会有一个生命周期：先是处于进行中（pending），结束后变为Fulfilled（成功状态）或Rejected（失败状态）。内部属性`[[PromiseState]]`的值表示这三种状态。

若对象实现了`then()`，则该对象称为 thenable 对象。所有`Promise`都是thenable对象，但不是所有thenable对象都是`Promise`。

每次调用`then()`或`catch()`都会创建一个新任务，当Promise完成时执行。这些任务都会被加入到一个为此量身定制的独立队列中。

## 创建 `Promise`

```js
let fs = require('fs');
// 创建待完成的 Promise
let promise = new Promise(function(resolve, reject) {
    // 异步操作
    fs.readFile(filename, { encoding: 'utf8' }, function(err, contents) {
        if (err) {
            // 执行失败时调用 reject，可传入错误对象
            reject(err);
            return;
        }
    })
    // 成功执行后需要调用 resolve，可传入一个成功信息的参数
    resolve(contents);
});

promise.then(function(contents) {
    // 完成
    console.log(contents);
}, function(err) {
    // 失败
    console.error(err.message);
});

// 创建已完成的 Promise
let promise = Promise.resolve(42);

promise.then(function(val) {
    console.log(val); // 42
});

let promise2 = Promise.reject(42);

promise.catch(function(val) {
    consolr.log(val); // 42
});


```

> 若向 `Promise.resolve()` 或 `Promise.reject()` 传入一个 `Promise`，该 `Promise` 会被直接返回。

新建的`Promise`会立即执行，然后才执行后续流程中的代码：

```js
let promise = new Promise(function(resolve, reject) {
    console.log('Promise');
    resolve();
});

// Promise完成后才被添加到任务队列末尾
promise.then(function() {
    console.log('Resolve');
});

console.log('Hi!');

// 输出顺序：
// Promise
// Hi!
// Resolve
```

## `thenable`对象

拥有`then()`且其有 resolve 和 reject 两参数的普通对象就是非 `Promise` 的 `thenable` 对象：

```js
let thenable = {
    then: function(resolve, reject) {
        resolve(42);
    }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
    console.log(value); // 42
});
```

`Promise.resolve()` 或 `Promise.reject()` 都接受非 `Promise` 的 `thenable` 对象为参数，此时会调用 `thenable.then()`，并返回一个新的 `Promise`（成功或失败）。

若不确定某对象是不是`Promise`对象，可将其传入 `Promise.resolve()` 或 `Promise.reject()` 中，如果是 `Promise` ，则返回值不会有变化。

## 错误处理

`Promise`内部发生错误（包括throw Error）时，会直接进入reject状态，调用执行失败时完成的函数。

标准中，若没有编写执行失败时完成的函数（如`catch()`），则`Promise`内部发生错误时不会有任何提示信息。

Node.js中，`process`对象上有相应两个事件：

- `unhandledRejection`：在一个事件循环中，当`Promise`执行失败，且没提供处理函数时，触发该事件。
- `rejectionHandled`：在一个事件循环后，当`Promise`执行失败，且调用了处理函数时，触发该事件。

浏览器也有相应的事件。

```js
// promise参数是执行失败的 Promise 对象
process.on('unhandledRejection', function(err, promise) {
    // ...
});

process.on('rejectionHandled', function(promise) {
    // ...
});

// 浏览器中：
window.onunhandledrejection = function(event) {
    console.log(event.type); // 事件名称
    console.log(event.reason); // 传入处理函数的参数值
    console.log(event.promise); // Promise对象
};

window.onrejectionhandled = function(event) {
    console.log(event.type); // 事件名称
    console.log(event.reason); // 传入处理函数的参数值
    console.log(event.promise); // Promise对象
};
```

## 在`Promise`链中返回`Promise`

```js
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

p1.then(function(val) {
    console.log(val); // 42
    return p2; // 返回 p2
}).catch(function(val) {
    // 是处理 p2 的函数
    console.log(val); // 43
});

// 等价于
let p3 = p1.then(function(val) {
    console.log(val); // 42
    return p2; // 返回 p2
});
p3.catch(function(val) {
    // 根据 p2 的结果执行函数
    console.log(val); // 43
});
```

## 响应多个`Promise`

- `Promise.all()`：传入一个可迭代对象，其中所有`Promise`都完成才成功完成，执行处理函数。
- `Promise.race()`：传入一个可迭代对象，其中任意一个`Promise`完成就算成功完成，执行处理函数。

## 继承`Promise`

```js
class MyPromise extends Promise {
    // 使用默认构造函数

    success(resolve, reject) {
        // ...
    }

    failure(resolve, reject) {
        // ...
    }
}
```

## 基于`Promise`的异步任务执行

```js
let fs = require('fs');

function run(taskDef) {
    // 创建迭代器
    let task = taskDef();

    // 开始执行任务
    let result = task.next();

    (function step() {
        // 如果有更多任务要做
        if (!result.done) {
            // 用一个Promise来解决会简化问题
            let promise = Promise.resolve(result.value);
            promise.then(function(value) {
                result = task.next(value);
                step();
            }).catch(function(error) {
                result = task.throw(error);
                step();
            });
        }
    }());
}

// 定义一个可用于任务执行的函数

function readFile(filename) {
    return new Promise(function(resolve, reject) {
        fs.readFile(filename, function(err, contents) {
            if (err) {
                reject(err);
            } else {
                resolve(contents);
            }
        });
    });
}

// 执行一个任务

run(function*() {
    let contents = yield readFile('config.json');
    doSomethingWith(contents);
    console.log('Done');
});
```

---

__参考资料__

- 《深入理解ES6》
