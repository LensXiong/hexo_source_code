---
layout: post
title: 【MySQL高级】查询截取分析-查询优化
date: 2019-01-29 08:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：查询截取分析主要包括查询优化、慢查询日志、批量数据脚本、`Show Profile`和全局查询日志。本篇文章只针对查询优化的的部分情况进行说明，第一使用小表驱动大表，使用`IN`和`EXISTS`进行案例分析；第二`order by`关键字优化，为排序使用索引；第三`group by`关键字优化，实质基本与`order by`关键字优化相同。
**
<!-- more -->
<The rest of contents | 余下全文>

# 优化步骤

数据库优化四步骤：
① 慢查询日志的开启并捕获。
② `EXPLAIN` + 慢`SQL`分析。
③ `SHOW PROFILE` 查询`SQL`在`MySQL`服务器里面的执行细节和生命周期情况。
④ `MySQL`数据库服务器的参数调优。

# 小表驱动大表（`IN`和`EXISTS`）

优化原则：小表驱动大表，即小的数据集驱动大的数据集。使用`IN`和`EXISTS`进行说明。

使用`IN`的情况：

> 当B表的数据集小于A表的数据集时，用`IN`优于`EXISTS`。

```
select * from A where id in (select id from B)
等价于：
for select id from B
for select * from A where A.id = B.id
```

使用`EXISTS`的情况：

> 当A表的数据集小于B表的数据集时，用`EXISTS`优于`IN`。

```
select * from A where exists (select 1 for B where B.id = A.id)
等价于：
for select * from A
for select * from B where B.id = A.id
```

`EXISTS`语法：

```
SELECT .. FROM table WHERE EXISTS(subquery)
```

该语法可以理解为：将主查询的数据，放到子查询中做条件验证，根据验证结果（`TRUE`或`FALSE`）来决定主查询的数据结果是否予以保留。

> ` EXISTS`(`subquery`)只返回`TRUE`或`FALSE`，因此子查询中的`SELECT * `也可以是`SELECT 1` 或者其他，官方说法是实际执行时会忽略`SELECT`清单，因此没有区别。

建表语句：

```
CREATE TABLE `TableA` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `deptName` varchar(30) DEFAULT NULL,
  `locAdd` varchar(40) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COMMENT='表A';
CREATE TABLE `TableB` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_dept_id` (`deptId`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8 COMMENT='表B';
```

数据插入：

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

使用`IN`的`SQL`：

```
select * from TableB b where b.`deptId` in (select id from TableA a);
```

使用`EXISTS`的`SQL`：

```
select * from TableB b where exists (select 1 from TableA a where b.`deptId` = a.id);
```
![mysql-query-optimization-01](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-query-optimization/01.jpg?raw=true)

# 排序优化（order by ）
`MySQL`支持`Index`和`FileSort`两种方式的排序，`Index`是指扫描索引本身完成排序，`FileSort`是扫描文件内容进行排序，`Index`效率高于`FileSort`。
建表语句：
```
CREATE TABLE tblA(
id int primary key not null auto_increment,
age INT,
birth TIMESTAMP NOT NULL
);
```
数据插入：
```
INSERT INTO tblA(age,birth) VALUES(28,NOW());
INSERT INTO tblA(age,birth) VALUES(27,NOW());
INSERT INTO tblA(age,birth) VALUES(26,NOW());
```
创建索引：
```
CREATE INDEX idx_A_ageBirth ON tblA(age,birth);
```
![mysql-query-optimization-02](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-query-optimization/02.jpg?raw=true)
第一种情况和第二种情况，未产生文件排序；第三种情况和第四种情况因为创建的索引顺序为`age`、`index`，未按照创建索引的顺序排序会导致查询时进行文件排序。
>  注：在进行`ORDER BY` 子句，尽量使用`Index`方式排序，避免使用`FileSort`方式排序。

![mysql-query-optimization-03](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-query-optimization/03.jpg?raw=true)
以上四种情况，只有第三种没有用到`FileSort`文件排序，效率是最高的。第一种和第二种情况没有满足索引创建时的最佳左前缀原则，直接忽略了第一层索引，跳到第二层索引。第四种情况是因为索引要升序都升序，要降序都降序，有升有降，导致索引部分失效。
> 尽可能在索引列上完成排序操作，遵照索引创建时的最佳左前缀原则。

如果不在索引列上，`FileSort`文件排序有两种算法：双路排序和单路排序。

双路排序：取一批数据，要对磁盘进行两次扫描。具体操作是读取行指针和所排序的列，进行排序，然后扫描已经排序好的列表按照表中的值重新输出，从磁盘取出排序字段，在`buffer`进行排序，再从磁盘取其他数据。双路排序多适用于`MySQL4.1`之前。

单路排序：从磁盘读取查询需要的所有列，按照`order by` 列在`buffer`对它们进行排序，然后扫描排序后的列表进行输出。单路排序的效率更快一些，避免了第二次读取数据，并且把随机IO变成了顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中。

单路排序可能导致的问题：
单路排序比多路排序要多占很多空间，因为单路排序一次性取出所有的字段，会导致取出的数据总大小超出`sort_buffer`的容量，导致每次只能取`sort_buffer`容量大小的数据进行排序，排完再取，再排.....从而导致多次I/O。本来想省一次I/O操作，反而导致了大量的I/O操作。

提高`Order By `的速度：
① 进行 `Order By ` 时应避免使用`select` *，应该只查询需要的字段。这是因为当`Query`的字段大总和小于`max_length_for_sort_data`而且排序字段不是`TEXT`|`BLOB`类型时，会使用改进后的算法-单路算法，否则会使用老算法-多路排序。查询少量的字段会使得占用的空间较小。
② 尝试提高`sort_buffer_size`。不管使用多路排序还是单路排序，两种算法的数据都有可能超出`sort_buffer_size`的容量，超出之后会创建`tmp`文件进行合并排序，导致多次I/O，所以应该尽可能的提高`sort_buffer_size`。当然，需要根据系统的能力去提高。
③  尝试提高`max_length_for_sort_data`。不管使用哪种算法，提高这个参数都会提高效率，但是如果设的太高，数据总容量超出`sort_buffer_size`的概率就增大，会导致高的磁盘I/O活动和低的处理器使用。
为排序使用索引结论：
① `MySQL`两种排序方式，文件排序和索引排序。
② `MySQL`能为排序和查询使用相同的索引。

```
KEY a_b_c(a,b,c)
order by 能使用索引最左前缀
-- ORDER BY a
-- ORDER BY a,b
-- ORDER BY a,b,c
-- ORDER BY a DESC,b DESC,c DESC

如果 WHERE 使用索引的最左前缀定义为常量，则order by 能使用索引
-- WHERE a = const ORDER BY b
-- WHERE a = const AND b = const ORDER BY c
-- WHERE a = const ORDER BY b,c
-- WHERE a = const AND b > const ORDER BY b,c

不能使用索引进行排序
-- ORDER BY a ASC,b DESC,c DESC /*排序不一致*/
-- WHERE g = const ORDER BY b,c /*丢失a索引*/
-- WHERE a = const ORDER BY c /*丢失b索引*/
-- WHERE a = const ORDER BY a,d /*d不是索引的一部分*/
-- WHERE a in(..) ORDER BY b,c /*对于排序来说，多个相等条件也是范围查询*/
```
![mysql-query-optimization-04](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-query-optimization/04.jpg?raw=true)
# 分组优化（group by）

① `group by` 实质是先排序后进行分组，遵照索引创建的最佳左前缀。
② 当无法使用索引列，增大`max_length_for_sort_data`参数的设置，增大`sort_buffer_size`参数的设置。
③ `where`高于`having`，能写在`where`限定的条件就不要在`having`限定。
