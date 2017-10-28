# IO

* [`[Doc]` Buffer](/sections/zh-cn/io.md#buffer)
* [`[Doc]` String Decoder](/sections/zh-cn/io.md#string-decoder)
* [`[Doc]` Stream](/sections/zh-cn/io.md#stream)
* [`[Doc]` Console](/sections/zh-cn/io.md#console)
* [`[Doc]` File System](/sections/zh-cn/io.md#file)
* [`[Doc]` Readline](/sections/zh-cn/io.md#readline)
* [`[Doc]` REPL](/sections/zh-cn/io.md#repl)

# Brief introduction

Node.js was famous as handling IO-intensive business. Then here are the questions, What do you really know about IO? What is it called IO intensive business?

## Buffer

Buffer is the class to handle binary data in Node.js, IO-related operations (network / file, etc.) are all based on Buffer. An instance of the Buffer class is very similar to an array of integers, ***but its size is fixed***, And its original memory space is allocated outside the V8 stack. After the instance of the Buffer class is created, the memory size occupied by it can no longer be adjusted.

`New Buffer ()` interface was deprecated from Node.js v6.x, The reason is that different types of parameters will return different types of Buffer objects, So when the developer does not correctly verify the parameters or does not correctly initialize the contents of the Buffer object, it will inadvertently introduce security and reliability problems to the code.

Interface |use
---|---
Buffer.from()|Creates a Buffer object based on the existing data
Buffer.alloc()|Creates an initialized Buffer object
Buffer.allocUnsafe()|Creates an uninitialized Buffer object

### TypedArray

After introducing TypedArray in ES6, Node.js modified the implementation of the original Buffer to Uint8Array in TypedArray, thus enhancing the performance.

Here are the things you need to know when using it:

```javascript
const arr = new Uint16Array(2);
arr[0] = 5000;
arr[1] = 4000;

const buf1 = Buffer.from(arr); // Copy the buffer
const buf2 = Buffer.from(arr.buffer); // Share memory with the array

console.log(buf1);
// Output: <Buffer 88 a0>, The copied buffer only contains to element
console.log(buf2);
// Output: <Buffer 88 13 a0 0f>

arr[1] = 6000;
console.log(buf1);
// Output: <Buffer 88 a0>
console.log(buf2);
// Output: <Buffer 88 13 70 17>
```

## String Decoder

String Decoder is a module to decode buffers to strings, as a supplement to Buffer.toString, it supports multi-byte UTF-8 and UTF-16 characters. Such as:

```javascript
const StringDecoder = require('string_decoder').StringDecoder;
const decoder = new StringDecoder('utf8');

const cent = Buffer.from([0xC2, 0xA2]);
console.log(decoder.write(cent)); // ¢

const euro = Buffer.from([0xE2, 0x82, 0xAC]);
console.log(decoder.write(euro)); // €
```

Of course can be done step by step.

```javascript
const StringDecoder = require('string_decoder').StringDecoder;
const decoder = new StringDecoder('utf8');

decoder.write(Buffer.from([0xE2]));
decoder.write(Buffer.from([0x82]));
console.log(decoder.end(Buffer.from([0xAC])));  // €
```

## Stream

Built-in `stream` module in Node.js is the basis of multiple core modules. But stream is a popular programming method very early. We can use the more familiar C language to see stream operation:

```c

int copy(const char *src, const char *dest)
{
    FILE *fpSrc, *fpDest;
    char buf[BUF_SIZE] = {0};
    int lenSrc, lenDest;

    // open src file
    if ((fpSrc = fopen(src, "r")) == NULL)
    {
        printf("file '%s' can not be opened\n", src);
        return FAILURE;
    }

    // open dest file
    if ((fpDest = fopen(dest, "w")) == NULL)
    {
        printf("file '%s' can not be opened\n", dest);
        fclose(fpSrc);
        return FAILURE;
    }
    
    // Read the BUF_SIZE data from src to buf
    while ((lenSrc = fread(buf, 1, BUF_SIZE, fpSrc)) > 0)
    {
        // write data in buf to dest 
        if ((lenDest = fwrite(buf, 1, lenSrc, fpDest)) != lenSrc)
        {
            printf("write file '%s' failed\n", dest);
            fclose(fpSrc);
            fclose(fpDest);
            return FAILURE;
        }
        // clean buf when success
        memset(buf, 0, BUF_SIZE);
    }
  
    // close file
    fclose(fpSrc);
    fclose(fpDest);
    return SUCCESS;
}
```

The application scenario is simple, When you need to copy a 20G file, if you have 20G of data read into memory at once, your memory may not be enough, or seriously affect performance. But if you use a 1MB size cache (buf), Read 1Mb, then write 1Mb, Then no matter how much of this file will only take up 1Mb of memory.

In Node.js, the principle is similar to the above C code, But its IO operation is implemented through libuv and EventEmitter with asynchronous features. You can use `|` to feel the stream operation in linux/unix.

### Type of Stream 


Class| Scenario |Overrided method
---|---|---
[Readable](https://github.com/substack/stream-handbook#readable-streams)|Read only|_read
[Writable](https://github.com/substack/stream-handbook#writable-streams)|Write only|_write
[Duplex](https://github.com/substack/stream-handbook#duplex)|Read and write|_read, _write
[Transform](https://github.com/substack/stream-handbook#transform)|Operate the writed data, and then read out the results|_transform, _flush


### Object mode

The stream created by the Node API can only manipulate strings or buffer objects. But the implementation of the stream can be based on other types of JavaScript(Except for null, it has a special meaning in stream). This stream is in the "object mode (objectMode)".
You can generate an object-mode stream by providing the `objectMode` parameter when creating a stream object. It is not safe to attempt to convert an existing stream to object mode.

### Buffer

The buffer of stream in Node.js, using the copy file code at the begining that written in C Languange as a template to discuss, (Despite the difference with asynchronous) is reading data from `src` to` buf`, and not directly written to `dest`, but first placed in a relatively large buffer, Waiting for be written into (comsumed) `dest` . That is, with the help of the buffer we can achieve Read and write separation.

Both the Readable and Writable streams store the data in an internal buffer. The buffers can be accessed by `writable._writableState.getBuffer ()` and `readable._readableState.buffer` respectively. The size of the buffer is specified by the `highWaterMark` flag when creating the stream, for `objectMode` stream, this flag indicates the number of objects that can be accommodated.

#### Readable stream

When a readable instance calls the `stream.push ()` method, the data will be pushed into the buffer. If the data is not consumed, That is, If you call the `stream.read ()` method to read the words, the data will remain in the buffer queue. When the data in the buffer reaches the threshold specified by `highWaterMark`, The readable stream will stop drawing data from the bottom until the current buffered report is successfully consumed.

#### Writable stream

The data is written to the buffer of the writable stream when a writable.write (chunk) is kept on a writable instance. If the buffer amount of the current buffer is less than the value set by `highWaterMark`, Calling the writable.write () method will return true (indicating that the data has been written to the buffer), Otherwise, the write method will return false when the amount of data buffered reaches the threshold and the data can not be written to the buffer, then you can continue to call write to write Until the drain event is triggered.

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

#### Duplex and Transform

Duplex stream and the Transform stream are both readable and writable simultaneously, They will maintain two internal buffer respectively, corresponding to read and write, so that you can allow both sides to operate at the same time independently, thus to maintain efficient data flow. Such as net.Socket is a Duplex stream, The Readable side allows you to get data from the socket and consume data, while the Writable side allows you to write data to it. The speed of data writing is likely to be different from the speed of consumption, so it is important to operate and buffer both ends independently.

### pipe

The `.pipe()` method of stream appends a writable stream to a readable stream while switching the writable stream to stream mode, and push all the data to the writable stream. In the process of passing data in the pipe, `objectMode` is passing by references, while non-`objectMode` is passing by value.

The main purpose of the pipe method is to buffer the flow of data to an acceptable level, so that the difference between the different data sources won't cause the memory to be filled. For more details about pipe, see David Cai's [Analyzes the implementation of pipe in Node.js by source code](https://cnodejs.org/topic/56ba030271204e03637a3870)

## Console

[Generally console.log is asynchronous, unless you use `new Console(stdout[, stderr])` to specify a file as a destination](https://nodejs.org/dist/latest-v6.x/docs/api/console.html#console_asynchronous_vs_synchronous_consoles). However, mostly it looks like this ([6.x source code](https://github.com/nodejs/node/blob/v6.x/lib/console.js#L42)):

```javascript
// As of v8 5.0.71.32, the combination of rest param, template string
// and .apply(null, args) benchmarks consistently faster than using
// the spread operator when calling util.format.
Console.prototype.log = function(...args) {
  this._stdout.write(`${util.format.apply(null, args)}\n`);
};
```

Refering to the following code if you want to implement a console.log by yourself:

```javascript
let print = (str) => process.stdout.write(str + '\n');

print('hello world');
```

Note: The code does not handle multiple arguments, nor does it handle placeholders (the function of util.format).

### The console.log.bind(console) problem

```javascript
// From https://github.com/nodejs/node/blob/v6.x/lib/console.js
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

“Everything is a file” is one of the basic philosophy of Unix/Linux, Not only normal files, but also directory, character device, block device, socket, and so on are all treated as files in Unix/Linux, that is, the operating objects of these resources are all fd (file descriptor), they can be read and written through the same set of system call. You can use ulimit to manage the fd resources in linux.

Node.js encapsulates the collection of standard POSIX file I / O operations. The module can be loaded by require ('fs'). All the methods in the module have both asynchronous execution and synchronous execution. You can get a file's file descriptor via fs.open.

### Encoding

// TODO

Supports for UTF8, GBK, es6 encoding, how to calculate the length of a Chinese character

BOM

### stdio

stdio (standard input output), includes stdin, stdout and stderr. Corresponding to `process.stdin` (Readable),` process.stdout` (Writable) and `process.stderr` (Writable) respectively in Node.js.

The output function is the first function that everyone needs to learn when learning a programming language. Such as `printf("hello, world!");` of C language, `print 'hello, world!'` of python/ruby and `console.log('hello, world!');` in JavaScript.

Here is the implementation of such an output function in the C language pseudo-code:

```c
int printf(FILE *stream, The content to be printed)
{
  // ...

  // 1. Apply for a temporary memory space
  char *s = malloc(4096);

  // 2. Handle the contents of the print, the value stored in the s
  //      ...

  // 3. Write the contents of s into the stream
  fwrite(s, stream);

  // 4. Release temporary space
  free(s);

  // ...
}
```

What we need to know is step 3, where stream refers to stdout (output stream). In fact, when running an application on the shell, the first operation of the shell is fork the current process (So, if you see the process you started from the shell through ps, its parent process pid is the current shell pid), in this process your current application process also inherited the shell stdio, so when you write the data to stdout in the current process, it is also written to the shell stdout, that is the current shell.

So it is the input, The current process inherits the shell's stdin, So when you read data from stdin, in fact, it's to get the data you enter in the shell. (PS: shell can be cmd, powershell in windows, or bash and zsh in linux)

When using ssh to run a command on a remote server, Although the command output on the server is also written to the shell on the server stdout, but the remote shell is forked from the sshd service, its stdout is a fd inherited from sshd, so the fd is actually a socket, and the data is actually written to a socket in the end, then being sent to the shell stdout on your local computer through the socket.

If you understand the things we mentioned above, then you can understand why the daemon needs to close stdio, if the daemon that is cut into the background does not close stdio, then when you using the shell to do some operation, the screen will come out some inexplicably output. Here is the code that written in C language in [daemon](/sections/zh-cn/process.md#daemon):

```c
for (; i < getdtablesize(); ++i) {
   close(i);  // close fd
}
```

fd in Linux/unix was designed as an integer number starts from 0. You can try running the following code to view it.

```
console.log(process.stdin.fd); // 0
console.log(process.stdout.fd); // 1
console.log(process.stderr.fd); // 2
```

So it looks very straightforward for the method that using the environment variable to pass fd mentioned in the previous section: [How did the parent process communicate with the child process before the IPC channel was established? How did the IPC build if there was no communication?](/sections/zh-cn/process.md#q-child), because the transmited fd is actually passed an integer number.

### How to get user input synchronizely?

If you already understood the content above, Getting the user's input is actually reading Node.js process in the input stream (ie process.stdin stream) data in Node.js.

And to read synchronously, it is not using the asynchronous read interface, but with the synchronous readSync interface to read the stdin data. The following comes from the Almighty stackoverflow:

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

The `readline` module provides an interface for reading a row from a stream of Readble (for example, process.stdin). Of course, you can also use it to read the file or net, http stream, for example:

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

For implementation, realine uses `input.on('keypress', onkeypress)` method to determine whether it is new line or not when reading TTY data', for normal stream, it caches the data and then uses the regular `.test` to determine whether it is new line.

PS: If you are not used to getting input asynchronously when writing a script and want to get the input synchronously, see this module [scanf](https://github.com/Lellansin/node-scanf/) (typescript supported).

## REPL

Read-Eval-Print-Loop (REPL)

Coming soon...
