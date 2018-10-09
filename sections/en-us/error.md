# Error handle & Debug

* [`[Doc]` Errors](/sections/en-us/error.md#errors)
* [`[Doc]` Domain](/sections/en-us/error.md#domain)
* [`[Doc]` Debugger](/sections/en-us/error.md#debugger)
* [`[Doc]` C/C++ Addon](/sections/en-us/error.md#cc-addon)
* [`[Doc]` V8](/sections/en-us/error.md#v8)
* [`[Point]` Memory snapshots](/sections/en-us/error.md#memory-snapshots)
* [`[Point]` CPU profiling](/sections/en-us/error.md#cpu-profiling)


## Errors

There are mainly four types of Errors in Node.js:

|Error| Triggered by |
|---|---|
|Standard JavaScript errors|error codes|
|System errors|operating system|
|User-specified errors|throw method|
|Assertion errors| `assert` module|

Here are the common standard JavaScript errors：

* [EvalError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/EvalError): Thrown when error occurs when calling eval().
* [SyntaxError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SyntaxError): Thrown when codes are not conforming to JavaScript syntax style.
* [RangeError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RangeError): Thrown when out of bounds.
* [ReferenceError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ReferenceError): Thrown when referencing undefined variables.
* [TypeError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError): Thrown when parameter types are error.
* [URIError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/URIError): Thrown when misusing global URI handling functions.

And the common system errors list can be viewed by os object in Node.js：

```javascript
const os = require('os');

console.log(os.constants.errno);
```

When searching interview questions of Node.js, We find that most of them are out-of-date. In a older post [Best practices about error handling in NodeJS](https://cnodejs.org/topic/55714dfac4e7fbea6e9a2e5d), which was translated from Joyent's official blog, we've found the words below:

>  In fact, the only commonly-used case where you'd use `try/catch` is `JSON.parse` and other user-input validation functions.

But in nowadays you can easily use `try/catch` to catch asynchronous exceptions in Node.js. and after using the upgraded v8 engine from Node.js v7.6, the problem that `try/catch` codes can not be optimized was also solved. Now let's see the question

> <a name="q-handle-error"></a> How should I handle unexpected errors? Should I use `try/catch`, domains, or something else?

Here are the error handling methods in Node.js:

* `callback(err, data)` Callback agreement
* throw / try / catch 
* Error event of EventEmitter

Using `callback(err, data)` to handle errors is cumbersome and does not have compulsion, so we recommend you understand it, but don't use it. As for the domain module, it's already half foot into the coffin.

1) Thank [co](https://github.com/visionmedia/co) for his leading forward in this domain, now you can use `try/catch` to protect key position easily, Such as koa's error processing can be done in the form of middleware, for more details, see [Koa error handling](https://github.com/koajs/koa/wiki/Error-Handling). async/await are the same way as koa.

2) Adding error callback for the key object through error listening form of EventEmitter. Such as `error` events of http server, tcp server and `uncaughtException`, `unhandledRejection` of process object.

3) Using Promise to encapsulate asynchronous, and the error handling of it to handle errors.

4) If the methods above can't play a good role, then you should learn how to [Let It Crash](http://wiki.c2.com/?LetItCrash) gracefully.

> Why is the first parameter of cb should be error? And why is the first parameter of some cb is not error, such as http.createServer?

TODO


### Error stack is missing

```javascript
function test() {
  throw new Error('test error');
}

function main() {  
  test();
}

main();
```

Then you get an error message:

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

You can find that the number of rows reported, call hierarchy of test and main function are all displayed clearly in the stack.

When we use timers such as setImmediate to set asynchronously:

```javascript
function test() {
  throw new Error('test error');
}

function main() {  
  setImmediate(() => test());
}

main();

```

We find this:

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

The error stack only outputs to the line where the function was called in `test` function, and the call information of `main` function is lost. That is if you have many layers of nested function calls, it is very hard to trace this asynchronous call when error occurs, because the up-layer stack is already lost. If you've used modules such as [async](https://github.com/caolan/async), You may also find that the error stack is very long and tortuous, so it's difficult to locate the error position through the stack.

This won't be a problem if the project is small / the coder knows all the thing, but it will become a big pain when the project growing bigger and there is more coders. We've talked about this issue in the `Suggestions for writing new functions` section of [best practices about error handling in Node.js](https://cnodejs.org/topic/55714dfac4e7fbea6e9a2e5d) mentioned above. Errors are packaged layer by layer through the way of using [verror](https://www.npmjs.com/package/verror) so that we can get the key information for locating error in the finally got Error.

Let's see the download statistics from yesterday (2017-3-13). Last month, the downloads of [verror](https://www.npmjs.com/package/verror) is `1100w`, higher than [express](https://www.npmjs.com/package/express) (`1070w`). Now you can feel how popular is this way of writing codes.

### Defensive programming

It's not terrible to make mistakes, what makes a terrible  mistake is you are not prepared to deal with it————[Introduction and skills of defensive programming](http://blog.jobbole.com/101651/)

### let it crash

[Let It Crash](http://wiki.c2.com/?LetItCrash)

### uncaughtException

The `uncaughtException` event of process object will be triggered  when the exception is not caught and bubbling to the Event Loop. By default, Node.js will ouput the stack trace information to the `stderr` and end process for such exceptions, And adding listener to `uncaughtException` event can override the default behavior, thus not end the process directly.

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

#### Using uncaughtException reasonably

The original intention of `uncaughtException` is to let you do some recycling processing and then process.exit after getting the error. Official comrades have discussed to remove this event. (See [issues](https://github.com/nodejs/node-v0.x-archive/issues/2582))

So you need to know `uncaughtException` is already a non-conventional means, try to avoid using it to handle errors. Because capturing the error through the event does not mean that `you can continue to run happily (On Error Resume Next)`. There is an unhandled exception inside the program, which means that the application is in an unknown state. If you can not properly restore its status, then it is likely to trigger unforeseen problems. (Even worse if using domain, and all kinds of puzzled questions will be produced)

If the error is not caught in the callback listener specified by the `.on` function, the process of Node.js will be interrupted and return a non-zero exit code, and finally output the corresponding stack information. Otherwise, there will be infinite recursion. In addition, memory crashes / underlying errors also can not be captured, **We currently guess** the reason is v8/C++ did not deal with the problem, while Node.js was unable to handle it (TODO: We suddenly found this idea has not been verified yet, please help us to verify it if convenient).

So the right way advised by the officials to use `uncaughtException` is cleaning up the used resources synchronously (file descriptors, handles, and so on) and then process.exit. 

Actually It's not safe to perform a normal restore operation after uncaughtException event. Officials advise you to prepare for a monitor process to do health checks, manage recoveries and restart when necessary (So the officials are reminding you to use tools such as pm2 implicitly).


### unhandledRejection

This event will be triggered When a Promise without binded handler is rejected. It is very useful when investigating and tracking Promise not handles reject behavior.

Here are the parameters of the callback of this event:

* `reason` [`<Error>`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) | `<any>` Rejected reason (Usually Error)
* `p` The rejected Promise

Such as:

```javascript
process.on('unhandledRejection', (reason, p) => {
  console.log('Unhandled Rejection at: Promise', p, 'reason:', reason);
  // application specific logging, throwing an error, or other logic here
});

somePromise.then((res) => {
  return reportToUser(JSON.pasre(res)); // note the typo (`pasre`)
}); // no `.catch` or `.then`
```

The following code also triggers the `unhandledRejection` event：

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

In the early Node.js, try/catch is unable to capture asynchronous errors, And the error first callback is just an agreement, without mandataries and very cumbersome to write. So in order to catch the exception very well, Node.js introduces domain module in v0.8.

domain is an EventEmitter object, The basic idea of capturing an asynchronous exception is to create a domain, The `cb` function will inherit the domain of the upper layer, and the errors will be passed through the error events triggered by `.emit('error', err)` function in current domain, so that asynchronous errors can be forced to capture. (For more details, see [Asynchronous exception handling in Node.js and analysis of domain module](https://cnodejs.org/topic/516b64596d38277306407936))

But domain also brought more new problems. Such as dependent modules can not inherit the domain you defined, Causing it can't cover errors in dependent modules. Furthermore, Many people (especially new bie) didn't understand memory / asynchronous processes and other issues in Node.js, they didn't do well when using domain to process errors and let the code continue, this is likely to cause **the project to be completely unserviceable** (Any problems are possible, And all kinds of shit...)

You can see the latest news about this module at: [deprecate domains](https://github.com/nodejs/node/issues/66)


## Debugger

![node-js-survey-debug](/assets/node-js-survey-debug.png)

Command line debug tool like gdb (Build-in debugger in the image above), it also supports remote debug (like [node-inspector](https://github.com/node-inspector/node-inspector), but still in  trial). Of course, many developers feel that [vscode](https://code.visualstudio.com/) has maken a better integration to the debug tools.

We recommend reading [official document](https://nodejs.org/dist/latest-v6.x/docs/api/debugger.html) to learn how to use this build-in debugger. If you want to dig deeper, see: [Modify the value of a variable in the NodeJS program dynamically](http://code.oneapm.com/nodejs/2015/06/27/intereference/)


## C/C++ Addon

The most painful thing when developing addon in Node.js is the incompatible of C/C++ codes caused by V8 upgrades, and it has lasted for a long time. So someone opened a project named [nan](https://github.com/nodejs/nan) to solve this problem.

To learn addon development, We recommend reading: [https://github.com/nodejs/node-addon-examples](https://github.com/nodejs/node-addon-examples) in addition to [official document](https://nodejs.org/docs/latest/api/addons.html)


## V8

We are not talking about V8, but V8 module in Node.js. It's used for opening built-in events and interfaces of V8 engine in Node.js. Because these interfaces are defined by underlying part of V8, so we can't say it's absolutely stable.

|Interface|Description|
|---|---|
|v8.getHeapStatistics()|Get heap informations|
|v8.getHeapSpaceStatistics()|Get heap space informations |
|v8.setFlagsFromString(string)|Settings V8 options dynamicly|

### v8.setFlagsFromString(string)

This method is used to add additional V8 command line flags. But be cautious, modifying the configuration after the VM starts may cause unpredictable behavior, crash, and data loss; Or nothing.

You can query the available V8 options in the current Node.js environment by `node --v8-options` command. Furthermore, you can also refer to an unofficial maintenance [V8 options list](https://github.com/thlorenz/v8-flags/blob/master/flags-0.11.md).

Example:

```javascript
// Print GC events to stdout for one minute.
const v8 = require('v8');
v8.setFlagsFromString('--trace_gc');
setTimeout(function() { v8.setFlagsFromString('--notrace_gc'); }, 60e3);
```

## Memory snapshots

Memory snapshots are commonly used to resolve memory leaks. We recommend you use [heapdump](https://github.com/bnoordhuis/node-heapdump) to save memory snapshots, and [devtool](https://github.com/Jam3/devtool) to view memory snapshots. When using heapdump to save memory snapshots, it only contains objects in Node.js (but for [node-inspector](https://github.com/node-inspector/node-inspector), there will be front-end variables in the snapshot).

For more details about memory leaks and how to resolve them, see: [How to analysis memory leaks in Node.js](https://zhuanlan.zhihu.com/p/25736931?group_id=825001468703674368).

## CPU profiling

CPU profiling is commonly used in performance optimization. And there are many third-party tools to do it, but in most cases, the easiest way is using the built-in one in Node.js - [V8 internal profiler](https://github.com/v8/v8/wiki/Using%20V8%E2%80%99s%20internal%20profiler), it can do interval sampling analysis during the execution of the program.

Using `--prof` to turn on built-in profilling.

```shell
node --prof app.js
```

It will generate one `isolate-0xnnnnnnnnnnnn-v8.log` file in the current run directory after the program runs.

You can use `--prof-process` to generate a report.

```
node --prof-process isolate-0xnnnnnnnnnnnn-v8.log
```

And the report is as followed:

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

|Field|Description|
|---|---|
|ticks|Time slice|
|total|The ratio of the current operation to the total time|
|nonlib|Current ratio of non-System library execution time|

Coming soon...
