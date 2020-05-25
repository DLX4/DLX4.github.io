---
title: vscode的remote功能
tags:
  - vscode
  - remote
categories:
  - 编程工具
  - 自言自语
date: 2020-05-23 21:58:03
---

今天对redis进行了考古，从google source上下了redis 1.0版本的源码。



这是一个pure C的项目（完全用C语言编写的），1.0版本所有的源码文件都在一个目录底下，包括一个Makefile，几个C源文件（不超过10个），还有其它的若干个源代码文件。



惊讶于现如今这个家喻户晓的甚至最出名的中间件的源代码实现居然这么简洁。



和往常一样准备hack一下redis的源码，单步调试什么的把玩把玩。



该用什么IDE好呢，显然我没法在windows下执行各种make操作（其实是可以的，基于mingw的环境，参考QT的项目就使用qmake或者cmake实现了跨平台的编译运行）。但是我不想惹事，最直接的还是在Linux环境下编译运行。但是Linux下的C语言IDE使用体验都不佳（我试过qtcreator，atom）。说起最好用的IDE，我心目中当然是vscode和JB全家桶啦，对于C语言的项目当然首选vscode。btw，现在是2020年啦。

当年我还是一个嵌入式Linux驱动程序员的时候，菊厂人人都用的SI（source insight），似乎这个圈子的程序员都用这东西（中兴也用）。后来听说菊厂又不用SI，现在用wecode，据说和vscode几乎一样。我个人揣测应该是个vscode抄的，但是这是合法的，多亏了微软大善人开源了vscode，许可证还是MIT的，因此菊厂可以抄并且名字都可以碰瓷，比如叫vscodex, hwcode。美帝的公司真是良心啊，JB的IDEA也有开源的社区版，许可证是Apache2.0的。

如果代码需要在Linux上运行，IDE又装在windows开发机上，这个咋办呢？在菊厂的时候通常的做法是在Linux上配samba，然后windows上把Linux上的整个目录映射成驱动器，这种做法没啥毛病，不过SI在建索引的时候经常会比较卡。

突然哪里看到vscode有个remote模式，于是试了下，确实很好用。

![](..\images\2020-05-23-vscode-remote-0.png)

比较可以爽快的看代码，改代码，还能在terminal里面执行任意命令。

真是66啊，以后可以考虑把代码库搞到服务上直接在服务器上开发了，想到这里我的2000多RMB的I7 8700 CPU突然不香了。







