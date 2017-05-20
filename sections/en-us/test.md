# 测试

* [`[Basic]` 测试方法](#测试方法)
* [`[Basic]` 单元测试](#单元测试)
* [`[Basic]` 基准测试](#集成测试)
* [`[Basic]` 集成测试](#基准测试)
* [`[Basic]` 压力测试](#压力测试)
* [`[Doc]` Assert (断言)](#assert)

## 简述

> <a name="q-why-write-test"></a> 为什么要写测试? 写测试是否会拖累开发进度?

项目在多人合作的时候, 为了某个功能修改了某个模块的某部分代码, 实际的情况中修改一个地方可能会影响到别人开发的多个功能, 在自己不知情的情况下想要保证自己修改的代码不影响到其他功能, 最简单的办法是通过测试来保证.

```
A
  \
    E
  /   \
B       H
  \   /
    F
  / 
C
  \
    G
  / 
D
```

如上述情况, ABCD 是逻辑层, EFGH 等是更低一次层 (比如工具层等), 当你为了功能 A 的 BUG 修改了 H 的代码, 那么实际受影响的功能除了 A 之外还有 BC, 如果你有针对每一个逻辑的测试, 那么修改了 H 的代码之后, 跑一遍测试即可保证对 H 的修改不会影响到 BC (如果有影响, 那么相应的测试会报错). 利用这种特性, 你还可以基于测试去做重构, 在通过原有测试的情况下, 即表明新的重构版本可以替代原有的版本.

而这样的效果, 只有当覆盖率达到了一定程度 (通常是 80% 以上, 90% 以上为最理想) 才能实现, 如果测试的覆盖率低, 无法覆盖到多种情况, 那么测试对你的项目可能是没有用甚至起到反作用的 (让你误以为你的修改没问题而发布等).

写测试是否会拖累开发进度要视具体情况而定. 需要考虑到, 开发进度包含功能和品质两个方面, 单纯写代码的速度不能完全代表开发进度. 测试在适当的情况下可以保证项目的品质从而得到更好的开发进度.

如上述的例子, 在修改功能 A 的 BUG 的时候, 如果你不知道 H 会影响到 BC 又没有测试的话, 那么开发 BC 的同学可能会出现十分经典的 **"昨天还好好的, 今天怎么就不能用了?"** 的情况. 

当然写测试拖累开发进度的情况也是客观存在的, 通常是有以下几种情况:

* 不会写测试
* 过度测试, 不必要的测试
* 为了迎合测试, 而忽略了实际需求


> <a name="q-death-loop"></a> 测试是如何保证业务逻辑中不会出现死循环的?

你可以通过测试来避免坑爹的同事在某些逻辑中写出死循环, 在通常的测试中加上超时的时间, 在覆盖率足够的情况下, 就可以通过跑出超时的测试来排查出现死循环以及低性能的情况.


## 测试方法

### 黑盒测试

黑盒测试 (Black-box Testing), 测试应用程序的功能, 而不是其内部结构或运作. 测试者不需了解代码、内部结构等, 只需知道什么是应用应该做的事, 即当键入特定的输入, 可得到一定的输出. 测试者通过选择`有效输入`和`无效输入`来验证是否正确的输出. 此测试方法可适合大部分的软件测试, 例如集成测试 (Integration Testing) 以及系统测试 (System Testing).

### 白盒测试

白盒测试 (White-box Testing) 测试应用程序的内部结构或运作, 而不是测试应用程序的功能 (即黑盒测试). 在白盒测试时, 以编程语言的角度来设计测试案例. 白盒测试可以应用于单元测试 (Unit Testing)、集成测试 (Integration Testing) 和系统的软件测试流程, 可测试在集成过程中每一单元之间的路径, 或者主系统跟子系统中的测试.


## 单元测试

单元测试 (Unit Testing) 是白盒测试的一种, 用于针对程序模块进行正确性检验的测试工作. 单元 (Unit) 是指**最小可测试的部件**. 在过程化编程中, 一个单元就是单个程序、函数、过程等; 对于面向对象编程, 最小单元就是方法, 包括基类、抽象类、或者子类中的方法.

另外, 每次修改代码之后, 通过单元测试来验证比把整个应用启动/重启验证要更快/更简单.

### 覆盖率

测试覆盖率 (Test Coverage) 是指代码中各项逻辑被测试覆盖到的比率, 比如 90% 的覆盖率, 是指代码中 90% 的情况都被测试覆盖到了.

覆盖率通常由四个维度贡献:

* 行覆盖率 (line coverage) 是否每一行都执行了？
* 函数覆盖率 (function coverage) 是否每个函数都调用了？
* 分支覆盖率 (branch coverage) 是否每个if代码块都执行了？
* 语句覆盖率 (statement coverage) 是否每个语句都执行了？

常用的测试覆盖率框架 [istanbul](https://github.com/gotwarlost/istanbul).

当然覆盖率并不完全是由单元测试贡献, 在单元测试之上还有集成测试等. 更多关于覆盖率的内容可以参见[测试覆盖（率）到底有什么用?](http://www.infoq.com/cn/articles/test-coverage-rate-role)

### Mock

Mock 主要用于单元测试中. 当一个测试的对象可能依赖其他 (也许复杂/多个) 的对象. 为了确保其行为不受其他对象的影响, 你可以通过模拟其他对象的行为来隔离你要测试的对象.

当你要测试的单元依赖了一些很难纳入单元测试的情况时 (例如要测试的单元依赖数据库/文件操作/第三方服务 等情况的返回时), 使用 mock 是非常有用的. 简而言之, Mock 是模拟其他依赖的 behaviour.

Mock 与 Stub 的区别参见: [Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)


### 常见测试工具

* [Mocha](https://github.com/mochajs/mocha)
* [ava](https://github.com/avajs/ava)
* [Jest](https://github.com/facebook/jest)


## 集成测试

集成测试也称综合测试、组装测试、联合测试, 将程序模块采用适当的集成策略组装起来, 对系统的接口及集成后的功能进行正确性检测的测试工作. 集成测试可以是黑盒的, 也可以是白盒的, 其主要目的是检查软件单位之间的接口是否正确, 而集成测试的对象是**已经经过单元测试的模块**.

例如你可以在本地将项目中的 web app 启动, 并模拟接口调用:

```javascript
describe('Path API', () => {
  // ...

  describe('GET /v2/path/:_id', () => {
    it('should return 200 GET /v2/path/:_id', () => {
      return request
        .get('/v2/path/' + pathId)
        .set('Cookie', 'common_user=xxx')
        .expect(200);
    });
  });

  describe('POST /v2/path', () => {
    it('should return 412 POST /v2/path lost params path', () => {
      return request
        .post('/v2/path')
        .set('Cookie', 'common_user=xxx')
        .expect(412);
    });

    it('should return 409 POST /v2/path when path exist', () => {
      return request
        .post('/v2/path')
        .send({path: '/'})
        .set('Cookie', 'common_user=xxx')
        .expect(409);
    });

    it('should return 200 POST /v2/path successfully', () => {
      return request
        .post('/v2/path')
        .send({path: '/comment'})
        .set('Cookie', 'common_user=xxx')
        .expect(200);
    });
  });

  // ...
});
```

## 基准测试

目前 Node.js 中流行的白盒级基准测试工具是 [benchmark](https://benchmarkjs.com/docs).

```javascript
const Benchmark = require('benchmark');
const suite = new Benchmark.Suite;

suite.add('RegExp#test', function() {
    /o/.test('Hello World!');
})
.add('String#indexOf', function() {
    'Hello World!'.indexOf('o') > -1;
})
.on('cycle', function(event) {
    console.log(String(event.target));
})
.on('complete', function() {
    console.log('Fastest is ' + this.filter('fastest').map('name'));
})
// run async
.run({ 'async': true });
```

你可以将同一个功能的不同实现基于同一个标准来比较不同实现的速度, 从而得到最优解.

黑盒级别的基准测试, 则推荐 [Apache ab](https://httpd.apache.org/docs/2.4/programs/ab.html) 以及 [wrk](https://github.com/wg/wrk) 等, 例如执行:

```
ab -n 100 -c 10 https://ele.me/
```

可以得到如下的详细数据:

```
Server Software:        Tengine/2.1.1
Server Hostname:        ele.me
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES256-GCM-SHA384,2048,256

Document Path:          /
Document Length:        284 bytes

Concurrency Level:      10
Time taken for tests:   1.775 seconds
Complete requests:      100
Failed requests:        0
Non-2xx responses:      100
Total transferred:      62400 bytes
HTML transferred:       28400 bytes
Requests per second:    56.33 [#/sec] (mean)
Time per request:       177.511 [ms] (mean)
Time per request:       17.751 [ms] (mean, across all concurrent requests)
Transfer rate:          34.33 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       88  116  26.0    104     234
Processing:    33   55  39.6     47     394
Waiting:       33   54  39.0     46     394
Total:        124  171  48.1    152     491

Percentage of the requests served within a certain time (ms)
  50%    152
  66%    184
  75%    193
  80%    199
  90%    224
  95%    242
  98%    288
  99%    491
 100%    491 (longest request)
```

与前者相比, ab 等工具可以设置规模以及并发情况. 在比规模不大/需求不复杂的情况下, ab 以及 wrk 也可以用于做压力测试.


## 压力测试

压力测试 (Stress testing), 是保证系统稳定性的一种测试方法. 通过预估系统所需要承载的 QPS, TPS 等指标, 然后通过如 [Jmeter](http://jmeter.apache.org/) 等压测工具模拟相应的请求情况, 来验证当前应能能否达到目标.

对于比较重要, 流量较高或者后期业务量会持续增长的系统, 进行压力测试是保证项目品质的重要环节. 常见的如负载是否均衡, 带宽是否合理, 以及磁盘 IO 网络 IO 等问题都可以通过比较极限的压力测试暴露出来.


## Assert

断言 (Assert) 是快速判断并对不符合预期的情况进行报错的模块. 是将:

```javascript
if (condition) {
  throw new Error('Sth wrong');
}
```

写成:

```javascript
assert(!condition, 'Sth wrong');
```

等等情况的一种简化. 并且提供了丰富了 `equal` 判断, 对于对象类型也有深度/严格判断等情况支持.

Node.js 中内置的 `assert` 模块也是属于断言模块的一种, 但是官方在文档中有注明, 该内置模块主要是用于内置代码编写时的基本断言需求, 并不是一个通用的断言库 (**not intended to be used as a general purpose assertion library**)

### 常见断言工具

* [Chai](https://github.com/chaijs/chai)
* [should.js](https://github.com/shouldjs/should.js)

