# IO

* [`[Doc]` Buffer](/sections/zh-cn/io.md#buffer)
* [`[Doc]` String Decoder (字符串解码)](/sections/zh-cn/io.md#string-decoder)
* [`[Doc]` Stream (流)](/sections/zh-cn/io.md#stream)
* [`[Doc]` Console (控制台)](/sections/zh-cn/io.md#console)
* [`[Doc]` File System (文件系统)](/sections/zh-cn/io.md#file)
* [`[Doc]` Readline](/sections/zh-cn/io.md#readline)
* [`[Doc]` REPL](/sections/zh-cn/io.md#repl)

# 简述

Node.js 是以 IO 密集型业务著称. 那么问题来了, 你真的了解什么叫 IO, 什么又叫 IO 密集型业务吗?

## Buffer

Buffer 是 Node.js 中用于处理二进制数据的类, 其中与 IO 相关的操作 (网络/文件等) 均基于 Buffer. Buffer 类的实例非常类似整数数组, ***但其大小是固定不变的***, 并且其内存在 V8 堆栈外分配原始内存空间. Buffer 类的实例创建之后, 其所占用的内存大小就不能再进行调整.

在 Node.js v6.x 之后 `new Buffer()` 接口开始被废弃, 理由是参数类型不同会返回不同类型的 Buffer 对象, 所以当开发者没有正确校验参数或没有正确初始化 Buffer 对象的内容时, 以及不了解的情况下初始化  就会在不经意间向代码中引入安全性和可靠性问题.

接口|用途
---|---
Buffer.from()|根据已有数据生成一个 Buffer 对象
Buffer.alloc()|创建一个初始化后的 Buffer 对象
Buffer.allocUnsafe()|创建一个未初始化的 Buffer 对象

### TypedArray

Node.js 的 Buffer 在 ES6 增加了 TypedArray 类型之后, 修改了原来的 Buffer 的实现, 选择基于 TypedArray 中 Uint8Array 来实现, 从而提升了一波性能.

使用上, 你需要了解如下情况:

```javascript
const arr = new Uint16Array(2);
arr[0] = 5000;
arr[1] = 4000;

const buf1 = Buffer.from(arr); // 拷贝了该 buffer
const buf2 = Buffer.from(arr.buffer); // 与该数组共享了内存

console.log(buf1);
// 输出: <Buffer 88 a0>, 拷贝的 buffer 只有两个元素
console.log(buf2);
// 输出: <Buffer 88 13 a0 0f>

arr[1] = 6000;
console.log(buf1);
// 输出: <Buffer 88 a0>
console.log(buf2);
// 输出: <Buffer 88 13 70 17>
```

## String Decoder

字符串解码器 (String Decoder) 是一个用于将 Buffer 拿来 decode 到 string 的模块, 是作为 Buffer.toString 的一个补充, 它支持多字节 UTF-8 和 UTF-16 字符. 例如

```javascript
const StringDecoder = require('string_decoder').StringDecoder;
const decoder = new StringDecoder('utf8');

const cent = Buffer.from([0xC2, 0xA2]);
console.log(decoder.write(cent)); // ¢

const euro = Buffer.from([0xE2, 0x82, 0xAC]);
console.log(decoder.write(euro)); // €
```

stringDecoder.write 会确保返回的字符串不包含 Buffer 末尾残缺的多字节字符，残缺的多字节字符会被保存在一个内部的 buffer 中用于下次调用 stringDecoder.write() 或 stringDecoder.end()。

```javascript
const StringDecoder = require('string_decoder').StringDecoder;
const decoder = new StringDecoder('utf8');

decoder.write(Buffer.from([0xE2]));
decoder.write(Buffer.from([0x82]));
console.log(decoder.end(Buffer.from([0xAC])));  // €
```

## Stream

Node.js 内置的 `stream` 模块是多个核心模块的基础. 但是流 (stream) 是一种很早之前流行的编程方式. 可以用大家比较熟悉的 C语言来看这种流式操作:

```c

int copy(const char *src, const char *dest)
{
    FILE *fpSrc, *fpDest;
    char buf[BUF_SIZE] = {0};
    int lenSrc, lenDest;

    // 打开要 src 的文件
    if ((fpSrc = fopen(src, "r")) == NULL)
    {
        printf("文件 '%s' 无法打开\n", src);
        return FAILURE;
    }

    // 打开 dest 的文件
    if ((fpDest = fopen(dest, "w")) == NULL)
    {
        printf("文件 '%s' 无法打开\n", dest);
        fclose(fpSrc);
        return FAILURE;
    }
    
    // 从 src 中读取 BUF_SIZE 长的数据到 buf 中
    while ((lenSrc = fread(buf, 1, BUF_SIZE, fpSrc)) > 0)
    {
        // 将 buf 中的数据写入 dest 中
        if ((lenDest = fwrite(buf, 1, lenSrc, fpDest)) != lenSrc)
        {
            printf("写入文件 '%s' 失败\n", dest);
            fclose(fpSrc);
            fclose(fpDest);
            return FAILURE;
        }
        // 写入成功后清空 buf
        memset(buf, 0, BUF_SIZE);
    }
  
    // 关闭文件
    fclose(fpSrc);
    fclose(fpDest);
    return SUCCESS;
}
```

应用的场景很简单, 你要拷贝一个 20G 大的文件, 如果你一次性将 20G 的数据读入到内存, 你的内存条可能不够用, 或者严重影响性能. 但是你如果使用一个 1MB 大小的缓存 (buf) 每次读取 1Mb, 然后写入 1Mb, 那么不论这个文件多大都只会占用 1Mb 的内存.

而在 Node.js 中, 原理与上述 C 代码类似, 不过在读写的实现上通过 libuv 与 EventEmitter 加上了异步的特性. 在 linux/unix 中你可以通过 `|` 来感受到流式操作.

### Stream 的类型


类|使用场景|重写方法
---|---|---
[Readable](https://github.com/substack/stream-handbook#readable-streams)|只读|_read
[Writable](https://github.com/substack/stream-handbook#writable-streams)|只写|_write
[Duplex](https://github.com/substack/stream-handbook#duplex)|读写|_read, _write
[Transform](https://github.com/substack/stream-handbook#transform)|操作被写入数据, 然后读出结果|_transform, _flush


### 对象模式

通过 Node API 创建的流, 只能够对字符串或者 buffer 对象进行操作. 但其实流的实现是可以基于其他的 JavaScript 类型(除了 null, 它在流中有特殊的含义)的. 这样的流就处在 "对象模式(objectMode)" 中.
在创建流对象的时候, 可以通过提供 `objectMode` 参数来生成对象模式的流. 试图将现有的流转换为对象模式是不安全的.

### 缓冲区

Node.js 中 stream 的缓冲区, 以开头的 C语言 拷贝文件的代码为模板讨论, (抛开异步的区别看) 则是从 `src` 中读出数据到 `buf` 中后, 并没有直接写入 `dest` 中, 而是先放在一个比较大的缓冲区中, 等待写入(消费) `dest` 中. 即, 在缓冲区的帮助下可以使读与写的过程分离.

Readable 和 Writable 流都会将数据储存在内部的缓冲区中. 缓冲区可以分别通过 `writable._writableState.getBuffer()` 和 `readable._readableState.buffer` 来访问. 缓冲区的大小, 由构造 stream 时候的 `highWaterMark` 标志指定可容纳的 byte 大小, 对于 `objectMode` 的 stream, 该标志表示可以容纳的对象个数.

#### 可读流

当一个可读实例调用 `stream.push()` 方法的时候, 数据将会被推入缓冲区. 如果数据没有被消费, 即调用 `stream.read()` 方法读取的话, 那么数据会一直留在缓冲队列中. 当缓冲区中的数据到达 `highWaterMark` 指定的阈值, 可读流将停止从底层汲取数据, 直到当前缓冲的报备成功消耗为止.

#### 可写流

在一个在可写实例上不停地调用 writable.write(chunk) 的时候数据会被写入可写流的缓冲区. 如果当前缓冲区的缓冲的数据量低于 `highWaterMark` 设定的值, 调用 writable.write() 方法会返回 true (表示数据已经写入缓冲区), 否则当缓冲的数据量达到了阈值, 数据无法写入缓冲区 write 方法会返回 false, 直到 drain 事件触发之后才能继续调用 write 写入.

```javascript
// Write the data to the supplied writable stream one million times.
// Be attentive to back-pressure.
function writeOneMillionTimes(writer, data, encoding, callback) {
  let i = 1000000;
  write();
  function write() {
    var ok = true;
    do {
      i--;
      if (i === 0) {
        // last time!
        writer.write(data, encoding, callback);
      } else {
        // see if we should continue, or wait
        // don't pass the callback, because we're not done yet.
        ok = writer.write(data, encoding);
      }
    } while (i > 0 && ok);
    if (i > 0) {
      // had to stop early!
      // write some more once it drains
      writer.once('drain', write);
    }
  }
}
```

#### Duplex 与 Transform

Duplex 流和 Transform 流都是同时可读写的, 他们会在内部维持两个缓冲区, 分别对应读取和写入, 这样就可以允许两边同时独立操作, 维持高效的数据流. 比如说 net.Socket 是一个 Duplex 流, Readable 端允许从 socket 获取、消耗数据, Writable 端允许向 socket 写入数据. 数据写入的速度很有可能与消耗的速度有差距, 所以两端可以独立操作和缓冲是很重要的.

### pipe

stream 的 `.pipe()`, 将一个可写流附到可读流上, 同时将可写流切换到流模式, 并把所有数据推给可写流. 在 pipe 传递数据的过程中, `objectMode` 是传递引用, 非 `objectMode` 则是拷贝一份数据传递下去.

pipe 方法最主要的目的就是将数据的流动缓冲到一个可接受的水平, 不让不同速度的数据源之间的差异导致内存被占满. 关于 pipe 的实现参见 David Cai 的 [通过源码解析 Node.js 中导流（pipe）的实现](https://cnodejs.org/topic/56ba030271204e03637a3870)

## Console

[console.log 同步还是异步取决于与谁相连和`os`](https://nodejs.org/dist/latest-v6.x/docs/api/process.html#process_a_note_on_process_i_o). 不过一般情况下的实现都是如下 ([6.x 源代码](https://github.com/nodejs/node/blob/v6.x/lib/console.js#L42))，其中`this._stdout`默认是`process.stdout`:

```javascript
// As of v8 5.0.71.32, the combination of rest param, template string
// and .apply(null, args) benchmarks consistently faster than using
// the spread operator when calling util.format.
Console.prototype.log = function(...args) {
  this._stdout.write(`${util.format.apply(null, args)}\n`);
};
```

自己实现一个 console.log 可以参考如下代码:

```javascript
let print = (str) => process.stdout.write(str + '\n');

print('hello world');
```

注意: 该代码并没有处理多参数, 也没有处理占位符 (即 util.format 的功能).

### console.log.bind(console) 问题

```javascript
// 源码出处 https://github.com/nodejs/node/blob/v6.x/lib/console.js
function Console(stdout, stderr) {
  // ... init ...

  // bind the prototype functions to this Console instance
  var keys = Object.keys(Console.prototype);
  for (var v = 0; v < keys.length; v++) {
    var k = keys[v];
    this[k] = this[k].bind(this);
  }
}
```

## File

“一切皆是文件”是 Unix/Linux 的基本哲学之一, 不仅普通的文件、目录、字符设备、块设备、套接字等在 Unix/Linux 中都是以文件被对待, 也就是说这些资源的操作对象均为 fd (文件描述符), 都可以通过同一套 system call 来读写. 在 linux 中你可以通过 ulimit 来对 fd 资源进行一定程度的管理限制.

Node.js 封装了标准 POSIX 文件 I/O 操作的集合. 通过 require('fs') 可以加载该模块. 该模块中的所有方法都有异步执行和同步执行两个版本. 你可以通过 fs.open 获得一个文件的文件描述符.

### 编码

// TODO

UTF8, GBK, es6 中对编码的支持, 如何计算一个汉字的长度

BOM

### stdio

stdio (standard input output) 标准的输入输出流, 即输入流 (stdin), 输出流 (stdout), 错误流 (stderr) 三者. 在 Node.js 中分别对应 `process.stdin` (Readable), `process.stdout` (Writable) 以及 `process.stderr` (Writable) 三个 stream.

输出函数是每个人在学习任何一门编程语言时所需要学到的第一个函数. 例如 C语言的 `printf("hello, world!");` python/ruby 的 `print 'hello, world!'` 以及 JavaScript 中的 `console.log('hello, world!');`

以 C语言的伪代码来看的话, 这类输出函数的实现思路如下:

```c
int printf(FILE *stream, 要打印的内容)
{
  // ...

  // 1. 申请一个临时内存空间
  char *s = malloc(4096);

  // 2. 处理好要打印的的内容, 其值存储在 s 中
  //      ...

  // 3. 将 s 上的内容写入到 stream 中
  fwrite(s, stream);

  // 4. 释放临时空间
  free(s);

  // ...
}
```

我们需要了解的是第 3 步, 其中的 stream 则是指 stdout (输出流). 实际上在 shell 上运行一个应用程序的时候, shell 做的第一个操作是 fork 当前 shell 的进程 (所以, 如果你通过 ps 去查看你从 shell 上启动的进程, 其父进程 pid 就是当前 shell 的 pid), 在这个过程中也把 shell 的 stdio 继承给了你当前的应用进程, 所以你在当前进程里面将数据写入到 stdout, 也就是写入到了 shell 的 stdout, 即在当前 shell 上显示了.

输入也是同理, 当前进程继承了 shell 的 stdin, 所以当你从 stdin 中读取数据时, 其实就获取到你在 shell 上输入的数据. (PS: shell 可以是 windows 下的 cmd, powershell, 也可以是 linux 下 bash 或者 zsh 等)

当你使用 ssh 在远程服务器上运行一个命令的时候, 在服务器上的命令输出虽然也是写入到服务器上 shell 的 stdout, 但是这个远程的 shell 是从 sshd 服务上 fork 出来的, 其 stdout 是继承自 sshd 的一个 fd, 这个 fd 其实是个 socket, 所以最终其实是写入到了一个 socket 中, 通过这个 socket 传输你本地的计算机上的 shell 的 stdout.

如果你理解了上述情况, 那么你也就能理解为什么守护进程需要关闭 stdio, 如果切到后台的守护进程没有关闭 stdio 的话, 那么你在用 shell 操作的过程中, 屏幕上会莫名其妙的多出来一些输出. 此处对应[守护进程](/sections/zh-cn/process.md#守护进程)的 C 实现中的这一段:

```c
for (; i < getdtablesize(); ++i) {
   close(i);  // 关闭打开的 fd
}
```

Linux/unix 的 fd 都被设计为整型数字, 从 0 开始. 你可以尝试运行如下代码查看.

```
console.log(process.stdin.fd); // 0
console.log(process.stdout.fd); // 1
console.log(process.stderr.fd); // 2
```

在上一节中的 [在 IPC 通道建立之前, 父进程与子进程是怎么通信的? 如果没有通信, 那 IPC 是怎么建立的?](/sections/zh-cn/process.md#q-child) 中使用环境变量传递 fd 的方法, 这么看起来就很直白了, 因为传递 fd 其实是直接传递了一个整型数字.

### 如何同步的获取用户的输入?

如果你理解了上述的内容, 那么放到 Node.js 中来看, 获取用户的输入其实就是读取 Node.js 进程中的输入流 (即 process.stdin 这个 stream) 的数据.

而要同步读取, 则是不用异步的 read 接口, 而是用同步的 readSync 接口去读取 stdin 的数据即可实现. 以下来自万能的 stackoverflow:

```javascript
/*
 * http://stackoverflow.com/questions/3430939/node-js-readsync-from-stdin
 * @mklement0
 */
var fs = require('fs');

var BUFSIZE = 256;
var buf = new Buffer(BUFSIZE);
var bytesRead;

module.exports = function() {
  var fd = ('win32' === process.platform) ? process.stdin.fd : fs.openSync('/dev/stdin', 'rs');
  bytesRead = 0;

  try {
    bytesRead = fs.readSync(fd, buf, 0, BUFSIZE);
  } catch (e) {
    if (e.code === 'EAGAIN') { // 'resource temporarily unavailable'
      // Happens on OS X 10.8.3 (not Windows 7!), if there's no
      // stdin input - typically when invoking a script without any
      // input (for interactive stdin input).
      // If you were to just continue, you'd create a tight loop.
      console.error('ERROR: interactive stdin input not supported.');
      process.exit(1);
    } else if (e.code === 'EOF') {
      // Happens on Windows 7, but not OS X 10.8.3:
      // simply signals the end of *piped* stdin input.
      return '';
    }
    throw e; // unexpected exception
  }

  if (bytesRead === 0) {
    // No more stdin input available.
    // OS X 10.8.3: regardless of input method, this is how the end 
    //   of input is signaled.
    // Windows 7: this is how the end of input is signaled for
    //   *interactive* stdin input.
    return '';
  }
  // Process the chunk read.

  var content = buf.toString(null, 0, bytesRead - 1);

  return content;
};
```

## Readline

`readline` 模块提供了一个用于从 Readble 的 stream (例如 process.stdin) 中一次读取一行的接口. 当然你也可以用来读取文件或者 net, http 的 stream, 比如:

```javascript
const readline = require('readline');
const fs = require('fs');

const rl = readline.createInterface({
  input: fs.createReadStream('sample.txt')
});

rl.on('line', (line) => {
  console.log(`Line from file: ${line}`);
});
```

实现上, realine 在读取 TTY 的数据时, 是通过 `input.on('keypress', onkeypress)` 时发现用户按下了回车键来判断是新的 line 的, 而读取一般的 stream 时, 则是通过缓存数据然后用正则 .test 来判断是否为 new line 的.

PS: 打个广告, 如果在编写脚本时, 不习惯这样异步获取输入, 想要同步获取同步的用户输入可以看一看这个 Node.js 版本类 C语言使用的 [scanf](https://github.com/Lellansin/node-scanf/) 模块 (支持 ts).

## REPL

Read-Eval-Print-Loop (REPL)

整理中
