---
title: 数据库表被误删了怎么办
tags:
  - 数据库
url: 226.html
id: 226
categories:
  - 数据库
date: 2019-10-31 13:24:57
---

单表被误删，oracle有个flashback特性可以帮助恢复被删除的表。

--1 模拟  删除一张测试表  
drop table TYPICAL_copy;

--2 查看回收站  
select object\_name,original\_name,type,droptime from user_recyclebin;

![](http://106.54.113.128/wordpress/wp-content/uploads/2019/10/企业微信截图_15724952788754.png)

--3 恢复表  
flashback table "BIN$PMNTTOLhQlSZKvrgid3wNg==$0" to before drop rename to TYPICAL_copy;

--4确认已经恢复  
select * from TYPICAL_copy;