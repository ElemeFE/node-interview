# 错误处理/调试

* `[Doc]` Errors (异常)
* `[Doc]` Domain (域)
* `[Doc]` Debugger (调试器)
* `[Doc]` C/C++ 插件
* `[Doc]` V8
* `[Point]` 内存快照
* `[Point]` CPU剖析


## Errors

在 Node.js 中的错误主要有一下四种类型：

|错误|名称|触发|
|---|---|---|
|Standard JavaScript errors|标准 JavaScript 错误|由错误代码触发|
|System errors|系统错误|由操作系统触发|
|User-specified errors|用户自定义错误|通过 throw 抛出|
|Assertion errors|断言错误|由 `assert` 模块触发|

其中标准的 JavaScript 错误常见有：

* [EvalError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/EvalError): 调用 eval() 出现错误时抛出该错误
* [SyntaxError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SyntaxError): 代码不符合 JavaScript 语法规范时抛出该错误
* [RangeError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RangeError): 数组越界时抛出该错误
* [ReferenceError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ReferenceError): 引用未定义的变量时抛出该错误
* [TypeError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError): 参数类型错误时抛出该错误
* [URIError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/URIError): 误用全局的 URI 处理函数时抛出该错误

而常见的系统错误列表可以通过 Node.js 的 os 对象常看列表：

```javascript
const os = require('os');

console.log(os.constants.errno);
```

目前搜索 Node.js 面试题, 发现很多题目已经跟不上 Node.js 的发展了.比较老的 [NodeJS 错误处理最佳实践](https://cnodejs.org/topic/55714dfac4e7fbea6e9a2e5d), 译自 Joyent 的官方博客, 其中有这样的描述:

> 实际上, `try/catch` 唯一常用的是在 `JSON.parse` 和类似验证用户输入的地方

然而实际上现在在 Node.js 中你已经可以轻松的使用 try/catch 去捕获异步的异常了. 并且在 Node.js v7.6 之后使用了升级引擎的新版 v8, 旧版中 try/catch 代码不能优化的问题也解决了. 所以我们现在再来看

> <a name="q-handle-error"></a> 怎么处理未预料的出错? 用 try/catch , domains 还是其它什么?

在 Node.js 中错误处理主要有一下几种方法:

* callback(err, data) 回调约定
* throw / try / catch 
* EventEmitter 的 error 事件

callback(err, data) 这种形式的错误处理起来繁琐, 并不具备强制性, 目前已经处于仅需要了解, 不推荐使用的情况. 而 domain 模块则是半只脚踏进棺材了.

1) 感谢 [co](https://github.com/visionmedia/co) 的先河, 现在的你已经简单的使用 try/catch 保护关键的位置, 以 koa 为例, 可以通过中间件的形式来进行错误处理, 详见 [Koa error handling](https://github.com/koajs/koa/wiki/Error-Handling). 之后的 async/await 均属于这种模式.

2) 通过 EventEmitter 的错误监听形式为各大关键的对象加上错误监听的回调. 例如监听 http server, tcp server 等对象的 `error` 事件以及 process 对象提供的 `uncaughtException` 和 `unhandledRejection` 等等.

3) 使用 Promise 来封装异步, 并通过 Promise 的错误处理来 handle 错误.

4) 如果上述办法不能起到良好的作用, 那么你需要学习如何优雅的 [Let It Crash](http://wiki.c2.com/?LetItCrash)

> 为什么要在 cb 的第一参数传 error? 为什么有的 cb 第一个参数不是 error, 例如 http.createServer?

TODO


### 错误栈丢失

```javascript
function test() {
  throw new Error('test error');
}

function main() {  
  test();
}

main();
```

可以收获报错:

```javascript
/data/node-interview/error.js:2
  throw new Error('test error');
  ^

Error: test error
    at test (/data/node-interview/error.js:2:9)
    at main (/data/node-interview/error.js:6:3)
    at Object.<anonymous> (/data/node-interview/error.js:9:1)
    at Module._compile (module.js:570:32)
    at Object.Module._extensions..js (module.js:579:10)
    at Module.load (module.js:487:32)
    at tryModuleLoad (module.js:446:12)
    at Function.Module._load (module.js:438:3)
    at Module.runMain (module.js:604:10)
    at run (bootstrap_node.js:394:7)
```

可以发现报错的行数, test 函数, main 函数的调用关系都在 stack 中清晰的体现.

当你使用 setImmediate 等定时器来设置异步的时候:

```javascript
function test() {
  throw new Error('test error');
}

function main() {  
  setImmediate(() => test());
}

main();

```

我们发现

```javascript
/data/node-interview/error.js:2
  throw new Error('test error');
  ^

Error: test error
    at test (/data/node-interview/error.js:2:9)
    at Immediate.setImmediate (/data/node-interview/error.js:6:22)
    at runCallback (timers.js:637:20)
    at tryOnImmediate (timers.js:610:5)
    at processImmediate [as _immediateCallback] (timers.js:582:5)
```

错误栈中仅输出到 test 函数内调用的地方位置, 再往上 main 的调用信息就丢失了. 也就是说如果你的函数调用深度比较深的情况下, 你使用异步调用某个函数出错了的情况下追溯这个异步的调用是一个很困难的事情, 因为其之上的栈都已经丢失了. 如果你用过 [async](https://github.com/caolan/async) 之类的模块, 你还可能发现, 报错的 stack 会非常的长而且曲折, 光看 stack 很难去定位问题.

这在项目不大/作者清楚的情况下不是问题, 但是当项目大起来, 开发人员多起来之后, 这样追溯错误会变得异常痛苦. 关于这个问题, 在上文中提到 [错误处理的最佳实践](https://cnodejs.org/topic/55714dfac4e7fbea6e9a2e5d) 中, 关于 `编写新函数的具体建议` 那一带的内容有描述到. 通过使用 [verror](https://www.npmjs.com/package/verror) 这样的方式, 让 Error 一层层封装, 并在每一层将错误的信息一层层的包上, 最后拿到的 Error 直接可以从 message 中获取用于定位问题的关键信息.

以昨天的数据为准（2017-3-13）各位只要对比一下看看 npm 上上个月 [verror](https://www.npmjs.com/package/verror) 的下载量 `1100w` 比 [express](https://www.npmjs.com/package/express) 的 `1070w` 还高. 应该就能感受到这种写法有多流行了.

### 防御性编程

错误并不可怕, 可怕的是你不去准备应对错误————[防御性编程的介绍和技巧](http://blog.jobbole.com/101651/)

### let it crash

[Let It Crash](http://wiki.c2.com/?LetItCrash)

### uncaughtException

当异常没有被捕获一路冒泡到 Event Loop 时就会触发该事件 process 对象上的 `uncaughtException` 事件. 默认情况下, Node.js 对于此类异常会直接将其堆栈跟踪信息输出给 `stderr` 并结束进程, 而为 `uncaughtException` 事件添加监听可以覆盖该默认行为, 不会直接结束进程.

```javascript
process.on('uncaughtException', (err) => {
  console.log(`Caught exception: ${err}`);
});

setTimeout(() => {
  console.log('This will still run.');
}, 500);

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
console.log('This will not run.');
```

#### 合理使用 uncaughtException

`uncaughtException` 的初衷是可以让你拿到错误之后可以做一些回收处理之后再 process.exit. 官方的同志们还曾经讨论过要移除该事件 (详见 [issues](https://github.com/nodejs/node-v0.x-archive/issues/2582))

所以你需要明白 `uncaughtException` 其实已经是非常规手段了, 应尽量避免使用它来处理错误. 因为通过该事件捕获到错误后, 并不代表 `你可以愉快的继续运行 (On Error Resume Next)`. 程序内部存在未处理的异常, 这意味着应用程序处于一种未知的状态. 如果不能适当的恢复其状态, 那么很有可能会触发不可预见的问题. (使用 domain 会很夸张的加剧这个现象, 并产生新人不能理解的各类幽灵问题)

如果在 `.on` 指定的监听回调中报错不会被捕获, Node.js 的进程会直接终端并返回一个非零的退出码, 最后输出相应的堆栈信息. 否则, 会出现无限递归. 除此之外, 内存崩溃/底层报错等情况也不会被捕获, **目前猜测**是 v8/C++ 那边撂担子不干了, Node.js 完全插不上话导致的 (TODO 整理到这里才想起来这个念头尚未验证, 如果有空的朋友帮忙验证下).

所以官方建议的使用 `uncaughtException` 的正确姿势是在结束进程前使用同步的方式清理已使用的资源 (文件描述符、句柄等) 然后 process.exit. 

在 uncaughtException 事件之后执行普通的恢复操作并不安全. 官方建议是另外在专门准备一个 monitor 进程来做健康检查并通过 monitor 来管理恢复情况, 并在必要的时候重启 (所以官方是含蓄的提醒各位用 pm2 之类的工具).


### unhandledRejection

当 Promise 被 reject 且没有绑定监听处理时, 就会触发该事件. 该事件对排查和追踪没有处理 reject 行为的 Promise 很有用.

该事件的回调函数接收以下参数：

* `reason` [`<Error>`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) | `<any>` 该 Promise 被 reject 的对象 (通常为 Error 对象)
* `p` 被 reject 的 Promise 本身

例如

```javascript
process.on('unhandledRejection', (reason, p) => {
  console.log('Unhandled Rejection at: Promise', p, 'reason:', reason);
  // application specific logging, throwing an error, or other logic here
});

somePromise.then((res) => {
  return reportToUser(JSON.pasre(res)); // note the typo (`pasre`)
}); // no `.catch` or `.then`
```

以下代码也会触发 `unhandledRejection` 事件：

```javascript
function SomeResource() {
  // Initially set the loaded status to a rejected promise
  this.loaded = Promise.reject(new Error('Resource not yet loaded!'));
}

var resource = new SomeResource();
// no .catch or .then on resource.loaded for at least a turn
```

> In this example case, it is possible to track the rejection as a developer error as would typically be the case for other 'unhandledRejection' events. To address such failures, a non-operational `.catch(() => { })` handler may be attached to resource.loaded, which would prevent the 'unhandledRejection' event from being emitted. Alternatively, the 'rejectionHandled' event may be used.


## Domain

Node.js 早期, try/catch 无法捕获异步的错误, 而错误优先的 callback 仅仅是一种约定并没有强制性并且写起来十分繁琐. 所以为了能够很好的捕获异常, Node.js 从 v0.8 开始引入 domain 这个模块.

domain 本身是一个 EventEmitter 对象, 其中文意思是 "域" 的意思, 捕获异步异常的基本思路是创建一个域, cb 函数会在定义时会继承上一层的域, 报错通过当前域的 `.emit('error', err)` 方法触发错误事件将错误传递上去, 从而使得异步错误可以被强制捕获. (更多内容详见 [Node.js 异步异常的处理与domain模块解析](https://cnodejs.org/topic/516b64596d38277306407936))

但是 domain 的引入也带来了更多新的问题. 比如依赖的模块无法继承你定义的 domain, 导致你写的 domain 无法 cover 依赖模块报错. 而且, 很多人 (特别是新人) 由于不了解 Node.js 的内存/异步流程等问题, 在使用 domain 处理报错的时候, 没有做到完善的处理并盲目的让代码继续走下去, 这很可能导致**项目完全无法维护** (可能出现的问题真是不胜枚举, 各种梦魇...)

该模块目前的情况: [deprecate domains](https://github.com/nodejs/node/issues/66)


## Debugger

![node-js-survey-debug](/assets/node-js-survey-debug.png)

类似 gdb 的命令行下 debug 工具 (上图中的 build-in debugger), 同时也支持远程 debug (类似 [node-inspector](https://github.com/node-inspector/node-inspector), 目前处于试验状态). 当然, 目前有不少同学觉得 [vscode](https://code.visualstudio.com/) 对 debug 工具集成的比较好.

关于这个 build-in debugger 使用推荐看[官方文档](https://nodejs.org/dist/latest-v6.x/docs/api/debugger.html). 如果要深入一点, 你可能对本文感兴趣: [动态修改 NodeJS 程序中的变量值](http://code.oneapm.com/nodejs/2015/06/27/intereference/)


## C/C++ Addon

在 Node.js 中开发 addon 最痛苦的地方莫过于升级 V8 导致的 C/C++ 代码不能兼容的问题, 这个问题在很早就出现了. 为了解决这个问题前人开了一个叫 [nan](https://github.com/nodejs/nan) 的项目.

要学习 addon 开发, 除了[官方文档](https://nodejs.org/docs/latest/api/addons.html)也推荐阅读这个: https://github.com/nodejs/node-addon-examples


## V8

这里并不是介绍 V8, 而是介绍 Node.js 中的 V8 这个模块. 该模块用于开放 Node.js 内建的 V8 引擎的事件和接口. 这些接口由 V8 底层决定, 所以无法保证绝对的稳定性.

|接口|描述|
|---|---|
|v8.getHeapStatistics()|获取 heap 信息|
|v8.getHeapSpaceStatistics()|获取 heap space 信息|
|v8.setFlagsFromString(string)|动态设置 V8 options|

### v8.setFlagsFromString(string)

该方法用于添加额外的 V8 命令行标志. 该方法需谨慎使用, 在 VM 启动后修改配置可能会发生不可预测的行为、崩溃和数据丢失; 或者什么反应都没有.

通过 `node --v8-options` 命令可以查询当前 Node.js 环境中有哪些可用的 V8 options. 此外, 还可以参考非官方维护的一个 [V8 options 列表](https://github.com/thlorenz/v8-flags/blob/master/flags-0.11.md).

用法:

```javascript
// Print GC events to stdout for one minute.
const v8 = require('v8');
v8.setFlagsFromString('--trace_gc');
setTimeout(function() { v8.setFlagsFromString('--notrace_gc'); }, 60e3);
```

## 内存快照

内存快照常用与解决内存泄漏的问题. 快照工具推荐使用 [heapdump](https://github.com/bnoordhuis/node-heapdump) 用来保存内存快照, 使用 [devtool](https://github.com/Jam3/devtool) 来查看内存快照. 使用 heapdump 保存内存快照时, 只会有 Node.js 环境中的对象, 不会受到干扰（如果使用 [node-inspector](https://github.com/node-inspector/node-inspector) 的话, 快照中会有前端的变量干扰）.

使用以及内存泄漏的常见原因详见: [如何分析 Node.js 中的内存泄漏](https://zhuanlan.zhihu.com/p/25736931?group_id=825001468703674368).

## CPU profiling

CPU profiling (剖析) 常用于性能优化. 有许多用于做 profiling 的第三方工具, 但是大部分情况下, 使用 Node.js 内置的是最简单的. 其内置调用的就是 [V8 本身的 profiler](https://github.com/v8/v8/wiki/Using%20V8%E2%80%99s%20internal%20profiler), 它可以在程序执行过程中中是对 stack 间隔性的抽样分析.

使用 `--prof` 开启内置的 profilling

```shell
node --prof app.js
```

程序运行之后会生成一个 `isolate-0xnnnnnnnnnnnn-v8.log` 在当前运行目录. 

你可以使用 `--prof-process` 来生成报告查看

```
node --prof-process isolate-0xnnnnnnnnnnnn-v8.log
```

报告形如:

```
Statistical profiling result from isolate-0x103001200-v8.log, (12042 ticks, 2634 unaccounted, 0 excluded).

 [Shared libraries]:
   ticks  total  nonlib   name
     35    0.3%          /usr/lib/system/libsystem_platform.dylib
     27    0.2%          /usr/lib/system/libsystem_pthread.dylib
      7    0.1%          /usr/lib/system/libsystem_c.dylib
      3    0.0%          /usr/lib/system/libsystem_kernel.dylib
      1    0.0%          /usr/lib/system/libsystem_malloc.dylib

 [JavaScript]:
   ticks  total  nonlib   name
    208    1.7%    1.7%  Stub: LoadICStub
    187    1.6%    1.6%  KeyedLoadIC: A keyed load IC from the snapshot
    104    0.9%    0.9%  Stub: VectorStoreICStub
     69    0.6%    0.6%  LazyCompile: *emit events.js:136:44
     68    0.6%    0.6%  Builtin: CallFunction_ReceiverIsNotNullOrUndefined
     65    0.5%    0.5%  KeyedStoreIC: A keyed store IC from the snapshot {2}
     47    0.4%    0.4%  Builtin: CallFunction_ReceiverIsAny
     43    0.4%    0.4%  LazyCompile: *storeHeader _http_outgoing.js:312:21
     34    0.3%    0.3%  LazyCompile: *removeListener events.js:315:28
     33    0.3%    0.3%  Stub: RegExpExecStub
     33    0.3%    0.3%  LazyCompile: *_addListener events.js:210:22
     32    0.3%    0.3%  Stub: CEntryStub
     32    0.3%    0.3%  Builtin: ArgumentsAdaptorTrampoline
     31    0.3%    0.3%  Stub: FastNewClosureStub
     30    0.2%    0.3%  Stub: InstanceOfStub
     ...

 [C++]:
   ticks  total  nonlib   name
    460    3.8%    3.8%  _mach_port_extract_member
    329    2.7%    2.7%  _openat$NOCANCEL
    199    1.7%    1.7%  ___bsdthread_register
    136    1.1%    1.1%  ___mkdir_extended
    116    1.0%    1.0%  node::HandleWrap::Close(v8::FunctionCallbackInfo<v8::Value> const&)
    112    0.9%    0.9%  void v8::internal::BodyDescriptorBase::IterateBodyImpl<v8::internal::StaticScavengeVisitor>(v8::internal::Heap*, v8::internal::HeapObject*, int, int)
    106    0.9%    0.9%  _http_parser_execute
    103    0.9%    0.9%  _szone_malloc_should_clear
     99    0.8%    0.8%  int v8::internal::BinarySearch<(v8::internal::SearchMode)1, v8::internal::DescriptorArray>(v8::internal::DescriptorArray*, v8::internal::Name*, int, int*)
     89    0.7%    0.7%  node::TCPWrap::Connect(v8::FunctionCallbackInfo<v8::Value> const&)
     86    0.7%    0.7%  v8::internal::LookupIterator::State v8::internal::LookupIterator::LookupInRegularHolder<false>(v8::internal::Map*, v8::internal::JSReceiver*)
     ...

 [Bottom up (heavy) profile]:
  Note: percentage shows a share of a particular caller in the total
  amount of its parent calls.
  Callers occupying less than 2.0% are not shown.

   ticks parent  name
   2634   21.9%  UNKNOWN
    764   29.0%    LazyCompile: *connect net.js:815:17
    764  100.0%      LazyCompile: ~<anonymous> net.js:966:30
    764  100.0%        LazyCompile: *_tickCallback internal/process/next_tick.js:87:25
    193    7.3%    LazyCompile: *createWriteReq net.js:732:24
    101   52.3%      LazyCompile: *Socket._writeGeneric net.js:660:42
     99   98.0%        LazyCompile: ~<anonymous> net.js:667:34
     99  100.0%          LazyCompile: ~g events.js:287:13
     99  100.0%            LazyCompile: *emit events.js:136:44
     92   47.7%      LazyCompile: ~Socket._writeGeneric net.js:660:42
     91   98.9%        LazyCompile: ~<anonymous> net.js:667:34
     91  100.0%          LazyCompile: ~g events.js:287:13
     91  100.0%            LazyCompile: *emit events.js:136:44
	 ...
```

|字段|描述|
|---|---|
|ticks|时间片|
|total|当前操作执行的时间占总时间的比率|
|nonlib|当前非 System library 执行时间比率|

整理中

