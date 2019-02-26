---
layout: post
title: 【MySQL高级】索引优化分析（三）
date: 2019-01-08 11:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：在索引优化分析的前两篇文章中，我们讨论了索引相关基础知识，也重点介绍了`EXPLAIN`执行计划。针对于索引的分析，本篇文章以单表优化、两表优化、三表优化为例，详细介绍了未创建索引和创建索引情况下语句的执行情况，尤其是在`Join`连接的情况下，如何更好的建立索引才能使性能更优化。
**
<!-- more -->
<The rest of contents | 余下全文>

# 索引分析

## 单表优化案例

相关建表语句和数据如下：

```
CREATE TABLE IF NOT EXISTS `article`(
`id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
`author_id` INT(10) UNSIGNED NOT NULL,
`category_id` INT(10) UNSIGNED NOT NULL,
`views` INT(10) UNSIGNED NOT NULL,
`comments` INT(10) UNSIGNED NOT NULL,
`title` VARBINARY(255) NOT NULL,
`content` TEXT NOT NULL
);
INSERT INTO `article`(`author_id`,`category_id`,`views`,`comments`,`title`,`content`) VALUES 
(1,1,1,1,'1','1'),
(2,2,2,2,'2','2'),
(1,1,3,3,'3','3');
```
要求：查询`category_id `为1且`comments`大于1的情况下，`views`最多的`article_id`。

```
EXPLAIN SELECT id,author_id FROM article WHERE `category_id` = 1 AND `comments` > 1 ORDER BY views DESC LIMIT 1;
```
未建索引的情况下，从上图可以看到上面的查询语句进行全表扫描（`All`）和文件内排序（`Using filesort`）。显然，这种查询是最坏的情况，优化是必须的。

```
-- ALTER TABLE `article` ADD INDEX idx_article_ccv(`category_id`,`comments`,`views`);
CREATE index idx_article_ccv on article(`category_id`,`comments`,`views`);
SHOW INDEX FROM `article`;
EXPLAIN SELECT id,author_id FROM article WHERE `category_id` = 1 AND `comments` > 1 ORDER BY views DESC LIMIT 1;
```

经过创建索引后再次执行`SQL`语句，会发现`type`访问类型由全表扫描`ALL`变成了`rang`范围，但是扫描的时候依然进行了文件内排序（`Using filesort`）。
> 我们已经建立了索引，为什么没有用？

这是因为按照`BTree`索引的工作原理，先排序`category_id`，如果遇到相同的`category_id`则再排序`comments`，如果遇到相同的`comments`则再排序`views`。而本次查询的`comments`字段在联合索引的中间位置，`comments` > 1条件是一个范围值（`type`值为`range`）,`MySQL`无法利用索引再对后面的`views`部分进行检索，也就是说`range`类型查询字段后面的索引无效。
```
DROP INDEX idx_article_ccv ON article;
CREATE INDEX idx_article_cv ON `article`(category_id,views);
SHOW INDEX FROM `article`;
EXPLAIN SELECT id,author_id FROM article WHERE `category_id` = 1 AND `comments` > 1 ORDER BY views DESC LIMIT 1;
```

经过第三次的分析并重新创建了索引，可将`Extra`中的`Using filesort`优化处理掉。

## 两表优化案例

建表语句如下：

```
CREATE TABLE IF NOT EXISTS `class`(
`id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY(`id`)
);
CREATE TABLE IF NOT EXISTS `book`(
`bookid` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`bookid`)
);
```

分别插入20条数据：

```
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO class(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
INSERT INTO book(`card`) VALUES(FLOOR(1+(RAND() * 20)));
```
未建索引时，`Explain`执行计划中的`type`为`ALL`：

左连接时分别加左表索引和右表索引时的执行计划如下：

左连接的特性决定左表必须全部都有，右表最好建立索引，查询优化效率最好。
同样，右连接的特性决定如何从左表搜索行，右边一定都会有，左边的表一定要建立索引。
> 结论：左连接，右表建索引；右连接，左表建索引。

## 三表优化案例

再增加一个表如下：

```
CREATE TABLE IF NOT EXISTS `phone`(
`phoneid` iNT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`phoneid`)
)ENGINE = INNODB;
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(floor(1 + (RAND() * 20)));
```

三表连接时，依然遵循之前两表的原则建立索引，左连接时所有的右表都建索引，最终执行计划如下：

```
-- 左连接时，右表继续添加索引
ALTER TABLE `phone` ADD INDEX p(`card`);
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card LEFT JOIN phone ON book.card = phone.card;
```

结论：
① 左连接时右表建索引，右连接时左边建索引的原则。
② 尽可能减少`Join`语句中的嵌套循环总次数，永远记住用小表驱动大表。
③ 优先优化嵌套循环的内层循环，保证`Join`语句中被驱动表上的`Join`条件字段已经被建立索引。
④ 当无法保证被驱动表的`Join`条件字段被索引且内存资源充足的前提下，`JoinBuffer`的设置可以设置大一些。
