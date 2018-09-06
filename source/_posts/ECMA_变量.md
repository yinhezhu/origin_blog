---
title: ECMA-变量
date: 2018-04-17 15:36:07
tags: [ECMA,变量]
category: "ECMA"
---
## 一、变量类型

### js的基本数据类型
```
  Undefined、Null、Boolean、Number、String、
  ECMAScript 2015 新增:Symbol(创建后独一无二且不可变的数据类型 )
```
### 变量类型区别
栈：原始数据类型（`Undefined`，`Null`，`Boolean`，`Number`、`String`）<br />
堆：引用数据类型（对象、数组和函数）

<p class="tip">两种类型的区别是：存储位置不同；
原始数据类型直接存储在栈(stack)中的简单数据段，占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储；
引用数据类型存储在堆(heap)中的对象,占据空间大、大小不固定。如果存储在栈中，将会影响程序运行的性能；引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体</p>

![](http://ou3l58em5.bkt.clouddn.com/varHeap.gif)

### 判断变量类型
#### 1、`typeof `
> `typeof ` 操作符返回的是类型字符串，它的返回值有6种取值：

```javascript
typeof 3 // "number"
typeof "abc" // "string"
typeof {} // "object"
typeof true // "boolean"
typeof undefined // "undefined"
typeof function () {} // "function"
typeof null // "object""
typeof [] // "object"
```
#### 2、`instanceof`
> `instanceof`操作符用于检查某个对象的原型链是否包含某个构造函数的`prototype`属性。例如：

```javascript
[1, 2, 3] instanceof Array // true
/abc/ instanceof RegExp // true
({}) instanceof Object // true
(function(){}) instanceof Function // true
(new Date) instanceof Date // true
```
> `instanceof`对基本数据类型不起作用，因为基本数据类型没有原型链。

```javascript
3 instanceof Number // false
true instanceof Boolean // false
'abc' instanceof String // false
null instanceof XXX  // always false
undefined instanceof XXX  // always false
```

#### 3、`toString`
> `toString`方法是最为可靠的类型检测手段，它会将当前对象转换为字符串并输出。 `toString`属性定义在`Object.prototype`上，因而所有对象都拥有`toString`方法。 但`Array`, `Date`等对象会重写从`Object.prototype`继承来的`toString`， 所以最好用`Object.prototype.toString`来检测类型。

```javascript
toString = Object.prototype.toString;

toString.call(new Date);    // [object Date]
toString.call(new String);  // [object String]
toString.call(Math);        // [object Math]
toString.call(3);           // [object Number]
toString.call([]);          // [object Array]
toString.call({});          // [object Object]

// Since JavaScript 1.8.5
toString.call(undefined);   // [object Undefined]
toString.call(null);        // [object Null]
```
### 总结
* `typeof`只能检测基本数据类型，对于`null`还有Bug；
* `instanceof`适用于检测对象，它是基于原型链运作的；
* `constructor`指向的是最初创建者，而且容易伪造，不适合做类型判断；
* `toString`适用于ECMA内置JavaScript类型（包括基本数据类型和内置对象）的类型判断；
* 基于引用判等的类型检查都有跨窗口问题，比如`instanceof`和`constructor`。

<p class="tip">总之，如果你要判断的是基本数据类型或JavaScript内置对象，使用`toString`； 如果要判断的时自定义类型，请使用`instanceof`</p>

* 另外Duck Typing的方式也非常可行
```javascript
isWindow:  function (obj) {
    return obj && typeof obj === 'object' && "setInterval" in obj;
}
```

## 二、变量作用域
JavaScript语言提供了函数作用域，同时JavaScript允许嵌套的函数定义。 有趣的是，内部函数（inner function）的生命周期可能超过父级函数。 这时便会显示出JavaScript的一个特殊现象：闭包。 闭包为面向对象编程提供了另外一种有趣的封装方式，我们无需为私有变量声明`private`。
### 函数作用域
程序设计语言中的作用域（scope）控制着变量和参数的可见性和生命周期。 它的重要性体现在避免命名冲突和自动内存管理两个方面，极大地减少了程序员的工作。
多数编程语言都拥有块作用域（block scope），由一对大括号限定其中变量的作用域。 比如C++的变量只在块内可见，变量被定义时执行内存申请和构造函数， 控制流退出代码块后内部的变量又被析构和内存回收。
不幸的是JavaScript提供了块语法，却不提供块作用域，而是提供函数作用域。 这意味着参数和变量在函数外部不可见，在函数内部始终可见。
### 闭包
JavaScript允许在函数中嵌套定义函数，这意味着内部函数（inner function）也可以访问父级函数的上下文（包括参数和变量）。 有趣的是，有时内部函数拥有更长的生命周期。来一个典型的面试题：
```javascript
function f() {
    for (var i = 0; i < 3, i++) {
        setTimeout(function () {
            console.log(i);
        }, 1000);
    }
}
f(); // 333
```
`i`是`function f`中定义的变量，在传递给`setTimeout`的函数中，`i`仍然有效（其实它叫做闭包变量）。 所以1s后`console.log`时i仍然是`function f`中定义的那个`i`，这时它的值为`3`。

这意味着内部函数可以访问真正的外部变量，而不是外部变量的副本。 函数可以访问它被创建时的所有上下文变量，这就叫做闭包现象。

如果我们希望上述代码输出`123`，我们需要开启一个子作用域。而JavaScript只提供函数作用域，所以我们需要添加一层函数：
```js
function f() {
    for (var i = 0; i < 3; i++) {
        !function (i) {
            setTimeout(function () {
                console.log(i);
            }, 1000);
        }(i);
    }
}
```
这时输出便是`123`了。其中的`!`是为了让JavaScript将这一句解析为表达式，而不是函数声明。 其实任何运算符都可以(`+-`），但如果没有便是语法错。
### 变量提升
一般编程语言建议将变量定义推迟，越晚越好。然而在JavaScript中不是这样， 应当将一切变量都在函数体最上方声明。例如：
```js
var p = {name: 'harttle'};
function foo() {
    console.log(p.name);
    var p = {name: 'alice'};
}
```
上述代码将产生运行时错误：
```bath
Uncaught TypeError: Cannot read property 'name' of undefined
```
这是因为在JavaScript的函数作用域实现中，所有变量声明（var xxx）都会被提升。 就像在函数体刚进入时声明这些变量一样。上述代码相当于：
```js
var p = {name: 'harttle'};
function foo() {
    var p;
    console.log(p.name);
    p = {name: 'alice'};
}
```
> Note：变量声明与初始化是两回事，变量提升指的是声明提升，初始化/赋值并不会提升。

### 闭包封装
基于类声明的面向对象语言不通，JavaScript基于原型继承。 JavaScript并未提供`public`, `protected`, `private`等关键字。 而JavaScript的闭包机制则可以完美地提供封装。你只需要：
* 1、将public属性赋值到对象属性。
* 2、将private属性声明为局部变量或内部函数。

```js
function Counter() {
    var i = 0;
    return {
        count: function () {i++;}
        get: function () {return i;}
    }
}
var counter = Counter();
counter.count();
counter.get();      // 1
```
其中`i`便是`private`；`count`,`get`便是`public`。

## 变量转换
### parseInt parseFloat
> parseInt() 函数可解析一个字符串，并返回一个整数。
```js
parseInt(string, radix)
```
> parseFloat() 函数可解析一个字符串，并返回一个浮点数。
```js
parseFloat(string)
```

考验你对map的了解，考验你对parseInt方法的了解
```js
["1","2","3"].map(parseInt)   // [1, NaN, NaN]
["1","2","3"].map(function (){ console.log(arguments) })

// ["1", 0, Array[3]]

// ["2", 1, Array[3]]

// ["3", 2, Array[3]]
```

### `toString`方法
它的作用是返回一个反映这个对象的字符串
<p class="tip">所有对象继承了两个转换方法：`toString()` and `valueOf()`</p>

```js
//返回相应的字符串
console.log(
  ({x: 1,
    y: 1
   }).toString()

  );  // [object Object]
console.log([1,2,3].toString()); // 1,2,3
console.log((function (x) {f(x);}).toString()); //function (x){f(x); }
console.log(/\d+/g.toString()); // /\d+/g
console.log(new Date(2015,4,4).toString()); // Mon May 04 2015 00:00:00 GMT+0800

console.log(new Date(2015,4,4).valueOf()); //  1430668800000
```

### `valueOf`方法
它的作用是返回它相应的原始值，一般都是隐式调用它
<p class="tip">每个JavaScript固有对象的 valueOf 方法定义不同</p>

| 对象      | 返回值   |
| :-------- | :-----  |
| Array    | 数组的元素被转换为字符串，这些字符串由逗号分隔，连接在一起。其操作与 Array.toString 和 Array.join 方法相同。     |
| Boolean  | Boolean 值。      |
| Date     | 存储的时间是从 1970 年 1 月 1 日午夜开始计的毫秒数 UTC。     |
| Function | 函数本身。      |
| Number   | 数字值。      |
| Object   | 对象本身。这是默认情况。     |
| String   | 字符串值。    |

Math 和 Error 对象没有 valueOf 方法。

### `===`运算符判断相等的流程

1. 如果两个值不是相同类型，它们不相等
2. 如果两个值都是null或者都是undefined，它们相等
3. 如果两个值都是布尔类型true或者都是false，它们相等
4. 如果其中有一个是NaN，它们不相等
5. 如果都是数值型并且数值相等，他们相等， -0等于0
6. 如果他们都是字符串并且在相同位置包含相同的16位值，他它们相等；如果在长度或者内容上不等，它们不相等；两个字符串显示结果相同但是编码不同==和===都认为他们不相等
7. 如果他们指向相同对象、数组、函数，它们相等；如果指向不同对象，他们不相等

### `==`运算符判断相等的流程
1. 如果两个值类型相同，按照===比较方法进行比较
2. 如果类型不同，使用如下规则进行比较
3. 如果其中一个值是null，另一个是undefined，它们相等
4. 如果一个值是数字另一个是字符串，将字符串转换为数字进行比较
5. 如果有布尔类型，将true转换为1，false转换为0，然后用==规则继续比较
6. 如果一个值是对象，另一个是数字或字符串，将对象转换为原始值然后用==规则继续比较
其他所有情况都认为不相等

### 对象到字符串的转换步骤
1. 如果对象有valueOf()方法并且返回元素值，javascript将返回值转换为数字作为结果
2. 否则，如果对象有toString()并且返回原始值，javascript将返回结果转换为数字作为结果
3. 否则，throws a TypeError

### <, >, <=, >=的比较规则
所有比较运算符都支持任意类型，但是比较只支持数字和字符串，所以需要执行必要的转换然后进行比较，转换规则如下:

1. 如果操作数是对象，转换为原始值：如果valueOf方法返回原始值，则使用这个值，否则使用toString方法的结果，如果转换失败则报错
2. 经过必要的对象到原始值的转换后，如果两个操作数都是字符串，按照字母顺序进行比较（他们的16位unicode值的大小）
3. 否则，如果有一个操作数不是字符串，将两个操作数转换为数字进行比较

### +运算符工作流程
1. 如果有操作数是对象，转换为原始值
2. 此时如果有一个操作数是字符串，其他的操作数都转换为字符串并执行连接
3. 否则：所有操作数都转换为数字并执行加法