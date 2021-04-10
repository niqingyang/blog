---
title: "ECMAScript 5 学习笔记"
date: 2019-05-10T00:24:11+08:00
draft: false
categories:
    - web
tags:
    - es5
    - js
coverColor: rgba(255, 193, 7, 0.97)
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410224839-es5.png
coverStyle: 1
---

## 前言

ECMAScript 5.1 (或仅 ES5) 是 ECMAScript (基于 JavaScript 的规范)标准最新修正。 与 HTML5 规范进程本质类似，ES5 通过对现有 JavaScript 方法添加语句和原生 ECMAScript 对象做合并实现标准化。

## 严格模式（Strict Mode）

严格模式（Strict Mode）是 ECMAScript 5 的新特性，它允许你把整个程序，或者某个函数，放置在“严格”的操作语境中。

在 JS 文件或是函数的顶部（前无语句，可有注释）添加"use strict"；即可启用严格模式。这种严格的语境会防止某些特定的操作并抛出更多的异常。

```js
// 全局开启严格模式
"use strict";

// 函数内部开启严格模式
function test(){
   "use strict";
   // other code ...
}
```

严格模式下主要注意事项：

- 使用未显示声明的变量会报错（正常模式下会创建新全局变量，之后有可能报错）

- 任何在正常模式下引起静默失败的赋值操作 (给不可写属性赋值, 给只读属性 (getter-only) 赋值, 给不可扩展对象的新属性赋值) 都会抛出异常 (正常模式下无提示)

```js
"use strict";
// 给不可写属性赋值
var obj1 = {};
Object.defineProperty(obj1, "x", { value: 42, writable: false });
// 抛出TypeError错误
obj1.x = 9;
// 给只读属性赋值
var obj2 = { get x() { return 17; } };
// 抛出TypeError错误
obj2.x = 5;
// 给不可扩展对象的新属性赋值
var fixed = {};
Object.preventExtensions(fixed);
// 抛出TypeError错
fixed.newProp = "ohai";
```

- 严格模式下, 试图删除不可删除的属性时会抛出异常 (正常模式下无提示)

```js
"use strict";
// 抛出TypeError错误
delete Object.prototype;
```

- 禁止对象属性重名 (正常模式下后者覆盖前者)
- 禁止函数参数重名 (正常模式下如有多个重名的参数，可用arguments[i]读取)
- 禁止用delete删除变量
- 禁止使用 arguments, callee, caller 属性
- 禁止使用 width 语句
- 不允许在非函数的代码块内声明函数
- 强制为eval创建新作用域
- 严格模式下,无法直接删除变量。只有configurable设置为true的对象属性，才能被删除
- 新增保留字：implements, interface, let, package, private, protected, public, static, yield。

## JSON

ES5提供一个全局的JSON对象，
用来序列化(JSON.stringify)和反序列化(JSON.parse)对象为JSON格式。

### JSON.parse

[JSON.parse()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse "JSON.parse()") 方法用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。提供可选的reviver函数用以在返回之前对所得到的对象执行变换(操作)。

语法：

```js
JSON.parse(text[, reviver])
```
- text 要被解析成JavaScript值的字符串。
- reviver 转换器, 如果传入该参数(函数)，可以用来修改解析生成的原始值，调用时机在parse函数返回之前。详细用法可以参考[官方文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse#Example:_Using_the_reviver_parameter "官方文档")。


### JSON.stringify

[JSON.stringify()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify "JSON.stringify()")方法是将一个JavaScript值(对象或者数组)转换为一个 JSON字符串，如果指定了replacer是一个函数，则可以替换值，或者如果指定了replacer是一个数组，可选的仅包括指定的属性。

语法：

```js
JSON.stringify(value[, replacer [, space]])
```

参数：

- `value` 将要序列化成 一个JSON 字符串的值。
- `replacer` 可选，如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理；详情可以参考[官方文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#replacer%E5%8F%82%E6%95%B0 "官方文档")。
- `space` 可选，指定缩进用的空白字符串，用于美化输出（pretty-print），可以为数字、字符串、制表符，数字和字符串的长度代表缩进的长度，上限为10。

**关于序列化，有下面五点注意事项：**

- 非数组对象的属性序列化后属性的排序顺序可能会变。
- 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
- undefined、任意的函数以及 symbol 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 null（出现在数组中时）。
- 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。
- 所有以 symbol 为属性键的属性都会被完全忽略掉，即便 replacer 参数中强制指定包含了它们。
- 不可枚举的属性会被忽略

## Function 新增方法

### bind

[bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind "function.bind()") 函数和 call、apply 有些类似，不过 call、apply 是用于执行某个函数，而 bind 是返回一个指定了函数中 this 对象的函数

```js
function.bind(thisArg[, arg1[, arg2[, ...]]])
```

## Array 新增方法

ES5 Array 增加了 `isArray`、`every`、`some` 、`forEach`、`filter` 、`indexOf`、`lastIndexOf`、`map`、`reduce`、`reduceRight` 方法

以下方法添加到了Array.prototype对象上（isArray除外）

### isArray

[Array.isArray()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray "Array.isArray()") 用于确定传递的值是否是一个 `Array`。

- 如果检测的对象是 Array ，则返回true，否则为false。
- 当检测Array实例时, Array.isArray 优于 instanceof,因为Array.isArray能检测iframes.

语法：

```js
Array.isArray(obj)

// 如果对象是 Array ，则返回true，否则为false。
```


### indexOf

indexOf() 方法返回在数组中可以找到一个给定元素的第一个索引，如果不存在，则返回-1。

```js
array.indexOf(searchElement)
array.indexOf(searchElement[, fromIndex = 0])
```

参数

- searchElement

要查找的元素

- fromIndex

开始查找的位置。如果该索引值大于或等于数组长度，意味着不会在数组里查找，返回-1。如果参数中提供的索引值是一个负值，则将其作为数组末尾的一个抵消，即-1表示从最后一个元素开始查找，-2表示从倒数第二个元素开始查找 ，以此类推。 注意：如果参数中提供的索引值是一个负值，并不改变其查找顺序，查找顺序仍然是从前向后查询数组。如果抵消后的索引值仍小于0，则整个数组都将会被查询。其默认值为0.

### lastIndexOf

类似 indexOf() 方法（顺序相反）

### from

[Array.from()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from "Array.from()") 方法从一个类似数组或可迭代对象中创建一个新的数组实例。

- Array.from() 可以根据伪数组对象（拥有一个 length 属性和若干索引属性的任意对象）和 可迭代对象（可以获取对象中的元素,如 Map和 Set 等）来创建数组。

语法：

```js
Array.from(arrayLike[, mapFn[, thisArg]])
```

拆分字符串为数组

```js
Array.from("Hello World!")
// Array ["H", "e", "l", "l", "o", " ", "W", "o", "r", "l", "d", "!"]
```

### map

[map()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map "map()") 方法会创建并返回一个新数组，其结果是被遍历数组中的每个元素都调用一个提供的函数后返回的结果。

- `map` 的用法与 `foreach` 的用法类似，但 `map` 的 `callback` 函数需要有 `return` 值，如果没有就为 `undefined`。
- map 方法会给原数组中的每个元素都按顺序调用一次 `callback` 函数。`callback` 每次执行后的返回值（包括 `undefined`）组合起来形成一个新数组。 `callback` 函数只会在有值的索引上被调用；那些从来没被赋过值或者使用 `delete` 删除的索引则不会被调用。
- `callback` 函数会被自动传入三个参数：数组元素，元素索引，原数组本身。
- map 不修改调用它的原数组本身（当然可以在 `callback` 执行时改变原数组）。

语法：

```js
var new_array = arr.map(function callback(currentValue[, index[, array]]) {
 // 为新数组返回一个元素
}[, thisArg])
```

使用 map 重新格式化数组中的对象

```js
var kvArray = [{key: 1, value: 10},
               {key: 2, value: 20},
               {key: 3, value: 30}];

var reformattedArray = kvArray.map(function(obj) {
   var rObj = {};
   rObj[obj.key] = obj.value;
   return rObj;
});

// reformattedArray 数组为： [{1: 10}, {2: 20}, {3: 30}],

// kvArray 数组未被修改:
// [{key: 1, value: 10},
//  {key: 2, value: 20},
//  {key: 3, value: 30}]
```

使用 map 遍历字符串

```js
var map = Array.prototype.map
var a = map.call("Hello World", function(x) {
  return x;
});
// a 数组为：["H", "e", "l", "l", "o", " ", "W", "o", "r", "l", "d"]
```

通常情况下，map 方法中的 callback 函数只需要接受一个参数，就是正在被遍历的数组元素本身。但这并不意味着 map 只给 callback 传了一个参数。这个思维惯性可能会让我们犯一个很容易犯的错误。

```js
// 下面的语句返回什么呢:
["1", "2", "3"].map(parseInt);
// 你可能觉的会是[1, 2, 3]
// 但实际的结果是 [1, NaN, NaN]

// 通常使用 parseInt 时,只需要传递一个参数.
// 但实际上,parseInt 可以有两个参数.第二个参数是进制数.
// 可以通过语句"alert(parseInt.length)===2"来验证.
// map方法在调用 callback 函数时,会给它传递三个参数:当前正在遍历的元素,
// 元素索引, 原数组本身.
// 第三个参数 parseInt 会忽视, 但第二个参数不会,也就是说,
// parseInt把传过来的索引值当成进制数来使用.从而返回了 NaN.
```

使用 map 将字符串类型的数字转换为数字类型

```js
function returnInt(element) {
  return parseInt(element, 10);
}

['1', '2', '3'].map(returnInt); // 返回的结果:[1, 2, 3]

// 也可以使用简单的箭头函数，结果同上
['1', '2', '3'].map( str => parseInt(str) );

// 一个更简单的方式:
['1', '2', '3'].map(Number); // [1, 2, 3]

// 与`parseInt` 不同，下面的结果会返回浮点数或指数:
['1.1', '2.2e2', '3e300'].map(Number); // [1.1, 220, 3e+300]
```

### reduce

[reduce()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce "reduce()") 方法对数组中的每个元素执行一个由您提供的reducer函数(升序执行)，将其结果汇总为单个返回值。

- 提供初始值通常更安全

语法：

```js
var result = arr.reduce(function(accumulator, currentValue[, currentIndex[, array]]){
	// do someting ...
	// return 下一个回调中的 accumulator
}[, initialValue])

// result 为函数累计处理的结果
```

`callback` 执行数组中每个值的函数，包含四个参数：

- `accumulator` 累计器累计回调的返回值; 它是上一次调用回调时返回的累积值，或 initialValue（见于下方）。
- `currentValue` 数组中正在处理的元素。
- `currentIndex` 可选，数组中正在处理的当前元素的索引。 如果提供了 initialValue，则起始索引号为0，否则为1。
- `array` 可选，调用reduce()的数组

<info>

> `initialValue` 可选，作为第一次调用 callback函数时的第一个参数的值。 如果没有提供初始值，则将使用数组中的第一个元素。 在没有初始值的空数组上调用 reduce 将报错。

</info>

<warning>

> 如果数组为空且没有提供 initialValue，会抛出 TypeError 。如果数组仅有一个元素（无论位置如何）并且没有提供 initialValue， 或者有提供 initialValue 但是数组为空，那么此唯一值将被返回并且 callback 不会被执行。

</warning>

求数组元素之和

```js
var sum = [0, 1, 2, 3].reduce(function (accumulator, currentValue) {
  return accumulator + currentValue;
}, 0);
// 和为 6
```

二维数组转为一维数组
```js
var flattened = [[0, 1], [2, 3], [4, 5]].reduce(
  function(a, b) {
    return a.concat(b);
  },
  []
);
// flattened is [0, 1, 2, 3, 4, 5]
```

### filter

[filter()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter "filter()") 方法如图这个函数名一样，能够对数组进行过滤筛选，其会为数组中的每一个元素调用一次 `callback` 函数，并利用所有使得 `callback` 返回 `true` 或等价于 `true` 的值的元素创建一个新数组。

- `callback` 只会在已经赋值的索引上被调用，对于那些已经被删除或者从未被赋值的索引不会被调用。那些没有通过 `callback` 测试的元素会被跳过，不会被包含在新数组中。
- `filter` 不会改变原数组，它返回过滤后的新数组。

语法：

```js
var new_array = arr.map(function callback(currentValue[, index[, array]]) {
	// 通过 return true 或者 return false 表示是否将当前元素放入新的数组中
}[, thisArg])
```

通过 `filter` 筛选排除所有较小的值

```js
var filtered = [12, 5, 8, 130, 44].filter(function(element){
	return element >= 10;
});
// filtered is [12, 130, 44]
```

### reduceRight

[reduceRight()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight "reduceRight()") 方法接受一个函数作为累加器（accumulator），让每个值（从右到左，亦即从尾到头）缩减为一个值。（与 reduce() 的执行方向相反）

### every (且)

[every()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/every "every()") 方法用于检测数组所有元素是否都符合指定条件（通过 `callback` 函数检测）。

- `every` 方法为数组中的每个元素执行一次 `callback` 函数，直到它找到一个使 `callback` 返回 `false`（表示可转换为布尔值 `false` 的值）的元素。如果发现了一个这样的元素，`every` 方法将会立即返回 `false`。否则，`callback` 为每一个元素返回 `true`，`every` 就会返回 `true`。
- `callback` 只会为那些已经被赋值的索引调用。不会为那些被删除或从来没被赋值的索引调用。
- `every` 不会对空数组进行检测。
- `every` 不会改变原数组。

语法：

```js
var result = arr.every(function callback(currentValue[, index[, array]]) {
	// return true 或者 return false
}[, thisArg])

// result is true or false
```

通过 `every` 判断数组中元素是否全部为 `Number`

```js
function isNumber(obj) {
  return obj !== undefined && typeof(obj) === 'number' && !isNaN(obj);
}

var a1 = [1, 2, 3];
console.log(a1.every(isNumber)); // logs true
var a2 = [1, '2', 3];
console.log(a2.every(isNumber)); // logs false
```

### some (或)

[some()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/some "some()") 方法，只要数组中有一个元素在 `callback` 上被返回 `true`，就返回 `true`。

- 如果有一个元素满足条件，则表达式返回true , 剩余的元素不会再执行检测。
- 如果没有满足条件的元素，则返回false。
- some() 不会对空数组进行检测。
- some() 不会改变原始数组。


语法：

```js
var result = arr.some(function callback(currentValue[, index[, array]]) {
	// return true 或者 return false
}[, thisArg])

// result is true or false
```

注意：对于放在空数组上的任何条件，此方法返回 `false`。

```js
var array = [];

var even = function(element) {
  return true;
};

// even is false
```

通过 `some` 判断是否包含 `Number`

```js
function isNumber(value){
	return obj !== undefined && typeof(obj) === 'number' && !isNaN(obj);
}
var a1 = [1, 2, 3];
console.log(a1.some(isNumber)); // logs true
var a2 = [1, '2', 3];
console.log(a2.some(isNumber)); // logs true
var a3 = ['1', '2', '3'];
console.log(a3.some(isNumber)); // logs false
```

### forEach

[forEach()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach "forEach()") 方法用于遍历数组，对数组的每个元素执行一次提供的函数。

- 方法按升序为数组中含有效值的每一项执行一次 `callback` 函数，`那些已删除或者未初始化的项将被跳过（例如在稀疏数组上）`。
- 方法还可以接受一个可选的上下文参数（用于改变 `callback` 函数中的 `this`），如果未指定第二个参数，则默认为全局对象（浏览器中为 window 对象，严格模式下甚至是 undefined）。
- 方法永远返回 undefined，所以无法进行链式操作，在其回调函数中也无法中止或者跳出循环。

语法：

```js
arr.forEach(function(value, index, array){
	// ...
}[, thisArg]);
```

使用 forEach 方法遍历数组元素

```js
// 未初始化的不会被输出
var words = [ , , 'a', 'b', 'c', ,];

words.forEach(function(item) {
  console.log(item);
});

// expected output: "a"
// expected output: "b"
// expected output: "c"
```

### find

[find()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/find "find()") 方法返回数组中满足提供的测试函数的第一个元素的`值`。否则返回 `undefined`。

findIndex() 方法，它返回数组中找到的元素的索引，而不是其值。

如果需要找到一个元素的位置或者一个元素是否存在于数组中，使用Array.prototype.indexOf() 或 Array.prototype.includes()。

语法：

```js
var result = arr.find(function callback(currentValue[, index[, array]]) {
	// return true 或者 return false
}[, thisArg])

// result 是数组中第一个满足条件的元素
```

### findIndex

[findIndex()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex "findIndex()") 方法返回数组中满足提供的测试函数的第一个元素的`索引`。否则返回`-1`。

语法：

```js
var result = arr.findIndex(function callback(currentValue[, index[, array]]) {
	// return true 或者 return false
}[, thisArg])

// result 是数组中第一个满足条件的元素的索引，否则返回-1
```

### includes

[includes()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/includes "includes()") 方法用来判断一个数组是否包含一个指定的`值`，根据情况，如果包含则返回 `true`，否则返回 `false`。

- 注意：对象数组不能使用 `includes` 方法来检测。
- 注意：使用 `includes()` 比较字符串和字符时是区分大小写。

```js
arr.includes(valueToFind[, fromIndex])
```

<info>

> 从 fromIndex 索引处开始查找 valueToFind。如果为负值，则按升序从 array.length + fromIndex 的索引开始搜 （即使从末尾开始往前跳 fromIndex 的绝对值个索引，然后往后搜寻）。默认为 0

</info>

示例

```js
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
[1, 2, NaN].includes(NaN); // true
// 如果 fromIndex 大于等于数组的长度，则会返回 false，且该数组不会被搜索
[1, 2, 3].includes(1, 100); // false
// 如果 fromIndex 为负值，计算出的索引将作为开始搜索searchElement的位置。如果计算出的索引小于 0，则整个数组都会被搜索。
[1, 2, 3].includes(1, -100); // true
```

`includes()` 方法有意设计为通用方法。它不要求 `this` 值是数组对象，所以它可以被用于其他类型的对象 (比如类数组对象)。下面的例子展示了 在函数的 `arguments` 对象上调用的 `includes()` 方法。

```js
(function() {
  console.log([].includes.call(arguments, 'a')); // true
  console.log([].includes.call(arguments, 'd')); // false
})('a','b','c');
```

## String 新增方法

### trim

[trim()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/trim "trim()") 方法会从一个字符串的两端删除空白字符。在这个上下文中的空白字符是所有的空白字符 (space, tab, no-break space 等) 以及所有行终止符字符（如 LF，CR）。

- trim() 方法并不影响原字符串本身，它返回的是一个新的字符串。

语法：

```js
str.trim()
```

## Date 新增方法

### Date.now()

[Date.now()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date/now "Date.now()") 方法返回自1970年1月1日 00:00:00 UTC到当前时间的毫秒数，类型为Number。

语法：

```js
var timeInMs = Date.now();
```

- now() 是Date的一个静态函数，所以必须以 Date.now() 的形式来使用

## Object 新增方法

- Object.create() - 使用指定的原型对象和属性创建一个新对象。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create "[更多...]")

- Object.defineProperty() - 给对象添加一个属性并指定该属性的配置。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty "[更多...]")

- Object.defineProperties() - 给对象添加多个属性并分别指定它们的配置。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties "[更多...]")

- Object.getOwnPropertyDescriptor() - 返回对象指定的属性配置。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor  "[更多...]")

- Object.getOwnPropertyNames() - 返回一个数组，它包含了指定对象所有的可枚举或不可枚举的属性名。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames "[更多...]")

- Object.getPrototypeOf() - 返回指定对象的原型对象。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf "[更多...]")

- Object.freeze() - 冻结对象：其他代码不能删除或更改任何属性。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze "[更多...]")

- Object.isFrozen() - 判断对象是否已经冻结。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/isFrozen "[更多...]")

- Object.seal() - 防止其他代码删除对象的属性。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/seal "[更多...]")

- Object.isSealed() - 判断对象是否已经密封。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/isSealed "[更多...]")

- Object.preventExtensions() - 防止对象的任何扩展。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions "[更多...]")

- Object.isExtensible() - 判断对象是否可扩展。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/isExtensible "[更多...]")

- Object.keys() - 返回一个包含所有给定对象自身可枚举属性名称的数组。[[更多...]](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys "[更多...]")






## 参考文档

- [https://www.jianshu.com/p/a19b6400b8f7](https://www.jianshu.com/p/a19b6400b8f7 "https://www.jianshu.com/p/a19b6400b8f7")
- [https://www.cnblogs.com/jesse131/p/6501812.html](https://www.cnblogs.com/jesse131/p/6501812.html "https://www.cnblogs.com/jesse131/p/6501812.html")
- [http://www.cnblogs.com/lovesong/p/4908871.html](http://www.cnblogs.com/lovesong/p/4908871.html "http://www.cnblogs.com/lovesong/p/4908871.html")