# Javascript 基础问题

* [`[Basic]` 类型判断](/sections/zh-cn/common.md#类型判断)
* [`[Basic]` 作用域](/sections/zh-cn/common.md#作用域)
* [`[Basic]` 引用传递](/sections/zh-cn/common.md#引用传递)
* [`[Basic]` 内存释放](/sections/zh-cn/common.md#内存释放)
* [`[Basic]` ES6 新特性](/sections/zh-cn/common.md#es6-新特性)


## 简述

与前端 Js 不同, 后端方面除了SSR/爬虫之外很少会接触 DOM, 所以关于 DOM 方面的各种知识基本不会讨论. 前端很少碰到内存问题, 但是后端几乎是直面服务器内存的, 更加偏向内存方面, 对于一些更基础的问题也会更加关注.

不过由于 Js 方面的知识点实在太多, 《Javascript 权威指南》的厚度完全可以说明问题, 所以本教程并不会完整的带大家过一遍 Js 的基础问题, 只是简单列举一些饿了么在面试 Node.js 程序的时候通常会问的一些 Js 基础问题, 有的详细的地方会直接留下书名或者博文链接, 以供大家深入了解, 这里就不赘述了.

> 希望大家更多的是带着本文抛出的问题去学习, 而不是期待本文把所有答案列出来.

## 类型判断

Javascript 的类型判断其实是个挺折磨人的话题, 不然也不会有 Typescript 出现了. 在类型判断的问题上, 基础上 推荐阅读 [lodash](https://github.com/lodash/lodash) 的源代码.

这类问题一般只是简单的开场, 不会因为说你不知道 `undefined == null` 的结果是 `true` 就一票否决一个人. 只是根据个人经验看来，这个问题答不清楚的有不小的概率属于基础较差. 如果你对这种问题没有任何概念, 也许要反思一下是不是该找本书过一下 Js 的基础了.

另外在这个问题上, 对使用 TypeScript 以及 flow 同学会有一定的加分.

## 作用域

在面试时, 作用域并不是一个很好问的知识点, 一般会问的是 `es6 中 let 与 var 的区别`, 或者列举代码, 然后通过对代码的解读来看你对作用域的掌握比较方便.

印象中那本 [《你不知道的 Javascript》](https://book.douban.com/subject/26351021/) 讲的很好了, 有兴趣可以去看那本书, 以下是该书的部分目录:

* 第1章 作用域是什么
* 第2章 词法作用域
* 第3章 函数作用域和块作用域
* 第4章 提升
* 第5章 作用域闭包
* ...

## 引用传递

> <a name="q-value"></a> js 中什么类型是引用传递, 什么类型是值传递? 如何将值类型的变量以引用的方式传递?

简单点说, 对象是引用传递, 基础类型是值传递, 通过将基础类型包装 (boxing) 可以以引用的方式传递.(复杂见注①)

引用传递和值传递是一个非常简单的问题, 也是理解 Javascript 中的内存方面问题的一个基础. 如果不了解引用可能很难去看很多问题.

面试写代码的话, 可以通过 `如何编写一个 json 对象的拷贝函数` 等类似的问题来考察对引用的了解.
不过笔者偶尔会有恶趣味, 喜欢先问应聘者对于 `==` 的 `===` 的区别的了解. 然后再问 `[1] == [1]` 是 `true` 还是 `false`. 如果基础不好的同学可能会被自己对于 `==` 和 `===` 的结论影响然后得出错误的结论.

注①: 对于技术好的, 希望能直接反驳这个问题本身是有问题的, 比如讲清楚 Javascript 中没有引用传递只是传递引用. 参见 [Is JavaScript a pass-by-reference or pass-by-value language?](http://stackoverflow.com/questions/518000/is-javascript-a-pass-by-reference-or-pass-by-value-language). 虽然说是复杂版, 但是这些知识对于 3年经验的同学真的应该是很简单的问题了.

另外如果简历中有写 C++, 则必问 `指针与引用的区别`.

## 内存释放

> <a name="q-mem"></a> Javascript 中不同类型以及不同环境下变量的内存都是何时释放?

引用类型是在没有引用之后, 通过 v8 的 GC 自动回收, 值类型如果是处于闭包的情况下, 要等闭包没有引用才会被 GC 回收, 非闭包的情况下等待 v8 的新生代 (new space) 切换的时候回收.

与前端 Js 不同, 2年以上经验的 Node.js 一定要开始注意内存了, 不说对 v8 的 GC 有多了解, 基础的内存释放一定有概念了, 并且要开始注意内存泄漏的问题了.

你需要了解哪些操作一定会导致内存泄漏, 或者可以崩掉内存. 比如如下代码能否爆掉 V8 的内存?

```javascript
let arr = [];
while(true)
  arr.push(1);
```

然后上述代码与下方的情况有什么区别?

```javascript
let arr = [];
while(true)
  arr.push();
```

如果 push 的是 `Buffer` 情况又会有什么区别?

```javascript
let arr = [];
while(true)
  arr.push(new Buffer(1000));
```

思考完之后可以尝试找找别的情况如何爆掉 V8 的内存. 以及来聊聊内存泄漏?

```javascript
var theThing = null  
var replaceThing = function () {
  var originalThing = theThing
  var unused = function () {
    if (originalThing)
      console.log("hi")
  }
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log(someMessage)
    }
  };
};
setInterval(replaceThing, 1000)
```

比如上述情况中 `unused` 的函数中持有了 `originalThing` 的引用, 使得每次旧的对象不会释放从而导致内存泄漏 (例子出自[《Node.js 垃圾回收》](https://eggggger.xyz/2016/10/22/node-gc/))

当然对于一些高水平的同学, 要求能清楚的了解 v8 内存 GC 的机制, 懂得内存快照等 (之后会在`调试/优化`的小结中讨论) 了. 比如 V8 中不同类型的数据存储的位置, 在内存释放的时候不同区域的不同策略等等.

## ES6 新特性

推荐阅读阮一峰的 [《ECMAScript 6 入门》](http://es6.ruanyifeng.com/)

比较简单的会问 `let` 与 `var` 的区别, 以及 `箭头函数` 与 `function` 的区别等等.

深入的话, es6 有太多细节可以深入了. 比如结合 `引用` 的知识点来询问 `const` 方面的知识. 结合 `{}` 的使用与缺点来谈 `Set, Map` 等. 比如私有化的问题与 `symbol` 等等.

其他像是 `闭包是什么?` 这种问烂了问题已经感觉没必要问了, 取而代之的是询问闭包应用的场景更加合理. 比如说, 如果回答者通常使用闭包实现数据的私有, 那么可以接着问 es6 的一些新特性 (例如 `class`, `symbol`) 能否实现私有, 如果能的话那为什么要用闭包? 亦或者是什么闭包中的数据/私有化的数据的内存什么时候释放? 等等.

`...` 的使用上, 如何实现一个数组的去重 (使用 Set 可以加分).

> <a name="q-const"></a> const 定义的 Array 中间元素能否被修改? 如果可以, 那 const 修饰对象有什么意义?

其中的值可以被修改. 意义上, 主要保护引用不被修改 (如用 [Map](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Map) 等接口对引用的变化很敏感, 使用 const 保护引用始终如一是有意义的), 也适合用在 immutable 的场景.

暂时写上这些, 之后会慢慢整理, 如果内容比较多可能单独归一类来讨论.
