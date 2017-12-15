# Basic

* [`[Common]` Type judgment](/sections/en-us/common.md#Type-judgement)
* [`[Common]` Scope](/sections/en-us/common.md#Scope)
* [`[Common]` Reference](/sections/en-us/common.md#Reference)
* `[Common]` Memory release
* `[Common]` ES6+ features

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
