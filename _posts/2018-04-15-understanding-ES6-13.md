---
layout: post
title:  "第13章 用模块封装代码"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 模块
mathjax: true
---

* content
{:toc}

<!-- # 第13章 用模块封装代码 -->

模块，指自动运行在严格模式下且没办法退出运行的 JavaScript 代码。模块必须导出外部代码可访问的元素，模块也可以从其它模块导入绑定。

在模块顶部，`this`值是`undefined`，不支持 HTML 风格的代码注释。

脚本，即不是模块的 JavaScript 代码，没有上述特性。

## `export`

用`export`导出想导出的模块内容：

```js
// exp.js

// 导出数据
export var color = 'red';
export let name = 'me';
export const num = '42';

// 导出函数
export function sum (num1, num2) {
    return num1 + num2;
}

// 导出类
export class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }
}

function sub (num1, num2) {
    return num1 - num2;
}

// 导出引用
export sub;
```

## `import`

用`import`导入想导入的模块内容

```js
import { sum, num } from './exp.js';

console.log(sum(num, num));

num = 1; // 报错，不能重新赋值。
```

> 导入的变量与用`const`定义相似，无法定义（或导入）另一个同名变量，无法在`import`语句前使用标识符或改变其值。

```js
// 导入所有变量，又叫命名空间导入（namespace import）
import * as exp from './exp.js';

console.log(exp.num); // 42

// 重复引入的模块只会执行一次，执行后保存在内存中供重复使用。
import { color } from './exp.js';
import { name } from './exp.js';
import { sum } from './exp.js';

// export 和 import 必须静态执行，不能动态使用
if (flag) {
    export flag; // 报错
}

function tryImport() {
    import flag from './exp.js'; // 报错
}
```

## 修改被导出模块内的值

```js
// ./exp.js
export var name = 'name';
export function setName(newName) {
    name = newName;
}
```

如上，可以使用`setName`修改`name`的值：

```js
import { name, setName } from './exp.js';

console.log(name); // 'me'
setName('you');
console.log(name); // 'you'

name = 'they' // 报错
```

## 重命名与默认值

可以用`as`重命名：

```js
// 使用 add 名称导入 sum
import { sum as add } from './exp.js';
console.log(typeof sum); // 'undefined'
// 使用 sum 名称导出
export { add as sum };
```

可以用`default`指定默认值：

```js
export default function(num1, num2) {
    return num1 + num2;
}
// 导出指定标识符
export default someName;
// 使用 as
export { sum as default };

// 导入默认值
import sum from './exp.js';
// 同时导入默认值和其它值
// exp.js：
// export let color = 'red';
// export default function(num1, num2) {
//     return num1 + num2;
// }
import sum, { color } from './exp.js';
import { default as sum, color } from './exp.js';
```

## 导出已导入的内容

```js
import { sum } from './exp.js';
export { sum } // 将导入的绑定再次导出

// 效果同上
export { sum } from 'exp.js';
export { sum as add } from 'exp.js';
export * from 'exp.js';
```

## 无绑定导入

有些模块没有导出任何内容，导入这些模块只是为了执行那些模块内的代码：

```js
// 可简化导入操作，可应用于 Polyfill 和 Shim。
import './exp.js';
```

## 加载模块

ECMAScript 6 只规定了模块的语法，并没有定义如何加载模块。浏览器中使用模块的机制如下：

### 在 `<script>` 中使用模块

指定使用模块的方式加载：

```html
<!-- 加载模块文件 -->
<script type="module" src="module.js"></script>

<!-- 内联加载模块 -->
<script type="module">
// 不会暴露变量到全局

import { sum } from './exp.js';

let result = sum(1, 2);

</script>
```

> 当浏览器无法识别`type`值时，浏览器将自动忽略该`<script>`元素，不支持`module`的浏览器也不会有事，向后兼容性良好。

### 模块加载顺序

加载模块时，`defer`默认必选（该属性会被忽略），所以模块会等页面加载完成才会执行。可以使用`async`属性。

```html
<!-- 先执行该文件 -->
<script type="module" src="module1.js"></script>

<!-- 再执行该文件 -->
<script type="module">
// 不会暴露变量到全局

import { sum } from './exp.js';

let result = sum(1, 2);

</script>

<!-- 最后执行 -->
<script type="module" src="module2.js"></script>
```

> 模块会等前一个文件加载完成（包括递归下载文件中导入的其它模块资源）再加载下一个文件。加载完成后，当文档完全解析后才会执行，也会等前一个文件执行完（先递归执行文件中导入的其它模块资源，再执行该文件），再执行下一个文件。

### 作为 Worker 加载

Worker内部也应用`import`导入模块。

```js
// 按脚本方式加载
let worker = new Worker('script.js');
// 按模块方式加载
let worker = new Worker('module.js', { type: 'module' });
```

### module specifier （模块说明符）

浏览器中可正常使用如`/`，`./`，`../`，URL形式的模块说明符，除了以下情况：

```js
// 都无效，即使作为 <script> src 属性的值有效（有意为之）
import { first } from 'exp.js';
import { second } from 'exp.js';
```

---

__参考资料__

- 《深入理解ES6》
