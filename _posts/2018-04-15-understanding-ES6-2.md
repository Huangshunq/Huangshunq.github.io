---
layout: post
title:  "第2章 字符串和正则表达式"
date:   2018-04-15 12:19:00
categories: 《深入理解ES6》
tags: ES6
excerpt: 字符串和正则表达式
mathjax: true
---

* content
{:toc}

# 第2章 字符串和正则表达式

## 关于Unicode的字符编码基本概念

> __码位__（code point），又叫代码点，指Unicode中为每个相应的字符分配的唯一编号（标识符），一个字符只会对应一个码位。
>
> 将每个字符用其对应的码位表示，并将该码位存储在计算机上的过程，叫做 __字符编码__（或简称编码，character encode）。
>
>__编码单元__（code unit）或叫代码单元，则是针对编码方式而言，它指进行编码的过程中对一个字符进行编码以后，对应的码位所要占用的最小存储单位，可能是一个字节，也可能是多个字节。编码/解码算法是以编码单元为最小单元来操作的。

## 更好的Unicode支持

在UTF-16中，前2^16^个码位均以16位的编码单元表示，这个范围被称作 __基本多文种平面__ （BMP，Basic Multilingual Plane）。超出这个范围的码位则要归属于某个辅助平面（supplementary plane），其中的码位仅用16位就无法表示了。为此，UTF-16引入了代理对（surrogate pair），其规定用两个16位编码单元表示一个码位。此时，字符串中的字符有两种，一种是由一个编码单元16位表示的BMP字符，另一种是由两个编码单元32位表示的 __辅助平面字符__ 。

在ES5中，JavaScript字符串一直基于16位字符编码（UTF-16）进行构建，所有字符串的操作都基于16位编码单元。如果采用这种方式处理包含代理对的UTF-16编码字符，可能得到错误的结果。

### `codePointAt()`

ES6增加了完全支持UTF-16的`codePointAt()`方法，这个方法接受编码单元的位置而非字符位置作为参数，返回与字符串中给定位置对应的码位，即一个整数值。

```js
let text = "𠮷a";

// ES5
console.log(text.charCodeAt(0)); //55362
console.log(text.charCodeAt(1)); //57271
console.log(text.charCodeAt(2)); //97

// ES6
console.log(text.codePointAt(0));//134071
console.log(text.codePointAt(1));//57271
console.log(text.codePointAt(2));//97
```

对于BMP字符集中的字符，`codePointAt()`方法返回值与`charCodeAt()`方法相同。而对于非BMP字符集来说，比如text的第一个字符，`charCodeAt()`返回位置0处的第一个编码单元，而`codePointAt()`则返回完整的码位。其它的位置，二者返回值相同。

### `String.fromCodePoint()`

ECMAScript通常会面向同一个操作提供正反两种方法。相对于`codePointAt()`，有`String.fromCodePoint()`方法根据指定的码位生成返回一个字符。

```js
console.log(String.fromCodePoint(134071));
// "𠮷"
```

这相当于`String.fromCharCode()`（不适用于非BMP码位）的拓展方法。

### `normalize()`

> 如果不需要开发多语种的应用，基本不用此方法。如果需要，在排序和比较操作前，将被操作字符串按照统一标准进行标准化，即`normalize()`。

在Unicode中，有可能使一个字符有多种字符编码方式，各种方式生成的码位是不同的，但是所表达的字符（包括语义、视觉上）是一样的。

比如，许多欧洲语言有语调符号和重音符号。为了表示它们，Unicode 提供了两种方法。一种是直接提供带重音符号的字符，比如Ǒ（\u01D1）。另一种是提供合成符号（combining character），即原字符与重音符号的合成，两个字符合成一个字符，比如O（\u004F）和ˇ（\u030C）合成Ǒ（\u004F\u030C）。

ES6 提供字符串实例的`normalize()`方法，用来将字符的不同表示方法统一为同样的形式，这称为 Unicode 正规化。

```js
'\u01D1'==='\u004F\u030C' //false

'\u01D1'.length // 1
'\u004F\u030C'.length // 2

'\u01D1'.normalize() === '\u004F\u030C'.normalize()
// true
```

`normalize`方法可以接受一个参数来指定normalize的方式，参数的四个可选值如下。

- `NFC`，默认参数，表示“标准等价合成”（Normalization Form Canonical Composition），返回多个简单字符的合成字符。所谓“标准等价”指的是视觉和语义上的等价。
- `NFD`，表示“标准等价分解”（Normalization Form Canonical Decomposition），即在标准等价的前提下，返回合成字符分解的多个简单字符。
- `NFKC`，表示“兼容等价合成”（Normalization Form Compatibility Composition），返回合成字符。所谓“兼容等价”指的是语义上存在等价，但视觉上不等价，比如“囍”和“喜喜”。（这只是用来举例，normalize方法不能识别中文。）
- `NFKD`，表示“兼容等价分解”（Normalization Form Compatibility Decomposition），即在兼容等价的前提下，返回合成字符分解的多个简单字符。

### 正则表达式u修饰符

正则表达式默认将字符串中每个字符按照16位编码单元处理。而ECMAScript 6 给正则表达式定义了一个支持Unicode的u修饰符。当一个正则表达式添加了u修饰符时，它就从编码单元操作模式切换为字符模式（正确处理非BMP字符）。

```js
let text = '𠮷';

console.log(text.length); // 2
console.log(/^.$/.test(text)); // false
console.log(/^.$/u.test(text));// true
```

可以通过以下方式检测当前引擎是否支持u修饰符：

```js
function hasRegExpU() {
    try {
        var pattern = new RegExp('.', 'u');
        return true;
    } catch (e) {
        return false;
    }
}
```

## 子串识别

在一段字符串中检测另一段子字符串，ES6提供了新方法：

- `includes()`方法：如果在字符串中检测到指定文本则返回`true`，否则返回`false`。
- `startsWith()`方法：如果在字符串起始部分检测到指定文本则返回`true`，否则返回`false`。
- `endsWith()`方法：如果在字符串结束部分检测到指定文本则返回`true`，否则返回`false`。

以上方法都接受两个参数：第一个指定要搜索的文本；第二个参数可选，指定一个开始搜索的位置的索引值。

```js
let msg = 'Hello world!';

console.log(msg.startWith('Hello')); // true
console.log(msg.endsWith('!')); // true
console.log(msg.includes('o')); // true

console.log(msg.startWith('o', 4));
// true. 从索引的第四位开始搜索，即从第一个o开始向后搜索。
console.log(msg.endsWith('o', 8));
// true. 从索引的第七位开始搜索，即从最后一个o处开始，搜索前7个字符的字符串。
console.log(msg.includes('o', 8));
// false. 从字符串"world"中的"r"开始匹配，它位于索引第八位。
```

> 如果你没有按照要求传入一个字符串，而是传入一个正则表达式，会触发一个错误。而对于`indexOf()`和`lastIndexOf()`，都会把正则表达式转化为一个字符串并搜索它。

## `repeat()`

ES6 增添的`repeat()`方法，其接受一个`Number`类型的参数，表示该字符串的重复次数，返回将当前字符串重复一定次数后的新字符串。

```js
console.log('x'.repeat(3));//'xxx'
console.log('hello'.repeat(2));//'hellohello'
console.log('abc'.repeat(4));//'abcabcabcabc'
```

## 正则表达式y修饰符

使用y运算符的正则表达式，只会在`lastIndex`属性值的位置开始搜索，如果在这个指定位置没有匹配到相应字符，就停止搜索，返回`null`。

```js
let text = 'hello1 hello2 hello3',
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);
    
console.log(result[0]);      // 'hello1'
console.log(globalResult[0]);// 'hello1'
console.log(stickyResult[0]);// 'hello1'

console.log(pattern.lastIndex);// 0
console.log(globalPattern.lastIndex);// 7
console.log(stickyPattern.lastIndex);// 7

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);      // 'hello1'
console.log(globalResult[0]);// 'hello2'
console.log(stickyResult[0]);// 'hello2'

console.log(pattern.lastIndex);// 0
console.log(globalPattern.lastIndex);// 14
console.log(stickyPattern.lastIndex);// 14

// 从1开始搜索
pattern.lastIndex = 1;
globalPattern.lastIndex = 1;
stickyPattern.lastIndex = 1;

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);      // 'hello1'
console.log(globalResult[0]);// 'hello2'

// 由于从第二个字符开始匹配不到相应字符串，
// 所以终止搜索，返回`null`。
console.log(stickyResult[0]);//  null
```

关于y运算符，还有两点要注意：

- 只有调用`exec()`和`test()`这些正则表达式对象的方法才会涉及`lastIndex`属性；调用字符串的方法，例如`match()`，则不会触发粘滞行为。
- 对于粘滞正则表达式而言，如果使用`^`字符匹配字符串开端，只会从字符串的起始位置或多行模式首行进行匹配。当`lastIndex`值为0时，如果正则表达式含有`^`，则是否使用粘滞正则表达式并无区别；如果`lastIndex`值在单行模式下不为0，或在多行模式下不对应行首，则粘滞表达式永远不会成功匹配。

可以通过`sticky`属性检测y修饰符是否存在：

```js
let pattern = /hello\d/y;

console.log(pattern.sticky); // true
```

可以通过以下方式检测当前引擎是否支持y修饰符：

```js
function hasRegExpY() {
    try {
        var pattern = new RegExp('.', 'y');
        return true;
    } catch (e) {
        return false;
    }
}
```

## 复制正则表达式

在ES6中，可以通过RegExp构造函数第二个参数修改其修饰符。

```js
let re1 = /ab/i,

// 在ES5中报错，在ES6中正常运行。
re2 = new RegExp(re1, 'g');

console.log(re1.toString()); // '/ab/i'
console.log(re2.toString()); // '/ab/g'
```

## flags属性

在ES6中，访问flags属性会返回所有应用于当前正则表达式的修饰字符串：

```js
let re = /ab/g;

console.log(re.source); // 'ab'
console.log(re.flags);  // 'g'
```

## 模板字面量

ECMAScript 6 新增的模板字面量语法支持创建领域专用语言（Domain Special Language，DSL）。DSL通常指专门针对某些具体且有限的问题而设计的语言。根据是否从宿主语言（指通过一种语言构建另一种语言，如Java是构建在C上的语言）构建而来，DSL分为：

- 内部DSL（从一种宿主语言构建而来）
- 外部DSL（从零开始构建的语言，需要实现语法分析器等）

### 多行字符串

ES6以前，创造多行字符串，需要手动加入换行符，并使用`\`承接上一行代码。

现在，可以使用正规的模板字面量语法，空白符、换行符都属于字符串的一部分：

```js
// ES5
var message = 'Multiline \n\
string';

// ES6
let message = `Multiline 
string`;
// 也可以显式声明
let message = `Multiline\nstring`;

console.log(message.length); // 16
```

### 字符串占位符

占位符由左侧`${`和右侧的`}`符号组成，中间可以包含任意的JavaScript表达式，通过占位符可以直接将该作用域的局部变量嵌入输出的字符串中：

```js
let count = 10,
    price = 0.25,
    message = `${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message); // '10 items cost $2.50.'
```

### 标签模板

标签指在模板字面量第一个反撇号前的字符串：

```js
let message = tag`Hello world`;
// 此时，模板标签是 tag
```

标签可以是一个函数，传入的字符串会被分成各部分，每部分作为一个参数。

```js
let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;
    
    // 等同于
    message = passthru(['', ' items cost $', '.'], 10, 2.50)
```

<!--此时，`passthru()`函数会接受3个参数：首先是一个数组，元素有：-->

<!--- 第一个占位符前的空字符串（''）-->
<!--- 第一、二个占位符之间的字符串（' items cost $'）-->
<!--- 第二个占位符后的字符串（'.'）-->

<!--然后是变量`count`的值，最后是`(count * price).toFixed(2)`的值。-->

### 访问转义前字符串

用`String.raw()`标签函数，可以访问字符转义转换前的原生字符串：

```js
let message1 = `Multiline\nstring`,
    message2 = String.raw`${message1}`;
    
console.log(message1); // 'Multiline
                      //  string'
console.log(message2); // 'Multiline\\nstring'
```

模板标签函数的第一个参数是数组，它有一个`raw`属性，也可以用其访问原生字符串信息。

---

__参考资料__

- 《深入理解ES6》
- [《ECMAScript 6 入门》](http://es6.ruanyifeng.com/#docs/string#%E6%A8%A1%E6%9D%BF%E5%AD%97%E7%AC%A6%E4%B8%B2)
- [DSL简介](http://blog.csdn.net/u010278882/article/details/50554299)
- [DSL编程技术的介绍](https://www.jianshu.com/p/17266c5b8d1c)