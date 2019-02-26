---
layout: post
title: 【MySQL高级】索引优化分析（一）
date: 2018-12-28 13:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：索引对于良好的性能非常关键，尤其是当表中的数据量越来越大时，索引对性能的影响愈发重要。相对于查询优化，索引优化应该是对查询性能优化最有效的手段。本篇文章介绍了`MySQL` 查询优化器的处理过程、`MySQL`常见的瓶颈、`Explain` 执行计划。其中，使用`EXPLAIN`关键字可以模拟优化器执行`SQL`查询语句，从而知道`MySQL`是如何处理你的`SQL`语句的。重点介绍了`EXPLAIN`关键字如何分析你的查询语句或是表结构的性能瓶颈。
**
<!-- more -->
<The rest of contents | 余下全文>

# 性能分析

## MySQL 查询优化器

`MySQL`中有专门负责优化`SELECT`语句的优化器模块，其主要功能：通过计算分析系统收集到的统计信息，为客户端请求的`Query`提供它认为最优的执行计划（`MySQL`认为最优的数据检索方式，但不见得是`DBA`认为最优的）。
当客户端向`MySQL`请求一条`Query`，命令解析器模块完成请求分类。区别出是`SELECT`并转发给`MySQL`查询优化器时，查询优化器首先会对整条`Query`进行优化，处理掉一些常量表达式的预算，直接换算成常量值，并对`Query`中的查询条件进行简化和转化，如去掉一些无用或显而易见的条件、结构调整等。接下来分析`Query`中的`Hint`信息，通过`Hint`信息是否可以完全确定该`Query`的执行计划，如果没有`Hint`信息或者`Hint`信息还不足以完全确定执行计划，则会读取所涉及对象的统计信息，根据`Query`进行相应的计算分析，最后得到最终的执行计划。

## MySQL 常见的瓶颈
`CPU`瓶颈：数据装入内存或者从磁盘上读取数据，`CPU`会出现饱和的情况。
`IO`瓶颈：磁盘`I/O`瓶颈发生在装入数据远大于内存容量的时候。
服务器硬件的性能瓶颈：`top`,`free`,`iostat`和`vmstat`来查看系统的性能状态。

## Explain 执行计划

使用`EXPLAIN`关键字可以模拟优化器执行`SQL`查询语句，从而知道`MySQL`是如何处理你的`SQL`语句的。通过`EXPLAIN`关键字可以分析你的查询语句或是表结构的性能瓶颈。以下为其作用：
① 表的读取顺序，通过`id`关键词来判断。
② 数据读取操作的操作类型，通过`type`关键词来判断。
③ 哪些索引可以使用，通过`possible_keys`关键词来判断。
④ 哪些索引被实际使用，通过`key`关键词来判断。
⑤ 表之间的引用，通过`ref`关键词来判断。
⑥ 每张表有多少行被优化器查询，通过`rows`关键词来判断。

执行计划的语法：`Explain` + `SQL`语句。
执行计划包含的字段及说明如下：

|字段|说明|
|--|--|
|id|表的加载和读取顺序|
|select_type|数据的访问类型|
|table|所加载的表|
|type|数据的查询类型|
|possible_keys|可能使用的索引|
|key|实际使用的索引|
|key_len|索引的字节数|
|ref|索引引用的列|
|rows|读取的行数|
|extra|其他额外的信息|

为了更好的分析每个字段的作用，以下为三个相关建表语句，目的仅用做分析使用：

```
CREATE TABLE `t1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `other_column` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='表t1';
CREATE TABLE `t2` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `other_column` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='表t2';
CREATE TABLE `t3` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `other_column` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='表t3';

INSERT INTO t1 (id,`other_column`) VALUES (1,'');
INSERT INTO t1 (id,`other_column`) VALUES (2,'');
INSERT INTO t1 (id,`other_column`) VALUES (3,'');
INSERT INTO t2 (id,`other_column`) VALUES (1,'');
INSERT INTO t2 (id,`other_column`) VALUES (2,'');
INSERT INTO t2 (id,`other_column`) VALUES (3,'');
INSERT INTO t3 (id,`other_column`) VALUES (1,1);
INSERT INTO t3 (id,`other_column`) VALUES (2,2);
INSERT INTO t3 (id,`other_column`) VALUES (3,'');
```

### id

`SELECT`查询的序列号，包含一组数字，表示查询中执行`SELECT`子句或操作表的顺序。

三种情况：

① id相同，执行顺序由上至下。

```
-- id相同，执行顺序由上至下
explain select t2.* from t1,t2,t3 where t1.id = t2.id and t1.id = t3.id and t1.other_column = '';
```
执行结果如下图所示：

② id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行。

```
-- id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
explain select t2.* from t2 where t2.id = (select t1.id from t1 where t1.id = (select t3.id from t3 where t3.other_column = ''));
```

执行结果如下图所示：

③ id相同不同，同时存在。

`id`如果相同，可以认为是一组，从上往下顺序执行，在所有组中，`id`值越大，优先级越高，越先执行。
```
-- id相同又不同，同时存在
explain select t2.* from (select t3.id from t3 where t3.other_column = '') s1,t2 where s1.id = t2.id;
```
### select_type

查询类型`select_type`主要用于区别普通查询、联合查询、子查询等复杂查询，分类和说明如下所示：

|id|类型|说明|
|--|--|--|
|1|SIMPLE|查询中不包含子查询或者UNION|
|2|PRIMARY|查询中若包含任何复杂的子部分，最外层查询则被标记为PRIMARY|
|3|SUBQUERY|在SELECT或WHERE列表中包含了子查询，该子查询被标记为SUBQUERY|
|4|DERIVED|在FROM列表中包含的子查询被标记为：DERIVED（衍生）|
|5|UNION|若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在  FROM子句的子查询中，外层SELECT将被标记为：DERIVED|
|6|UNION RESULT|从UNION表获取结果的SELECT被标记为：UNION RESULT|

### table

`table`主要用于显示这一行的数据是关于哪张表，这里不做过多说明。

### type

` type`显示查询使用了何种类型，总共七种类型，从最好到最差，具体类型及说明如下所示：

|类型|说明|

|--|--|
|system|表只有一行记录（等于MySQL自带的系统表），这是const类型的特例，平时不会出现，可以忽略|
|const|表示通过索引一次就找到了，const用于primary key或者unique索引|
|eq_ref|唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引|
|ref|非唯一性索引扫描，返回匹配某个单独值得所有行|
|range|只检索给定范围的行，使用一个索引来选择行，属于范围扫描索引，一般出现在where语句中使用between,<,>,in等查询|
|index|Full Index Scan，只遍历索引树|
|all|Full Table Scan，将遍历全表以找到匹配的行|

> 从最好到最差依次是`system`>`const`>`eq_ref`>`ref`>`range`>`index`>`all`。

一般来说，要保证查询至少达到`range`级别，最好能达到`ref`。

### possible_keys

指出`MySQL`能使用哪个索引在表中找到行，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用。

### key
显示`MySQL`在查询中实际使用的索引，若没有使用索引，显示为`NULL`。

> 查询中若使用了覆盖索引，则该索引和查询的`SELECT`字段重合。（覆盖索引指的是创建的索引字段和查询的字段一致）

# key_len
表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好。

`key_len`显示的值为索引字段的最大可能长度，并非实际使用长度，即`key_len`是根据表定义计算而得，不是通过表内检索出的。

### ref

该字段显示索引的哪一列被使用了，有可能为一个常量。简单来讲，就是指哪些列或常量被用于查找索引列上的值。

上图所示，根据`table`和`key`字段可知，`t1`表的`idx_col1_col2`被充分使用；通过`ref`可知，`t1`表的`col1`匹配`t2`表的`col1`，`t1`表的`col2`匹配一个常量，即`const`等于`ac`。

### rows

根据表统计信息及索引选用情况，大致估算出最终找到所需记录需要读取的行数。

通过以上示例，未建立复合索引`idx_col1_col2`前，`MySQL`优化器查询的行数为640+1=641条数据，建立复合索引后，可将优化器的查询行数降低至195+4=199条。

### Extra

包含不适合在其他列中显示但十分重要的额外信息。主要包含以下内容：

|类型|说明|
|--|--|
|Using filesort|文件排序|
|Using temporary|临时表排序|
|Using index|覆盖索引|
|Using where|使用条件过滤|
|Using join buffer|使用连接缓存|
|impossible where|where子句值为false|

① Using filesort

`MySQL`会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。`MySQL`中无法利用索引完成的排序操作称为“文件排序”。

由上图可知，虽然两次查询的结果是相同的，但实现方式却是有很大的区别。两次的区别在于第一次使用了`Using filesort`，第二次的效率较高。

② Using temporary

`MySQL`在对查询结果排序时，使用临时表保存中间结果，常见于`order by` 排序`和group by `分组查询。

使用临时表后的执行速度比文件内排序更慢。通过上图第一次中可见`Using temporary`和`Using filesort`，原因是因为所创建的索引为`col1`和`col2`，`group by`时却直接用到了`col2`，没有按照索引的顺序进行排序，优化为第二次的语句，可见效率得到提高。

> 在使用`group by`时，应该考虑到所创建的索引尽量与`group by`后面的顺序及个数一致。这样，执行`SQL`语句的效率才不会被拖慢。

③ Using index

`Using index`表示相应的`select`操作中使用了覆盖索引（`Convering Index`），避免了表的数据行，效率比较好。
如果同时出现了`Using where`，表明索引被用来执行索引键值的查找。
如果没有同时出现`Using where`，表明索引被用来读取数据而非执行查找动作。

覆盖索引的两种理解方式：

方式一：`MySQL`可以利用索引返回`select`列表中的字段，而不必根据索引再次读取数据文件，换句话说，查询的列要被所建的索引覆盖。
包含所有满足查询需要的数据的索引称为覆盖索引（`Covering Index`）。

方式二：索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据，既然能通过读取索引就可以得到想要的数据，那就不需要再读取行。一个索引包含了（或覆盖了）满足查询结果的数据就叫做覆盖索引。

> 如果要使用覆盖索引，一定要注意`select`列表中只取出需要的列，不可`select *`，因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降。
④ Using where

表示`MySQL`服务器在存储引擎搜到记录后进行“后过滤”（`Post-filter`），如果查询未能使用索引，`Using where`的作用只是提醒我们`MySQL`将用`where`子句来过滤结果集。

⑤ Using join buffer
使用了连接缓存。
⑥ impossible where
`where`子句的值总是`false`，不能获取任何元素，如一个人的性别既是男又是女。

案例分析：

针对以下案例，请分析出`MySQL`的执行顺序：

分析：
执行顺序1：`id`为4的行，`select_type`为`union`，说明第四个`select`是`union`里的第二个`select`，最先执行【select name,id from t2】。
执行顺序2：`id`为3的行，是整个查询中第三个`select`的一部分。因为查询包含在`from`中，所以为`derived`。【select id,name from t1 where other_column = ''】。
执行顺序3：`id`为2的行，`select`列表中的子查询`select_type`为`subquery`，为整个查询中的第二个`select`。【select id from t3】
执行顺序4：`id`为1的行，`select_type`为`primary`，表示该查询为外层查询，`table`为`<derived3>`，表示查询结果来自一个衍生表，其中`derived3`中的3代表该查询衍生自id为3的`select`查询。【select d1.name......】
执行顺序5：`id`为NULL，代表从`union`的临时表中读取行的阶段，`table`列中的`<union1,4>`表示用第1个和第4个`select`的结果进行`union`操作。【两个结果union操作】
