# Process

* `[Doc]` Process
* `[Doc]` Child Processes
* `[Doc]` Cluster
* `[Basic]` IPC
* `[Basic]` Daemon

## Introduction

For Process, we will discuss two concepts,① the process of the operating system, ② the Process object in Node.js. Operation process is basis for the server-side just like Html for the Front-end. No one can do server-side programming without Unix/Linux. Excuting `ps -ef` command in Linux/Unix/Mac system, and you will see the running processes of the current system. 
Each parameter is as follows:

|Column name|Meaning|
|-----|---|
|UID|User ID of the process's owner|
|PID|Process ID number|
|PPID|ID number of the process's parent process|
|C|CPU usage|
|STIME|Time when the process started|
|TTY|Terminal associated with the process|
|TIME|Total CPU time used by the process since it started|
|CMD|Command and arguments for the process|

For more details about the process and the operating system, you can read the APUE(Advanced Programming in the UNIX® Environment).

## Process


Here we will discuss the `process` object in Node.js. It can be printed out by using `console.log (process)` in the code. You can see the process object exposed a lot of useful properties and methods. For more details you can refer [Official document](https://nodejs.org/dist/latest-v6.x/docs/api/process.html), which has been very detailed, 
including but not limited to:

* The basic information of the process
* The usage of the process
* Process Events
* Dependencies/versions
* The basic information of the operating system platform
* The information of the user
* Signal Events
* The three standard streams

### process.nextTick

The previous chapter has already mentioned `process.nextTick`, which is an important, basic method you have to know.

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

`process.nextTick` is not technically part of the event loop. Instead, the `nextTickQueue` will be processed after the current operation completes, regardless of the current phase of the event loop. So the question is coming, what will happen if you call process.nextTick recursively?(doge

```javascript
function test() { 
  process.nextTick(() => test());
}
```

What is the difference between this situation and the following? Why?

```javascript
function test() { 
  setTimeout(() => test(), 0);
}
```

### Configuration

Configuration is a very common problem in development deployments. As usual, there are two ways for configuration, one is to define configuration file, another is to  to use the environment variables.

 ![node-configuration](https://blog-assets.risingstack.com/2016/Sep/node-js-survey/node-js-survey-envvar-config-new.png)

You can specify the configuration by [Setting Environment Variables](http://cn.bing.com/search?q=linux+Setting+Environment+Variable&qs=n&form=QBRE&sp=-1&pq=linux+setting+environment+variable&sc=1-34&sk=&cvid=1027E58E457E42DEB5A4A4E495EEC4A9), then obtain the configuration item by using `process.env`. In addition, you can obtain by reading the configuration file. There are many excellent libraries such as `Dotenv`, ` node-config`, etc. in this field. But when loading the configuration file by using these libraries, it usually encounters a problem with the current working directory.

> <a name="q-cwd"></a> What's the current working directory of the process? What's it for?

You can obtain the current working directory by using `process.cwd()`. It usually is the directory when the command line starts. It can also be specified at startup. File operations, etc. obtain the file by using the relative path which is relative to the current working directory.

Some of the third-party modules that can obtain the configuration look for the configuration file through your current directory. So if running the start script in the wrong directory, you will get wrong rusults. You can change working directory by using `process.chdir()` in your code.

### Standard Stream

The process object also exposes three standard stream, `process.stderr`, `process.stdout`, `process.stdin`. If you have used C/C++/Java before, you will not feel unfamiliar with this. So the common interview questions come: **Is `console.log` synchronous or asynchronous? How to implement a `console.log`?**

If there are keywords such as C/C++ in your resume, you will be asked how to implement a synchronous input(similar to the `scanf` in the C, `cin` in C++, `raw_input` in Python, etc.).

## Maintaining

Familiar with basic commands about process, such as top, ps, pstree , etc.

## Child Process

Child Process is an important concept in the process.  In Node.js, you can use `child_process` module to execute executable files, call commands in command line , such as programs in other languages, etc. You can also execute js code as a sub-process by using this module. The well-known Netease's distributed architecture [pomelo](https://github.com/NetEase/pomelo) is based on the module(not `cluster`) to implement the multi-process distributed architecture.

> <a name="q-fork"></a> What're the difference between child_process.fork and fork in POSIX?

In Node.js, `child_process.fork()` calls POSIX [fork(2)](http://man7.org/linux/man-pages/man2/fork.2.html). You need manually manage the release of resources in the child process for fork POSIX. You don't need to care about this problem when using `child_process.fork`, beacuse Node.js will automatic release, and provide options whether the child process can survive after the parent process is destroyed. 

* spawn() - Spawns a child process to execute the command
  * options.detached - Whether the child process can survive after the parent process is destroyed
  * options.stdio - Configure the three pipes that are established between the parent and child process
* spawnSync() - A synchronous version of spawn().
 You can set the timeout, and obtain the child process by the return object
* exec() - Spawns a child process to execute the command, with the callback parameters to get information of the child process. You can set the timeout for the process
* execSync() - A synchronous version of exec(). You can set the timeout. It returns stdout of the child process
* execFile() - Spawns a child process to execute an executable file. You can set the timeout for the process
* execFileSync() - A synchronous version of execFile(). It returns stdout of the child process. It will throw Error if it is timeout or its exit code is not 0
* fork() - Enhanced version of spawn(). It returns a child process object and allows sending messages between parent and child.

The exec/execSync method will directly call bash to explain the command. So if there are external parameters of the command, you need to pay attention to the situation was injected.

### child.kill and child.send

The common interview question is what are the differences between `child.kill` and `child.send`. 
One is based on the signal system, the other is based on IPC.

> <a name="q-child"></a> Does the death of parent process or child process  affect each other? What is an orphan process?

The death of a child process will not affect the parent process. When the child process dies (the last thread of the thread group, usually when the "lead" thread dies), it will send a death signal to its parent process. On the other hand, when the parent process dies, by default, the child process will follow the death. But at this time, if the child process is in the operational state, dead state, etc., it will be adopted by process identifier 1(the init system process) and become an orphaned process. In addition, when the child process dies("terminated" state), the parent process does not call `wait()` or `waitpid()` to return the child's infomation in time, there is a `PCB` remaining in the process table. The child process is called a zombie process.

## Cluster

Cluster is a common way to use multi-core systems in Node.js. It is based on `child_process.fork ()` implementation. For this reason, the worker processes can communicate with the parent via IPC, and do not copy parent's memory space. You can distinguish between the parent process and the child process by adding `cluster.isMaster`, making it  
similar to the [fork](http://man7.org/linux/man-pages/man2/fork.2.html) in POSIX.

```javascript
const cluster = require('cluster');            // | | 
const http = require('http');                  // | | 
const numCPUs = require('os').cpus().length;   // | |    Both executed
                                               // | | 
if (cluster.isMaster) {                        // |-|-----------------
  // Fork workers.                             //   | 
  for (var i = 0; i < numCPUs; i++) {          //   | 
    cluster.fork();                            //   | 
  }                                            //   |   Only the parent process is executed (a.js)
  cluster.on('exit', (worker) => {             //   | 
    console.log(`${worker.process.pid} died`); //   | 
  });                                          //   |
} else {                                       // |-------------------
  // Workers can share any TCP connection      // | 
  // In this case it is an HTTP server         // | 
  http.createServer((req, res) => {            // | 
    res.writeHead(200);                        // |   Only the child process is executed (b.js)
    res.end('hello world\n');                  // | 
  }).listen(8000);                             // | 
}                                              // |-------------------
                                               // | |
console.log('hello');                          // | |    Both executed
```

In the code above numCPUs is a global variable. However,  it will not change in the child process when modified in the parent process, because the child process and the parent process run in separate memory spaces. The so-called shared is that they both run, but in separate memory spaces. 

The execution of the parent process can be seen as `a.js`, and the execution of the child process seen as `b.js`. You can imagine that it executes `node a.js` first, and then `cluster.fork` several times(execute `node b.js` several times). The cluster module is a bridge between them. They two can communicate between each other by using methods provided by cluster. 

### How It Works

The worker processes are spawned using the child_process.fork() method, so that they can communicate with the parent via IPC and pass server handles back and forth.

The cluster module supports two methods of distributing incoming connections.

The first one (and the default one on all platforms except Windows), is the round-robin approach, where the master process listens on a port, accepts new connections and distributes them across the workers in a round-robin fashion, with some built-in smarts to avoid overloading a worker process.

The second approach is where the master process creates the listen socket and sends it to interested workers. The workers then accept incoming connections directly.

The second approach should, in theory, give the best performance. In practice however, distribution tends to be very unbalanced due to operating system scheduler vagaries. Loads have been observed where over 70% of all connections ended up in just two processes, out of a total of eight.


## IPC(Inter-process communication)

Inter-process communication techniques can be divided into various types. These are:

Type|Without Connnection|Stability|Flow Control|Priority
---|-----|----|-----|-----
Pipe|N|Y|Y|N
Named Pipe|N|Y|Y|N
Message Queues|N|Y|Y|N
Semaphores|N|Y|Y|Y
Shared Memory|N|Y|Y|Y
UNIX Stream Socket|N|Y|Y|N
UNIX Datagram Socket|Y|Y|N|N

IPC in Node.js is implemented through Pipe based on libuv. It is implemented by Named Pipe(the second item in the list above) in windows, and UDS (Unix Domain Socket) in *nix.

Ordinary socket is designed for network communications, which itself is unreliable. But the socket for the IPC is not the case, because local network environment is reliable by default. So you can simplify much unnecessary encode/decode and calculate the verification, etc., get more efficient UDS communication.


If understanding the IPC in Node.js, you will be asked an interesting question:

> <a name="q-ipc-fd"></a> Before IPC channel was 
set up, how the parent process and the child process 
communicate between each other? If there is no communication, how is IPC set up?

This question is very simple, just a problem of thinking. When you create a child process via child_process, you can specify the env (environment variable) of the child process. When starting the child process in Node.js, the main process sets up the IPC channel first, then pass the fd(file descriptor) of the IPC channel to the child process via environment variable (`NODE_CHANNEL_FD`). Then the child process connects to the parent process via fd.

Finally, for the issue of inter-process communication (IPC), we generally do not directly ask the IPC implementation, but will ask under what conditions you need IPC, and the use of IPC to deal with any business scene.

## Daemon Process

Daemon Process is a very basic concept of the server side. Many people may only know that we can start a process as a daemon by using tools such as pm2, but not what is a process and why using it. For excellent guys, daemon process implement should be known.

The normal process will be directly shut down after the user exits the terminal. The Process starting with `&` and running in the background will be shut down when the session (session group) is released. The daemon process is not dependent on the terminal(tty) process and will not be shut down because of the user exiting the terminal.

```c
// Daemon Process Implement (Written in C)
void init_daemon()
{
    pid_t pid;
    int i = 0;

    if ((pid = fork()) == -1) {
        printf("Fork error !\n");
        exit(1);
    }

    if (pid != 0) {
        exit(0);        // parent process exits
    }

    setsid();           // the child process opens a new session and becomes the session header and the process group leader
    if ((pid = fork()) == -1) {
        printf("Fork error !\n");
        exit(-1);
    }
    if (pid != 0) {
        exit(0);        // End the first process, and the second process is not the session header any more.
                        // avoid the current session group to re-connect with the tty
    }
    chdir("/tmp");      // change the working directory
    umask(0);           // reset the file umask
    for (; i < getdtablesize(); ++i) {
       close(i);        // close the file descriptor
    }

    return;
}
```

[Code for Daemon Process in Node.js](https://cnodejs.org/topic/57adfadf476898b472247eac)
