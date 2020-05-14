---
title: 如何聚合多张业务大表生成大宽表
url: 417.html
id: 417
categories:
  - 数据库
date: 2019-12-31 16:50:57
tags:
  - 物化视图
  - cdc
  - 宽表
---

**背景：**某业务场景的查询需求，需要关联6张业务大表，单表数据量亿级，原方案使用oracle的物化视图方案。后来由于各种原因，oracle不让用了，需要基于云上资源（rds-postgresql等）来实现同样需求，重点优化查询性能。有什么好的方法呢？

 **方法1：**使用阿里云的流批计算神器blink，只需要写出dml语句（类似insert into xxx values select xx from table1 join table2 join table3...） 还有没有其它的方法呢？



下面介绍一种基于CDC的大宽表生产方案。 **关键词**：CDC（change data capture） 

**方案示意图：** ![](http://106.54.113.128/wordpress/wp-content/uploads/2019/12/cdc.png) ![](http://106.54.113.128/wordpress/wp-content/uploads/2019/12/cdc1.png) ![](http://106.54.113.128/wordpress/wp-content/uploads/2019/12/cdc2.png) **实现原理（基本）：** 

1、 针对PostgreSQL有成熟的方案来实现CDC（change data capture）\[2\]，用来捕获数据库指定表的变更流（INSERT OR UPDATE OR DELETE OR TRUNCATE），其中一种方法就是使用audit trigger 开源触发器（https://raw.githubusercontent.com/2ndQuadrant/audit-trigger/master/audit.sql） 

2、 将指定表的change写入到一张CDC表中，保存变更流。 

3、 application或者ETL通过查询CDC表，根据应用的逻辑来更新宽表数据。 

4、 以上步骤全部都是实时触发的（而物化视图在数据实时性，性能，scale之间无法得兼） 



**实现原理（进阶优化1）** 

5、由于触发器对业务数据库的影响（性能、稳定性、维护性），考虑配置一个REPLICA数据库，订阅复制宽表用到的原表，并且在REPLICA数据库上设置触发器，生成CDC表， 

6、application或者ETL通过查询REPLICA数据库的CDC表，更新宽表数据。 

7、也可以不用6的方案，而直接在REPLICA数据库的CDC表上编写触发器来更新宽表数据。 **实现原理（进阶优化2）** 

8、宽表的查询性能必然会是瓶颈，REPLICA数据库的宽表可以考虑只写数据，可以再配置多个RDS-PG实例，只复制并保存一张宽表，分发承受宽表的查询请求。 



**实现工作量：** 

1、 需要两套RDS-PG环境，一个作为主库存放全量数据，一个作为REPLICA库存放宽表，宽表相关源表，CDC表。RDS-PG上配置逻辑订阅。（工作量极小1%） 

2、 编写application轮询CDC表，更新宽表数据。同时处理多张大表的修改并更新到宽表，这里可能是性能瓶颈。Application的输入数据只需要CDC表数据，字典表数据（可缓存），理论上只需要读写一张宽表（可以预测读写宽表的时候数据局部性较强，被修改的警单大部分都集中在某个时间段，因此数据库缓存可以充分利用）。此外还可以多线程按照行政区划map并行处理来更新宽表。（工作量99%） **可行性：** 1、 性能：CDC表的数据被处理过之后失去时效性可以删除了，因此不会占用太多空间；replica数据库的JJD，FKD，CJD表只是用来trigger，较短时效之后就可以TRUNCATE掉，因此占用空间也将几乎为0；application更新宽表数据理论上可以并行处理； 2、 扩展性：上述方案有非常棒的扩展性，后续假如要新增一张新表的字段到宽表中，只需要在replica库上新增一张源表的trigger，同时修改application上线即可。 

3、 宽表对主业务库没有任何影响，只需要单独配置一套RDS-PG资源。

 **CDC表结构示意图：** ![](http://106.54.113.128/wordpress/wp-content/uploads/2019/12/cdc-table.png) **参考资料：** 1、https://medium.com/@ramesh.esl/change-data-capture-cdc-in-postgresql-7dee2d467d1b 2、（audit logging）https://severalnines.com/database-blog/postgresql-audit-logging-best-practices