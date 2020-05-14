---
title: 拯救我的服务器
url: 434.html
id: 434
categories:
  - 折腾
date: 2020-03-03 12:35:05
tags:
  - 云服务器
---

因为贫穷，买了最廉价的云服务器，自从买来就问题不断真可谓多灾多难。 可怜的服务器只有1核 1G内存 120KB的带宽。 亲眼见着他一步步变卡： 



1、我先装了wordpress，mysql 

2、为了贪图好看的皮肤，参考了别人的皮肤然后定制了一下。而这个皮肤做了SSR（服务端渲染），所以在服务器上跑了node和pm2。这时候问题就出现了，yarn build或者run serve的时候内存不够报错。这时候我机智的加了swap，现在能编过了，但是node就成了CPU，内存头号玩家。 

3、装nginx代理，现在我的博客跑起来了，虽然卡但是10秒开还是阔以的 

4、又在上面部署了我的selfchat，前端是vue build出来的静态dist，后端是springboot，简单运行java -jar，有点卡，但是几十秒内内打开也还行，要啥自行车。 

5、突然有一天，变得巨卡无比，博客和selfchat都是打不开了，上去一看一个叫javaupdate的进程占用了99%的cpu。kill -9 之后马上又出来，就像九头蛇似的。最后用了杀招pkill -u 强行杀了。终于恢复了原来慢吞吞能打开。 

6、突然有一天又打不开了，一开CPU正常的没啥占用，原来java占了200多M的内存，jmap jstack一通操作之后就觉得java可疑，杀掉之后果然博客10秒能打开了。估计是swap太多了，内存透支严重，也可能java线程也太多了，反正杀了就好了。 

7、为了彻底拯救服务器，忍痛把博客皮肤重构了，不用nuxt了，也告别node和SSR了，这下果然快了很多，至于java当然要阉割。加了一堆虚拟机参数之后java也安分了： `nohup java -Xmx64m -Xms64m -XX:ParallelGCThreads=2 -Djava.compiler=NONE -XX:+PrintGCDetails -Xloggc:/home/dlx/logs/gc.log -jar ./selfchat-0.0.1-SNAPSHOT.jar &` 主要是把内存限制到了64M，限制了垃圾回收线程数，JIT线程关了，再限制了springboot的tomcat线程数。 

8、尝试过把swap分区去掉，发现swapoff报错了，可能是很多内存分到了swap收回来没地方放了，覆水难收。 经过上面一通操作之后，我终于拯救了我的服务器。现在在上面愉快的运行着：wordpress，php，mysql，nginx（两个vue项目还有wordpress，selfchat代理），springboot。