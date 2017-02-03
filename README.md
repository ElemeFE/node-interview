![ElemeFE-background](https://github.com/ElemeFE/node-interview/blob/master/assets/ElemeFE-background.png)

# 如何通过饿了么 Node.js 面试

Hi, 欢迎来到 ElemeFE, 如标题所示本教程的目的是教你如何通过饿了么大前端的面试, 职位是 2~3 年经验的 Node.js 服务端程序员, 如果你对这个职位感兴趣或者想了解一下相关情况，那么欢迎阅读.

需要注意的是, 本文针对的并不是零基础的同学, 你需要有一定的 JavaScript/Node 基础, 并且有一定的工作经验.

如果你觉得大多不了解, 就不用投简历了 <del>(这样两边都节约了时间)</del>, 如果你觉得大都有了解或者<font color="blue">光看大纲都都觉得很简单那么欢迎投递简历至 ElemeFe (fe.job@ele.me)</font>.

### 导读

虽然说目的是要通过面试, 但是本教程并不是简单的把所有面试题列出来, 而<font color="red">主要是将面试中需要确认你是否懂的点列举出来</font>, 并进行一定程度的讨论. 本文将一些常见的问题划分归类, 每类标明涵盖的一些`覆盖点`, 并且列举几个`常见问题`, 通常这些问题都是 2~3 年工作经验需要了解或者面对的. 如果你对某类问题感兴趣, 或者想知道其中列举问题的答案, 可以通过该类下方的 `阅读更多` 查看更多的内容.

大纲列举的并不是很全面, 细节上覆盖率不高, 很多讨论只是点到即止, 就是希望大家带着问题去思考.

## [Js 基础问题](https://github.com/ElemeFE/node-interview/blob/master/sections/js-basic.md)

### 覆盖点

* `[Basic]` 类型判断
* `[Basic]` 作用域
* `[Basic]` 引用传递
* `[Basic]` 内存释放
* `[Basic]` ES6 新特性

### 常见问题

* js 中什么类型是引用传递, 什么类型是值传递? 如何将值类型的变量以引用的方式传递?
* js 中， 0.1 + 0.2 === 0.3 是否为 true ? 在不知道浮点数位数时应该怎样判断两个浮点数之和与第三数是否相等？
* const 定义的 Array 中间元素能否被修改? 如果可以, 那 const 的意义是?
* Javascript 中不同类型以及不同环境下变量的内存都是何时释放?

<font color="blue">[阅读更多](https://github.com/ElemeFE/node-interview/blob/master/sections/js-basic.md)</font>

## [node 基础问题](https://github.com/ElemeFE/node-interview/blob/master/sections/node-basic.md)

> 与前端 Js 不同, 后端是直面服务器的, 更加偏向内存方面, 对于一些更基础的问题也会更加关注.

* [`[Doc]` Modules (模块)](https://nodejs.org/dist/latest-v6.x/docs/api/modules.html)
* [`[Doc]` Globals (全局变量)](https://nodejs.org/dist/latest-v6.x/docs/api/globals.html)
* [`[Doc]` VM (虚拟机)](https://nodejs.org/dist/latest-v6.x/docs/api/vm.html)

### 常见问题

* a.js 和 b.js 两个文件互相 require 是否会死循环? 双方是否能导出变量? 如何从设计上避免这种问题?
* 如果 a.js require 了 b.js, 那么在 b 中定义全局变量 `t = 111` 能否在 a 中直接打印出来?
* 如何在不重启 node 进程的情况下热更新一个 js/json 文件? 这个问题本身是否有问题?

<font color="blue">[阅读更多](https://github.com/ElemeFE/node-interview/blob/master/sections/node-basic.md)</font>

## 事件/异步

* `[Basic]` Promise
* `[Doc]` Events (事件)
* `[Doc]` Timers (定时器)
* `[Point]` 并行
* `[Point]` 阻塞/异步

### 常见问题

* Promise 中 .then 的第二参数与 .catch 有什么区别?
* Eventemitter 的 emit 是同步还是异步?
* 如何判断接口是否异步? 是否只要有回调函数就是异步?
* nextTick, setTimeout 以及 setImmediate 三者有什么区别?
* 如何实现一个 sleep 函数?
* 如何实现一个异步的 reduce? (注:不是异步完了之后同步 reduce)

`更多整理中`

## 进程

* `[Doc]` Process (进程)
* `[Doc]` Child Processes (子进程)
* `[Doc]` Cluster (集群)
* `[Basic]` 进程间通信
* `[Basic]` 守护进程

### 常见问题

* 进程的执行目录是什么? 有什么作用?
* fork 是什么操作? 什么是子进程, 僵死进程?
* 父进程或子进程的死亡是否会影响对方?
* 什么是守护进程? 如何实现守护进程?

`更多整理中`

## IO

* `[Doc]` Stream (流)
* `[Doc]` Buffer
* `[Doc]` String Decoder (字符串解码)
* `[Doc]` Console (控制台)
* `[Doc]` File System (文件系统)
* `[Doc]` Readline
* `[Doc]` REPL

### 常见问题

* Stream 的 pipe 是如何使用? 在 pipe 的过程中数据是引用传递还是拷贝传递?
* 什么是文件句柄? 输入流/输出流/错误流是什么?
* console.log 是同步还是异步? 如何实现一个 console.log?
* Readline 是如何实现的? 如何实现一个同步的 Readline?

`更多整理中`

## Network

### 常见问题

* HTTP 协议中的 POST 和 PUT 有什么区别?
* `TCP/UDP` 的区别? `TCP` 粘包是怎么回事，如何处理? `UDP` 有粘包吗?
* `time_wait` 是什么情况?出现过多的 `close_wait` 可能是什么原因?
* socket hang up 是什么意思? 一般什么情况下出现?
* 列举几个提高网络传输速度的办法?

`更多整理中`

## OS

* `[Doc]` TTY
* `[Doc]` OS (操作系统)
* `[Doc]` 命令行参数
* `[Basic]` 负载
* `[Basic]` 指标
* `[Point]` CheckList

### 常见问题

* 服务器负载是什么概念? 如何计算负载?
* ulimit 是用来干什么的?

`更多整理中`

## 错误处理/调试/优化

* `[Doc]` Errors (异常)
* `[Doc]` Domain (域)
* `[Doc]` Debugger (调试器)
* `[Doc]` C/C++ 插件
* `[Doc]` V8
* `[Point]` 内存快照
* `[Point]` CPU剖析

### 常见问题

* 怎么处理未预料的出错?用 try/catch ，domains 还是其它什么?
* domain 的原理是? 为什么要弃用 domain?
* 为什么要在 cb 的第一参数传 error? 为什么有的 cb 第一个参数不是 error, 例如 http.createServer?

`更多整理中`

## 测试

* `[Basic]` 单元测试
* `[Basic]` 基准测试
* `[Basic]` 集成测试
* `[Doc]` Assert (断言)
* `[Module]` 常见测试工具
* `[Module]` 常见断言工具

### 常见问题

* 为什么要写测试? 写测试是否会拖累开发进度?
* 单元测试的单元是指什么? 什么是覆盖率?
* 测试是如何保证业务逻辑中不会出现死循环的?
* mock 是指什么? 一般如何实现?

`更多整理中`

## util

* `[Doc]` URL
* `[Doc]` Path (路径)
* `[Doc]` Utilities (实用函数)
* `[Doc]` Query Strings (查询字符串)
* `[Basic]` 正则表达式

* 如何获取某个文件夹下所有的文件名?

`更多整理中`

## 存储

* `[Point]` Sql
* `[Point]` NoSql
* `[Point]` 缓存
* `[Point]` 数据一致性

### 常见问题

* 索引有什么用，大致原理是什么?设计索引有什么注意点?
* Session/Cookie 有什么区别?
* 连接超时有可能是什么问题导致的?
* 什么情况下数据会出现脏读? 如何避免?

`更多整理中`

## 安全

* `[Point]` XSS
* `[Point]` CSRF
* `[Point]` 中间人攻击
* `[Point]` Sql/Nosql 注入攻击
* `[Doc]` Crypto (加密)

### 常见问题

* CSRF 的攻击和防范方法?
* 加密如何保证用户密码的安全性?
* 如何避免中间人攻击?

`更多整理中`
