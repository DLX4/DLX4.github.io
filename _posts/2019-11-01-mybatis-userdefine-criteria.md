---
title: Mybatis 通用mapper怎么加自定义复杂sql
tags:
  - java
  - mybatis
url: 251.html
id: 251
categories:
  - JAVA
date: 2019-11-01 18:10:14
---

用mbg自动生成的通用mapper构造条件CRUD时，遇到有些复杂一点的条件比如where clumA = clumB 再比如 xx and yy and (a = b or ...) 时，标配的andXxEqualTo()等方法就捉襟见肘，难道要裸写xml SQL吗？





这里有一个小技巧可以在MBG的通用mapper基础上整合部分自定义的sql。

```java
/**
      * 添加一个用户自定义的与条件
      * 
      * @param criteria
      * @param sql
      */
     public static  void addCriterionUserDefined(T criteria, String sql) {
         Class criteriaCls = criteria.getClass();
         Class superclass = criteriaCls.getSuperclass();
         Method addCriterion;
         try {
             addCriterion = superclass.getDeclaredMethod("addCriterion", String.class);
             addCriterion.setAccessible(true);
             addCriterion.invoke(criteria, sql);
         } catch (NoSuchMethodException | SecurityException | IllegalAccessException | IllegalArgumentException
                 | InvocationTargetException e) {
             // TODO Auto-generated catch block
             e.printStackTrace();
             logger.error("error: {}", e.getMessage(), e);
         }}
```

**client代码：**
         `Util.addCriterionUserDefined(criteria, "(jjdbh=jjd_zjjdbh or jjd_zjjdbh is null)");`