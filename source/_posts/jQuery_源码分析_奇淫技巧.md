---
title: jQuery_源码分析_奇淫技巧
date: 2017-04-17 16:32:53
tags: [jQuery,源码]
category: "jQuery"
---
系列第一篇： [【深入浅出jQuery】源码浅析--整体架构](/2018/04/17/jQuery-源码分析/)

本篇是系列第二篇，标题起得有点大，希望内容对得起这个标题，这篇文章主要总结一下在 jQuery 中一些十分讨巧的 coding 方式，将会由浅及深，可能会有一些基础，但是我希望全面一点，对看文章的人都有所帮助，源码我还一直在阅读，也会不断的更新本文。

即便你不想去阅读源码，看看下面的总结，我想对提高编程能力，转换思维方式都大有裨益，废话少说，进入正题。
## 短路表达式 与 多重短路表达式
`短路表达式`这个应该人所皆知了。在 `jQuery` 中，大量的使用了短路表达式与多重短路表达式。

`短路表达式`：作为"&&"和"||"操作符的操作数表达式，这些表达式在进行求值时，只要最终的结果已经可以确定是真或假，求值过程便告终止，这称之为短路求值。这是这两个操作符的一个重要属性。
```javascript
// ||短路表达式
var foo = a || b;
// 相当于
if(a){
    foo = a;
}else{
    foo = b;
}

// &&短路表达式
var bar = a && b;
// 相当于
if(a){
    bar = b;
}else{
    bar = a;
}
```
当然，上面两个例子是短路表达式最简单是情况，多数情况下，`jQuery` 是这样使用它们的：
```javascript
// 选自 jQuery 源码中的 Sizzle 部分
function siblingCheck(a, b) {
    var cur = b && a,
        diff = cur && a.nodeType === 1 && b.nodeType === 1 &&
        (~b.sourceIndex || MAX_NEGATIVE) -
        (~a.sourceIndex || MAX_NEGATIVE);

    // other code ...
}
```
嗯，可以看到，`diff` 的值经历了多重短路表达式配合一些全等判断才得出，这种代码很优雅，但是可读性下降了很多，使用的时候权衡一下，多重短路表达式和简单短路表达式其实一样，只需要先把后面的当成一个整体，依次推进，得出最终值。
```javascript
var a = 1, b = 0, c = 3;

var foo = a && b && c, // 0 ,相当于 a && (b && c)
  bar = a || b || c;  // 1
```
这里需要提出一些值得注意的点：

1、在 `Javascript` 的逻辑运算中，`0、""、null、false、undefined、NaN` 都会判定为 `false` ，而其他都为 `true` ；

2、因为 `Javascript` 的内置弱类型域 ``(weak-typing domain)``，所以对严格的输入验证这一点不太在意，即便使用 && 或者 || 运算符的运算数不是布尔值，仍然可以将它看作布尔运算。虽然如此，还是建议如下：
```javascript
if(foo){ ... }     //不够严谨
if(!!foo){ ... }   //更为严谨，!!可将其他类型的值转换为boolean类型
```
注重细节，`JavaScript` 既不弱也不低等，我们只是需要更努力一点工作以使我们的代码变得真正健壮。
## 预定义常用方法的入口
在 `jQuery` 的头几十行，有这么一段有趣的代码：
```javascript
(function(window, undefined) {
    var
        // 定义了一个对象变量，一个字符串变量，一个数组变量
        class2type = {},
        core_version = "1.10.2",
        core_deletedIds = [],

        // 保存了对象、字符串、数组的一些常用方法 concat push 等等...
        core_concat = core_deletedIds.concat,
        core_push = core_deletedIds.push,
        core_slice = core_deletedIds.slice,
        core_indexOf = core_deletedIds.indexOf,
        core_toString = class2type.toString,
        core_hasOwn = class2type.hasOwnProperty,
        core_trim = core_version.trim;

})(window);
```
不得不说，`jQuery` 在细节上做的真的很好，这里首先定义了一个对象变量、一个字符串变量、数组变量，要注意这 3 个变量本身在下文是有自己的用途的（可以看到，`jQuery` 作者惜字如金，真的是去压榨每一个变量的作用，使其作用最大化）；其次，借用这三个变量，再定义些常用的核心方法，从上往下是数组的 `concat、push 、slice 、indexOf` 方法，对象的 `toString 、hasOwnProperty` 方法以及字符串的 `trim` 方法，`core_xxxx` 这几个变量事先存储好了这些常用方法的入口，如果下文行文当中需要调用这些方法，将会：
```javascript
jQuery.fn = jQuery.prototype = {
    // ...

    // 将 jQuery 对象转换成数组类型
    toArray: function() {
        // 调用数组的 slice 方法，使用预先定义好了的 core_slice ，节省查找内存地址时间，提高效率
        // 相当于 return Array.prototype.slice.call(this)
        return core_slice.call(this);
    }
}
```
可以看到，当需要使用这些预先定义好的方法，只需要借助 `call` 或者 `apply`（ [戳我详解](/2018/04/19/ECMA-理解call和apply/) ）进行调用。
那么 `jQuery` 为什么要这样做呢，我觉得：

1、以数组对象的 `concat` 方法为例，如果不预先定义好 `core_concat = core_deletedIds.concat` 而是调用实例 `arr` 的方法 `concat` 时，首先需要辨别当前实例 `arr` 的类型是 `Array`，在内存空间中寻找 `Array` 的 `concat` 内存入口，把当前对象 `arr` 的指针和其他参数压入栈，跳转到 `concat` 地址开始执行，而当保存了 `concat` 方法的入口 `core_concat` 时，完全就可以省去前面两个步骤，从而提升一些性能；

2、另外一点，借助 `call` 或者 `apply` 的方式调用，让一些类数组可以直接调用数组的方法。就如上面是示例，`jQuery` 对象是类数组类型，可以直接调用数组的 `slice` 方法转换为数组类型。又譬如，将参数 `arguments` 转换为数组类型：
```javascript
function test(a,b,c){
    // 将参数 arguments 转换为数组
    // 使之可以调用数组成员方法
    var arr = Array.prototype.slice.call(arguments);

    ...
}
```
## 钩子机制（hook）
在 `jQuery 2.0.0` 之前的版本，对兼容性做了大量的处理，正是这样才让广大开发人员能够忽略不同浏览器的不同特性的专注于业务本身的逻辑。而其中，钩子机制在浏览器兼容方面起了十分巨大的作用。

钩子是编程惯用的一种手法，用来解决一种或多种特殊情况的处理。

简单来说，钩子就是适配器原理，或者说是表驱动原理，我们预先定义了一些钩子，在正常的代码逻辑中使用钩子去适配一些特殊的属性，样式或事件，这样可以让我们少写很多 `else if` 语句。

如果还是很难懂，看一个简单的例子，举例说明 `hook` 到底如何使用：

现在考公务员，要么靠实力，要么靠关系，但领导肯定也不会弄的那么明显，一般都是暗箱操作，这个场景用钩子实现再合理不过了。
```javascript
// 如果不用钩子的情况
// 考生分数以及父亲名
function examinee(name, score, fatherName) {
    return {
        name: name,
        score: score,
        fatherName: fatherName
    };
}

// 审阅考生们
function judge(examinees) {
    var result = {};
    for (var i in examinees) {
        var curExaminee = examinees[i];
        var ret = curExaminee.score;
        // 判断是否有后门关系
        if (curExaminee.fatherName === 'xijingping') {
            ret += 1000;
        } else if (curExaminee.fatherName === 'ligang') {
            ret += 100;
        } else if (curExaminee.fatherName === 'pengdehuai') {
            ret += 50;
        }
        result[curExaminee.name] = ret;
    }
    return result;
}


var lihao = examinee("lihao", 10, 'ligang');
var xida = examinee('xida', 8, 'xijinping');
var peng = examinee('peng', 60, 'pengdehuai');
var liaoxiaofeng = examinee('liaoxiaofeng', 100, 'liaodaniu');

var result = judge([lihao, xida, peng, liaoxiaofeng]);

// 根据分数选取前三名
for (var name in result) {
    console.log("name:" + name);
    console.log("score:" + score);
}
```
可以看到，在中间审阅考生这个函数中，运用了很多 `else if` 来判断是否考生有后门关系，如果现在业务场景发生变化，又多了几名考生，那么 `else if` 势必越来越复杂，往后维护代码也将越来越麻烦，成本很大，那么这个时候如果使用钩子机制，该如何做呢？
```javascript
// relationHook 是个钩子函数，用于得到关系得分
var relationHook = {
    "xijinping": 1000,
    "ligang": 100,
    "pengdehuai": 50,
　　 // 新的考生只需要在钩子里添加关系分
}

// 考生分数以及父亲名
function examinee(name, score, fatherName) {
    return {
        name: name,
        score: score,
        fatherName: fatherName
    };
}

// 审阅考生们
function judge(examinees) {
    var result = {};
    for (var i in examinees) {
        var curExaminee = examinees[i];
        var ret = curExaminee.score;
        if (relationHook[curExaminee.fatherName] ) {
            ret += relationHook[curExaminee.fatherName] ;
        }
        result[curExaminee.name] = ret;
    }
    return result;
}


var lihao = examinee("lihao", 10, 'ligang');
var xida = examinee('xida', 8, 'xijinping');
var peng = examinee('peng', 60, 'pengdehuai');
var liaoxiaofeng = examinee('liaoxiaofeng', 100, 'liaodaniu');

var result = judge([lihao, xida, peng, liaoxiaofeng]);

// 根据分数选取前三名
for (var name in result) {
    console.log("name:" + name);
    console.log("score:" + score);
}
```
可以看到，使用钩子去处理特殊情况，可以让代码的逻辑更加清晰，省去大量的条件判断，上面的钩子机制的实现方式，采用的就是表驱动方式，就是我们事先预定好一张表（俗称打表），用这张表去适配特殊情况。当然 `jQuery` 的 `hook` 是一种更为抽象的概念，在不同场景可以用不同方式实现。

看看 `jQuery` 里的表驱动 `hook` 实现，`$.type` 方法：
```javascript
(function(window, undefined) {
    var
        // 用于预存储一张类型表用于 hook
        class2type = {};

    // 原生的 typeof 方法并不能区分出一个变量它是 Array 、RegExp 等 object 类型，jQuery 为了扩展 typeof 的表达力，因此有了 $.type 方法
    // 针对一些特殊的对象（例如 null，Array，RegExp）也进行精准的类型判断
    // 运用了钩子机制，判断类型前，将常见类型打表，先存于一个 Hash 表 class2type 里边
    jQuery.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
        class2type["[object " + name + "]"] = name.toLowerCase();
    });

    jQuery.extend({
        // 确定JavaScript 对象的类型
        // 这个方法的关键之处在于 class2type[core_toString.call(obj)]
        // 可以使得 typeof obj 为 "object" 类型的得到更进一步的精确判断
        type: function(obj) {

            if (obj == null) {
                return String(obj);
            }
            // 利用事先存好的 hash 表 class2type 作精准判断
            // 这里因为 hook 的存在，省去了大量的 else if 判断
            return typeof obj === "object" || typeof obj === "function" ?
                class2type[core_toString.call(obj)] || "object" :
                typeof obj;
        }
    })
})(window);
```
这里的 `hook` 只是 `jQuery` 大量使用钩子的冰山一角，在对 `DOM` 元素的操作一块，`attr 、val 、prop 、css 方法大量运用了钩子，用于兼容 IE 系列下的一些怪异行为。在遇到钩子函数的时候，要结合具体情境具体分析，这些钩子相对于表驱动而言更加复杂，它们的结构大体如下，只要记住钩子的核心原则，保持代码整体逻辑的流畅性，在特殊的情境下去处理一些特殊的情况：
```javascript
var someHook = {
    get: function(elem) {
        // obtain and return a value
        return "something";
    },
    set: function(elem, value) {
        // do something with value
    }
}
```
从某种程度上讲，钩子是一系列被设计为以你自己的代码来处理自定义值的回调函数。有了钩子，你可以将差不多任何东西保持在可控范围内。

从设计模式的角度而言，这种钩子运用了策略模式。
> 策略模式：将不变的部分和变化的部分隔开是每个设计模式的主题，而策略模式则是将算法的使用与算法的实现分离开来的典型代表。使用策略模式重构代码，可以消除程序中大片的条件分支语句。在实际开发中，我们通常会把算法的含义扩散开来，使策略模式也可以用来封装一系列的“业务规则”。只要这些业务规则指向的目标一致，并且可以被替换使用，我们就可以使用策略模式来封装他们。

策略模式的优点：

- 策略模式利用组合，委托和多态等技术思想，可以有效的避免多重条件选择语句；
- 策略模式提供了对开放-封闭原则的完美支持，将算法封装在独立的函数中，使得它们易于切换，易于理解，易于扩展。
- 策略模式中的算法也可以复用在系统的其它地方，从而避免许多重复的复制粘贴工作。

## 连贯接口
无论 `jQuery` 如今的流行趋势是否在下降，它用起来确实让人大呼过瘾，这很大程度归功于它的链式调用，接口的连贯性及易记性。很多人将连贯接口看成链式调用，这并不全面，我觉得连贯接口包含了链式调用且代表更多。而 `jQuery` 无疑是连贯接口的佼佼者。

`1、链式调用`：链式调用的主要思想就是使代码尽可能流畅易读，从而可以更快地被理解。有了链式调用，我们可以将代码组织为类似语句的片段，增强可读性的同时减少干扰。
```javascript
// 传统写法
var elem = document.getElementById("foobar");
elem.style.background = "red";
elem.style.color = "green";
elem.addEventListener('click', function(event) {
  alert("hello world!");
}, true);

// jQuery 写法
$('xxx')
    .css("background", "red")
    .css("color", "green")
    .on("click", function(event) {
    　　alert("hello world");
    });
```
`2、命令查询同体`：这个上一章也讲过了，就是函数重载。正常而言，应该是命令查询分离（Command and Query Separation，CQS），是源于命令式编程的一个概念。那些改变对象的状态（内部的值）的函数称为命令，而那些检索值的函数称为查询。原则上，查询函数返回数据，命令函数返回状态，各司其职。而 `jQuery` 将 `getter` 和 `setter` 方法压缩到单一方法中创建了一个连贯的接口，使得代码暴露更少的方法，但却以更少的代码实现同样的目标。

`3、参数映射及处理`：`jQuery` 的接口连贯性还体现在了对参数的兼容处理上，方法如何接收数据比让它们具有可链性更为重要。虽然方法的链式调用是非常普遍的，你可以很容易地在你的代码中实现，但是处理参数却不同，使用者可能传入各种奇怪的参数类型，而 `jQuery` 作者想的真的很周到，考虑了用户的多种使用场景，提供了多种对参数的处理。
```javascript
// 传入键值对
jQuery("#some-selector")
  .css("background", "red")
  .css("color", "white")
  .css("font-weight", "bold")
  .css("padding", 10);

// 传入 JSON 对象
jQuery("#some-selector").css({
  "background" : "red",
  "color" : "white",
  "font-weight" : "bold",
  "padding" : 10
});
```
`jQuery` 的 `on()`` 方法可以注册事件处理器。和 `CSS()` 一样它也可以接收一组映射格式的事件，但更进一步地，它允许单一处理器可以被多个事件注册：
```javascript
// binding events by passing a map
jQuery("#some-selector").on({
  "click" : myClickHandler,
  "keyup" : myKeyupHandler,
  "change" : myChangeHandler
});

// binding a handler to multiple events:
jQuery("#some-selector").on("click keyup change", myEventHandler);
```
## setTimeout in Jquery
写到这里，发现上文的主题有些飘忽，接近于写成了 如何写出更好的 Javascript 代码，下面介绍一些 jQuery 中我觉得很棒的小技巧。

熟悉 `jQuery` 的人都知道 `DOM Ready` 事件，传`Javascript`原生的 `window.onload` 事件是在页面所有的资源都加载完毕后触发的。如果页面上有大图片等资源响应缓慢, 会导致 `window.onload` 事件迟迟无法触发，所以出现了`DOM Ready` 事件。此事件在 `DOM` 文档结构准备完毕后触发，即在资源加载前触发。另外我们需要在 `DOM` 准备完毕后，再修改`DOM`结构，比如添加`DOM`元素等。而为了完美实现 `DOM Ready` 事件，兼容各浏览器及低版本`IE`（针对高级的浏览器，可以使用 DOMContentLoaded 事件，省时省力），在 `jQuery.ready()` 方法里，运用了 `setTimeout()` 方法的一个特性， 在 `setTimeout` 中触发的函数, 一定是在 `DOM` 准备完毕后触发。
```javascript
jQuery.extend({
    ready: function(wait) {
        // 如果需要等待，holdReady()的时候，把hold住的次数减1，如果还没到达0，说明还需要继续hold住，return掉
        // 如果不需要等待，判断是否已经Ready过了，如果已经ready过了，就不需要处理了。异步队列里边的done的回调都会执行了
        if (wait === true ? --jQuery.readyWait : jQuery.isReady) {
            return;
        }

        // 确定 body 存在
        if (!document.body) {
            // 如果 body 还不存在 ，DOMContentLoaded 未完成，此时
            // 将 jQuery.ready 放入定时器 setTimeout 中
            // 不带时间参数的 setTimeout(a) 相当于 setTimeout(a,0)
            // 但是这里并不是立即触发 jQuery.ready
            // 由于 javascript 的单线程的异步模式
            // setTimeout(jQuery.ready) 会等到重绘完成才执行代码，也就是 DOMContentLoaded 之后才执行 jQuery.ready
            // 所以这里有个小技巧：在 setTimeout 中触发的函数, 一定会在 DOM 准备完毕后触发
            return setTimeout(jQuery.ready);
        }

        // Remember that the DOM is ready
        // 记录 DOM ready 已经完成
        jQuery.isReady = true;

        // If a normal DOM Ready event fired, decrement, and wait if need be
        // wait 为 false 表示ready事情未触发过，否则 return
        if (wait !== true && --jQuery.readyWait > 0) {
            return;
        }

        // If there are functions bound, to execute
        // 调用异步队列，然后派发成功事件出去（最后使用done接收，把上下文切换成document，默认第一个参数是jQuery。
        readyList.resolveWith(document, [jQuery]);

        // Trigger any bound ready events
        // 最后jQuery还可以触发自己的ready事件
        // 例如：
        //    $(document).on('ready', fn2);
        //    $(document).ready(fn1);
        // 这里的fn1会先执行，自己的ready事件绑定的fn2回调后执行
        if (jQuery.fn.trigger) {
            jQuery(document).trigger("ready").off("ready");
        }
    }
})
```