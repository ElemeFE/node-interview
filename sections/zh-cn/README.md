![ElemeFE-background](/assets/ElemeFE-background.png)

# 如何通过饿了么 Node.js 面试

Hi, 欢迎来到 ElemeFE, 如标题所示本教程的目的是教你如何通过饿了么大前端的面试, 职位是 2~3 年经验的 Node.js 服务端程序员 (并不是全栈), 如果你对这个职位感兴趣或者学习 Node.js 一些进阶的内容, 那么欢迎围观.

需要注意的是, 本文针对的并不是零基础的同学, 你需要有一定的 JavaScript/Node.js 基础, 并且有一定的工作经验. 另外本教程的重点更准确的说是服务端基础中 Node.js 程序员需要了解的部分.

如果你觉得大多不了解, 就不用投简历了 <del>(这样两边都节约了时间)</del>, 如果你觉得大都有了解或者**光看大纲都都觉得很简单那么欢迎投递简历至 ElemeFe (fe.job@ele.me)**.

## 导读

虽然说目的是要通过面试, 但是本教程并不是简单的把所有面试题列出来, 而**主要是将面试中需要确认你是否懂的点列举出来**, 并进行一定程度的讨论.

本文将一些常见的问题划分归类, 每类标明涵盖的一些`覆盖点`, 并且列举几个`常见问题`, 通常这些问题都是 2~3 年工作经验需要了解或者面对的. 如果你对某类问题感兴趣, 或者想知道其中列举问题的答案, 可以通过该类下方的 `阅读更多` 查看更多的内容.

整体上大纲列举的并不是很全面, 细节上覆盖率不高, 很多讨论只是点到即止, 希望大家带着问题去思考.

## [Js 基础问题](/sections/zh-cn/common.md)

> 与前端 Js 不同, 后端是直面服务器的, 更加偏向内存方面.

* [`[Basic]` 类型判断](/sections/zh-cn/common.md#类型判断)
* [`[Basic]` 作用域](/sections/zh-cn/common.md#作用域)
* [`[Basic]` 引用传递](/sections/zh-cn/common.md#引用传递)
* [`[Basic]` 内存释放](/sections/zh-cn/common.md#内存释放)
* [`[Basic]` ES6 新特性](/sections/zh-cn/common.md#es6-新特性)

**常见问题**

* js 中什么类型是引用传递, 什么类型是值传递? 如何将值类型的变量以引用的方式传递? [[more]](/sections/zh-cn/common.md#q-value)
* js 中， 0.1 + 0.2 === 0.3 是否为 true ? 在不知道浮点数位数时应该怎样判断两个浮点数之和与第三数是否相等？
* const 定义的 Array 中间元素能否被修改? 如果可以, 那 const 修饰对象的意义是? [[more]](/sections/zh-cn/common.md#q-const)
* JavaScript 中不同类型以及不同环境下变量的内存都是何时释放? [[more]](/sections/zh-cn/common.md#q-mem)

[阅读更多](/sections/zh-cn/common.md)

## [模块](/sections/zh-cn/module.md)

* [`[Basic]` 模块机制](/sections/zh-cn/module.md#模块机制)
* [`[Basic]` 热更新](/sections/zh-cn/module.md#热更新)
* [`[Basic]` 上下文](/sections/zh-cn/module.md#上下文)
* [`[Basic]` 包管理](/sections/zh-cn/module.md#包管理)

**常见问题**

* a.js 和 b.js 两个文件互相 require 是否会死循环? 双方是否能导出变量? 如何从设计上避免这种问题? [[more]](/sections/zh-cn/module.md#q-loop)
* 如果 a.js require 了 b.js, 那么在 b 中定义全局变量 `t = 111` 能否在 a 中直接打印出来? [[more]](/sections/zh-cn/module.md#q-global)
* 如何在不重启 node 进程的情况下热更新一个 js/json 文件? 这个问题本身是否有问题? [[more]](/sections/zh-cn/module.md#q-hot)

[阅读更多](/sections/zh-cn/module.md)

## [事件/异步](/sections/zh-cn/event-async.md)

* [`[Basic]` Promise](/sections/zh-cn/event-async.md#promise)
* [`[Doc]` Events (事件)](/sections/zh-cn/event-async.md#events)
* [`[Doc]` Timers (定时器)](/sections/zh-cn/event-async.md#timers)
* [`[Point]` 阻塞/异步](/sections/zh-cn/event-async.md#阻塞异步)
* [`[Point]` 并行/并发](/sections/zh-cn/event-async.md#并行并发)

**常见问题**

* Promise 中 .then 的第二参数与 .catch 有什么区别? [[more]](/sections/zh-cn/event-async.md#q-1)
* Eventemitter 的 emit 是同步还是异步? [[more]](/sections/zh-cn/event-async.md#q-2)
* 如何判断接口是否异步? 是否只要有回调函数就是异步? [[more]](/sections/zh-cn/event-async.md#q-3)
* nextTick, setTimeout 以及 setImmediate 三者有什么区别? [[more]](/sections/zh-cn/event-async.md#q-4)
* 如何实现一个 sleep 函数? [[more]](/sections/zh-cn/event-async.md#q-5)
* 如何实现一个异步的 reduce? (注:不是异步完了之后同步 reduce) [[more]](/sections/zh-cn/event-async.md#q-6)

[阅读更多](/sections/zh-cn/event-async.md)

## [进程](/sections/zh-cn/process.md)

* [`[Doc]` Process (进程)](/sections/zh-cn/process.md#process)
* [`[Doc]` Child Processes (子进程)](/sections/zh-cn/process.md#child-process)
* [`[Doc]` Cluster (集群)](/sections/zh-cn/process.md#cluster)
* [`[Basic]` 进程间通信](/sections/zh-cn/process.md#进程间通信)
* [`[Basic]` 守护进程](/sections/zh-cn/process.md#守护进程)

**常见问题**

* 进程的当前工作目录是什么? 有什么作用? [[more]](/sections/zh-cn/process.md#q-cwd)
* child_process.fork 与 POSIX 的 fork 有什么区别? [[more]](/sections/zh-cn/process.md#q-fork)
* 父进程或子进程的死亡是否会影响对方? 什么是孤儿进程? [[more]](/sections/zh-cn/process.md#q-child)
* cluster 是如何保证负载均衡的? [[more]](/sections/zh-cn/process.md#how-it-works)
* 什么是守护进程? 如何实现守护进程? [[more]](/sections/zh-cn/process.md#守护进程)

[阅读更多](/sections/zh-cn/process.md)


## [IO](/sections/zh-cn/io.md)

* [`[Doc]` Buffer](/sections/zh-cn/io.md#buffer)
* [`[Doc]` String Decoder (字符串解码)](/sections/zh-cn/io.md#string-decoder)
* [`[Doc]` Stream (流)](/sections/zh-cn/io.md#stream)
* [`[Doc]` Console (控制台)](/sections/zh-cn/io.md#console)
* [`[Doc]` File System (文件系统)](/sections/zh-cn/io.md#file)
* [`[Doc]` Readline](/sections/zh-cn/io.md#readline)
* [`[Doc]` REPL](/sections/zh-cn/io.md#repl)

**常见问题**

* Buffer 一般用于处理什么数据? 其长度能否动态变化? [[more]](/sections/zh-cn/io.md#buffer)
* Stream 的 highWaterMark 与 drain 事件是什么? 二者之间的关系是? [[more]](/sections/zh-cn/io.md#缓冲区)
* Stream 的 pipe 的作用是? 在 pipe 的过程中数据是引用传递还是拷贝传递? [[more]](/sections/zh-cn/io.md#pipe)
* 什么是文件描述符? 输入流/输出流/错误流是什么? [[more]](/sections/zh-cn/io.md#file)
* console.log 是同步还是异步? 如何实现一个 console.log? [[more]](/sections/zh-cn/io.md#console)
* 如何同步的获取用户的输入?  [[more]](/sections/zh-cn/io.md#如何同步的获取用户的输入)
* Readline 是如何实现的? (有思路即可) [[more]](/sections/zh-cn/io.md#readline)

[阅读更多](/sections/zh-cn/io.md)

## [Network](/sections/zh-cn/network.md)

* [`[Doc]` Net (网络)](/sections/zh-cn/network.md#net)
* [`[Doc]` UDP/Datagram](/sections/zh-cn/network.md#udp)
* [`[Doc]` HTTP](/sections/zh-cn/network.md#http)
* [`[Doc]` DNS (域名服务器)](/sections/zh-cn/network.md#dns)
* [`[Doc]` ZLIB (压缩)](/sections/zh-cn/network.md#zlib)
* [`[Point]` RPC](/sections/zh-cn/network.md#rpc)

**常见问题**

* cookie 与 session 的区别? 服务端如何清除 cookie? [[more]](/sections/zh-cn/network.md#q-cookie-session)
* HTTP 协议中的 POST 和 PUT 有什么区别? [[more]](/sections/zh-cn/network.md#q-post-put)
* 什么是跨域请求? 如何允许跨域? [[more]](/sections/zh-cn/network.md#q-cors)
* TCP/UDP 的区别? TCP 粘包是怎么回事，如何处理? UDP 有粘包吗? [[more]](/sections/zh-cn/network.md#q-tcp-udp)
* `TIME_WAIT` 是什么情况? 出现过多的 `TIME_WAIT` 可能是什么原因? [[more]](/sections/zh-cn/network.md#q-time-wait)
* ECONNRESET 是什么错误? 如何复现这个错误?
* socket hang up 是什么意思? 可能在什么情况下出现? [[more]](/sections/zh-cn/network.md#socket-hang-up)
* hosts 文件是什么? 什么叫 DNS 本地解析?
* 列举几个提高网络传输速度的办法?

[阅读更多](/sections/zh-cn/network.md)

## [OS](/sections/zh-cn/os.md)

* [`[Doc]` TTY](/sections/zh-cn/os.md#tty)
* [`[Doc]` OS (操作系统)](/sections/zh-cn/os.md#os-1)
* [`[Doc]` Path](/sections/zh-cn/os.md#path)
* [`[Doc]` 命令行参数](/sections/zh-cn/os.md#命令行参数)
* [`[Basic]` 负载](/sections/zh-cn/os.md#负载)
* [`[Point]` CheckList](/sections/zh-cn/os.md#checklist)

**常见问题**

* 什么是 TTY? 如何判断是否处于 TTY 环境? [[more]](/sections/zh-cn/os.md#tty)
* 不同操作系统的换行符 (EOL) 有什么区别? [[more]](/sections/zh-cn/os.md#os)
* 服务器负载是什么概念? 如何查看负载? [[more]](/sections/zh-cn/os.md#负载)
* ulimit 是用来干什么的? [[more]](/sections/zh-cn/os.md#ulimit)

[阅读更多](/sections/zh-cn/os.md)

## [错误处理/调试](/sections/zh-cn/error.md)

* [`[Doc]` Errors (异常)](/sections/zh-cn/error.md#errors)
* [`[Doc]` Domain (域)](/sections/zh-cn/error.md#domain)
* [`[Doc]` Debugger (调试器)](/sections/zh-cn/error.md#debugger)
* [`[Doc]` C/C++ 插件](/sections/zh-cn/error.md#c-c++-addon)
* [`[Doc]` V8](/sections/zh-cn/error.md#v8)
* [`[Point]` 内存快照](/sections/zh-cn/error.md#内存快照)
* [`[Point]` CPU profiling](/sections/zh-cn/error.md#cpu-profiling)

**常见问题**

* 怎么处理未预料的出错? 用 try/catch ，domains 还是其它什么? [[more]](/sections/zh-cn/error.md#q-handle-error)
* 什么是 `uncaughtException` 事件? 一般在什么情况下使用该事件? [[more]](/sections/zh-cn/error.md#uncaughtexception)
* domain 的原理是? 为什么要弃用 domain? [[more]](/sections/zh-cn/error.md#domain)
* 什么是防御性编程? 与其相对的 let it crash 又是什么?
* 为什么要在 cb 的第一参数传 error? 为什么有的 cb 第一个参数不是 error, 例如 http.createServer?
* 为什么有些异常没法根据报错信息定位到代码调用? 如何准确的定位一个异常? [[more]](/sections/zh-cn/error.md#错误栈丢失)
* 内存泄漏通常由哪些原因导致? 如何分析以及定位内存泄漏? [[more]](/sections/zh-cn/error.md#内存快照)

[阅读更多](/sections/zh-cn/error.md)

## [测试](/sections/zh-cn/test.md)

* [`[Basic]` 测试方法](/sections/zh-cn/test.md#测试方法)
* [`[Basic]` 单元测试](/sections/zh-cn/test.md#单元测试)
* [`[Basic]` 集成测试](/sections/zh-cn/test.md#集成测试)
* [`[Basic]` 基准测试](/sections/zh-cn/test.md#基准测试)
* [`[Basic]` 压力测试](/sections/zh-cn/test.md#压力测试)
* [`[Doc]` Assert (断言)](/sections/zh-cn/test.md#assert)

**常见问题**

* 为什么要写测试? 写测试是否会拖累开发进度?[[more]](/sections/zh-cn/test.md#q-why-write-test)
* 单元测试的单元是指什么? 什么是覆盖率?[[more]](/sections/zh-cn/test.md#单元测试)
* 测试是如何保证业务逻辑中不会出现死循环的?[[more]](/sections/zh-cn/test.md#q-death-loop)
* mock 是什么? 一般在什么情况下 mock?[[more]](/sections/zh-cn/test.md#mock)

[阅读更多](/sections/zh-cn/test.md)

## [util](/sections/zh-cn/util.md)

* [`[Doc]` URL](/sections/zh-cn/util.md#url)
* [`[Doc]` Query Strings (查询字符串)](/sections/zh-cn/util.md#query-strings)
* [`[Doc]` Utilities (实用函数)](/sections/zh-cn/util.md#util-1)
* [`[Basic]` 正则表达式](/sections/zh-cn/util.md#正则表达式)

**常见问题**

* HTTP 如何通过 GET 方法 (URL) 传递 let arr = [1,2,3,4] 给服务器? [[more]](/sections/zh-cn/util.md#get-param)
* Node.js 中继承 (util.inherits) 的实现? [[more]](/sections/zh-cn/util.md#utilinherits)
* 如何递归获取某个文件夹下所有的文件名? [[more]](/sections/zh-cn/util.md#q-traversal)

[阅读更多](/sections/zh-cn/util.md)

## [存储](/sections/zh-cn/storage.md)

* [`[Point]` Mysql](/sections/zh-cn/storage.md#mysql)
* [`[Point]` Mongodb](/sections/zh-cn/storage.md#mongodb)
* [`[Point]` Replication](/sections/zh-cn/storage.md#replication)
* [`[Point]` 数据一致性](/sections/zh-cn/storage.md#数据一致性)
* [`[Point]` 缓存](/sections/zh-cn/storage.md#缓存)

**常见问题**

* 备份数据库与 M/S, M/M 等部署方式的区别? [[more]](/sections/zh-cn/storage.md#replication)
* 索引有什么用，大致原理是什么? 设计索引有什么注意点? [[more]](/sections/zh-cn/storage.md#索引)
* Monogdb 连接问题(超时/断开等)有可能是什么问题导致的? [[more]](/sections/zh-cn/storage.md#Mongodb)
* 什么情况下数据会出现脏数据? 如何避免? [[more]](/sections/zh-cn/storage.md#数据一致性)
* redis 与 memcached 的区别? [[more]](/sections/zh-cn/storage.md#缓存)

[阅读更多](/sections/zh-cn/storage.md)

## [安全](/sections/zh-cn/security.md)

* [`[Doc]` Crypto (加密)](/sections/zh-cn/security.md#crypto)
* [`[Doc]` TLS/SSL](/sections/zh-cn/security.md#tlsssl)
* [`[Doc]` HTTPS](/sections/zh-cn/security.md#https)
* [`[Point]` XSS](/sections/zh-cn/security.md#xss)
* [`[Point]` CSRF](/sections/zh-cn/security.md#csrf)
* [`[Point]` 中间人攻击](/sections/zh-cn/security.md#中间人攻击)
* [`[Point]` Sql/Nosql 注入](/sections/zh-cn/security.md#sqlnosql-注入)

**常见问题**

* 加密是如何保证用户密码的安全性? [[more]](/sections/zh-cn/security.md#crypto)
* TLS 与 SSL 有什么区别? [[more]](/sections/zh-cn/security.md#tlsssl)
* HTTPS 能否被劫持? [[more]](/sections/zh-cn/security.md#https)
* XSS 攻击是什么? 有什么危害? [[more]](/sections/zh-cn/security.md#xss)
* 过滤 Html 标签能否防止 XSS? 请列举不能的情况? [[more]](/sections/zh-cn/security.md#xss)
* CSRF 是什么? 如何防范? [[more]](/sections/zh-cn/security.md#csrf)
* 如何避免中间人攻击? [[more]](/sections/zh-cn/security.md#中间人攻击)

[阅读更多](/sections/zh-cn/security.md)

## 最后

目前 repo 处于施工现场的情况，如果发现问题欢迎在 [issues](https://github.com/ElemeFE/node-interview/issues) 中指出。如果有比较好的问题/知识点/指正，也欢迎提 PR。

另外关于 Js 基础 是个比较大的话题，在本教程不会很细致深入的讨论，更多的是列出一些重要或者跟服务端更相关的地方，所以如果你拿着《JavaScript 权威指南》给教程提 PR 可能不会采纳。本教程的重点更准确的说是服务端基础中 Node.js 程序员需要了解的部分。
