![ElemeFE-background](../../assets/ElemeFE-background.png)

## Guide


## [Common](/sections/en-us/common.md)

> It's much more diff between frontend and backend.

* `[Common]` Type judgment
* `[Common]` Scope
* `[Common]` Reference
* `[Common]` Memory release
* `[Common]` ES6+ features

**Common Problem**

[View more](/sections/en-us/common.md)

## [Module](/sections/en-us/module.md)

* `[Common]` Module
* `[Common]` Hotfix
* `[Common]` Context
* `[Common]` Package Manager

**Common Problem**


[View more](/sections/en-us/module.md)

## [Event & Async](/sections/en-us/event-async.md)

* `[Common]` Promise
* `[Doc]` Events
* `[Doc]` Timers
* `[Bonus]` Blocking & Non-blocking
* `[Bonus]` Parallel & Concurrent

**Common Problem**

* What's the difference between the Promise's second argument of '.then' function and '.catch' function? [[more]](/sections/en-us/event-async.md#q-1)
* Is the Eventemitter.emit synchronous or asynchronous? [[more]](/sections/en-us/event-async.md#q-2)
* How to judge whether a interface is asynchronous? Is it asynchronous while a callback provided? [[more]](/sections/en-us/event-async.md#q-3)
* Diff among nextTick, setTimeout and setImmediate? [[more]](/sections/en-us/event-async.md#q-4)
* How to implement a Sleep function? [[more]](/sections/en-us/event-async.md#q-5)
* How to implement an async.reduce? [[more]](/sections/en-us/event-async.md#q-6)

[View more](/sections/en-us/event-async.md)

## [Process](/sections/en-us/process.md)

* `[Doc]` Process
* `[Doc]` Child Processes
* `[Doc]` Cluster
* `[Bonus]` IPC
* `[Bonus]` Daemon

**Common Problem**

* What's the current working directory of the process? What's it for? [[more]](/sections/en-us/process.md#q-cwd)
* Difference between child_process.fork and fork in POSIX? [[more]](/sections/en-us/process.md#q-fork)
* Does the death of parent process or child process  affect each other? What is an orphan process? [[more]](/sections/en-us/process.md#q-child)
* How does the cluster load balance work? [[more]](/sections/en-us/process.md#how-it-works)
* What's daemon process? how to implement? [[more]](/sections/en-us/process.md#daemon-process)

[View more](/sections/en-us/process.md)


## [IO](/sections/en-us/io.md)

* `[Doc]` Buffer
* `[Doc]` String Decoder
* `[Doc]` Stream
* `[Doc]` Console
* `[Doc]` File System
* `[Doc]` Readline
* `[Doc]` REPL

**Common Problem**

* What does Buffer for? Can we change the buffer's size? [[more]](/sections/en-us/io.md#buffer)
* What's the highWaterMark & drain event of Stream? What's their relation? [[more]](/sections/en-us/io.md#buffer-2)
* What's Stream.pipe for? Is it make copy or pass object while piping? [[more]](/sections/en-us/io.md#pipe)
* What is stdio, stdout, stderr and file descriptor? [[more]](/sections/en-us/io.md#file)
* Is console.log asynchronous? How to implement console.log? [[more]](/sections/en-us/io.md#console)
* How to get user input synchronously?  [[more]](/sections/en-us/io.md#how-to-get-user-input-synchronizely)
* How to implement 'Readline'? [[more]](/sections/en-us/io.md#readline)

[View more](/sections/en-us/io.md)

## [Network](/sections/en-us/network.md)

* `[Doc]` Net
* `[Doc]` UDP/Datagram
* `[Doc]` HTTP
* `[Doc]` DNS
* `[Doc]` ZLIB
* `[Common]` RPC

**Common Problem**


[View more](/sections/en-us/network.md)

## [OS](/sections/en-us/os.md)

* `[Doc]` TTY
* `[Doc]` OS
* `[Doc]` Command Line Options
* `[Common]` Load
* `[Bonus]` CheckList
* `[Common]` Indicators

**Common Problem**

* What's TTY? How to check if terminal is TTY? [[more]](/sections/en-us/os.md#tty)
* Is there different among operating system's EOL(end of line)? [[more]](/sections/en-us/os.md#os)
* What is system load? how to check it? [[more]](/sections/en-us/os.md#load)
* What's ulimit for? [[more]](/sections/en-us/os.md#ulimit)

[View more](/sections/en-us/os.md)

## [Error handle & Debug](/sections/en-us/error.md)

* `[Doc]` Errors
* `[Doc]` Domain
* `[Doc]` Debugger
* `[Doc]` C/C++ Addon
* `[Doc]` V8
* `[Bonus]` Memory snapshot
* `[Bonus]` CPU Profilling

**Common Problem**

* How to handle unexpected errors? With try/catch, domains or something eles? [[more]](/sections/en-us/error.md#q-handle-error)
* What is `uncaughtException` event? when shoud we use it? [[more]](/sections/en-us/error.md#uncaughtexception)
* What is domain's principle? why domain is deprecated? [[more]](/sections/en-us/error.md#domain)
* What's defensive programming? how about 'let it crash'?
* Why we need error-first callback? why there are callback not error-first, such as http.createServer?
* Why there are errors can't location? how to locate accurately? [[more]](/sections/en-us/error.md#error-stack-is-missing)
* What cause memory leak? how to locate and analyse it? [[more]](/sections/en-us/error.md#memory-snapshots)

[View more](/sections/en-us/error.md)

## [Test](/sections/en-us/test.md)

* `[Common]` Methods
* `[Common]` Unit Test
* `[Common]` Benchmarks
* `[Common]` Integration Test
* `[Common]` Pressure Test
* `[Doc]` Assert

**Common Problem**


[View more](/sections/en-us/test.md)

## [Util](/sections/en-us/util.md)

* `[Doc]` URL
* `[Doc]` Query Strings
* `[Doc]` Utilities
* `[Common]` Regex

**Common Problem**

* How does HTTP pass `let arr = [1,2,3,4]` to the server by GET method? [[more]](/sections/en-us/util.md#get-param)
* How to implement util.inherits in Node.js? [[more]](/sections/en-us/util.md#utilinherits)
* How do I get all the file names under a folder? [[more]](/sections/en-us/util.md#q-traversal)

[View more](/sections/en-us/util.md)

## [Storage](/sections/en-us/storage.md)

* `[Common]` Sql
* `[Common]` NoSql
* `[Bonus]` Cache
* `[Bonus]` Consistency

**Common Problem**


[View more](/sections/en-us/storage.md)

## [Security](/sections/en-us/security.md)

* `[Doc]` Crypto
* `[Doc]` TLS/SSL
* `[Doc]` HTTPS
* `[Bonus]` XSS
* `[Bonus]` CSRF
* `[Bonus]` MITM
* `[Bonus]` Sql/Nosql Injection

**Common Problem**


[View more](/sections/en-us/security.md)

## Final

Current repo is translating, you can report on [issues](https://github.com/ElemeFE/node-interview/issues) freely if there is typo or reading problem.
