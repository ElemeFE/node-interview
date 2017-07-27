# util

* `[Doc]` URL
* `[Doc]` Query Strings
* `[Doc]` Utilities
* `[Basic]` Regex


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

### Characters escape

A common list of characters that need to be escaped:

|Character|encodeURI|
|---|---|
|`' '`|`'%20'`|
|`<`|`'%3C'`|
|`>`|`'%3E'`|
|`"`|`'%22'`|
|<code>\`</code>|`'%60'`|
|`\r`|`'%0D'`|
|`\n`|`'%0A'`|
|`\t`|`'%09'`|
|`{`|`'%7B'`|
|`}`|`'%7D'`|
|`|`|`'%7C'`|
|`\\`|`'%5C'`|
|`^`|`'%5E'`|
|`'`|`'%27'`|

Want to know more? Try this:

```javascript
Array(range).fill(0)
  .map((_, i) => String.fromCharCode(i))
  .map(encodeURI)
```

Try to set the range to 255 first (doge.

## Query Strings

A query string is the part of a URL referring to the table above. Node.js provides a module called `querystring`.

|Method|Description|
|---|---|
|.parse(str[, sep[, eq[, options]]])|Parse a query string into a json object|
|.unescape(str)|Inner method used by .parse(). It is exported primarily to allow application code to provide a replacement decoding implementation if necessary|
|.stringify(obj[, sep[, eq[, options]]])|
Converts a json object to a query string|
|.escape(str)|Inner method used by .stringify(). It is exported primarily to allow application code to provide a replacement percent-encoding implementation if necessary.|

So far, the Node.js built-in querystring does not support for the deep structure:

```javascript
const qs = require('qs'); // 
Third party
const querystring = require('querystring'); // Node.js built-in

let obj = { a: { b: { c: 1 } } };

console.log(qs.stringify(obj)); // 'a%5Bb%5D%5Bc%5D=1'
console.log(querystring.stringify(obj)); // 'a='

let str = 'a%5Bb%5D%5Bc%5D=1';

console.log(qs.parse(str)); // { a: { b: { c: '1' } } }
console.log(querystring.parse(str)); // { 'a[b][c]': '1' }
```

> <a name="q-get-param"></a> How does HTTP pass `let arr = [1,2,3,4]` to the server by GET method?

```javascript
const qs = require('qs');

let arr = [1,2,3,4];
let str = qs.stringify({arr});

console.log(str); // arr%5B0%5D=1&arr%5B1%5D=2&arr%5B2%5D=3&arr%5B3%5D=4
console.log(decodeURI(str)); // 'arr[0]=1&arr[1]=2&arr[2]=3&arr[3]=4'
console.log(qs.parse(str)); // { arr: [ '1', '2', '3', '4' ] }
```

You can pass arr Array to the server vir `https://your.host/api/?arr[0]=1&arr[1]=2&arr[2]=3&arr[3]=4`.

## util

In v4.0.0 or later, util.is*() is not recommended and deprecated. Maybe it is because that maintaining the library is thankless and there are so many popular libraries. The following is the list:

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

Most of them can be used as an interview to ask how to implement.

### util.inherits

> How to implement util.inherits in Node.js?

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

## Regex

At first, regular expression is a biological expression that used to describe the brain neurons, GNU beard used to do the string match after the original road drifting away. Then it is used by men of GNU to match string, and 
deviates from the original road.

Collecting...

## Common Modules

[Awesome Node.js](https://github.com/sindresorhus/awesome-nodejs)
[Most depended-upon packages](https://www.npmjs.com/browse/depended)

> <a name="q-traversal"></a> How do I get all the file names under a folder?

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
      if (err.code === 'ENOENT' && // can not open link file
          !!fs.readlinkSync(filepath)) { // if it is a link file
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


Of course you can also use Oh my [glob](https://github.com/isaacs/node-glob):


```javascript
const glob = require("glob");

glob("**/*.js", (err, files) {
  if (err) {
    throw new Error(err);
  }
  console.log('Here you are:', files.map(path.basename));
});
