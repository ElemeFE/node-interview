# event/Asynchronous

* [`[Basic]` Promise](/sections/en-us/event-async.md#promise)
* [`[Doc]` Events ](/sections/en-us/event-async.md#events)
* [`[Doc]` Timers ](/sections/en-us/event-async.md#timers)
* [`[Point]` Blocking/non-blocking](/sections/en-us/event-async.md#blocking-non-blocking)
* [`[Point]` Paraller/Concurrent](/sections/en-us/event-async.md#paraller-concurrent)

## Summary

Synchronous or Asynchronous ？That is a question.

## Promise

![callback-hell](/assets/callback-hell.jpg)


I believe that in the interview, many students have been asked such a question: how to handle `Callback Hell`. In the early years, there are lots of solutions like [Q](https://www.npmjs.com/package/q), [async](https://www.npmjs.com/package/async), [EventProxy](https://www.npmjs.com/package/eventproxy). Finally from the prevalence point of view, promise has been the winner of them, and has been the part of the ECMAScript 6 specification.


To learn more basic knowledge about `promise`, we recommend this article. [Promise](http://javascript.ruanyifeng.com/advanced/promise.html#toc9)

> <a name="q-1"></a> What's the difference between the second argument of '.then' function and '.catch' function?

To distinguish the difference, you can read this article [We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)

About Synchronous or Asynchronous, I hope you can pay attention to this question, a simple `promise` example below.

```javascript
let doSth = new Promise((resolve, reject) => {
  console.log('hello');
  resolve();
});

doSth.then(() => {
  console.log('over');
});
```

there is no doubt that you can get the output 

```
hello
over
```


the first question is that the code wrapped by Promise is certainly synchronized, but whether the execution of `then` is Asynchronous?

the second quesiton is `setTimeout` and `then` will be called after 10s, but how about `hello`? will `hello` be printed after 10s or at the beginning?

```javascript
let doSth = new Promise((resolve, reject) => {
  console.log('hello');
  resolve();
});

setTimeout(() => {
  doSth.then(() => {
    console.log('over');
  })
}, 10000);
```


and how to understand the execution order of the code below: ([resource](https://zhuanlan.zhihu.com/p/25407758))


```javascript
setTimeout(function() {
  console.log(1)
}, 0);
new Promise(function executor(resolve) {
  console.log(2);
  for( var i=0 ; i<10000 ; i++ ) {
    i == 9999 && resolve();
  }
  console.log(3);
}).then(function() {
  console.log(4);
});
console.log(5);
```


If you don't kown the answers of these questions, you can print the output at local environment. I hope you can understand the change of  `promise` status, includes the relationship between `promise` and asynchronous, how promise help you to handle async situation , it would be better if you know the implementations of `promise` 

## Events


`Events` module is a very important core module in Node.js. There are many important core APIs in the node that depend on `Events` , for example, `Stream` is implemented based on `Events`, and `fs`, `net`, 'http' are implemented based on 'Stream', 'Events' is so important to Node.js.

A class or a Object can get basic `events` methods by extending `EventEmitter` class, and we call it 'emitter', and the callback funciton that emit a kind of event is called as 'listener'. It is diffrent from DOM tree in browser, there are no bubble and capture actions or methods to handle event.


><a name="q-2">Is the Eventemitter.emit synchronous or asynchronous?   

The answer is **synchronous**, there are some description on Node.js documentation:

>The EventListener calls all listeners synchronously in the order in which they were registered. This is important to ensure the proper sequencing of events and to avoid race conditions or logic errors.


let's discuss the output is 'hi 1' or 'hi 2'?

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('myEvent', () => {
  console.log('hi 1');
});

emitter.on('myEvent', () => {
  console.log('hi 2');
});

emitter.emit('myEvent');
```
and whether there is a endless loop?

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('myEvent', () => {
  console.log('hi');
  emitter.emit('myEvent');
});

emitter.emit('myEvent');
```

and how about this case?

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('myEvent', function sth () {
  emitter.on('myEvent', sth);
  console.log('hi');
});

emitter.emit('myEvent');
```

Emitter can handle many complex state scenarios, such as TCP complex state machine, and if  you are handling a multiple asynchronous operation and each step may throw an error,  at this time .emit error and the excute some .once operations can save you from the mud. 


Pay attention to that some students prefer to monitor the status of certain class, but when you destroy this class, don't forget to destroy these emitters too , because inside the class, some listener may cause memory leak.

## Blocking/non-blocking

> <a name="q-3"></a> How to judge whether a interface is asynchronous? Is it asynchronous while a callback provided?


This is a open question, you can have your own way to judge.

* review documentation
* console.log and print the output
* whether there is IO operation


Simply use the callback function is not asynchronous, IO operation may be asynchronous, in addition to the use of setTimeout and other ways are asynchronous.


> if you have built a website by koa, this website has a interface A, and in some cases, interface A can be the endless loop, unfortunately, if you triggered this endless loop, what will be the impact on your website? 


In Node.js environment javascript code has only one single thread. Only the current code has been excuted, the process will cut into the event loop, and then  pop out the next callback function from the event queue to start the implementation of the code. so ① to implement a Sleep function, as long as an infinite loop can block the execution of the entire js process (on how to avoid the  colleagues write deadless loop, see the chapter of `test`.)


> <a name="q-5"></a> How to implement a Sleep function? ①

```javascript
function sleep(ms) {
  var start = Date.now(), expire = start + ms;
  while (Date.now() < expire) ;
  return;
}
```

Asynchronous in Node.js means an event queue in other thread achived by libuv module.

If endless loop logic trigger in your website, the whole process will be blocked, and all request will timeout, asynchronous code will never be excuted, and your website will be crashed.

> <a name="q-6"></a> How to implement an async.reduce?

You need to konw that reduce is analyze a recursive data structure and through use of a given combining operation, recombine the results of recursively processing its constituent parts, building up a return value.

## Timers

The writter think there are two kinds of 'asynchronous' in Node.js:  `hard asynchronous` and `soft asynchronous`.

`hard asynchronous` means IO operation or some cases that you need  libuv module externally and of course includes `readFileSync` or `execSync`. Because of the single thread feature of Node.js, it is unwise to do some IO operation in synchronous way as it will block the excutation of other code. 

`soft asynchronous` is that some asynchronous cases implemented by `setTimeout`. To understand the diffrence among nextTick, setTimeout and setImmediate , you can see this article. <a name="q-4"></a> [article](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)




**Event loop example** 

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

To know more about event loop, Timers, nextTick, we recommend the Node.js documentation, [*The Node.js Event Loop, Timers, and process.nextTick()*](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/) and [*Tasks, microtasks, queues and schedules*](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/).


## Paraller/Concurrent


Parallelism and Concurrency are two very common concepts. You can read this blog of Joe Armstrong(the creator of Erlang) ([Concurrent and Parallel](http://joearms.github.io/2013/04/05/concurrent-and-parallel-programming.html))

![con_and_par](http://joearms.github.io/images/con_and_par.jpg)

Coucurrent = 2 queues with 1 coffee machine 

Parallel = 2 queues with 2 coffee machines


Node.js executes each task of events queue one by one by event loop, by this way, it avoids that in some traditional multithreading situation, when '2 queues with 1 coffee machine', the context switch and resource scramble/synchronize problems, and achives high concurrent。


you can add another 'coffee machine' by using `cluster` module to achieve paraller in Node.js.
