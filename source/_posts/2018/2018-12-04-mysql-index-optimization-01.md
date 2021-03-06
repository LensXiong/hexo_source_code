---
layout: post
title: 【MySQL高级】索引优化分析（一）
date: 2018-12-04 18:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：作为一名开发者，完成业务功能只是第一步，如何将代码和SQL语句写的优雅，执行效率高，运行速度快是区别平庸和优秀最直接的因素。如果在你的认知里面，索引就像字典中的目录一样，是为了快速查找内容，显然，你还需要努力去学习索引更多的知识。索引，作为`MySQL`优化最重要的一部分，需要掌握索引的基础知识，索引带来的优劣，基本语法和分类，除此之外，只有正确理解索引的检索原理，在后期进行创建和使用索引才能更轻松自如的应对。最后，也对适合创建索引的场景和不适合创建索引的场景进行相关的阐述。
**
<!-- more -->
<The rest of contents | 余下全文>

# 索引基础

`MySQL`官方对索引的定义为：索引（`Index`）是存储引擎用于快速找到记录的一种数据结构。
索引的本质：一种数据结构。
索引的目的：提高查询效率。
索引解决：① `where`条件后面的字段拼装查询效率。②`order by ` 后面的字段排序如何查询快。
简而言之，可以将索引理解为：**排好序的快速查找结构。**

> `Index`在数组中称为下标，在数据库中称为索引，在`Git`中称为暂存区。

在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式（比如指针）引用（指向）数据。这样就可以在这些数据结构上实现高级查找算法，而这种数据结构，就是索引。

> 频繁的`update`除了会更新数据，也会更新索引。

一般来说，索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。
我们平常说的索引，如果没有特别指明，都是指B树（多路搜索树，并不一定是二叉的，也有可能是三叉）结构组织的索引，其中聚集索引、次要索引、复合索引、前缀索引、唯一索引默认都是使用B+树索引，统称为索引。当然，除了B+树这种类型的索引之外，还有哈希索引（`hash index`）。

# 索引优劣

索引的优势：
① 索引大大减少了服务器需要扫描的数据量。
② 索引可以帮助服务器避免排序和临时表，通过索引列对数据进行排序，降低数据排序的成本，降低了`CPU`的消耗。
③ 索引可以将随机`IO`变为顺序`IO`，类似图书馆书目索引，提高数据检索的效率，降低数据库的`IO`成本。

索引的劣势：
① 占用空间：实际上，索引也是一张表，该表保留了主键与索引字段，并指向实体表的记录，所以索引列也是需要占用空间的。
② 降低更新表的速度：虽然索引大大提高了查询速度，但是也会降低更新表的速度，如对表进行`INSERT`、`UPDATE`和`DELETE`时，都会调整因为更新所带来的键值变化后的索引信息。
③ 索引优化需要花费大量的时间：索引只是提高效率的一个因素，如果表的数据量较大，就需要花时间研究去建立最优的索引，也要进行`SQL`优化查询。因为各种业务不同，对于所建的索引也是不同的，也需要根据实际业务去寻找最优的方式。

# 索引分类

① 单值索引：一个索引只包含单个列，一个表可以有多个单列索引。
② 唯一索引：索引列的值必须唯一，但允许有空值。
③ 复合索引：一个索引包含多个列。


# 基本语法

```
-- 创建
CREATE [UNIQUE] INDEX indexName ON table (columnname(length));
-- 更新
ALTER table ADD [UNIQUE] INDEX indexName ON table (columnname(length));
-- 删除
DROP INDEX [indexName] ON table;
-- 查看
SHOW INDEX FROM table\G;
```

使用`ALTER`命令为数据表添加索引语法如下：

```
-- 主键索引，意味着索引值必须是唯一的，且不能为NULL
ALTER TABLE table_name ADD PRIMARY KEY(column_list);
-- 唯一索引，该索引的值必须是唯一的，除了NULL外，NULL可能会出现多次。
ALTER TABLE table_name ADD UNIQUE index_name(column_list);
-- 普通索引，索引值可出现多次
ALTER TABLE table_name ADD INDEX index_name(column_list);
-- 全文索引
ALTER TABLE table_name ADD FULLTEXT index_name(column_list);
```

# 检索原理

`MySQL`支持的索引类型，主要有`B-Tree`索引、`Hash`索引、`R-Tree`（空间数据）索引、`Full-Text`全文索引等，其中主要介绍`B-Tree`索引的检索原理：

## 初始化介绍
![检索原理](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2018/mysql-index-optimization-01/01.jpg?raw=true)
如上图所示的一颗`B+`树，浅蓝色的称为磁盘块，每个磁盘块包含几个数据项（深蓝色所示）和指针（黄色所示）。如磁盘块1包含数据项17和35，包含指针P1、P2、P3，P1表示小于17的磁盘块，P2表示在17和35之间的磁盘块，P3表示大于35的磁盘块。真实的数据存在于叶子节点即3、5、9、10、13、15、23、29、36、60、75、79、90、99。非叶子节点不存储真实的数据，只存储指引搜索方向的数据项，如17、35并不真实存在于数据表。

## 查找过程
如果要查找
数据项29，那么首先会把磁盘块1由磁盘加载到内存，此时发生一次IO，在内存中用二分查找确定29在17和35之间，锁定磁盘块1的P2指针，内存时间非常短（相比于磁盘IO）可以忽略不计，通过磁盘块1的P2指针的磁盘地址把磁盘块3加载到内存，发生第二次IO，29在26和30之间，锁定磁盘块3的P2指针，通过指针加载磁盘块8到内存，发生第三次IO，同时内存中做二分查找找到29，结束查询，总计三次IO。

真实的情况是，3层的`B+`树可以表示上百万的数据，如果上百万的数据查找只需要三次IO，性能提高将是巨大的，如果没有索引，每个数据项都要发生一次IO，那么总共需要百万次的IO，显然成本非常非常的高。

## 索引场景

适合创建索引的场景如下：
① 主键自动建立唯一索引。
② 频繁作为查询条件的字段应该创建索引。
③ 查询中与其他表关联的字段，外键关系建立索引。
④ 单键/组合索引的选择时，在高并发下倾向创建组合索引。
⑤ 查询中排序的字段，排序字段如果通过索引去访问将大大提高排序速度。
⑥ 查询中统计或者分组字段。

不适合创建索引的场景如下：
① 频繁更新的字段不适合创建索引，因为每次更新不单单是更新记录还会更新索引。
② `where`条件中用不到的字段不创建索引。
③ 表数据太少，实际上，三百万左右数据以下属于表数据太少。
④ 经常增删改的表，建立索引虽然增加了查询速度，但是会降低更新表的速度。
⑤ 数据重复且分布平均的表字段，为它建立索引就没有太大的实际效果。

> 假如一个表有10万行记录，有一个字段只有F和M两种值，且每个值得分布概率大约为50%，那么对这个字段建立索引一般不会提高数据库的查询速度。

索引的选择性是指索引列中不同值得数目与表中记录数的比。如果一个表中有2000条数据，表索引列有1980个不同的值，那么这个索引的选择性就是1980/2000=0.99。一个索引的选择性越接近1，这个索引的效率就越高。
