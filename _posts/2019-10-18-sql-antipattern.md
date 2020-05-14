---
title: 查询反模式
url: 140.html
id: 140
categories:
  - 数据库
date: 2019-10-18 00:29:03
tags:
  - 反模式
---

反模式：害怕使用null

建议做法：将null视为特殊值。老的SQL标准定义了一个IS NULL的断言，在一个给定的操作值为NULL时，它将返回TRUE。与之对应的，IS NOT NULL在操作值是NULL时，返回FALSE。



反模式：group语句需要遵循单值原则



反模式：

```sql
SELECT * FROM Bugs ORDER BY RAND() LIMIT 1;
```

获取随机行的性能问题。  



反模式：全文搜索。最好的方案就是使用特殊的搜索引擎技术，而不是SQL。另一个可选方案是将结果保存起来从而减少重复的搜索开销。



反模式：意大利面条式查询。拆分一个复杂的SQL查询。





反模式：select *。应该明确列出列名。