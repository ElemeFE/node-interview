# OS

* `[Doc]` TTY
* `[Doc]` OS (操作系统)
* `[Doc]` 命令行参数
* `[Basic]` 负载
* `[Point]` CheckList
* `[Basic]` 指标

## TTY

"tty" 原意是指 "teletype" 即打字机, "pty" 则是 "pseudo-teletype" 即伪打字机. 在 Unix 中, `/dev/tty*` 是指任何表现的像打字机的设备, 例如终端 (terminal).

你可以通过 `w` 命令查看当前登录的用户情况, 你会发现每登录了一个窗口就会有一个新的 tty.

```shell
$ w
 11:49:43 up 482 days, 19:38,  3 users,  load average: 0.03, 0.08, 0.07
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
dev      pts/0    10.0.128.252     10:44    1:01m  0.09s  0.07s -bash
dev      pts/2    10.0.128.252     11:08    2:07   0.17s  0.14s top
root     pts/3    10.0.240.2       11:43    7.00s  0.04s  0.00s w
```

使用 ps 命令查看进程信息中也有 tty 的信息:

```shell
$ ps -x
  PID TTY      STAT   TIME COMMAND
 5530 ?        S      0:00 sshd: dev@pts/3
 5531 pts/3    Ss+    0:00 -bash
11296 ?        S      0:00 sshd: dev@pts/4
11297 pts/4    Ss     0:00 -bash
13318 pts/4    R+     0:00 ps -x
23733 ?        Ssl    2:53 PM2 v1.1.2: God Daemon
```

其中为 `?` 的是没有依赖 TTY 的进程, 即[守护进程](/sections/zh-cn/process.md#%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B).

在 Node.js 中你可以通过 stdio 的 isTTY 来判断当前进程是否处于 TTY (如终端) 的环境.

```shell
$ node -p -e "Boolean(process.stdout.isTTY)"
true
$ node -p -e "Boolean(process.stdout.isTTY)" | cat
false
```

## OS

通过 OS 模块可以获取到当前系统一些基础信息的辅助函数.

|属性|描述|
|---|---|
|os.EOL|根据当前系统, 返回当前系统的 `End Of Line`|
|os.arch()|返回当前系统的 CPU 架构, 如 `'x86'` 和 `'x64'`|
|os.constants|返回系统常量|
|os.cpus()|返回 CPU 每个核的信息|
|os.endianness()|返回 CPU 字节序, 如果是大端字节序返回 `BE`, 小端字节序则 `LE`|
|os.freemem()|返回系统空闲内存的大小, 单位是字节|
|os.homedir()|返回当前用户的根目录|
|os.hostname()|返回当前系统的主机名|
|os.loadavg()|返回负载信息|
|os.networkInterfaces()|返回网卡信息 (类似 `ifconfig`)|
|os.platform()|返回编译时指定的平台信息, 如 `win32`, `linux`, 同 `process.platform()`|
|os.release()|返回操作系统的分发版本号|
|os.tmpdir()|返回系统默认的临时文件夹|
|os.totalmem()|返回总内存大小(同内存条大小)|
|os.type()|根据 `[uname](https://en.wikipedia.org/wiki/Uname#Examples)` 返回系统的名称|
|os.uptime()|返回系统的运行时间，单位是秒|
|os.userInfo([options])|返回当前用户信息|

> 不同操作系统的换行符 (EOL) 有什么区别?

end of line (EOL) 同 newline, line ending, 以及 line break.

通常由 line feed (LF, `\n`) 和 carriage return (CR, `\r`) 组成. 常见的情况:

|符号|系统|
|---|---|
|LF|在 Unix 或 Unix 相容系统 (GNU/Linux, AIX, Xenix, Mac OS X, ...)、BeOS、Amiga、RISC OS|
|CR+LF|MS-DOS、微软视窗操作系统 (Microsoft Windows)、大部分非 Unix 的系统|
|CR|Apple II 家族, Mac OS 至版本9|

如果不了解 EOL 跨系统的兼容情况, 那么在处理文件的行分割/行统计等情况时可能会被坑.

### OS 常量

* 信号常量 (Signal Constants), 如 `SIGHUP`, `SIGKILL` 等.
* POSIX 错误常量 (POSIX Error Constants), 如 `EACCES`, `EADDRINUSE` 等.
* Windows 错误常量 (Windows Specific Error Constants), 如 `WSAEACCES`, `WSAEBADF` 等.
* libuv 常量 (libuv Constants), 仅 `UV_UDP_REUSEADDR`.


## Path

Node.js 内置的 path 是用于处理路径问题的模块. 不过众所周知, 路径在不同操作系统下又不可调和的差异.

### Windows vs. POSIX

|POSIX|值|Windows|值|
|---|---|---|---|
|path.posix.sep|`'/'`|path.win32.sep|`'\\'`|
|path.posix.normalize('/foo/bar//baz/asdf/quux/..')|`'/foo/bar/baz/asdf'`|path.win32.normalize('C:\\temp\\\\foo\\bar\\..\\')|`'C:\\temp\\foo\\'`|
|path.posix.basename('/tmp/myfile.html')|`'myfile.html'`|path.win32.basename('C:\\temp\\myfile.html')|`'myfile.html'`|
|path.posix.join('/asdf', '/test.html')|`'/asdf/test.html'`|path.win32.join('/asdf', '/test.html')|`'\\asdf\\test.html'`|
|path.posix.relative('/root/a', '/root/b')|`'../b'`|path.win32.relative('C:\\a', 'c:\\b')|`'..\\b'`
|path.posix.isAbsolute('/baz/..')|`true`|path.win32.isAbsolute('C:\\foo\\..')|`true`|
|path.posix.delimiter|`':'`|path.win32.delimiter|`','`|
|process.env.PATH|`'/usr/bin:/bin'`|process.env.PATH|`C:\Windows\system32;C:\Program Files\node\'`|
|PATH.split(path.posix.delimiter)|`['/usr/bin', '/bin']`|PATH.split(path.win32.delimiter)|`['C:\\Windows\\system32', 'C:\\Program Files\\node\\']`|


看了上表之后, 你应该了解到当你处于某个平台之下的时候, 所使用的 `path` 模块的方法其实就是对应的平台的方法, 例如笔者这里用的是 mac, 所以:

```javascript
const path = require('path');
console.log(path.basename === path.posix.basename); // true
```

如果你处于其中某一个平台, 但是要处理另外一个平台的路径, 需要注意这个跨平台的问题.

### path 对象

on POSIX:

```javascript
path.parse('/home/user/dir/file.txt')
// Returns:
// {
//    root : "/",
//    dir : "/home/user/dir",
//    base : "file.txt",
//    ext : ".txt",
//    name : "file"
// }
```

```javascript
┌─────────────────────┬────────────┐
│          dir        │    base    │
├──────┬              ├──────┬─────┤
│ root │              │ name │ ext │
"  /    home/user/dir / file  .txt "
└──────┴──────────────┴──────┴─────┘
```

on Windows:

```javascript
path.parse('C:\\path\\dir\\file.txt')
// Returns:
// {
//    root : "C:\\",
//    dir : "C:\\path\\dir",
//    base : "file.txt",
//    ext : ".txt",
//    name : "file"
// }
```

```javascript
┌─────────────────────┬────────────┐
│          dir        │    base    │
├──────┬              ├──────┬─────┤
│ root │              │ name │ ext │
" C:\      path\dir   \ file  .txt "
└──────┴──────────────┴──────┴─────┘
```

### path.extname(path)

|case|return|
|---|---|
|path.extname('index.html')|`'.html'`|
|path.extname('index.coffee.md')|`'.md'`|
|path.extname('index.')|`'.'`|
|path.extname('index')|`''`|
|path.extname('.index')|`''`|


## 命令行参数

命令行参数 (Command Line Options), 即对 CLI 使用上的一些文档. 关于 CLI 主要有 4 种使用方式:

* node [options] [v8 options] [script.js | -e "script"] [arguments]
* node debug [script.js | -e "script" | <host>:<port>] …
* node --v8-options
* 无参数直接启动 REPL 环境

### Options

|参数|简介|
|---|---|
|-v, --version|查看当前 node 版本|
|-h, --help|查看帮助文档|
|-e, --eval "script"|将参数字符串当做代码执行
|-p, --print "script"|打印 `-e` 的返回值
|-c, --check|检查语法并不执行
|-i, --interactive|即使 stdin 不是终端也打开 REPL 模式
|-r, --require module|在启动前预先 `require` 指定模块
|--no-deprecation|关闭废弃模块警告
|--trace-deprecation|打印废弃模块的堆栈跟踪信息
|--throw-deprecation|执行废弃模块时抛出错误
|--no-warnings|无视报警（包括废弃警告）
|--trace-warnings|打印警告的 stack （包括废弃模块）
|--trace-sync-io|只要检测到异步 I/O 出于 Event loop 的开头就打印 stack trace
|--zero-fill-buffers|自动初始化(zero-fill) **Buffer** 和 **SlowBuffer**
|--preserve-symlinks|在解析和缓存模块时指示模块加载程序保存符号链接
|--track-heap-objects|为堆快照跟踪堆对象的分配情况
|--prof-process|使用 v8 选项 `--prof` 生成 Profilling 报告
|--v8-options|显示 v8 命令行选项
|--tls-cipher-list=list|指明替代的默认 TLS 加密器列表
|--enable-fips|在启动时开启 FIPS-compliant crypto
|--force-fips|在启动时强制实施 FIPS-compliant
|--openssl-config=file|启动时加载 OpenSSL 配置文件
|--icu-data-dir=file|指定ICU数据加载路径

### 环境变量

|环境变量|简介|
|----|----|
|`NODE_DEBUG=module[,…]`|指定要打印调试信息的核心模块列表
|`NODE_PATH=path[:…]`|指定搜索目录模块路径的前缀列表
|`NODE_DISABLE_COLORS=1`|关闭 REPL 的颜色显示
|`NODE_ICU_DATA=file`|ICU (Intl object) 数据路径
|`NODE_REPL_HISTORY=file`|持久化存储REPL历史文件的路径
|`NODE_TTY_UNSAFE_ASYNC=1`|设置为1时, 将同步操作 stdio (如 console.log 变成同步)
|`NODE_EXTRA_CA_CERTS=file`|指定 CA (如 VeriSign) 的额外证书路径

## 负载

负载是衡量服务器运行状态的一个重要概念. 通过负载情况, 我们可以知道服务器目前状态是清闲, 良好, 繁忙还是即将 crash.

通常我们要查看的负载是 CPU 负载, 详细一点的情况你可以通过阅读这篇博客: [Understanding Linux CPU Load](http://blog.scoutapp.com/articles/2009/07/31/understanding-load-averages) 来了解.

命令行上可以通过 `uptime`, `top` 命令, Node.js 中可以通过 `os.loadavg()` 来获取当前系统的负载情况:

```
load average: 0.09, 0.05, 0.01
```

其中分别是最近 1 分钟, 5 分钟, 15 分钟内系统 CPU 的平均负载. 当 CPU 的一个核工作饱和的时候负载为 1, 有几核 CPU 那么饱和负载就是几.

在 Node.js 中单个进程的 CPU 负载查看可以使用 [pidusage](https://github.com/soyuka/pidusage) 模块.

除了 CPU 负载, 对于服务端 (偏维护) 还需要了解网络负载, 磁盘负载等.

## CheckList

> 有一个醉汉半夜在路灯下徘徊，路过的人奇怪地问他：“你在路灯下找什么？”醉汉回答：“我在找我的KEY”,路人更奇怪了：“找钥匙为什么在路灯下?”，醉汉说：“因为这里最亮！”。

很多服务端的同学在说到检查服务器状态时只知道使用 `top` 命令, 其实情况就和上面的笑话一样, 因为对于他们而言 `top` 是最亮的那盏路灯.

对于服务端程序员而言, 完整的服务器 checklist 首推 [《性能之巅》](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B0140I5WPK) 第二章中讲述的 [USE 方法](http://www.brendangregg.com/USEmethod/use-linux.html).

The USE Method provides a strategy for performing a complete check of system health, identifying common bottlenecks and errors. For each system resource, metrics for utilization, saturation and errors are identified and checked. Any issues discovered are then investigated using further strategies.

This is an example USE-based metric list for Linux operating systems (eg, Ubuntu, CentOS, Fedora). This is primarily intended for system administrators of the physical systems, who are using command line tools. Some of these metrics can be found in remote monitoring tools.

### Physical Resources

<table border="1" cellpadding="2" width="100%">
<tbody><tr><th>component</th><th>type</th><th>metric</th></tr>

<tr><td>CPU</td><td>utilization</td><td>system-wide: <tt>vmstat 1</tt>, "us" + "sy" + "st"; <tt>sar -u</tt>, sum fields except "%idle" and "%iowait"; <tt>dstat -c</tt>, sum fields except "idl" and "wai"; per-cpu: <tt>mpstat -P ALL 1</tt>, sum fields except "%idle" and "%iowait"; <tt>sar -P ALL</tt>, same as <tt>mpstat</tt>; per-process: <tt>top</tt>, "%CPU"; <tt>htop</tt>, "CPU%"; <tt>ps -o pcpu</tt>; <tt>pidstat 1</tt>, "%CPU"; per-kernel-thread: <tt>top</tt>/<tt>htop</tt> ("K" to toggle), where VIRT == 0 (heuristic). [1]</td></tr>
<tr><td>CPU</td><td>saturation</td><td>system-wide: <tt>vmstat 1</tt>, "r" &gt; CPU count [2]; <tt>sar -q</tt>, "runq-sz" &gt; CPU count; <tt>dstat -p</tt>, "run" &gt; CPU count; per-process: /proc/PID/schedstat 2nd field (sched_info.run_delay); <tt>perf sched latency</tt> (shows "Average" and "Maximum" delay per-schedule); dynamic tracing, eg, SystemTap schedtimes.stp "queued(us)" [3]</td></tr>
<tr><td>CPU</td><td>errors</td><td><tt>perf</tt> (LPE) if processor specific error events (CPC) are available; eg, AMD64's "04Ah Single-bit ECC Errors Recorded by Scrubber" [4]</td></tr>

<tr><td>Memory capacity</td><td>utilization</td><td>system-wide: <tt>free -m</tt>, "Mem:" (main memory), "Swap:" (virtual memory); <tt>vmstat 1</tt>, "free" (main memory), "swap" (virtual memory); <tt>sar -r</tt>, "%memused"; <tt>dstat -m</tt>, "free"; <tt>slabtop -s c</tt> for kmem slab usage; per-process: <tt>top</tt>/<tt>htop</tt>, "RES" (resident main memory), "VIRT" (virtual memory), "Mem" for system-wide summary</td></tr>
<tr><td>Memory capacity</td><td>saturation</td><td>system-wide: <tt>vmstat 1</tt>, "si"/"so" (swapping); <tt>sar -B</tt>, "pgscank" + "pgscand" (scanning); <tt>sar -W</tt>; per-process: 10th field (min_flt) from /proc/PID/stat for minor-fault rate, or dynamic tracing [5]; OOM killer: <tt>dmesg | grep killed</tt></td></tr>
<tr><td>Memory capacity</td><td>errors</td><td><tt>dmesg</tt> for physical failures; dynamic tracing, eg, SystemTap uprobes for failed malloc()s</td></tr>

<tr><td>Network Interfaces</td><td>utilization</td><td><tt>sar -n DEV 1</tt>, "rxKB/s"/max "txKB/s"/max; <tt>ip -s link</tt>, RX/TX tput / max bandwidth; /proc/net/dev, "bytes" RX/TX tput/max; nicstat "%Util" [6]</td></tr>
<tr><td>Network Interfaces</td><td>saturation</td><td><tt>ifconfig</tt>, "overruns", "dropped"; <tt>netstat -s</tt>, "segments retransmited"; <tt>sar -n EDEV</tt>, *drop and *fifo metrics; /proc/net/dev, RX/TX "drop"; nicstat "Sat" [6]; dynamic tracing for other TCP/IP stack queueing [7]</td></tr>
<tr><td>Network Interfaces</td><td>errors</td><td><tt>ifconfig</tt>, "errors", "dropped"; <tt>netstat -i</tt>, "RX-ERR"/"TX-ERR"; <tt>ip -s link</tt>, "errors"; <tt>sar -n EDEV</tt>, "rxerr/s" "txerr/s"; /proc/net/dev, "errs", "drop"; extra counters may be under /sys/class/net/...; dynamic tracing of driver function returns 76]</td></tr>

<tr><td>Storage device I/O</td><td>utilization</td><td>system-wide: <tt>iostat -xz 1</tt>, "%util"; <tt>sar -d</tt>, "%util"; per-process: iotop; <tt>pidstat -d</tt>; /proc/PID/sched "se.statistics.iowait_sum"</td></tr>
<tr><td>Storage device I/O</td><td>saturation</td><td><tt>iostat -xnz 1</tt>, "avgqu-sz" &gt; 1, or high "await"; <tt>sar -d</tt> same; LPE block probes for queue length/latency; dynamic/static tracing of I/O subsystem (incl. LPE block probes)</td></tr>
<tr><td>Storage device I/O</td><td>errors</td><td>/sys/devices/.../ioerr_cnt; <tt>smartctl</tt>; dynamic/static tracing of I/O subsystem response codes [8]</td></tr>

<tr><td>Storage capacity</td><td>utilization</td><td>swap: <tt>swapon -s</tt>; <tt>free</tt>; /proc/meminfo "SwapFree"/"SwapTotal"; file systems: "df -h"</td></tr>
<tr><td>Storage capacity</td><td>saturation</td><td>not sure this one makes sense - once it's full, ENOSPC</td></tr>
<tr><td>Storage capacity</td><td>errors</td><td><tt>strace</tt> for ENOSPC; dynamic tracing for ENOSPC; /var/log/messages errs, depending on FS</td></tr>

<tr><td>Storage controller</td><td>utilization</td><td><tt>iostat -xz 1</tt>, sum devices and compare to known IOPS/tput limits per-card</td></tr>
<tr><td>Storage controller</td><td>saturation</td><td>see storage device saturation, ...</td></tr>
<tr><td>Storage controller</td><td>errors</td><td>see storage device errors, ...</td></tr>

<tr><td>Network controller</td><td>utilization</td><td>infer from <tt>ip -s link</tt> (or /proc/net/dev) and known controller max tput for its interfaces</td></tr>
<tr><td>Network controller</td><td>saturation</td><td>see network interface saturation, ...</td></tr>
<tr><td>Network controller</td><td>errors</td><td>see network interface errors, ...</td></tr>

<tr><td>CPU interconnect</td><td>utilization</td><td>LPE (CPC) for CPU interconnect ports, tput / max</td></tr>
<tr><td>CPU interconnect</td><td>saturation</td><td>LPE (CPC) for stall cycles</td></tr>
<tr><td>CPU interconnect</td><td>errors</td><td>LPE (CPC) for whatever is available</td></tr>

<tr><td>Memory interconnect</td><td>utilization</td><td>LPE (CPC) for memory busses, tput / max; or CPI greater than, say, 5; CPC may also have local vs remote counters</td></tr>
<tr><td>Memory interconnect</td><td>saturation</td><td>LPE (CPC) for stall cycles</td></tr>
<tr><td>Memory interconnect</td><td>errors</td><td>LPE (CPC) for whatever is available</td></tr>

<tr><td>I/O interconnect</td><td>utilization</td><td>LPE (CPC) for tput / max if available; inference via known tput from iostat/ip/...</td></tr>
<tr><td>I/O interconnect</td><td>saturation</td><td>LPE (CPC) for stall cycles</td></tr>
<tr><td>I/O interconnect</td><td>errors</td><td>LPE (CPC) for whatever is available </td></tr>
</tbody></table>


### Software Resources

<table border="1" width="100%">
<tbody><tr><th>component</th><th>type</th><th>metric</th></tr>

<!--SW-START-->
<tr><td>Kernel mutex</td><td>utilization</td><td>With CONFIG_LOCK_STATS=y, /proc/lock_stat "holdtime-totat" / "acquisitions" (also see "holdtime-min", "holdtime-max") [8]; dynamic tracing of lock functions or instructions (maybe)</td></tr>
<tr><td>Kernel mutex</td><td>saturation</td><td>With CONFIG_LOCK_STATS=y, /proc/lock_stat "waittime-total" / "contentions" (also see "waittime-min", "waittime-max"); dynamic tracing of lock functions or instructions (maybe); spinning shows up with profiling (<tt>perf record -a -g -F 997 ...</tt>, <tt>oprofile</tt>, dynamic tracing)</td></tr>
<tr><td>Kernel mutex</td><td>errors</td><td>dynamic tracing (eg, recusive mutex enter); other errors can cause kernel lockup/panic, debug with kdump/<tt>crash</tt></td></tr>

<tr><td>User mutex</td><td>utilization</td><td><tt>valgrind --tool=drd --exclusive-threshold=...</tt> (held time); dynamic tracing of lock to unlock function time</td></tr>
<tr><td>User mutex</td><td>saturation</td><td><tt>valgrind --tool=drd</tt> to infer contention from held time; dynamic tracing of synchronization functions for wait time; profiling (oprofile, PEL, ...) user stacks for spins</td></tr>
<tr><td>User mutex</td><td>errors</td><td><tt>valgrind --tool=drd</tt> various errors; dynamic tracing of pthread_mutex_lock() for EAGAIN, EINVAL, EPERM, EDEADLK, ENOMEM, EOWNERDEAD, ...</td></tr>

<tr><td>Task capacity</td><td>utilization</td><td><tt>top</tt>/<tt>htop</tt>, "Tasks" (current); <tt>sysctl kernel.threads-max</tt>, /proc/sys/kernel/threads-max (max)</td></tr>
<tr><td>Task capacity</td><td>saturation</td><td>threads blocking on memory allocation; at this point the page scanner should be running (sar -B "pgscan*"), else examine using dynamic tracing</td></tr>
<tr><td>Task capacity</td><td>errors</td><td>"can't fork()" errors; user-level threads: pthread_create() failures with EAGAIN, EINVAL, ...; kernel: dynamic tracing of kernel_thread() ENOMEM</td></tr>

<tr><td>File descriptors</td><td>utilization</td><td>system-wide: <tt>sar -v</tt>, "file-nr" vs /proc/sys/fs/file-max; <tt>dstat --fs</tt>, "files"; or just /proc/sys/fs/file-nr; per-process: <tt>ls /proc/PID/fd | wc -l</tt> vs <tt>ulimit -n</tt></td></tr>
<tr><td>File descriptors</td><td>saturation</td><td>does this make sense?  I don't think there is any queueing or blocking, other than on memory allocation.</td></tr>
<tr><td>File descriptors</td><td>errors</td><td><tt>strace</tt> errno == EMFILE on syscalls returning fds (eg, open(), accept(), ...).</td></tr>
</tbody></table>

#### ulimit

ulimit 用于管理用户对系统资源的访问.

```
-a   显示目前全部限制情况
-c   设定 core 文件的最大值, 单位为区块
-d   <数据节区大小> 程序数据节区的最大值, 单位为KB
-f   <文件大小> shell 所能建立的最大文件, 单位为区块
-H   设定资源的硬性限制, 也就是管理员所设下的限制
-m   <内存大小> 指定可使用内存的上限, 单位为 KB
-n   <文件描述符数目> 指定同一时间最多可开启的 fd 数
-p   <缓冲区大小> 指定管道缓冲区的大小, 单位512字节
-s   <堆叠大小> 指定堆叠的上限, 单位为 KB
-S   设定资源的弹性限制
-t   指定CPU使用时间的上限, 单位为秒
-u   <进程数目> 用户最多可开启的进程数目
-v   <虚拟内存大小> 指定可使用的虚拟内存上限, 单位为 KB
```

例如:

```
$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 127988
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 655360
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

注意, open socket 等资源拿到的也是 fd, 所以 `ulimit -n` 比较小除了文件打不开, 还可能建立不了 socket 链接.

