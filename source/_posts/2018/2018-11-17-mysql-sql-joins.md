---
layout: post
title: ﻿【MySQL高级】七种 SQL JOINS 文氏图解
date: 2018-11-17 21:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：在进行索引优化、查询截取分析、锁机制等高级部分之前，需要对基本的核心知识点进行重点归纳和总结。首先，需要明确手写`SQL`顺序和机读`SQL`顺序是不一样的，只有理解`SQL`的执行顺序才能写出较好的`SQL`语句。再次，依次介绍了内连接`INNER JOIN`，左连接`LEFT JOIN`，右连接`RIGHT JOIN`，全连接` FULL OUTER JOIN`等`SQL JOINS`的文氏图，通过详细的案例进行对比和说明，目的在于深入理解各种`SQL JOINS`的相关知识。
![七种SQL Joins 文氏图解](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2018/mysql-sql-joins/02.png?raw=true)
**
<!-- more -->
<The rest of contents | 余下全文>

# SQL的执行顺序

手写`SQL`顺序：

```
SELECT DISTINCT
    <select_list>
FROM
    <left_table> 
    <join_type>
JOIN <right_table> ON <join_condition>
WHERE
    <where_condition>
GROUP BY
    <group_by_list>
HAVING
    <having_condition>
ORDER BY
    <order_by_condition>
LIMIT
    <limit_number>
```

机读`SQL`数据：
![机读SQL顺序](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2018/mysql-sql-joins/01.jpg?raw=true)

```
FROM 
    <left_table>
ON 
    <join_condition>
    <join_type> 
JOIN 
    <right_table> 
WHERE
    <where_condition>
GROUP BY
    <group_by_list>
HAVING
    <having_condition>
SELECT DISTINCT
    <select_list>
ORDER BY
    <order_by_condition>
LIMIT
    <limit_number>
```

# 七种Join文氏图
![七种SQL Joins 文氏图解](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2018/mysql-sql-joins/02.png?raw=true)

建表语句：

```
-- 创建表A
CREATE TABLE `TableA`(
`id` INT(11) NOT NULL AUTO_INCREMENT,
`deptName` VARCHAR(30) DEFAULT NULL,
`locAdd` VARCHAR(40) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='表A';
-- 创建表B
CREATE TABLE `TableB`(
`id` INT(11) NOT NULL AUTO_INCREMENT,
`name` VARCHAR(20) DEFAULT NULL,
`deptId` INT(11) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `fk_dept_id` (`deptId`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='表B';
```

插入数据：

```
-- 插入表A和表B数据
INSERT INTO TableA(deptName,locAdd) VALUES('PD',11);
INSERT INTO TableA(deptName,locAdd) VALUES('HR',12);
INSERT INTO TableA(deptName,locAdd) VALUES('MK',13);
INSERT INTO TableA(deptName,locAdd) VALUES('MIS',14);
INSERT INTO TableA(deptName,locAdd) VALUES('FD',15);
INSERT INTO TableB(name,deptId) VALUES('z3',1);
INSERT INTO TableB(name,deptId) VALUES('z4',1);
INSERT INTO TableB(name,deptId) VALUES('z5',1);
INSERT INTO TableB(name,deptId) VALUES('w5',2);
INSERT INTO TableB(name,deptId) VALUES('w6',2);
INSERT INTO TableB(name,deptId) VALUES('s7',3);
INSERT INTO TableB(name,deptId) VALUES('s8',4);
INSERT INTO TableB(name,deptId) VALUES('s9',51);
```

根据上面文氏图，依次顺序为`INNER JOIN`，`LEFT JOIN`，`RIGHT JOIN`，` FULL OUTER JOIN`。

## 内连接

`INNER JOIN`（内连接），产生的结果集中，是A和B的交集。
```
-- INNER JOIN（内连接），产生的结果集中，是A和B的交集。
SELECT *
FROM TABLEA AS a 
INNER JOIN TABLEB AS b ON a.`id` = b.`deptId`;
-- CROSS JOIN
SELECT *
FROM TABLEA AS a 
CROSS JOIN TABLEB AS b ON a.`id` = b.`deptId`;
-- JOIN
SELECT *
FROM TABLEA AS a 
JOIN TABLEB AS b ON a.`id` = b.`deptId`;
```

> In MySQL, JOIN, CROSS JOIN, and INNER JOIN are syntactic equivalents (they can replace each other). In standard SQL, they are not equivalent. INNER JOIN is used with an ON clause, CROSS JOIN is used otherwise.

## 左连接

`LEFT JOIN`（左连接），产生的结果集中，是A的全部结果集。
```
-- LEFT [OUTER] JOIN
SELECT *
FROM TABLEA AS a 
LEFT JOIN TABLEB AS b ON a.`id` = b.`deptId`;
```
## 右连接

`RIGHT JOIN`（右连接），产生的结果集中，是B的全部结果集。
```
-- RIGHT [OUTER] JOIN
SELECT *
FROM TABLEA AS a 
RIGHT JOIN TABLEB AS b ON a.`id` = b.`deptId`;
```
关于`LEFT JOIN`和 `RIGHT JOIN`，`MySQL`官方文档原文如下：

> RIGHT JOIN works analogously to LEFT JOIN. To keep code portable across databases, it is recommended that you use LEFT JOIN instead of RIGHT JOIN.

## 左连接不含B

` LEFT [OUTER] JOIN AND b.deptId IS NULL`，产生的结果集中，是A中不包含B的结果集。
```
-- LEFT [OUTER] JOIN AND b.deptId IS NULL
SELECT *
FROM TABLEA AS a 
LEFT JOIN TABLEB AS b ON a.id = b.deptId
WHERE b.deptId IS NULL;
```
## 右连接不含A

`RIGHT [OUTER] JOIN AND a.id IS NULL`，产生的结果集中，是B中不包含A的结果集。
```
-- RIGHT [OUTER] JOIN AND a.id IS NULL
SELECT *
FROM TABLEA AS a 
RIGHT JOIN TABLEB AS b ON a.id = b.deptId
WHERE a.id IS NULL;
```
## 全连接

`FULL [OUTER] JOIN`，产生的结果集中，是包含AB的所有结果集。
`MySQL`不支持全连接，因此文氏图中的全连接语句不适用于`MySQL`，但对于`Oracle`适用。
```
-- FULL [OUTER] JOIN 
SELECT * 
FROM TABLEA AS a 
FULL OUTER JOIN TABLEB AS b ON a.id = b.deptId
```

>  `MySQL`不支持全连接，以上全连接语句不适用于`MySQL`。

`MySQL`不支持全连接，但可以通过`LEFT JOIN + UNION + RIGHT JOIN `来实现`FULL JOIN`：
```
-- MySQL不支持全连接，可以通过LEFT JOIN + UNION + RIGHT JOIN 来实现FULL JOIN
SELECT *
FROM TABLEA AS a 
LEFT JOIN TABLEB AS b ON a.id = b.deptId
UNION
SELECT *
FROM TABLEA AS a 
RIGHT JOIN TABLEB AS b ON a.id = b.deptId;
```
## 全连接A的独有与B的独有

本次产生的结果集，是A的独有结果集和B的独有结果集之和。

在`Oracle`中，全连接写法如下：

```
SELECT *
FROM
TABLEA a 
FULL OUTER JOIN 
TABLEB b
ON a.id = b.deptId
WHERE a.id IS NULL 
OR b.deptId IS NULL;
```

因为`MySQL`不支持全连接，通过`LEFT JOIN + UNION + RIGHT JOIN `来实现：

```
SELECT *
FROM TABLEA AS a 
LEFT JOIN TABLEB AS b ON a.id = b.deptId
WHERE b.deptId IS NULL
UNION
SELECT *
FROM TABLEA AS a 
RIGHT JOIN TABLEB AS b ON a.id = b.deptId
WHERE a.id IS NULL ;
```
