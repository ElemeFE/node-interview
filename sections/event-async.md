# 事件/异步

* `[Basic]` Promise
* [`[Doc]` Events (事件)](https://nodejs.org/dist/latest-v6.x/docs/api/events.html)
* [`[Doc]` Timers (定时器)](https://nodejs.org/dist/latest-v6.x/docs/api/timers.html)
* `[Point]` 并行
* `[Point]` 阻塞/异步

> <a name="q-1"></a> Promise 中 .then 的第二参数与 .catch 有什么区别?

参见 [We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)

> <a name="q-2"></a> Eventemitter 的 emit 是同步还是异步?

同步.

> <a name="q-3"></a> 如何判断接口是否异步? 是否只要有回调函数就是异步? 

开放性问题, 每个写 node 的人都有一套自己的判断方式.

* 看文档
* console.log 打印看看
* 看是否有IO操作

回调函数并没有异步, IO 操作才可能会异步, 除此之外还有使用 setTimeout 等方式实现异步.

> <a name="q-4"></a> nextTick, setTimeout 以及 setImmediate 三者有什么区别?

参见[该帖](https://cnodejs.org/topic/5556efce7cabb7b45ee6bcac)

> <a name="q-5"></a> 如何实现一个 sleep 函数? ①

```javascript
function sleep(ms) {
  var start = Date.now(), expire = start + ms;
  while (Date.now() < expire) ;
  return;
}
```

> <a name="q-6"></a> 如何实现一个异步的 reduce? (注:不是异步完了之后同步 reduce)

需要了解 reduce 的情况, 是第 n 个与 n+1 的结果异步处理完之后, 在用新的结果与第 n+2 个元素继续依次异步下去. 不贴答案, 期待诸君的版本.

## 简述

异步还是不异步? 这是一个问题.

## Promise

![callback-hell](https://github.com/ElemeFE/node-interview/blob/master/assets/callback-hell.png)

相信很多同学在面试的时候都碰到过这样一个问题, `如何处理 Callback Hell`. 在早些年的时候, 大家会看到有很多的解决方案例如 [Q](https://www.npmjs.com/package/q), [async](https://www.npmjs.com/package/async), [EventProxy](https://www.npmjs.com/package/eventproxy) 等等. 最后从流行程度来看 `Promise` 当之无愧的独领风骚, 并且是在 ES6 的 Javascript 标准上赢得了支持.

关于它的基础知识/概念推荐看阮一峰的 [Promise 对象](http://javascript.ruanyifeng.com/advanced/promise.html#toc9) 这里就不多不赘述, 直接来看问题.

一下是一个很简单的 Promise 的使用例子:

```javascript
let doSth = new Promise((resolve, reject) => {
  console.log('hello');
  resolve();
});

doSth.then(() => {
  console.log('over');
});
```

毫无疑问的可以得到一下输出结果:

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

如果你不了解这两个问题, 可以自己在本地尝试研究一下打印的结果. 这里希望你掌握的是 Promise 的状态转换, 以及异步与 Promise 的关系, Promise 如何帮助你处理异步, 如果你研究过 Promise 的实现那就更好了.

## Events

`Events` 是 Node.js 中一个非常重要的 core 模块, 在 node 中有许多重要的 core API 都是依赖其建立的. 比如 `Stream` 是基于 `Events` 实现的, 而 `fs`, `net`, `http` 等模块都依赖 `Stream`, 所以 `Events` 模块的重要性可见一斑.

通过继承 EventEmitter 来使得一个类具有 node 提供的基本的 event 方法, 这样的对象可以称作 emitter, 而触发(emit)事件的 cb 则称作 listener. 与前端 DOM 树上的事件并不相同,  emitter 的触发不存在冒泡, 逐层捕获等事件行为, 也没有处理事件传递的方法.

Node.js 中 Eventemitter 的 emit 是同步的. 在官方文档中有说明:

> The EventListener calls all listeners synchronously in the order in which they were registered. This is important to ensure the proper sequencing of events and to avoid race conditions or logic errors.

使用 emitter 处理问题可以处理比较复杂的状态场景, 比如 TCP 的复杂状态机, 做多项异步操作的时候每一步都可能报错, 这个时候 .emit 错误并且执行某些 .once 的操作可以将你从泥沼中拯救出来.

另外可以注意一下的是, 有些同学喜欢用 emitter 来监控某些类的状态, 但是在这些类释放的时候可能会忘记释放 emitter, 而 emitter 的 listener 可能一直这些类的内部持有其引用从而可能导致内存泄漏.

## 阻塞/异步

> 有这样一个场景, 你在线上使用 koa 搭建了一个网站, 这个网站项目中有一个你同事写的接口 A, 而 A 接口中在特殊情况下会变成死循环. 那么首先问题是, 如果触发了这个死循环, 会对网站造成什么影响?

Node.js 中执行 js 代码的过程是单线程的. 只有当前代码都执行完, 才会切入事件循环, 然后从事件队列中 pop 出下一个回调函数开始执行代码. 所以 ① 实现一个 sleep 函数, 只要通过一个死循环就可以阻塞整个 js 的执行流程. (关于如何避免坑爹的同事写出死循环, 在后面的测试环节有写到.)

而异步, 是使用 libuv 来实现的 (C/C++的同学可以参见 libev 和 libevent) 另一个线程里的事件队列.

如果在线上的网站中出现了死循环的逻辑被触发, 整个进程就会一直卡在死循环中, 如果没有多进程部署的话, 之后的网站请求全部会超时, js 代码没有结束那么事件队列就会停下等待不会执行异步, 整个网站无法响应.

整理中

## 并行/并发

整理中
