---
title: redis 1.0版本源码
tags:
  - redis
categories:
  - 编程
  - 源码
date: 2020-05-24 23:39:03
---

# 在线debug/分析redis 1.0 源码

- [x] 1、benchmark分析redis对于性能很佛系，基于单核的event loop。
- [x] 2、rdb文件格式，了解持久化原理
- [ ] 3、从redis的104个testcase开始按照dfs顺序走读代码
- [x] 4、redis的事件循环，了解redis-cli和redis-server的通信
- [ ] 5、彩蛋，基于redis的山寨版推特实现
- [ ] 6、master slave的实现原理

## RDB文件格式

```
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1  flushall
OK
[dlx@localhost redis-1.0]$ 
[dlx@localhost redis-1.0]$ 
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 set key1 abc
OK
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 set key2 abc
OK
[dlx@localhost redis-1.0]$ 
[dlx@localhost redis-1.0]$ 
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 rpush key3 1
OK
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 rpush key3 2
OK
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 rpush key3 3
OK
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 llen key3
3
[dlx@localhost redis-1.0]$ 
[dlx@localhost redis-1.0]$ 
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 sadd key4 a
1
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 sadd key4 b
1
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 sadd key4 a
0
[dlx@localhost redis-1.0]$ 
[dlx@localhost redis-1.0]$ ./redis-cli  -h 127.0.0.1 keys key*
key2 key3 key4 key1
[dlx@localhost redis-1.0]$ save
```



如下图所示，上面这一通操作之后rdb文件中保存了4个key以及对应的数据。

```
  Offset: 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 	
00000000: 52 45 44 49 53 30 30 30 31{FE}[00]00 04 6B 65 79    REDIS0001~...key
00000010: 32 03 61 62 63[01]04 6B 65 79 33 03 C0 01 C0 02    2.abc..key3.@.@.
00000020: C0 03[02]04 6B 65 79 34 02 01 61 01 62 00 04 6B    @...key4..a.b..k
00000030: 65 79 31 03 61 62 63{FF}                            ey1.abc.
```

上面中括号括起来的字段标记了数据类型。1.0版本只有string，list，set三种数据类型。上面还标记了两处大括号括起来的标记，用来标记是SELECTDB的东西还是EXPIRETIME相关的东西，以及EOF。

具体怎么解析的，大家看代码就了然了 `static int rdbLoad(char *filename)`

```c
/* Object types */
#define REDIS_STRING 0
#define REDIS_LIST 1
#define REDIS_SET 2
#define REDIS_HASH 3

/* Object types only used for dumping to disk */
#define REDIS_EXPIRETIME 253
#define REDIS_SELECTDB 254
#define REDIS_EOF 255
```

至于save的时候做了啥，注意这里的`rename`操作。也就是先写到一个临时的rdb文件然后rename过去。![1590419337048](https://raw.githubusercontent.com/DLX4/DLX4.github.io/master/images/1590419337048.png)



## redis的事件循环

redis里面的事件分为两类：文件事件和时间事件。

```c
/* State of an event based program */
typedef struct aeEventLoop {
    long long timeEventNextId;
    aeFileEvent *fileEventHead;
    aeTimeEvent *timeEventHead;
    int stop;
} aeEventLoop;
```

### 文件事件（file event）

这里的文件就是1 + N个fd。即1个redis server的fd（会触发accept事件），以及N个redis client连接的fd（会触发read和reply事件）。



文件事件本质上是文件的读写事件，是通过select系统调用获取的。

```c
#include <sys/select.h>

int select(int nfds, fd_set *readfds, fd_set *writefds,
                  fd_set *exceptfds, struct timeval *timeout);

select() allows a program to monitor multiple file descriptors,
       waiting until one or more of the file descriptors become "ready" for
       some class of I/O operation (e.g., input possible).  A file
       descriptor is considered ready if it is possible to perform a
       corresponding I/O operation (e.g., read(2), or a sufficiently small
       write(2)) without blocking.
```

![1590513741792](https://raw.githubusercontent.com/DLX4/DLX4.github.io/master/images/1590513741792.png)



事件都有对应的事件处理函数，可以在对应的事件handler中加断点，观察事件时怎么被处理的。

情况1：读请求内容（read）

![1590426188384](https://raw.githubusercontent.com/DLX4/DLX4.github.io/master/images/1590426188384.png)



情况2：接受请求（accept）

![1590426265615](https://raw.githubusercontent.com/DLX4/DLX4.github.io/master/images/1590426265615.png)

情况3：回复（reply）

![1590426333167](C:\Users\ADMINI~1\AppData\Local\Temp\1590426333167.png)



Accept处理过程中会衍生出Read事件（因为**接受了客户端请求，紧接着就是读取客户端请求内容**），如下图所示，注意下图中的handler为`readQueryFromClient`。

![1590427918273](https://raw.githubusercontent.com/DLX4/DLX4.github.io/master/images/1590427918273.png)

Read事件的处理过程中又会衍生出Reply事件（因为**读取请求了，紧接着就是执行请求内容，紧接着考虑是否要回复结果**）。

我们随便选择一个命令（比如dbsize），从调用栈能够清晰看出执行的流程。

![1590503934322](https://raw.githubusercontent.com/DLX4/DLX4.github.io/master/images/1590503934322.png)

可以看出`sendReplyToClient`将会在另外单独的一个Event中进行处理。事实上也确实如此：

![1590426916735](https://raw.githubusercontent.com/DLX4/DLX4.github.io/master/images/1590426916735.png)

从调用栈可以发现有个叫`redisClient`的数据结构，在`processEvents`之后的重重调用中都充当着入参的角色，这个玩意应该就是服务端给每个client存的会话数据了。

`aeEventLoop`这个数据结构就是事件列表了，某一时刻redis的所有事件（accept，read，reply，time）都挂在这里面。

这4种事件又分为两类：

- file event（数据结构为`aeFileEvent`）：包括accept，read，reply
- time event（数据结构为`aeTimeEvent`）：只有time

显然read，reply这些事件是关联到某个client的（因此`aeFileEvent.clientData`指向某个redisClient）。并且read，reply事件在handler处理结束之后就会从事件列表中删除（通过指定redisClient的fd从列表中删除相同fd的事件）。

accept事件是redis server初始化的时候加入到事件列表中的，fd为server的fd（`server.fd = anetTcpServer(server.neterr, server.port, server.bindaddr);`），因此accept事件将会一直存在与列表中，从始至终。

![1590427569512](https://raw.githubusercontent.com/DLX4/DLX4.github.io/master/images/1590427569512.png)



### 时间事件（time event）

可以搜索相关代码 `aeCreateTimeEvent(server.el, 1000, serverCron, NULL, NULL);`  主要是redis sever的定时任务。

## benchmark

redis benchmark的代码也是基于上面的事件循环写的。先构造若干个client，同时加入到event loop列表，循环N次之后通过stop标记停止event loop。（注意：benchmark的时候因为是模拟压测，因此没有删除事件的操作）。

redis本身叫做远程字典服务（远程+字典）目前看来redis使用事件循环来实现了所谓的远程，即多客户端并发连接操作。

显然，单线程的event loop再牛皮也只能把一个cpu核跑满，虽然现在吞吐可以跑到300000 requests per second。

**所以redis显然不是从极致性能出发去设计的。**

```
====== SET ======
  10004 requests completed in 0.03 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

83.90% <= 0 milliseconds
100.00% <= 1 milliseconds
303151.53 requests per second

====== GET ======
  10013 requests completed in 0.04 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

80.81% <= 0 milliseconds
100.00% <= 1 milliseconds
250325.00 requests per second

====== INCR ======
  10004 requests completed in 0.04 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

81.07% <= 0 milliseconds
100.00% <= 1 milliseconds
263263.16 requests per second

====== LPUSH ======
  10006 requests completed in 0.04 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

82.06% <= 0 milliseconds
100.00% <= 1 milliseconds
277944.47 requests per second

====== LPOP ======
  10005 requests completed in 0.04 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

82.12% <= 0 milliseconds
100.00% <= 1 milliseconds
277916.69 requests per second
```

## testcase



