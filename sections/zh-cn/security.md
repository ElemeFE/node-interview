# 安全

* `[Doc]` Crypto (加密)
* `[Doc]` TLS/SSL
* `[Doc]` HTTPS
* `[Point]` XSS
* `[Point]` CSRF
* `[Point]` 中间人攻击
* `[Point]` Sql/Nosql 注入攻击


## Crypto

Node.js 的 `crypto` 模块封装了诸多的加密功能, 包括 OpenSSL 的哈希、HMAC、加密、解密、签名和验证函数等.

Node.js 的加密貌似有点问题, 某些算法算出来跟别的语言 (比如 Python) 不一样. 具体情况还在整理中 (时间不定), 欢迎补充.

> 加密是如何保证用户密码的安全性?

在客户端加密, 是增加传输的过程中被第三方嗅探到密码后破解的成本. 对于游戏, 在客户端加密是防止外挂/破解等. 在服务端加密 (如 md5) 是避免管理数据库的 DBA 或者攻击者攻击数据库之后直接拿到明文密码, 从而提高安全性.


## TLS/SSL

早期的网络传输协议由于只在大学内使用, 所以是默认互相信任的. 所以传统的网络通信可以说是没有考虑网络安全的. 早年的浏览器大厂网景公司为了应对这个情况设计了 SSL (Secure Socket Layer), SSL 的主要用途是:

1. 认证用户和服务器, 确保数据发送到正确的客户机和服务器;
2. 加密数据以防止数据中途被窃取;
3. 维护数据的完整性, 确保数据在传输过程中不被改变.

存在三个特性:

* 机密性：SSL协议使用密钥加密通信数据
* 可靠性：服务器和客户都会被认证, 客户的认证是可选的
* 完整性：SSL协议会对传送的数据进行完整性检查

1999年, SSL 因为应用广泛, 已经成为互联网上的事实标准. IETF 就在那年把 SSL 标准化/强化. 标准化之后的名称改为传输层安全协议 (Transport Layer Security, TLS). 很多相关的文章都把这两者并列称呼 (TLS/SSL), 因为这两者可以视作同一个东西的不同阶段.


## HTTPS

在网络上, 每个网站都在各自的服务器上, 想要确保你访问的是一个正确的网站, 并且访问到这个网站正确的数据 (没有被劫持/篡改), 除了需要传输安全之外, 还需要安全的认证, 认证不能由目标网站进行, 否则恶意/钓鱼网站也可以自己说自己是对的, 所以为了能在网络上维护网络之间的基本信任, 早期的大厂们合力推动了一项名为 PKI 的基础设施, 通过第三方来认证网站.

公钥基础设施 (Public Key Infrastructure, PKI) 是一种遵循标准的, 利用公钥加密技术为电子商务的开展提供一套安全基础平台的技术和规范. 其基础建置包含认证中心 (Certification Authority, CA) 、注册中心 (Register Authority, RA) 、目录服务 (Directory Service, DS) 服务器. 

由 RA 统筹、审核用户的证书申请, 将证书申请送至 CA 处理后发出证书, 并将证书公告至 DS 中. 在使用证书的过程中, 除了对证书的信任关系与证书本身的正确性做检查外, 并透过产生和发布证书废止列表 (Certificate Revocation List, CRL) 对证书的状态做确认检查, 了解证书是否因某种原因而遭废弃. 证书就像是个人的身分证, 其内容包括证书序号、用户名称、公开金钥 (Public Key) 、证书有效期限等.

在 TLS/SSL 中你可以使用 OpenSSL 来生成 TLS/SSL 传输时用来认证的 public/private key. 不过这个 public/private key 是自己生成的, 而通过 PKI 基础设施可以获得权威的第三方证书 (key) 从而加密 HTTP 传输安全. 目前博客圈子里比较流行的是 [Let's Encrypt 签发免费的 HTTPS 证书](https://imququ.com/post/letsencrypt-certificate.html).

需要注意的是, 如果 PKI 受到攻击, 那么 HTTPS 也一样不安全. 可以参见 [HTTPS 劫持 - 知乎讨论](https://www.zhihu.com/question/22795329) 中的情况, 证书由 CA 机构签发, 一般浏览器遇到非权威的 CA 机构是会告警的 (参见 [12306](https://kyfw.12306.cn/otn/)), 但是如果你在某些特殊的情况下信任了某个未知机构/证书, 那么也可能被劫持.

此外有的 CA 机构以邮件方式认证, 那么当某个网站的邮件服务收到攻击/渗透, 那么攻击者也可能以此从 CA 机构获取权威的正确的证书.


## XSS

跨站脚本 (Cross-Site Scripting, XSS) 是一种代码注入方式, 为了与 CSS 区分所以被称作 XSS. 早期常见于网络论坛, 起因是网站没有对用户的输入进行严格的限制, 使得攻击者可以将脚本上传到帖子让其他人浏览到有恶意脚本的页面, 其注入方式很简单包括但不限于 JavaScript / VBScript / CSS / Flash 等.

当其他用户浏览到这些网页时, 就会执行这些恶意脚本, 对用户进行 Cookie 窃取/会话劫持/钓鱼欺骗等各种攻击. 其原理, 如使用 js 脚本收集当前用户环境的信息 (Cookie 等), 然后通过 img.src, Ajax, onclick/onload/onerror 事件等方式将用户数据传递到攻击者的服务器上. 钓鱼欺骗则常见于使用脚本进行视觉欺骗, 构建假的恶意的 Button 覆盖/替换真实的场景等情况 (该情况在用户上传 CSS 的时候也可能出现, 如早期淘宝网店装修, 使用 CSS 拼接假的评分数据等覆盖在真的评分数据上误导用户).

> 过滤 Html 标签能否防止 XSS? 请列举不能的情况?

用户除了上传

```html
<script>alert('xss');</script>
```

还可以使用图片 url 等方式来上传脚本进行攻击

```html
<table background="javascript:alert(/xss/)"></table>
<img src="javascript:alert('xss')">
```

还可以使用各种方式来回避检查, 例如空格, 回车, Tab

```html
<img src="javas cript:
alert('xss')">
```

还可以通过各种编码转换 (URL 编码, Unicode 编码, HTML 编码, ESCAPE 等) 来绕过检查

```
<img%20src=%22javascript:alert('xss');%22>
<img src="javascrip&#116&#58alert(/xss/)">
```

### CSP 策略

在百般无奈, 没有统一解决方案的情况下, 厂商们推出了 CSP 策略. 

以 Node.js 为例, 计算脚本的 hashes 值:
```
const crypto = require('crypto');

function getHashByCode(code, algorithm = 'sha256') {
  return algorithm + '-' + crypto.createHash(algorithm).update(code, 'utf8').digest("base64");
}

getHashByCode('console.log("hello world");'); // 'sha256-wxWy1+9LmiuOeDwtQyZNmWpT0jqCUikqaqVlJdtdh/0='
```

设置 CSP 头:

```
content-security-policy: script-src 'sha256-wxWy1+9LmiuOeDwtQyZNmWpT0jqCUikqaqVlJdtdh/0='
```

```html
<script>console.log('hello geemo')</script> <!-- 不执行 -->
<script>console.log('hello world');</script> <!-- 执行 -->
```

策略指令可以参见 [CSP Policy Directives](https://developer.mozilla.org/en-US/docs/Web/Security/CSP/CSP_policy_directives)以及[阮一峰的博文](http://www.ruanyifeng.com/blog/2016/09/csp.html), [屈大神的博文](https://imququ.com/post/content-security-policy-reference.html)


## CSRF

跨站请求伪造 (Cross-Site Request Forgery, CSRF, https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet) 是一种伪造跨站请求的攻击方式. 例如利用你在 A 站 (攻击目标) 的 cookie / 权限等, 在 B 站 (恶意/钓鱼网站) 拼装 A 站的请求.

比如 Q 君是某论坛管理员. 已知这个论坛 A 删除的接口是 post 到某个地址, 并指定一个帖子的 id.  那么我可以在自己的博客 B 上组织一个 CSRF 请求. 然后诱使 Q 君来访问我的博客. 就可以在 Q 君不知情的情况下删除掉我想删的某个帖子.

钓鱼方式包括但不限于公开网站 (xss), 攻击者的恶意网站, email 邮件, 微博, 微信, 短信等及时消息.

同源策略是最早用于防止 CSRF 的一种方式, 即关于跨站请求 (Cross-Site Request) 只有在同源/信任的情况下才可以请求. 但是如果一个网站群, 在互相信任的情况下, 某个网站出现了问题:

```
a.public.com
b.public.com
c.public.com
...
```

以上情况下, 如果 c.public.com 上没有预防 xss 等情况, 使得攻击者可以基于此站对其他信任的网站发起 CSRF 攻击.

另外同源策略主要是浏览器来进行验证的, 并且不同浏览器的实现又各自不同, 所以在某些浏览器上可以直接绕过, 而且也可以直接通过短信等方式直接绕过浏览器.

预防:

1. A 站 (预防站) 检查 http 请求的 header 确认其 origin
2. 检查 CSRF token

### 1.同源检查 

通过检查来过滤简单的 CSRF 攻击, 主要检查一下两个 header:

* Origin Header
* Referer Header

### 2.CSRF token 

简单来说, 对需要预防的请求, 通过特别的算法生成 token 存在 session 中, 然后将 token 隐藏在正确的界面表单中, 正式请求时带上该 token 在服务端验证, 避免跨站请求.


## 中间人攻击

中间人 (Man-in-the-middle attack, MITM) 是指攻击者与通讯的两端分别创建独立的联系, 并交换其所收到的数据, 使通讯的两端认为他们正在通过一个私密的连接与对方直接对话, 但事实上整个会话都被攻击者完全控制. 在中间人攻击中, 攻击者可以拦截通讯双方的通话并插入新的内容.

目前比较常见的是在公共场所放置精心准备的免费 wifi, 劫持/监控通过该 wifi 的流量. 或者攻击路由器, 连上你家 wifi 攻破你家 wifi 之后在上面劫持流量等.

对于通信过程中的 MITM, 常见的方案是通过 PKI / TLS 预防, 及时是通过存在第三方中间人的 wifi 你通过 HTTPS 访问的页面依旧是安全的. 而 HTTP 协议是明文传输, 则没有任何防护可言.

不常见的还有强力的互相认证, 你确认他之后, 他也确认你一下; 延迟测试, 统计传输时间, 如果通讯延迟过高则认为可能存在第三方中间人; 等等.

## SQL/NoSQL 注入

注入攻击是指当所执行的一些操作中有部分由用户传入时, 用户可以将其恶意逻辑注入到操作中. 当你使用 eval, new Function 等方式执行的字符串中有用户输入的部分时, 就可能被注入攻击. 上文中的 XSS 就属于一种注入攻击. 前面的章节中也提到过 Node.js 的 child_process.exec 由于调用 bash 解析, 如果执行的命令中有部分属于用户输入, 也可能被注入攻击.

### SQL

Sql 注入是网站常见的一种注入攻击方式. 其原因主要是由于登录时需要验证用户名/密码, 其执行 sql 类似:

```sql
SELECT * FROM users WHERE usernae = 'myName' AND password = 'mySecret';
```

其中的用户名和密码属于用户输入的部分, 那么在未做检查的情况下, 用户可能拼接恶意的字符串来达到其某种目的, 例如上传密码为 `'; DROP TABLE users; --` 使得最终执行的内容为:

```sql
SELECT * FROM users WHERE usernae = 'myName' AND password = ''; DROP TABLE users; --';
```

其能实现的功能, 包括但不限于删除数据 (经济损失), 篡改数据 (密码等), 窃取数据 (网站管理权限, 用户数据) 等. 防治手段常见于:

* 给表名/字段名加前缀 (避免被猜到)
* 报错隐藏表信息 (避免被看到, 12306 早期就出现过的问题)
* 过滤可以拼接 SQL 的关键字符
* 对用户输入进行转义
* 验证用户输入的类型 (避免 limit, order by 等注入)
* 等...

### NoSQL

看个简单的情况:

```javascript
let {user, pass, age} = ctx.query;

db.collection.find({
  user, pass,
  $where: `this.age >= ${age}`
})
```

那么这里的 age 就可以注入了. 另外 GET/POST 还可以传递深层结构 (比如 ?name[0]=alan 传递上来), 通过 qs 之类的模块解析后导致注入, 如 [cnodejs 遭遇 mongodb 注入](https://github.com/cnodejs/nodeclub/commit/0f6cc14f6bcbbe6b4de3199c6896efaec637693e).

