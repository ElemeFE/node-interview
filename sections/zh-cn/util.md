# util

* `[Doc]` URL
* `[Doc]` Query Strings (查询字符串)
* `[Doc]` Utilities (实用函数)
* `[Basic]` 正则表达式


## URL

```javascript
┌─────────────────────────────────────────────────────────────────────────────┐
│                                    href                                     │
├──────────┬┬───────────┬─────────────────┬───────────────────────────┬───────┤
│ protocol ││   auth    │      host       │           path            │ hash  │
│          ││           ├──────────┬──────┼──────────┬────────────────┤       │
│          ││           │ hostname │ port │ pathname │     search     │       │
│          ││           │          │      │          ├─┬──────────────┤       │
│          ││           │          │      │          │ │    query     │       │
"  http:   // user:pass @ host.com : 8080   /p/a/t/h  ?  query=string   #hash "
│          ││           │          │      │          │ │              │       │
└──────────┴┴───────────┴──────────┴──────┴──────────┴─┴──────────────┴───────┘
```

### 转义字符

常见的需要转义的字符列表:

|字符|encodeURI|
|---|---|
|`' '`|`'%20'`|
|`<`|`'%3C'`|
|`>`|`'%3E'`|
|`"`|`'%22'`|
|```|`'%60'`|
|`\r`|`'%0D'`|
|`\n`|`'%0A'`|
|`\t`|`'%09'`|
|`{`|`'%7B'`|
|`}`|`'%7D'`|
|`|`|`'%7C'`|
|`\\`|`'%5C'`|
|`^`|`'%5E'`|
|`'`|'%27'|

想了解更多? 你可以这样:

```javascript
Array(range).fill(0)
  .map((_, i) => String.fromCharCode(i))
  .map(encodeURI)
```

range 先来个 255 试试 (doge


## Query Strings

query string 属于 URL 的一部分, 见上方 URL 的表. 在 Node.js 中有内置提供一个 `querystring` 的模块.

|方法|描述|
|---|---|
|.parse(str[, sep[, eq[, options]]])|将一个 query string 解析为 json 对象|
|.unescape(str)|供 .parse 调用的内置解转义方法, 暴露出来以供用户自行替代|
|.stringify(obj[, sep[, eq[, options]]])|将一个 json 对象转换成 query string|
|.escape(str)|供 .stringify 调用的内置转义方法, 暴露出来以供用户自行替代|

Node.js 内置的 querystring 目前对于有深度的结构尚不支持. 见如下:

```javascript
const qs = require('qs'); // 第三方
const querystring = require('querystring'); // Node.js 内置

let obj = { a: { b: { c: 1 } } };

console.log(qs.stringify(obj)); // 'a%5Bb%5D%5Bc%5D=1'
console.log(querystring.stringify(obj)); // 'a='

let str = 'a%5Bb%5D%5Bc%5D=1';

console.log(qs.parse(str)); // { a: { b: { c: '1' } } }
console.log(querystring.parse(str)); // { 'a[b][c]': '1' }
```

> <a name="q-get-param"></a> HTTP 如何通过 GET 方法 (URL) 传递 let arr = [1,2,3,4] 给服务器?

```javascript
const qs = require('qs');

let arr = [1,2,3,4];
let str = qs.stringify({arr});

console.log(str); // arr%5B0%5D=1&arr%5B1%5D=2&arr%5B2%5D=3&arr%5B3%5D=4
console.log(decodeURI(str)); // 'arr[0]=1&arr[1]=2&arr[2]=3&arr[3]=4'
console.log(qs.parse(str)); // { arr: [ '1', '2', '3', '4' ] }
```

通过 `https://your.host/api/?arr[0]=1&arr[1]=2&arr[2]=3&arr[3]=4` 即可传递把 arr 数组传递给服务器


## util
 
util.is*() 从 v4.0.0 开始被不建议使用即将废弃 (deprecated). 大概的废弃原因, 笔者个人认为是维护这些功能吃力不讨好, 而且现在流行的轮子那么多. 那么一下是具体列表:

* util.debug(string)
* util.error([...strings])
* util.isArray(object)
* util.isBoolean(object)
* util.isBuffer(object)
* util.isDate(object)
* util.isError(object)
* util.isFunction(object)
* util.isNull(object)
* util.isNullOrUndefined(object)
* util.isNumber(object)
* util.isObject(object)
* util.isPrimitive(object)
* util.isRegExp(object)
* util.isString(object)
* util.isSymbol(object)
* util.isUndefined(object)
* util.log(string)
* util.print([...strings])
* util.puts([...strings])
* util._extend(target, source)

其中大部分都可以作为面试题来问如何实现.

### util.inherits

> Node.js 中继承 (util.inherits) 的实现?

https://github.com/nodejs/node/blob/v7.6.0/lib/util.js#L960

```javascript
/**
 * Inherit the prototype methods from one constructor into another.
 *
 * The Function.prototype.inherits from lang.js rewritten as a standalone
 * function (not on Function.prototype). NOTE: If this file is to be loaded
 * during bootstrapping this function needs to be rewritten using some native
 * functions as prototype setup using normal JavaScript does not work as
 * expected during bootstrapping (see mirror.js in r114903).
 *
 * @param {function} ctor Constructor function which needs to inherit the
 *     prototype.
 * @param {function} superCtor Constructor function to inherit prototype from.
 * @throws {TypeError} Will error if either constructor is null, or if
 *     the super constructor lacks a prototype.
 */
exports.inherits = function(ctor, superCtor) {

  if (ctor === undefined || ctor === null)
    throw new TypeError('The constructor to "inherits" must not be ' +
                        'null or undefined');

  if (superCtor === undefined || superCtor === null)
    throw new TypeError('The super constructor to "inherits" must not ' +
                        'be null or undefined');

  if (superCtor.prototype === undefined)
    throw new TypeError('The super constructor to "inherits" must ' +
                        'have a prototype');

  ctor.super_ = superCtor;
  Object.setPrototypeOf(ctor.prototype, superCtor.prototype);
};
```

## 正则表达式

正则表达式最早生物学上用来描述大脑神经元的一种表达式, 被 GNU 的大胡子拿来做字符串匹配之后在原本的道路上渐行渐远.

整理中..

## 常用模块

[Awesome Node.js](https://github.com/sindresorhus/awesome-nodejs)
[Most depended-upon packages](https://www.npmjs.com/browse/depended)

> <a name="q-traversal"></a> 如何获取某个文件夹下所有的文件名?

一个简单的例子:

```javascript
const fs = require('fs');
const path = require('path');

function traversal(dir) {
  let res = []
  for (let item of fs.readdirSync(dir)) {
    let filepath = path.join(dir, item);
    try {
      let fd = fs.openSync(filepath, 'r');
      let flag = fs.fstatSync(fd).isDirectory();
      fs.close(fd); // TODO
      if (flag) {
        res.push(...traversal(filepath));
      } else {
        res.push(filepath);
      }
    } catch(err) {
      if (err.code === 'ENOENT' && // link 文件打不开
          !!fs.readlinkSync(filepath)) { // 判断是否 link 文件
        res.push(filepath);
      } else {
        console.error('err', err);
      }
    } 
  }
  return res.map((file) => path.basename(file));
}

console.log(traversal('.'));


```

当然也可以 Oh my [glob](https://github.com/isaacs/node-glob):

```javascript
const glob = require("glob");

glob("**/*.js", (err, files) => {
  if (err) {
    throw new Error(err);
  }
  files.map((filename) => {
    console.log('Here you are:', filename);
  });
});
```
