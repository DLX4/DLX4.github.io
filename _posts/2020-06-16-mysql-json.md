---
title: 孤陋寡闻了，mysql居然也原生支持json
tags:
  - mysql
  - json
  - nosql
categories:
  - 编程
date: 2020-06-16 00:00:00
---

我对mysql了解的太少了，连mysql版本特性的changelog都不知道的。

> 2015年，**MySQL 5.7**发布，其包括如下重要特性及更新。
>
> ...
>
> **原生支持JSON类型，并引入了众多JSON函数。**
>
> ...



## Sql 语法

试一试，建一个表，插入几条数据。

```sql
CREATE TABLE test (
    `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    `category` JSON,
    `tags` JSON,
    PRIMARY KEY (`id`)
);

mysql> desc test
    -> ;
+----------+------------------+------+-----+---------+----------------+
| Field    | Type             | Null | Key | Default | Extra          |
+----------+------------------+------+-----+---------+----------------+
| id       | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| category | json             | YES  |     | NULL    |                |
| tags     | json             | YES  |     | NULL    |                |
+----------+------------------+------+-----+---------+----------------+

 INSERT INTO `test` (category, tags) VALUES ('{"id": 1, "name": "dlx"}', '[1, 2, 3]');
 INSERT INTO `test` (category, tags) VALUES (JSON_OBJECT("id", 2, "name", "gd");
```



看一下数据：

```sql
mysql> select * from test
    -> ;
+----+--------------------------+-----------+
| id | category                 | tags      |
+----+--------------------------+-----------+
|  1 | {"id": 1, "name": "dlx"} | [1, 2, 3] |
|  2 | {"id": 2, "name": "gd"}  | [1, 3, 5] |
+----+--------------------------+-----------+
```



查一查，select指定字段：

```sql
mysql>  SELECT id, category->'$.id', category->'$.name', tags->'$[0]', tags->'$[2]' FROM test;
+----+------------------+--------------------+--------------+--------------+
| id | category->'$.id' | category->'$.name' | tags->'$[0]' | tags->'$[2]' |
+----+------------------+--------------------+--------------+--------------+
|  1 | 1                | "dlx"              | 1            | 3            |
|  2 | 2                | "gd"               | 1            | 5            |
+----+------------------+--------------------+--------------+--------------+
```

按照json条件查：

```sql
mysql>  SELECT * FROM test WHERE category = CAST('{"id": 1, "name": "dlx"}' as JSON);
+----+--------------------------+-----------+
| id | category                 | tags      |
+----+--------------------------+-----------+
|  1 | {"id": 1, "name": "dlx"} | [1, 2, 3] |
+----+--------------------------+-----------+
```

按照某个json属性查：

```sql
mysql>  SELECT * FROM test WHERE category->'$.name' = 'gd';
+----+-------------------------+-----------+
| id | category                | tags      |
+----+-------------------------+-----------+
|  2 | {"id": 2, "name": "gd"} | [1, 3, 5] |
+----+-------------------------+-----------+


mysql>  SELECT * FROM test WHERE category->>'$.name' = 'dlx';
+----+--------------------------+-----------+
| id | category                 | tags      |
+----+--------------------------+-----------+
|  1 | {"id": 1, "name": "dlx"} | [1, 2, 3] |
+----+--------------------------+-----------+


mysql>  SELECT * FROM test WHERE category->'$.id' = 1;
+----+--------------------------+-----------+
| id | category                 | tags      |
+----+--------------------------+-----------+
|  1 | {"id": 1, "name": "dlx"} | [1, 2, 3] |
+----+--------------------------+-----------+


mysql>  SELECT * FROM test WHERE JSON_CONTAINS(category, '1', '$.id');
+----+--------------------------+-----------+
| id | category                 | tags      |
+----+--------------------------+-----------+
|  1 | {"id": 1, "name": "dlx"} | [1, 2, 3] |
+----+--------------------------+-----------+
```



## 如何索引JSON字段

下面的例子基于5.7.29版本

```sql
CREATE TABLE `players` (  
    `id` INT UNSIGNED NOT NULL,
    `player_and_games` JSON NOT NULL,
    PRIMARY KEY (`id`)
);

-- 增加虚拟列
CREATE TABLE `players` (  
   `id` INT UNSIGNED NOT NULL,
   `player_and_games` JSON NOT NULL,
   `names_virtual` VARCHAR(20) GENERATED ALWAYS AS (`player_and_games` ->> '$.name') NOT NULL, 
   PRIMARY KEY (`id`)
);

-- 插入数据
INSERT INTO `players` (`id`, `player_and_games`) VALUES (1, '{  
    "id": 1,  
    "name": "Sally",
    "games_played":{    
       "Battlefield": {
          "weapon": "sniper rifle",
          "rank": "Sergeant V",
          "level": 20
        },                                                                                                                          
       "Crazy Tennis": {
          "won": 4,
          "lost": 1
        },  
       "Puzzler": {
          "time": 7
        }
      }
   }'
);

INSERT INTO `players` (`id`, `player_and_games`) VALUES (2, '{  
    "id": 2,  
    "name": "Sall111y",
    "games_played":{    
       "Battlefield": {
          "weapon": "sniper rifle",
          "rank": "Sergeant V",
          "level": 20
        },                                                                                                                          
       "Crazy Tennis": {
          "won": 4,
          "lost": 1
        },  
       "Puzzler": {
          "time": 7
        }
      }
   }'
);

-- 查看数据
SELECT * FROM `players`;
mysql> SELECT * FROM `players`;
+----+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+
| id | player_and_games                                                                                                                                                                             | names_virtual |
+----+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+
|  1 | {"id": 1, "name": "Sally", "games_played": {"Puzzler": {"time": 7}, "Battlefield": {"rank": "Sergeant V", "level": 20, "weapon": "sniper rifle"}, "Crazy Tennis": {"won": 4, "lost": 1}}}    | Sally         |
|  2 | {"id": 2, "name": "Sall111y", "games_played": {"Puzzler": {"time": 7}, "Battlefield": {"rank": "Sergeant V", "level": 20, "weapon": "sniper rifle"}, "Crazy Tennis": {"won": 4, "lost": 1}}} | Sall111y      |
+----+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+

```

加索引前：

```sql
mysql> EXPLAIN SELECT * FROM `players` WHERE `names_virtual` = "Sally"\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: players
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 2
     filtered: 50.00
        Extra: Using where
```

添加索引：

```sql
CREATE INDEX `names_idx` ON `players`(`names_virtual`);  
```

加索引后：

```sql
mysql> EXPLAIN SELECT * FROM `players` WHERE `names_virtual` = "Sally"\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: players
   partitions: NULL
         type: ref
possible_keys: names_idx
          key: names_idx
      key_len: 62
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

牛皮，json字段也能加索引。

## Mysql也能和NoSql PK了吗

参考这篇文章：<https://lefred.be/content/top-10-reasons-for-nosql-with-mysql/>

> As you know, one of the great new feature in MySQL 8.0 is the Document Store. Now with MySQL you can store your JSON documents in collections and manage them using CRUD operations. NoSQL is now part of MySQL ! Instead of a mix of MongoDB and MySQL, now you can eliminate MongoDB and consolidate with MySQL ! 🙂
>
> This is a **historical meeting of NoSQL and SQL** in the same database server!

真是了不得，按照上面说的Mysql 8.0已经吊打MongoDB了。