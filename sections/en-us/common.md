# Basic

* [`[Common]` Type judgment](/sections/en-us/common.md#Type-judgement)
* [`[Common]` Scope](/sections/en-us/common.md#Scope)
* [`[Common]` Reference](/sections/en-us/common.md#Reference)
* [`[Common]` Memory release](/sections/en-us/common.md#Memory-release)
* [`[Common]` ES6+ features](/sections/en-us/common.md#ES6-features)

## Summary

In contrast to frontend, there are few chances to work with DOM in backend, unless we deal with SSR or web crawlers. So we won't discuss about it. Unlike browser side, backend faces memory directly, we concern more about the fundamental knowledge.

## Type judgement

We suffer tortuously from type judgement in JavaScript. Otherwise, TypeScript may not be created. Basically, we recommend to read the source code of [lodash](https://github.com/lodash/lodash).

Generally, this is a simple opening of an interview. We won't deny a candidate only because of not knowing the value of `undefined == null` is `true`. According to our personal experiences, candidate who cannot answer this question is probably to have a poor foundation. If you have no concept of such kind of question, you may reflect on whether to find a JavaScript book to review for basis.

Additionally, it is a bonus point if candidate understands TypeScript or flow.

## Scope

In an interview, scope is not an easy-to-ask knowledge point but critical in JavaScript. Eleme typically asks questions like `what's the difference between let and var in es6` or asks candidate to interpret a given code example in the beginning, in order to assess how much does a candidate master scope.

[You Don't Know JS](https://github.com/getify/You-Dont-Know-JS) has a great explanation on scope. Here it is the TOC of the book, we recommend you to do some intensive reading.

* Chapter 1: What is Scope?
* Chapter 2: Lexical Scope
* Chapter 3: Function vs. Block Scope
* Chapter 4: Hoisting
* Chapter 5: Scope Closures
* ...

## Reference

> <a name="q-value"></a> In JavaScript, which types are pass by reference? And which types are pass by value? How to pass a variable by reference?

Simply speaking, objects are pass by reference. Basic types are pass by value. We can pass basic types by reference using boxing technique. (More information at note 1)

Pass by reference and pass by value is a basic question. It is fundamental part to understand how does JavaScript's memory work. It is hardly to have further discussion without understanding reference.

In coding session, we use questions like `how to write a json object copy function` to assess candidate.

Sometimes, we ask about the difference between `==` and `===`. And then, `true` or `false` of `[1] == [1]`. Without a good foundation, candidate may make a wrong conclusion because of the wrong understanding of `==` and `===`.

Note 1: For senior candidates, you are expected to question directly on the question. e.g. There is no pass by reference in JavaScript. There is call by sharing. Read about [Is JavaScript a pass-by-reference or pass-by-value language?](http://stackoverflow.com/questions/518000/is-javascript-a-pass-by-reference-or-pass-by-value-language). Though it is advanced, it is common for senior developer with more than 3 years experiences.

If C++ is mentioned in resume, it is certain to ask `what is the difference between pointer and reference`.

## Memory release

> <a name="q-mem"></a> When will each types and each scope of variables be released in JavaScript?

If reference was no longer referenced, it would be collected by the GC of V8. If a value variable was inside a closure, it wouldn't be release until the closure was no longer referenced. In non-closure scope, it will be collected when V8 is switched to new space.

In contrast to frontend JavaScript, a Node.js developer with more than 2 years experience should care about memory. Though you may not understand in depth, you had better have a basic concept of memory release and start to pay attention to memory leaks.

You need to know which operations lead to memory leaks, or even crash the memory. For example, will the code segment given below fill up with all of V8's memory?

```javaScript
let arr = [];
while(true)
  arr.push(1);
```

Then, what's the difference between this one and the above?

```javaScript
let arr = [];
while(true)
  arr.push();
```

If a `Buffer` was pushed, what would happen?

```javaScript
let arr = [];
while(true)
  arr.push(new Buffer(1000));
```

After thinking about the aboves, try to figure out what else can fill up with V8's memory. And then let's talk about memory leaks.

```javaScript
function out() {
  const bigData = new Buffer(100);
  inner = function () {
    void bigData;
  }
}
```

Closure references variable from its parent. If it is not released, a memory leak happens. The example above shows `inner` is under the root, which causes a memory leak (`bigData` is not released).

For senior candidates, you need to know the mechanism of GC in V8 and know how memory snapshot (which will be discussed in chapter of `Debug/Optimization`) works. e.g. Where do V8 store different types of data? What are the specific optimizing strategies for different areas when doing memory release?

## ES6 features

We recommend a [ECMAScript 6 Tutorial](http://es6.ruanyifeng.com/) book from @ruanyifeng (in Chinese).

The basic questions can be the differences between `let` and `var`, and between `arrow function` and `function`.

To go deeper, there are lots of details in es6, such as `reference` together with `const`. Talk about `Set` and `Map` in context of usage and disadvantages of `{}`. Or it can be about the privatization and `symbol`.

However, it is unnecessary to ask `what is a closure?`. Instead, we'd like to ask about the application of closures. e.g. If interviewer usually uses closure to make data private, then we may ask can new features (e.g. `class` and `symbol`) be private? If true, then why we need closure here? When will data in a closure be released? And so on.

For `...`, how to implement deletion of duplicated for an array (Bonus point for using Set).

> <a name="q-const"></a> Is it possible for an element in a const Array be modified? If possible, what's the effect of const?

The elements can be modified. And it protects the reference, which cannot be modified (e.g. [Map](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Map) is sensitive to reference and it need const. Besides, it is also suitable for immutable).
