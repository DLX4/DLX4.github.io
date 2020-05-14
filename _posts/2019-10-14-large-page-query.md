---
title: 利用延迟关联优化超多分页查询
url: 78.html
id: 78
categories:
  - 数据库
date: 2019-10-14 14:45:38
tags:
  - 分页
---

经常用到分页查询，一直有个问题悬而未解，不论是mysql（limit 子句）分页查询还是oracle

```sql
（select A.*, ROWNUM rn from (" + sql + ") A where ROWNUM <=:endNum) where rn >=:startNum"）
```

分页查询都会存在 随着页号变大，查询耗时显著变大的情况。





有人做了[实验验证了](https://www.cnblogs.com/allenli263/p/7351107.html)通过延迟关联可以使mysql的limit查询性能提升一个数量级。

现在还有一个问题，不知道github的pagehelper插件有没有使用这个优化技术，先挖坑，后面去确认下。