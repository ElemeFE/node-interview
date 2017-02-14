# 进程

* `[Doc]` Process (进程)
* `[Doc]` Child Processes (子进程)
* `[Doc]` Cluster (集群)
* `[Basic]` 进程间通信
* `[Basic]` 守护进程

> 进程的当前工作目录是什么? 有什么作用?

当前进程启动的目录, 通过 process.cwd() 获取, 通常是命令行启动的时候所在的目录 (也可以在启动时指定), 文件操作等使用相对路径的时候会相对当前工作目录来获取文件.

> fork 是什么操作?

fork 是拷贝当前进程的状态来 clone 一个新的进程, 这种进程被称为子进程.

> 父进程或子进程的死亡是否会影响对方? 什么是僵死进程?

子进程死亡不会影响父进程, 不过 node 中父进程会收到子进程死亡的信号. 反之父进程死亡, 子进程也会跟着死亡, 如果子进程没有随之终止而继续存在的状态, 被称作僵死进程.

> 什么是守护进程? 如何实现守护进程?

守护进程是不依赖终端（tty）的进程，不会因为用户退出终端而停止运行的进程。普通的进程，在用户退出终端之后就会直接关闭。通过 & 启动到后台的进程，之后会由于会话（session组）被回收而终止进程。实现可以参见 [Nodejs编写守护进程](https://cnodejs.org/topic/57adfadf476898b472247eac), 面试的时候能说清楚原理就行了.

## 简述

关于 Process, 我们需要讨论的是两个概念, ①操作系统的进程, ② Node.js 中的 Process 对象. 操作进程对于服务端而言, 好比 html 之于前端一样基础. 想做服务端编程是不可能绕过 Unix/Linux 的. 在 Linux/Unix/Mac 系统中运行 `ps -ef` 命令可以看到当前系统中运行的进程. 各个参数如下:

|列名称|意义|
|-----|---|
|UID|执行该进程的用户ID|
|PID|进程编号|
|PPID|该进程的父进程编号|
|C|该进程所在的CPU利用率|
|STIME|进程执行时间|
|TTY|进程相关的终端类型|
|TIME|进程所占用的CPU时间|
|CMD|创建该进程的指令|

关于进程以及操作系统一些更深入的细节推荐阅读 APUE, 即《Unix 高级编程》等书籍来了解.

## Process

这里来讨论 Node.js 中的 `process` 对象. 直接在代码中通过 `console.log(process)` 即可打印出来. 可以看到 process 对象暴露了非常多有用的属性以及方法, 具体的细节见[官方文档](https://nodejs.org/dist/latest-v6.x/docs/api/process.html), 已经说的挺详细了. 其中包括但不限于:

* 进程基础信息
* 进程 Usage
* 进程级事件
* 依赖模块/版本信息
* OS 基础信息
* 账户信息
* 信号收发
* 三个标准流

### process.nextTick

上一节已经提到过 `process.nextTick` 了, 这是一个你需要了解的, 重要的, 基础方法.


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

`process.nextTick` 并不属于 Event loop 中的某一个阶段, 而是在 Event loop 的每一个阶段结束后, 直接执行 `nextTickQueue` 中插入的 "Tick", 并且直到整个 Queue 处理完. 所以面试时又有可以问的问题了, 递归调用 process.nextTick 会怎么样?

![](http://imgcdn.thecover.cn/FguJbLPxBE8-xlpM9TgO-Ed0OqFx?imageMogr2/quality/80/ignore-error/1)

```javascript
function test() { 
  process.nextTick(() => test());
}
```

### 配置

配置是开发部署中一个很常见的问题. 普通的配置有两种方式, 一是定义配置文件, 二是使用环境变量.

![node-configuration](https://blog-assets.risingstack.com/2016/Sep/node-js-survey/node-js-survey-envvar-config-new.png)

你可以通过[设置环境变量](http://cn.bing.com/search?q=linux+%E8%AE%BE%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)来指定配置, 然后通过 `process.env` 来获取配置项. 另外也可以通过读取定义好的配置文件来获取, 在这方面有很多不错的库例如 `dotenv`, `node-config` 等, 而在使用这些库来加载配置文件的时候, 通常都会碰到一个当前工作目录的问题.

通过 `process.cwd()` 获取当前目录 (current working directory), 一些获取配置的第三方模块就是通过你的当前目录来找配置文件的. 所以如果你错误的目录启动脚本, 可能没法得到正确的结果. 在程序中可以通过 `process.chdir()` 来改变当前的工作目录.

### 标准流

在 process 对象上还暴露了 `process.stderr`, `process.stdout` 以及 `process.stdin` 三个标准流, 熟悉 C/C++/Java 的同学应该对此比较熟悉. 关于这几个流, 常见的面试问题是问 **console.log 是同步还是异步? 如何实现一个 console.log?**

如果简历中有出现 C/C++ 关键字, 一般都会问到如何实现一个同步的输入 (类似实现C语言的 `scanf`, C++ 的 `cin`, Python 的 `raw_input` 等).

### 维护方面

熟悉与进程有关的基础命令, 如 top, ps, pstree 等命令.

## Child Process

子进程 (Child Process) 是进程中一个重要的概念. 你可以通过 Node.js 的 `child_process` 模块来执行可执行文件, 调用命令行命令, 比如其他语言的程序等. 也可以通过该模块来将 .js 代码以子进程的方式启动. 比较有名的网易的分布式架构 [pomelo](https://github.com/NetEase/pomelo) 就是基于该模块 (而不是 `cluster`) 来实现多进程分布式架构的.

Node.js 的 `child_process.fork()` 不像 POSIX [fork(2)](http://man7.org/linux/man-pages/man2/fork.2.html) 系统调用, 不会拷贝当前父进程. 这里对于其他语言转过的同学可能比较误导, 可以作为一个比较偏的面试题.

### child.kill 与 child.send

常见会问的面试题, 如 `child.kill` 与 `child.send` 的区别. 二者一个是基于信号系统, 一个是基于 IPC.

中李忠

## Cluster

Cluster 是常见的 Node.js 利用多核的办法. 它是基于 `child_process.fork()` 实现的, 所以 cluster 产生的进程之间是通过 IPC 来通信的, 并且它也没有拷贝父进程的空间, 而是通过加入 cluster.isMaster 这个标识, 来区分父进程以及子进程, 达到类似 POSIX 的 fork 的效果.

整理中

## 进程间通信

Node.js 的进程是有 IPC 频道 来产生了, 当 IPC 频道 close 就会触发 `disconnect` 事件.

而后是关于进程间通信 (IPC) 的问题, 一般不会直接问 IPC 的实现, 而是会问什么情况下需要 IPC, 使用 IPC 处理过什么业务场景等.

整理中

## 守护进程

最后的守护进程, 是服务端方面一个很基础的概念了. 很多人可能只知道通过 pm2 之类的工具可以将进程以守护进程的方式启动, 却不了解什么是守护进程, 为什么要用守护进程. 对于水平好的同学, 我们是希望能了解守护进程的实现的.

守护进程是不依赖终端（tty）的进程，不会因为用户退出终端而停止运行的进程。

普通的进程，在用户退出终端之后就会直接关闭。通过 & 启动到后台的进程，之后会由于会话（session组）被回收而终止进程。

```c
// 守护进程初始化函数
void init_daemon()
{
    pid_t pid;
    int i = 0;

    if ((pid = fork()) == -1) {
        printf("Fork error !\n");
        exit(1);
    }

    if (pid != 0) {
        exit(0);        // 父进程退出
    }

    setsid();           // 子进程开启新会话，并成为会话首进程和组长进程
    if ((pid = fork()) == -1) {
        printf("Fork error !\n");
        exit(-1);
    }
    if (pid != 0) {
        exit(0);        // 结束第一子进程，第二子进程不再是会话首进程
                        // 避免当前会话组重新与tty连接
    }
    chdir("/tmp");      // 改变工作目录
    umask(0);           // 重设文件掩码
    for (; i < getdtablesize(); ++i) {
       close(i);        // 关闭打开的文件描述符
    }

    return;
}
```

[Node.js 编写守护进程](https://cnodejs.org/topic/57adfadf476898b472247eac)


