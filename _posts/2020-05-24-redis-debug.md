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

- [ ] 1、从redis-benchmark出发剖析redis性能为何如此强悍
- [ ] 2、rdb文件格式，持久化原理
- [ ] 3、从redis的104个testcase开始按照dfs顺序走读代码
- [ ] 4、redis-cli和redis-server的通信
- [ ] 5、基于redis的山寨版推特实现



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



至于save的时候做了啥，注意这里的rename操作。也就是先写到一个临时的rdb文件然后rename过去。![1590419337048](D:\code\DLX4.github.io\images\1590419337048.png)



