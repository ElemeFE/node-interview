# OS

* `[Doc]` TTY
* `[Doc]` OS (Operating System)
* `[Doc]` Command Line Options
* `[Basic]` Load
* `[Point]` CheckList
* `[Basic]` Indicators

## TTY

"TTY" means "teletype", a typewriter, and "pty" is "pseudo-teletype", a pseudo typewriter. In Unix, `/dev/tty*` refers to any device that acts as a typewriter, such as the terminal.

You can view the currently logged in user through the `w` command, and you'll find a new tty every time you login to a window.

```shell
$ w
 11:49:43 up 482 days, 19:38,  3 users,  load average: 0.03, 0.08, 0.07
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
dev      pts/0    10.0.128.252     10:44    1:01m  0.09s  0.07s -bash
dev      pts/2    10.0.128.252     11:08    2:07   0.17s  0.14s top
root     pts/3    10.0.240.2       11:43    7.00s  0.04s  0.00s w
```

Using the ps command to see process information, there is also information about tty:

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

The process marked with `?` is not depending on TTY, which is called [Daemon](/sections/en-us/process.md#%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B).

In Node.js, you can use stdio's isTTY attribute to determine whether the current process is in a TTY (such as terminal) environment.

```shell
$ node -p -e "Boolean(process.stdout.isTTY)"
true
$ node -p -e "Boolean(process.stdout.isTTY)" | cat
false
```

## OS

You can get some auxiliary functions to the basic information of the current system through the OS module.

|Attribute|Description|
|---|---|
|os.EOL|Returns the current system's `End Of Line`, based on the current system|
|os.arch()|Returns the CPU architecture of the current system, such as `'x86'` or `'x64'`|
|os.constants|Returns system constants|
|os.cpus()|Returns the information for each kernel of the CPU|
|os.endianness()|Returns byte order of CPU, return `BE` if it is big endian, return `LE` if it is little endian.|
|os.freemem()|Returns the size of the system's free memory, in bytes|
|os.homedir()|Returns the root directory of the current user|
|os.hostname()|Returns the hostname of the current system|
|os.loadavg()|Returns load information|
|os.networkInterfaces()|Returns the NIC information (similar to `ifconfig`)|
|os.platform()|Returns the platform information specified at compile time, such as `win32`, `linux`, same as `process.platform()`|
|os.release()|Returns the distribution version number of the operating system|
|os.tmpdir()|Returns the default temporary folder of the system|
|os.totalmem()|Returns the total memory size (the same as the memory bar size)|
|os.type()|Returns the name of the system according to [`uname`](https://en.wikipedia.org/wiki/Uname#Examples)|
|os.uptime()|Returns the running time of the system, in seconds|
|os.userInfo([options])|Returns the current user information|

> What's the difference between the line breaks (EOL) of different operating systems?

End of line (EOL) is the same as newline, line ending and line break.

And it's usually composed of line feed (LF, `\n`) and carriage return (CR, `\r`). Here are some common cases:

|Symbol|System|
|---|---|
|LF|In Unix or Unix compatible systems (GNU/Linux, AIX, Xenix, Mac OS X, ...), BeOS, Amiga, RISC OS|
|CR+LF|MS-DOS, Microsoft Windows, Most non Unix systems|
|CR|Apple II family, Mac OS to version 9|

If you don't understand the cross-system compatibility of EOL, you might have problems dealing with the line segmentation/row statistics of the file.

### OS Constants

* Signal Constants, such as `SIGHUP`, `SIGKILL`, etc.
* POSIX Error Constants, such as `EACCES`, `EADDRINUSE`, etc.
* Windows Specific Error Constants, such as `WSAEACCES`, `WSAEBADF`, etc.
* libuv Constants, only `UV_UDP_REUSEADDR`.


## Path

The built-in path in Node.js is a module for handling path problems, but as we all know, the paths are irreconcilable in different operating systems.

### Windows vs. POSIX

|POSIX|Value|Windows|Value|
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


After looking at the table, you should realize that when under a certain platform, the `path` module is actually the method of the corresponding platform. For example, I uses Mac here, so:

```javascript
const path = require('path');
console.log(path.basename === path.posix.basename); // true
```

If you are on one of these platforms, but you need to deal with the path of another platform, you need to be aware of this cross platform issue.

### path Object

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


## Command Line Options

Command Line Options is some documentation on the use of CLI. There are 4 main ways of using CLI:

* node [options] [v8 options] [script.js | -e "script"] [arguments]
* node debug [script.js | -e "script" | <host>:<port>] …
* node --v8-options
* Starts REPL environment without parameters directly

### Options

|Parameter|Introduction|
|---|---|
|-v, --version|Shows the version of current node|
|-h, --help|Shows help documentation|
|-e, --eval "script"|The parameter string is executed as code
|-p, --print "script"|Prints the return value of `-e`
|-c, --check|Checks syntax without executing the code
|-i, --interactive|Opens REPL mode even if stdin is not the terminal
|-r, --require module|`require` the Specified module before startup
|--no-deprecation|Closes the scrap module warning
|--trace-deprecation|Prints stack trace information for an obsolete module
|--throw-deprecation|Throws errors while executing an obsolete module
|--no-warnings|Ignores warnings (including obsolete warnings)
|--trace-warnings|Prints warning stack (including discarded modules)
|--trace-sync-io|As soon as the asynchronous I/O is detected at the beginning of the event loop, the stack trace will be printed
|--zero-fill-buffers|Zero-fill **Buffer** and **SlowBuffer**
|--preserve-symlinks|Instructs the module loader to save symbolic links when parsing and caching modules
|--track-heap-objects|Tracks the allocation of heap objects for heap snapshot
|--prof-process|Using the V8 option `--prof` to generate the Profilling Report
|--v8-options|Shows the V8 command line options
|--tls-cipher-list=list|Specifies the list of alternative default TLS encryption devices
|--enable-fips|Turns on FIPS-compliant crypto at startup
|--force-fips|Enforces FIPS-compliant at startup
|--openssl-config=file|Loads the OpenSSL configuration file at startup
|--icu-data-dir=file|Specifies the loading path of ICU data

### Environment Variable

|Environment variable|Introduction|
|----|----|
|`NODE_DEBUG=module[,…]`|Specifies the list of core modules to print debug information
|`NODE_PATH=path[:…]`|Specifies prefix list of the module search directory
|`NODE_DISABLE_COLORS=1`|Closes the color display for REPL
|`NODE_ICU_DATA=file`|ICU (Intl, object) data path
|`NODE_REPL_HISTORY=file`|Path of persistent storage REPL history file
|`NODE_TTY_UNSAFE_ASYNC=1`|When set to 1, The stdio operation will proceed synchronously (such as console.log becomes synchronous)
|`NODE_EXTRA_CA_CERTS=file`|Specifies an extra certificate path for CA (such as VeriSign)

## Load

Load is an important concept to measure the running state of the server. Through the load situation, we can know whether the server is idle, good, busy or about to crash.

Typically, the load we want to look at is the CPU load, for more information you can read this blog: [Understanding Linux CPU Load](http://blog.scoutapp.com/articles/2009/07/31/understanding-load-averages).

To get the current system load, you can use `uptime`, `top` command in terminal or `os.loadavg()` in Node.js:

```
load average: 0.09, 0.05, 0.01
```

Here are the average load on the system of the last 1 minutes, 5 minutes, 15 minutes. When one of the CPU core is working in full load, the value of load will be 1, so the value represents how many CPU cores are in full load.

In Node.js, the CPU load of a single process can be viewed using the [pidusage](https://github.com/soyuka/pidusage) module.

In addition to the CPU load, the server (prefer maintain) needs to know about the network load, disk load, and so on.

## CheckList

> A police officer sees a drunken man intently searching the ground near a lamppost and asks him the goal of his quest. The inebriate replies that he is looking for his car keys, and the officer helps for a few minutes without success then he asks whether the man is certain that he dropped the keys near the lamppost.   
“No,” is the reply, “I lost the keys somewhere across the street.” “Why look here?” asks the surprised and irritated officer. “The light is much better here,” the intoxicated man responds with aplomb.

When it comes to checking server status, many server-side friends only know how to use the `top` command. In fact, the situation is the same as the jokes above, because `top` is the brightest street lamp for them.

For server-side programmers, the full server-side checklist is the [USE Method](http://www.brendangregg.com/USEmethod/use-linux.html) described in the second chapter of [《Systems Performance》](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/0133390098).

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

ulimit is used to manage user access to system resources.

```
-a   All current limits are reported
-c   The maximum size of core files created, take block as a unit
-d   <Data segment size> The maximum size of a process's data segment, take KB as a unit
-f   <File size> The maximum size of files written by the shell and its children, take block as a unit
-H   Set a hard limit to the resource, that is the limits set by the administrator
-m   <Memory size> The maximum resident set size, take KB as a unit
-n   <Number of file descriptors> The maximum number of open file descriptors at the same time
-p   <Cache size> The pipe size in 512-byte blocks, take 512-byte as a unit
-s   <Stack size> The maximum stack size, take KB as a unit
-S   Set flexible limits for resources
-t   The maximum amount of cpu time, in seconds
-u   <Number of processes> The maximum number of processes available to a single user
-v   <Virtual memory size> The maximum amount of virtual memory available to the shell, take KB as a unit
```

For example:

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

Note: open socket and other resources are also kind of file descriptor, if `ulimit -n` is too small, not only will you not open the file, but also can not establish a socket link.