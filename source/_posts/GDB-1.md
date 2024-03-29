---
title: GDB 常用命令详解（上）
comments: true
mathjax: true
tags:
  - Linux
  - C++ 开发
  - GDB
categories:
  - Linux GDB 调试指南
date: 2021-10-10 16:16:02
---
#### 音乐小港
{% meting "532776335" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# GDB 常用命令详解（上）
本课的核心内容如下：
- run 命令
- continue 命令
- break 命令
- backtrace 与 frame 命令
- info break、enable、disable 和 delete 命令
- list 命令
- print 和 ptype 命令

为了结合实践，这里以调试 Redis 源码为例来介绍每一个命令，先介绍一些常用命令的基础用法，某些命令的高级用法会在后面讲解。

## Redis 源码下载与 debug 版本编译
Redis 的最新源码下载地址可以在 Redis 官网获得，使用 wget 命令将 Redis 源码文件下载下来：
```
[root@localhost gdbtest]# wget http://download.redis.io/releases/redis-4.0.11.tar.gz
--2018-09-08 13:08:41--  http://download.redis.io/releases/redis-4.0.11.tar.gz
Resolving download.redis.io (download.redis.io)... 109.74.203.151
Connecting to download.redis.io (download.redis.io)|109.74.203.151|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1739656 (1.7M) [application/x-gzip]
Saving to: ‘redis-4.0.11.tar.gz’

54% [==================================================================>                                                         ] 940,876     65.6KB/s  eta 9s
```
解压：
```
[root@localhost gdbtest]# tar zxvf redis-4.0.11.tar.gz
```
进入生成的 redis-4.0.11 目录使用 **makefile** 命令进行编译。**makefile** 命令是 Linux 程序编译基本的命令，由于本课程的重点是 Linux 调试，如果读者不熟悉 Linux 编译可以通过互联网或相关书籍补充一下相关知识。

为了方便调试，我们需要生成调试符号并且关闭编译器优化选项，操作如下：
```
[root@localhost gdbtest]# cd redis-4.0.11
[root@localhost redis-4.0.11]# make CFLAGS="-g -O0" -j 4
```
> 注意：由于 redis 是纯 C 项目，使用的编译器是 gcc，因而这里设置编译器的选项时使用的是 CFLAGS 选项；如果项目使用的语言是 C++，那么使用的编译器一般是 g++，相对应的编译器选项是 CXXFLAGS。这点请读者注意区别。
>
> 另外，这里 makefile 使用了 -j 选项，其值是 4，表示开启 4 个进程同时编译，加快编译速度。

编译成功后，会在 src 目录下生成多个可执行程序，其中 redis-server 和 redis-cli 是需要调试的程序。

进入 src 目录，使用 GDB 启动 redis-server 这个程序：
```
[root@localhost src]# gdb redis-server
Reading symbols from /root/gdbtest/redis-4.0.11/src/redis-server...done.
```
## run 命令

默认情况下，前面的课程中我们说 **gdb filename** 命令只是附加的一个调试文件，并没有启动这个程序，需要输入 **run** 命令（简写为 r）启动这个程序：
```
(gdb) r
Starting program: /root/gdbtest/redis-4.0.11/src/redis-server
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
46455:C 08 Sep 13:43:43.957 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
46455:C 08 Sep 13:43:43.957 # Redis version=4.0.11, bits=64, commit=00000000, modified=0, pid=46455, just started
46455:C 08 Sep 13:43:43.957 # Warning: no config file specified, using the default config. In order to specify a config file use /root/gdbtest/redis-4.0.11/src/redis-server /path/to/redis.conf
46455:M 08 Sep 13:43:43.957 * Increased maximum number of open files to 10032 (it was originally set to 1024).
[New Thread 0x7ffff07ff700 (LWP 46459)]
[New Thread 0x7fffefffe700 (LWP 46460)]
[New Thread 0x7fffef7fd700 (LWP 46461)]
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 4.0.11 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 46455
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |     http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

46455:M 08 Sep 13:43:43.965 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
46455:M 08 Sep 13:43:43.965 # Server initialized
46455:M 08 Sep 13:43:43.965 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
46455:M 08 Sep 13:43:43.965 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
46455:M 08 Sep 13:43:43.965 * Ready to accept connections
```
这就是 redis-server 启动界面，假设程序已经启动，再次输入 run 命令则是重启程序。我们在 GDB 界面按 Ctrl + C 快捷键让 GDB 中断下来，再次输入 r 命令，GDB 会询问我们是否重启程序，输入 yes 确认重启。
```
^C
Program received signal SIGINT, Interrupt.
0x00007ffff73ee923 in epoll_wait () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install glibc-2.17-196.el7_4.2.x86_64
(gdb) r
The program being debugged has been started already.
Start it from the beginning? (y or n) yes
Starting program: /root/gdbtest/redis-4.0.11/src/redis-server
```
## continue 命令
当 GDB 触发断点或者使用 Ctrl + C 命令中断下来后，想让程序继续运行，只要输入 **continue** 命令即可（简写为 c）。当然，如果 **continue** 命令继续触发断点，GDB 就会再次中断下来。
```
^C
Program received signal SIGINT, Interrupt.
0x00007ffff73ee923 in epoll_wait () from /lib64/libc.so.6
(gdb) c
Continuing.
```
## break 命令
**break** 命令（简写为 b）即我们添加断点的命令，可以使用以下方式添加断点：
- break functionname，在函数名为 functionname 的入口处添加一个断点；
- break LineNo，在当前文件行号为 LineNo 处添加一个断点；
- break filename:LineNo，在 filename 文件行号为 LineNo 处添加一个断点。
这三种方式都是我们常用的添加断点的方式。举个例子，对于一般的 Linux 程序来说，main() 函数是程序入口函数，redis-server 也不例外，我们知道了函数的名字，就可以直接在 main() 函数处添加一个断点：
```
(gdb) b main
Breakpoint 1 at 0x423450: file server.c, line 3709.
```
添加好了以后，使用 run 命令重启程序，就可以触发这个断点了，GDB 会停在断点处。
```
(gdb) r
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /root/gdbtest/redis-4.0.11/src/redis-server
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".

Breakpoint 1, main (argc=1, argv=0x7fffffffe648) at server.c:3709
3709    int main(int argc, char **argv) {
(gdb)
```
redis-server 默认端口号是 6379 ，我们知道这个端口号肯定是通过操作系统的 socket API bind() 函数创建的，通过文件搜索，找到调用这个函数的文件，其位于 anet.c 441 行。

![gdb1](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20211010/gdb1.5js4ek6xxes0.jpg)

我们使用 **break** 命令在这个地方加一个断点：
```
(gdb) b anet.c:441
Breakpoint 3 at 0x426cf0: file anet.c, line 441
```
由于程序绑定端口号是 redis-server 启动时初始化的，为了能触发这个断点，再次使用 run 命令重启下这个程序，GDB 第一次会触发 main() 函数处的断点，输入 continue 命令继续运行，接着触发 anet.c:441 处的断点：
```
(gdb) r
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /root/gdbtest/redis-4.0.11/src/redis-server
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".

Breakpoint 1, main (argc=1, argv=0x7fffffffe648) at server.c:3709
3709    int main(int argc, char **argv) {
(gdb) c
Continuing.
46699:C 08 Sep 15:30:31.403 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
46699:C 08 Sep 15:30:31.403 # Redis version=4.0.11, bits=64, commit=00000000, modified=0, pid=46699, just started
46699:C 08 Sep 15:30:31.403 # Warning: no config file specified, using the default config. In order to specify a config file use /root/gdbtest/redis-4.0.11/src/redis-server /path/to/redis.conf
46699:M 08 Sep 15:30:31.404 * Increased maximum number of open files to 10032 (it was originally set to 1024).

Breakpoint 3, anetListen (err=0x746bb0 <server+560> "", s=10, sa=0x75edb0, len=28, backlog=511) at anet.c:441
441         if (bind(s,sa,len) == -1) {
(gdb)
```
anet.c:441 处的代码如下：

![gdb2](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20211010/gdb2.1rsl9bdfej34.jpg)

现在断点停在第 441 行，所以当前文件就是 anet.c，可以直接使用“break 行号”添加断点。例如，可以在第 444 行、450 行、452 行分别加一个断点，看看这个函数执行完毕后走哪个 return 语句退出，则可以执行：
```
440     static int anetListen(char *err, int s, struct sockaddr *sa, socklen_t len, int                               backlog) {
441         if (bind(s,sa,len) == -1) {
442             anetSetError(err, "bind: %s", strerror(errno));
443             close(s);
444             return ANET_ERR;
(gdb) l
445         }
446
447         if (listen(s, backlog) == -1) {
448             anetSetError(err, "listen: %s", strerror(errno));
449             close(s);
450             return ANET_ERR;
451         }
452         return ANET_OK;
453     }
454
(gdb) b 444
Breakpoint 3 at 0x426cf5: file anet.c, line 444.
(gdb) b 450
Breakpoint 4 at 0x426d06: file anet.c, line 450.
(gdb) b 452
Note: breakpoint 4 also set at pc 0x426d06.
Breakpoint 5 at 0x426d06: file anet.c, line 452.
(gdb)
```
添加好这三个断点以后，我们使用 **continue** 命令继续运行程序，发现程序运行到第 452 行中断下来（即触发 Breakpoint 5）：
```
(gdb) c
Continuing.

Breakpoint 5, anetListen (err=0x746bb0 <server+560> "", s=10, sa=0x7e34e0, len=16, backlog=511) at anet.c:452
452         return ANET_OK;
```
说明 redis-server 绑定端口号并设置侦听（listen）成功，我们可以再打开一个 SSH 窗口，验证一下，发现 6379 端口确实已经处于侦听状态了：
```
[root@localhost src]# lsof -i -Pn | grep redis
redis-ser 46699    root   10u  IPv6 245844      0t0  TCP *:6379 (LISTEN)
```
## backtrace 与 frame 命令
**backtrace** 命令（简写为 bt）用来查看当前调用堆栈。接上，redis-server 现在中断在 anet.c:452 行，可以通过 backtrace 命令来查看当前的调用堆栈：
```
(gdb) bt
#0  anetListen (err=0x746bb0 <server+560> "", s=10, sa=0x7e34e0, len=16, backlog=511) at anet.c:452
#1  0x0000000000426e35 in _anetTcpServer (err=err@entry=0x746bb0 <server+560> "", port=port@entry=6379, bindaddr=bindaddr@entry=0x0, af=af@entry=10, backlog=511)
    at anet.c:487
#2  0x000000000042793d in anetTcp6Server (err=err@entry=0x746bb0 <server+560> "", port=port@entry=6379, bindaddr=bindaddr@entry=0x0, backlog=511)
    at anet.c:510
#3  0x000000000042b0bf in listenToPort (port=6379, fds=fds@entry=0x746ae4 <server+356>, count=count@entry=0x746b24 <server+420>) at server.c:1728
#4  0x000000000042fa77 in initServer () at server.c:1852
#5  0x0000000000423803 in main (argc=1, argv=0x7fffffffe648) at server.c:3862
(gdb)
```
这里一共有 6 层堆栈，最顶层是 main() 函数，最底层是断点所在的 anetListen() 函数，堆栈编号分别是 #0 ~ #5 ，如果想切换到其他堆栈处，可以使用 frame 命令（简写为 f），该命令的使用方法是“**frame 堆栈编号**（编号不加 #）”。在这里依次切换至堆栈顶部，然后再切换回 #0 练习一下：
```
(gdb) f 1
#1  0x0000000000426e35 in _anetTcpServer (err=err@entry=0x746bb0 <server+560> "", port=port@entry=6379, bindaddr=bindaddr@entry=0x0, af=af@entry=10, backlog=511)
    at anet.c:487
487             if (anetListen(err,s,p->ai_addr,p->ai_addrlen,backlog) == ANET_ERR) s = ANET_ERR;
(gdb) f 2
#2  0x000000000042793d in anetTcp6Server (err=err@entry=0x746bb0 <server+560> "", port=port@entry=6379, bindaddr=bindaddr@entry=0x0, backlog=511)
    at anet.c:510
510         return _anetTcpServer(err, port, bindaddr, AF_INET6, backlog);
(gdb) f 3
#3  0x000000000042b0bf in listenToPort (port=6379, fds=fds@entry=0x746ae4 <server+356>, count=count@entry=0x746b24 <server+420>) at server.c:1728
1728                fds[*count] = anetTcp6Server(server.neterr,port,NULL,
(gdb) f 4
#4  0x000000000042fa77 in initServer () at server.c:1852
1852            listenToPort(server.port,server.ipfd,&server.ipfd_count) == C_ERR)
(gdb) f 5
#5  0x0000000000423803 in main (argc=1, argv=0x7fffffffe648) at server.c:3862
3862        initServer();
(gdb)
```
通过查看上面的各个堆栈，可以得出这里的调用层级关系，即：
- main() 函数在第 3862 行调用了 initServer() 函数
- initServer() 在第 1852 行调用了 listenToPort() 函数
- listenToPort() 在第 1728 行调用了 anetTcp6Server() 函数
- anetTcp6Server() 在第 510 行调用了 _anetTcpServer() 函数
- _anetTcpServer() 函数在第 487 行调用了 anetListen() 函数
- 当前断点正好位于 anetListen() 函数中
## info break、enable、disable 和 delete 命令
在程序中加了很多断点，而我们想查看加了哪些断点时，可以使用 **info break** 命令（简写为 info b）：
```
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000423450 in main at server.c:3709
        breakpoint already hit 1 time
2       breakpoint     keep y   0x000000000049c1f0 in _redisContextConnectTcp at net.c:267
3       breakpoint     keep y   0x0000000000426cf0 in anetListen at anet.c:441
        breakpoint already hit 1 time
4       breakpoint     keep y   0x0000000000426d05 in anetListen at anet.c:444
        breakpoint already hit 1 time
5       breakpoint     keep y   0x0000000000426d16 in anetListen at anet.c:450
        breakpoint already hit 1 time
6       breakpoint     keep y   0x0000000000426d16 in anetListen at anet.c:452
        breakpoint already hit 1 time
```
通过上面的内容片段可以知道，目前一共增加了 6 个断点，除了断点 2 以外，其他的断点均被触发一次，其他信息比如每个断点的位置（所在的文件和行号）、内存地址、断点启用和禁用状态信息也一目了然。如果我们想禁用某个断点，使用“**disable 断点编号**”就可以禁用这个断点了，被禁用的断点不会再被触发；同理，被禁用的断点也可以使用“**enable 断点编号**”重新启用。
```
(gdb) disable 1
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep n   0x0000000000423450 in main at server.c:3709
        breakpoint already hit 1 time
2       breakpoint     keep y   0x000000000049c1f0 in _redisContextConnectTcp at net.c:267
3       breakpoint     keep y   0x0000000000426cf0 in anetListen at anet.c:441
        breakpoint already hit 1 time
4       breakpoint     keep y   0x0000000000426d05 in anetListen at anet.c:444
        breakpoint already hit 1 time
5       breakpoint     keep y   0x0000000000426d16 in anetListen at anet.c:450
        breakpoint already hit 1 time
6       breakpoint     keep y   0x0000000000426d16 in anetListen at anet.c:452
        breakpoint already hit 1 time
```
使用 **disable 1** 以后，第一个断点的 Enb 一栏的值由 y 变成 n，重启程序也不会再次触发：
```
(gdb) r
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /root/gdbtest/redis-4.0.11/src/redis-server
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
46795:C 08 Sep 16:15:55.681 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
46795:C 08 Sep 16:15:55.681 # Redis version=4.0.11, bits=64, commit=00000000, modified=0, pid=46795, just started
46795:C 08 Sep 16:15:55.681 # Warning: no config file specified, using the default config. In order to specify a config file use /root/gdbtest/redis-4.0.11/src/redis-server /path/to/redis.conf
46795:M 08 Sep 16:15:55.682 * Increased maximum number of open files to 10032 (it was originally set to 1024).

Breakpoint 3, anetListen (err=0x746bb0 <server+560> "", s=10, sa=0x75edb0, len=28, backlog=511) at anet.c:441
441         if (bind(s,sa,len) == -1) {
```
如果 **disable** 命令和 **enable** 命令不加断点编号，则分别表示禁用和启用所有断点：
```
(gdb) disable
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep n   0x0000000000423450 in main at server.c:3709
2       breakpoint     keep n   0x000000000049c1f0 in _redisContextConnectTcp at net.c:267
3       breakpoint     keep n   0x0000000000426cf0 in anetListen at anet.c:441
        breakpoint already hit 1 time
4       breakpoint     keep n   0x0000000000426d05 in anetListen at anet.c:444
5       breakpoint     keep n   0x0000000000426d16 in anetListen at anet.c:450
6       breakpoint     keep n   0x0000000000426d16 in anetListen at anet.c:452
(gdb) enable
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000423450 in main at server.c:3709
2       breakpoint     keep y   0x000000000049c1f0 in _redisContextConnectTcp at net.c:267
3       breakpoint     keep y   0x0000000000426cf0 in anetListen at anet.c:441
        breakpoint already hit 1 time
4       breakpoint     keep y   0x0000000000426d05 in anetListen at anet.c:444
5       breakpoint     keep y   0x0000000000426d16 in anetListen at anet.c:450
6       breakpoint     keep y   0x0000000000426d16 in anetListen at anet.c:452
(gdb)
```
使用“**delete 编号**”可以删除某个断点，如 **delete 2 3** 则表示要删除的断点 2 和断点 3：
```
(gdb) delete 2 3
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000423450 in main at server.c:3709
4       breakpoint     keep y   0x0000000000426d05 in anetListen at anet.c:444
5       breakpoint     keep y   0x0000000000426d16 in anetListen at anet.c:450
6       breakpoint     keep y   0x0000000000426d16 in anetListen at anet.c:452
```
同样的道理，如果输入 delete 不加命令号，则表示删除所有断点。

## list 命令
**list** 命令和后面介绍的 **print** 命令都是 GDB 调试中用到的频率最高的命令，**list** 命令（简写为 l）可以查看当前断点处的代码。使用 **frame** 命令切换到刚才的堆栈 #3 处，然后输入 **list** 命令看下效果：
```
(gdb) f 4
#4  0x000000000042fa77 in initServer () at server.c:1852
1852            listenToPort(server.port,server.ipfd,&server.ipfd_count) == C_ERR)
(gdb) l
1847        }
1848        server.db = zmalloc(sizeof(redisDb)*server.dbnum);
1849
1850        /* Open the TCP listening socket for the user commands. */
1851        if (server.port != 0 &&
1852            listenToPort(server.port,server.ipfd,&server.ipfd_count) == C_ERR)
1853            exit(1);
1854
1855        /* Open the listening Unix domain socket. */
1856        if (server.unixsocket != NULL) {
(gdb)
```
断点停在第 1852 行，输入 list 命令以后，会显示第 1852 行前后的 10 行代码，再次输入 list 命令试一下：
```
(gdb) l
1857            unlink(server.unixsocket); /* don't care if this fails */
1858            server.sofd = anetUnixServer(server.neterr,server.unixsocket,
1859                server.unixsocketperm, server.tcp_backlog);
1860            if (server.sofd == ANET_ERR) {
1861                serverLog(LL_WARNING, "Opening Unix socket: %s", server.neterr);
1862                exit(1);
1863            }
1864            anetNonBlock(NULL,server.sofd);
1865        }
1866
(gdb) l
1867        /* Abort if there are no listening sockets at all. */
1868        if (server.ipfd_count == 0 && server.sofd < 0) {
1869            serverLog(LL_WARNING, "Configured to not listen anywhere, exiting.");
1870            exit(1);
1871        }
1872
1873        /* Create the Redis databases, and initialize other internal state. */
1874        for (j = 0; j < server.dbnum; j++) {
1875            server.db[j].dict = dictCreate(&dbDictType,NULL);
1876            server.db[j].expires = dictCreate(&keyptrDictType,NULL);
```
代码继续往后显示 10 行，也就是说，第一次输入 **list** 命令会显示断点处前后的代码，继续输入 **list** 指令会以递增行号的形式继续显示剩下的代码行，一直到文件结束为止。当然 list 指令还可以往前和往后显示代码，命令分别是“**list + **（加号）”和“**list - **（减号）”：
```
(gdb) list -
1857            unlink(server.unixsocket); /* don't care if this fails */
1858            server.sofd = anetUnixServer(server.neterr,server.unixsocket,
1859                server.unixsocketperm, server.tcp_backlog);
1860            if (server.sofd == ANET_ERR) {
1861                serverLog(LL_WARNING, "Opening Unix socket: %s", server.neterr);
1862                exit(1);
1863            }
1864            anetNonBlock(NULL,server.sofd);
1865        }
1866
(gdb) l -
1847        }
1848        server.db = zmalloc(sizeof(redisDb)*server.dbnum);
1849
1850        /* Open the TCP listening socket for the user commands. */
1851        if (server.port != 0 &&
1852            listenToPort(server.port,server.ipfd,&server.ipfd_count) == C_ERR)
1853            exit(1);
1854
1855        /* Open the listening Unix domain socket. */
1856        if (server.unixsocket != NULL) {
```
**list** 默认显示多少行可以通过修改相关的 GDB 配置，由于我们一般不会修改这个默认显示行数，这里就不再浪费篇幅介绍了。**list** 不仅可以显示当前断点处的代码，也可以显示其他文件某一行的代码，更多的用法可以在 GDB 中输入 **help list** 查看（也可以通过）：
```
(gdb) help list
List specified function or line.
With no argument, lists ten more lines after or around previous listing.
"list -" lists the ten lines before a previous ten-line listing.
One argument specifies a line, and ten lines are listed around that line.
Two arguments with comma between specify starting and ending lines to list.
Lines can be specified in these ways:
  LINENUM, to list around that line in current file,
  FILE:LINENUM, to list around that line in that file,
  FUNCTION, to list around beginning of that function,
  FILE:FUNCTION, to distinguish among like-named static functions.
  *ADDRESS, to list around the line containing that address.
With two args if one is empty it stands for ten lines away from the other arg.
```
上面的帮助信息中，介绍了可以使用 **list FILE:LINENUM** 来显示某个文件的某一行处的代码，这里不再演示是因为我觉得实用性不大。使用 GDB 的目的是调试，因此更关心的是断点附近的代码，而不是通过 GDB 阅读代码，GDB 并不是一个好的阅读工具。以我自己为例，调试 Redis 时用 GDB 调试，而阅读代码使用的却是 Visual Studio，如下图所示：

![gdb3](https://cdn.jsdelivr.net/gh/1258360186/image_hosting@master/20211010/gdb3.7hywlopi9fo0.jpg)

## print 和 ptype 命令
通过 **print** 命令（简写为 p）我们可以在调试过程中方便地查看变量的值，也可以修改当前内存中的变量值。切换当前断点到堆栈 #4 ，然后打印以下三个变量。
```
(gdb) bt
#0  anetListen (err=0x746bb0 <server+560> "", s=10, sa=0x7e34e0, len=16, backlog=511) at anet.c:447
#1  0x0000000000426e35 in _anetTcpServer (err=err@entry=0x746bb0 <server+560> "", port=port@entry=6379, bindaddr=bindaddr@entry=0x0, af=af@entry=10, backlog=511)
    at anet.c:487
#2  0x000000000042793d in anetTcp6Server (err=err@entry=0x746bb0 <server+560> "", port=port@entry=6379, bindaddr=bindaddr@entry=0x0, backlog=511)
    at anet.c:510
#3  0x000000000042b0bf in listenToPort (port=6379, fds=fds@entry=0x746ae4 <server+356>, count=count@entry=0x746b24 <server+420>) at server.c:1728
#4  0x000000000042fa77 in initServer () at server.c:1852
#5  0x0000000000423803 in main (argc=1, argv=0x7fffffffe648) at server.c:3862
(gdb) f 4
#4  0x000000000042fa77 in initServer () at server.c:1852
1852            listenToPort(server.port,server.ipfd,&server.ipfd_count) == C_ERR)
(gdb) l
1847        }
1848        server.db = zmalloc(sizeof(redisDb)*server.dbnum);
1849
1850        /* Open the TCP listening socket for the user commands. */
1851        if (server.port != 0 &&
1852            listenToPort(server.port,server.ipfd,&server.ipfd_count) == C_ERR)
1853            exit(1);
1854
1855        /* Open the listening Unix domain socket. */
1856        if (server.unixsocket != NULL) {
(gdb) p server.port
$15 = 6379
(gdb) p server.ipfd
$16 = {0 <repeats 16 times>}
(gdb) p server.ipfd_count
$17 = 0
```
这里使用 **print** 命令分别打印出 server.port 、server.ipfd 、server.ipfd_count 的值，其中 server.ipfd 显示 “{0 \}”，这是 GDB 显示字符串或字符数据特有的方式，当一个字符串变量或者字符数组或者连续的内存值重复若干次，GDB 就会以这种模式来显示以节约空间。

**print** 命令不仅可以显示变量值，也可以显示进行一定运算的表达式计算结果值，甚至可以显示一些函数的执行结果值。

举个例子，我们可以输入 **p &server.port** 来输出 server.port 的地址值，如果在 C++ 对象中，可以通过 p this 来显示当前对象的地址，也可以通过 p *this 来列出当前对象的各个成员变量值，如果有三个变量可以相加（ 假设变量名分别叫 a、b、c ），可以使用 **p a + b + c** 来打印这三个变量的结果值。

假设 func() 是一个可以执行的函数，p func() 命令可以输出该变量的执行结果。举一个最常用的例子，某个时刻，某个系统函数执行失败了，通过系统变量 errno 得到一个错误码，则可以使用 p strerror(errno) 将这个错误码对应的文字信息打印出来，这样就不用费劲地去 man 手册上查找这个错误码对应的错误含义了。

print 命令不仅可以输出表达式结果，同时也可以修改变量的值，我们尝试将上文中的端口号从 6379 改成 6400 试试：
```
(gdb) p server.port=6400
$24 = 6400
(gdb) p server.port
$25 = 6400
(gdb)
```
当然，一个变量值修改后能否起作用要看这个变量的具体位置和作用，举个例子，对于表达式 int a = b / c ; 如果将 c 修改成 0 ，那么程序就会产生除零异常。再例如，对于如下代码：
```
int j = 100;
for (int i = 0; i < j; ++i) {
    printf("i = %d\n", i);
}
```
如果在循环的过程中，利用 **print** 命令将 j 的大小由 100 改成 1000 ，那么这个循环将输出 i 的值 1000 次。

总结起来，利用 **print** 命令，我们不仅可以查看程序运行过程中的各个变量的状态值，也可以通过临时修改变量的值来控制程序的行为。

GDB 还有另外一个命令叫 **ptype** ，顾名思义，其含义是“print type”，就是输出一个变量的类型。例如，我们试着输出 Redis 堆栈 #4 的变量 server 和变量 server.port 的类型：
```
(gdb) ptype server
type = struct redisServer {
    pid_t pid;
    char *configfile;
    char *executable;
    char **exec_argv;
    int hz;
    redisDb *db;
    ...省略部分字段...
(gdb) ptype server.port
type = int
```
可以看到，对于一个复合数据类型的变量，ptype 不仅列出了这个变量的类型（ 这里是一个名叫 redisServer 的结构体），而且详细地列出了每个成员变量的字段名，有了这个功能，我们在调试时就不用刻意去代码文件中查看某个变量的类型定义了。

## 小结
### 本节课介绍了 **run、continue、break、backtrace、frame、info break、list、print** 和 **ptype** 等命令，这些都是 GDB 调试过程中非常常用的命令，尤其是一些复合命令（如 info、break）是调试多线程程序的核心命令，请读者务必掌握。

---
来源:[范蠡《Linux GDB 调试指南》](https://gitbook.cn/gitchat/column/5c0e149eedba1b683458fd5f)