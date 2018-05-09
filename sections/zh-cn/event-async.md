# 事件/异步

* [`[Basic]` Promise](/sections/zh-cn/event-async.md#promise)
* [`[Doc]` Events (事件)](/sections/zh-cn/event-async.md#events)
* [`[Doc]` Timers (定时器)](/sections/zh-cn/event-async.md#timers)
* [`[Point]` 阻塞/异步](/sections/zh-cn/event-async.md#阻塞异步)
* [`[Point]` 并行/并发](/sections/zh-cn/event-async.md#并行并发)

## 简述

异步还是不异步? 这是一个问题.

## Promise

![callback-hell](/assets/callback-hell.jpg)

相信很多同学在面试的时候都碰到过这样一个问题, `如何处理 Callback Hell`. 在早些年的时候, 大家会看到有很多的解决方案例如 [Q](https://www.npmjs.com/package/q), [async](https://www.npmjs.com/package/async), [EventProxy](https://www.npmjs.com/package/eventproxy) 等等. 最后从流行程度来看 `Promise` 当之无愧的独领风骚, 并且是在 ES6 的 JavaScript 标准上赢得了支持.

关于它的基础知识/概念推荐看阮一峰的 [Promise 对象](http://javascript.ruanyifeng.com/advanced/promise.html#toc9) 这里就不多不赘述.

> <a name="q-1"></a> Promise 中 .then 的第二参数与 .catch 有什么区别?

参见 [We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)

另外关于同步与异步, 有个问题希望大家看一下, 这是很简单的 Promise 的使用例子:

```javascript
let doSth = new Promise((resolve, reject) => {
  console.log('hello');
  resolve();
});

doSth.then(() => {
  console.log('over');
});
```

毫无疑问的可以得到以下输出结果:

```
hello
over
```

但是首先的问题是, 该 Promise 封装的代码肯定是同步的, 那么这个 then 的执行是异步的吗?

其次的问题是, 如下代码, `setTimeout` 到 10s 之后再 `.then` 调用, 那么 `hello` 是会在 10s 之后在打印吗, 还是一开始就打印?

```javascript
let doSth = new Promise((resolve, reject) => {
  console.log('hello');
  resolve();
});

setTimeout(() => {
  doSth.then(() => {
    console.log('over');
  })
}, 10000);
```

以及理解如下代码的执行顺序 ([出处](https://zhuanlan.zhihu.com/p/25407758)):

```javascript
setTimeout(function() {
  console.log(1)
}, 0);
new Promise(function executor(resolve) {
  console.log(2);
  for( var i=0 ; i<10000 ; i++ ) {
    i == 9999 && resolve();
  }
  console.log(3);
}).then(function() {
  console.log(4);
});
console.log(5);
```

如果你不了解这些问题, 可以自己在本地尝试研究一下打印的结果. 这里希望你掌握的是 Promise 的状态转换, 包括异步与 Promise 的关系, 以及 Promise 如何帮助你处理异步, 如果你研究过 Promise 的实现那就更好了.

## Events

`Events` 是 Node.js 中一个非常重要的 core 模块, 在 node 中有许多重要的 core API 都是依赖其建立的. 比如 `Stream` 是基于 `Events` 实现的, 而 `fs`, `net`, `http` 等模块都依赖 `Stream`, 所以 `Events` 模块的重要性可见一斑.

通过继承 EventEmitter 来使得一个类具有 node 提供的基本的 event 方法, 这样的对象可以称作 emitter, 而触发(emit)事件的 cb 则称作 listener. 与前端 DOM 树上的事件并不相同,  emitter 的触发不存在冒泡, 逐层捕获等事件行为, 也没有处理事件传递的方法.

> <a name="q-2"></a> Eventemitter 的 emit 是同步还是异步?

Node.js 中 Eventemitter 的 emit 是同步的. 在官方文档中有说明:

> The EventListener calls all listeners synchronously in the order in which they were registered. This is important to ensure the proper sequencing of events and to avoid race conditions or logic errors.

另外, 可以讨论如下的执行结果是输出 `hi 1` 还是 `hi 2`?

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('myEvent', () => {
  console.log('hi 1');
});

emitter.on('myEvent', () => {
  console.log('hi 2');
});

emitter.emit('myEvent');
```

或者如下情况是否会死循环?

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('myEvent', () => {
  console.log('hi');
  emitter.emit('myEvent');
});

emitter.emit('myEvent');
```

以及这样会不会死循环?

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('myEvent', function sth () {
  emitter.on('myEvent', sth);
  console.log('hi');
});

emitter.emit('myEvent');
```

使用 emitter 处理问题可以处理比较复杂的状态场景, 比如 TCP 的复杂状态机, 做多项异步操作的时候每一步都可能报错, 这个时候 .emit 错误并且执行某些 .once 的操作可以将你从泥沼中拯救出来.

另外可以注意一下的是, 有些同学喜欢用 emitter 来监控某些类的状态, 但是在这些类释放的时候可能会忘记释放 emitter, 而这些类的内部可能持有该 emitter 的 listener 的引用从而导致内存泄漏.

## 阻塞/异步

> <a name="q-3"></a> 如何判断接口是否异步? 是否只要有回调函数就是异步? 

开放性问题, 每个写 node 的人都有一套自己的判断方式.

* 看文档
* console.log 打印看看
* 看是否有 IO 操作

单纯使用回调函数并不会异步, IO 操作才可能会异步, 除此之外还有使用 setTimeout 等方式实现异步.

> 有这样一个场景, 你在线上使用 koa 搭建了一个网站, 这个网站项目中有一个你同事写的接口 A, 而 A 接口中在特殊情况下会变成死循环. 那么首先问题是, 如果触发了这个死循环, 会对网站造成什么影响?

Node.js 中执行 js 代码的过程是单线程的. 只有当前代码都执行完, 才会切入事件循环, 然后从事件队列中 pop 出下一个回调函数开始执行代码. 所以 ① 实现一个 sleep 函数, 只要通过一个死循环就可以阻塞整个 js 的执行流程. (关于如何避免坑爹的同事写出死循环, 在后面的测试环节有写到.)

> <a name="q-5"></a> 如何实现一个 sleep 函数? ①

```javascript
function sleep(ms) {
  var start = Date.now(), expire = start + ms;
  while (Date.now() < expire) ;
  return;
}
```

而异步, 是使用 libuv 来实现的 (C/C++的同学可以参见 libev 和 libevent) 另一个线程里的事件队列.

如果在线上的网站中出现了死循环的逻辑被触发, 整个进程就会一直卡在死循环中, 如果没有多进程部署的话, 之后的网站请求全部会超时, js 代码没有结束那么事件队列就会停下等待不会执行异步, 整个网站无法响应.

> <a name="q-6"></a> 如何实现一个异步的 reduce? (注:不是异步完了之后同步 reduce)

需要了解 reduce 的情况, 是第 n 个与 n+1 的结果异步处理完之后, 在用新的结果与第 n+2 个元素继续依次异步下去. 不贴答案, 期待诸君的版本.

## Timers

在笔者这里将 Node.js 中的异步简单的划分为两种, 硬异步和软异步. 

硬异步是指由于 IO 操作或者外部调用走 libuv 而需要异步的情况. 当然, 也存在 readFileSync, execSync 等例外情况, 不过 node 由于是单线程的, 所以如果常规业务在普通时段执行可能比较耗时同步的 IO 操作会使得其执行过程中其他的所有操作都不能响应, 有点作死的感觉. 不过在启动/初始化以及一些工具脚本的应用场景下是完全没问题的. 而一般的场景下 IO 操作都是需要异步的.

软异步是指, 通过 setTimeout 等方式来实现的异步. <a name="q-4"></a> 关于 nextTick, setTimeout 以及 setImmediate 三者的区别参见[该帖](https://cnodejs.org/topic/5556efce7cabb7b45ee6bcac)

**Event loop 示例**

```
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

关于事件循环, Timers 以及 nextTick 的关系详见官方文档 The Node.js Event Loop, Timers, and process.nextTick(): [英文](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)， [论坛中文讨论](https://cnodejs.org/topic/57d68794cb6f605d360105bf) 以及 [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

## 并行/并发

并行 (Parallel) 与并发 (Concurrent) 是两个很常见的概念.

可以看 Erlang 作者 Joe Armstrong 的博客 ([Concurrent and Parallel](http://joearms.github.io/2013/04/05/concurrent-and-parallel-programming.html))

![con_and_par](http://joearms.github.io/images/con_and_par.jpg)

并发 (Concurrent) = 2 队列对应 1 咖啡机.

并行 (Parallel) = 2 队列对应 2 咖啡机.

Node.js 通过事件循环来挨个抽取事件队列中的一个个 Task 执行, 从而避免了传统的多线程情况下 `2个队列对应 1个咖啡机` 的时候上下文切换以及资源争抢/同步的问题, 所以获得了高并发的成就.

至于在 node 中并行, 你可以通过 cluster 来再添加一个咖啡机.
