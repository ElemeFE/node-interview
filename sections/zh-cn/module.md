# 模块

* [`[Basic]` 模块机制](#模块机制)
* [`[Basic]` 热更新](#热更新)
* [`[Basic]` 上下文](#上下文)
* [`[Basic]` 包管理](#包管理)

## 常见问题


> <a name="q-hot"></a> 如何在不重启 node 进程的情况下热更新一个 js/json 文件? 这个问题本身是否有问题?

可以清除掉 `require.cache` 的缓存重新 `require(xxx)`, 视具体情况还可以用 VM 模块重新执行.

当然这个问题可能是典型的 [`X-Y Problem`](http://coolshell.cn/articles/10804.html), 使用 js 实现热更新很容易碰到 v8 优化之后各地拿到缓存的引用导致热更新 js 没意义. 当然热更新 json 还是可以简单一点比如用读取文件的方式来热更新, 但是这样也不如从 redis 之类的数据库中读取比较合理.

## 简述

其他还有很多内容也是属于很 '基础' 的 Node.js 问题 (例如异步/线程等等), 但是由于归类的问题并没有放在这个分类中. 所以这里只简单讲几个之后没归类的基础问题.


## 模块机制

node 的基础中毫无疑问的应该是有关于模块机制的方面的, 也即 `require` 这个内置功能的一些原理的问题.

关于模块互相引用之类的, 不了解的推荐先好好读读[官方文档](https://nodejs.org/dist/latest-v6.x/docs/api/modules.html).

其实官方文档已经说得很清楚了, 每个 node 进程只有一个 VM 的上下文, 不会跟浏览器相差多少, 模块机制在文档中也描述的非常清楚了:

```javascript
function require(...) {
  var module = { exports: {} };
  ((module, exports) => {
    // Your module code here. In this example, define a function.
    function some_func() {};
    exports = some_func;
    // At this point, exports is no longer a shortcut to module.exports, and
    // this module will still export an empty default object.
    module.exports = some_func;
    // At this point, the module will now export some_func, instead of the
    // default object.
  })(module, module.exports);
  return module.exports;
}
```

> <a name="q-global"></a> 如果 a.js require 了 b.js, 那么在 b 中定义全局变量 `t = 111` 能否在 a 中直接打印出来?

① 每个 `.js` 能独立一个环境只是因为 node 帮你在外层包了一圈自执行, 所以你使用 `t = 111` 定义全局变量在其他地方当然能拿到. 情况如下:

```javascript

// b.js
(function (exports, require, module, __filename, __dirname) {
  t = 111;
})();

// a.js
(function (exports, require, module, __filename, __dirname) {
  // ...
  console.log(t); // 111
})();
```

> <a name="q-loop"></a> a.js 和 b.js 两个文件互相 require 是否会死循环? 双方是否能导出变量? 如何从设计上避免这种问题?

② 不会, 先执行的导出其 **未完成的副本**, 通过导出工厂函数让对方从函数去拿比较好避免. 模块在导出的只是 `var module = { exports: {...} };` 中的 exports, 以从 a.js 启动为例, a.js 还没执行完会返回一个 a.js 的 exports 对象的 **未完成的副本** 给 b.js 模块。 然后 b.js 完成加载，并将 exports 对象提供给 a.js 模块。

另外还有非常基础和常见的问题, 比如 module.exports 和 exports 的区别这里也能一并解决了 exports 只是 module.exports 的一个引用. 没看懂可以在细看我以前发的[帖子](https://cnodejs.org/topic/5734017ac3e4ef7657ab1215).

再晋级一点, 众所周知, node 的模块机制是基于 [`CommonJS`](http://javascript.ruanyifeng.com/nodejs/module.html) 规范的. 对于从前端转 node 的同学, 如果面试官想问的难一点会考验关于 [`CommonJS`](http://javascript.ruanyifeng.com/nodejs/module.html) 的一些问题. 比如比较 `AMD`, `CMD`, [`CommonJS`](http://javascript.ruanyifeng.com/nodejs/module.html) 三者的区别, 包括询问关于 node 中 `require` 的实现原理等.

## 热更新

从面试官的角度看, `热更新` 是很多程序常见的问题. 对客户端而言, 热更新意味着不用换包, 当然也包含着 md5 校验/差异更新等复杂问题; 对服务端而言, 热更新意味着服务不用重启, 这样可用性较高<del>同时也优雅和有逼格</del>. 问的过程中可以一定程度的暴露应聘程序员的水平.

从 PHP 转 node 的同学可能会有些想法, 比如 PHP 的代码直接刷上去就好了, 并没有所谓的重启. 而 node 重启看起来动作还挺大. 当然这里面的区别, 主要是与同时有 PHP 与 node 开发经验的同学可以讨论, 也是很好的切入点.

在 Node.js 中做热更新代码, 牵扯到的知识点可能主要是 `require` 会有一个 `cache`, 有这个 `cache` 在, 即使你更新了 `.js` 文件, 在代码中再次 `require` 还是会拿到之前的编译好缓存在 v8 内存 (code space) 中的的旧代码. 但是如果只是单纯的清除掉 `require` 中的 `cache`, 再次 `require` 确实能拿到新的代码, 但是这时候很容易碰到各地维持旧的引用依旧跑的旧的代码的问题. 如果还要继续推行这种热更新代码的话, 可能要推翻当前的架构, 从头开始从新设计一下目前的框架.

不过热更新 json 之类的配置文件的话, 还是可以简单的实现的, 更新 `require` 的 `cache` 可以实现, 不会有持有旧引用的问题, 可以参见我 2 年前写着玩的[例子](https://www.npmjs.com/package/auto-reload), 但是如果旧的引用一直被持有很容易出现内存泄漏, 而要热更新配置的话, 为什么不存数据库? 或者用 `zookeeper` 之类的服务? 通过更新文件还要再发布一次, 但是存数据库直接写个接口配个界面多爽你说是不是?

所以这个问题其实本身其实是值得商榷的, 可能是典型的 [`X-Y Problem`](http://coolshell.cn/articles/10804.html), 不过聊起来确实是可以暴露水平.

## 上下文

如果你已经了解 ①② 那么你也应该了解, 对于 Node.js 而言, 正常情况下只有一个上下文, 甚至于内置的很多方面例如 `require` 的实现只是在启动的时候运行了[内置的函数](https://github.com/nodejs/node/tree/master/lib). 

每个单独的 `.js` 文件并不意味着单独的上下文, 在某个 `.js` 文件中污染了全局的作用域一样能影响到其他的地方.

而目前的 Node.js 将 VM 的接口暴露了出来, 可以让你自己创建一个新的 js 上下文, 这一点上跟前端 js 还是区别挺大的. 在执行外部代码的时候, 通过创建新的上下文沙盒 (sandbox) 可以避免上下文被污染:

```javascript
'use strict';
const vm = require('vm');

let code =
`(function(require) {

  const http = require('http');

  http.createServer( (request, response) => {
    response.writeHead(200, {'Content-Type': 'text/plain'});
    response.end('Hello World\\n');
  }).listen(8124);

  console.log('Server running at http://127.0.0.1:8124/');
})`;

vm.runInThisContext(code)(require);
```

这种执行方式与 eval 和 Function 有明显的区别. 关于 VM 更多的一些接口可以先阅读[官方文档 VM (虚拟机)](https://nodejs.org/dist/latest-v6.x/docs/api/vm.html)

讲完这个知识点, 这里留下一个简单的问题, 既然可以通过新的上下文来避免污染, 那么`为什么 Node.js 不给每一个 `.js` 文件以独立的上下文来避免作用域被污染?` <del>(反应不过来的同学还是别投简历了, 微笑脸)</del>


## 包管理


整理中...

为什么我装了全局, 但是提示我 not found

npm
yarn

锁版本

lerna：一个用户管理多个包模块的工具。

left-pad事件

greenkeeper 等
