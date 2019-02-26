---
layout: post
title: 【MySQL高级】索引优化分析（四）
date: 2019-01-16 11:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：`MySQL` 索引通常是被用于提高` WHERE`条件的数据行匹配时的搜索速度，在索引的使用过程中，存在一些使用细节和注意事项。为了避免索引失效，本篇文章主要分析了十种情况，其中包括最佳全值匹配、最左前缀原则（中间索引不能跳）、索引列少计算、存储引擎不能使用索引中范围条件右边的列、尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），避免` select *`、` like` 以通配符开头（`%abc...`）`MySQL`索引会失效变成全表扫描等情况。
**
<!-- more -->
<The rest of contents | 余下全文>

建表语句如下：

```
CREATE TABLE staffs(
`id` INT PRIMARY KEY AUTO_INCREMENT,
`name` VARCHAR(24) NOT NULL DEFAULT '' COMMENT '姓名',
`age` INT NOT NULL DEFAULT 0 COMMENT '年龄',
`pos` VARCHAR(20) NOT NULL DEFAULT '' COMMENT '职位',
`add_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间'
) CHARSET utf8 COMMENT '员工记录表';
```

数据插入：

```
INSERT INTO staffs(name,age,pos,add_time) VALUES('z3',22,'manager',NOW());
INSERT INTO staffs(name,age,pos,add_time) VALUES('July',23,'dev',NOW());
INSERT INTO staffs(name,age,pos,add_time) VALUES('2000',23,'dev',NOW());
```

建立索引语句如下：

```
ALTER TABLE staffs ADD INDEX idx_staffs_nameAgePos(name,age,pos);
SHOW INDEX FROM staffs;
```

# 全值匹配最佳
![mysql-index-optimization-04-01](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/01.jpg?raw=true)
# 最左前缀原则（中间索引不能跳）
![mysql-index-optimization-04-02](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/02.jpg?raw=true)
由上面的执行分析可知，如果索引为多列，要遵守最左前缀法则。
查询从索引的最左前列开始并且不能跳过索引中的列。
# 索引列少计算
![mysql-index-optimization-04-03](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/03.jpg?raw=true)
不在索引列上做任何操作（计算、函数、（自动或手动）类型转换），否则会导致索引失效而转向全表扫描。

# 存储引擎不能使用索引中范围条件右边的列
![mysql-index-optimization-04-04](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/04.jpg?raw=true)
根据上面的执行计划可知，当索引中查询条件为范围查询时，索引范围查询右边列会失效。
# 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），避免select*
![mysql-index-optimization-04-05](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/05.jpg?raw=true)
建立索引的列`name`,`age`,`pos`尽量与查询的列保持顺序一致和个数一致。
# `MySQL`在使用不等于（!=或者<>）的时候无法使用索引，会导致全表扫描
![mysql-index-optimization-04-06](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/06.jpg?raw=true)
# `is null`，`is not null` 也无法使用索引
![mysql-index-optimization-04-07](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/07.jpg?raw=true)
# like以通配符开头（`'%abc...'`）`MySQL`索引会失效变成全表扫描
![mysql-index-optimization-04-08](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/08.jpg?raw=true)
> 当模糊查询`LIKE`的百分号写在右边时索引才有效。但实际生产环境往往需要左右都是模糊查询，那如何解决`like`'%查询字符串%'时索引不被使用的方法'？
建表语句和数据如下：
```
CREATE TABLE tbl_user(
`id` INT(11) NOT NULL AUTO_INCREMENT,
`name` VARCHAR(20) DEFAULT NULL,
`age` INT(11) DEFAULT NULL,
`email` VARCHAR(20) DEFAULT NULL,
PRIMARY KEY (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;
INSERT INTO tbl_user(name,age,email) VALUES('1aa1',21,'w@gmail.com');
INSERT INTO tbl_user(name,age,email) VALUES('2aa2',211,'x@gmail.com');
INSERT INTO tbl_user(name,age,email) VALUES('3aa3',985,'wx@gmail.com');
INSERT INTO tbl_user(name,age,email) VALUES('4aa4',95,'xw@gmail.com');
```
创建索引语句如下：
```
CREATE INDEX idx_user_nameAge ON tbl_user(name,age);
SHOW INDEX FROM tbl_user;
```
![mysql-index-optimization-04-09](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/09.jpg?raw=true)
单独的`id`，单独的`name`，单独的`age`都会用到索引，查询的字段为索引的全部或者部分字段，会使用索引查询，如果查询的字段比所建索引字段多，索引会失效。
因此，对于`like`'%查询字符串%'时，使用覆盖索引来进行查询可以解决索引失效的问题。

# 字符串不加单引号索引失效
![mysql-index-optimization-04-10](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/10.jpg?raw=true)
# 少用or，用它来连接时索引会失效
![mysql-index-optimization-04-11](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/11.jpg?raw=true)
案例1：
假设`index(a,b,c)`

|where语句|索引是否被使用|
|--|--|
|where a = 3|Y，使用到a|
|where a = 3 and b = 5|Y，使用到a，b|
|where a = 3 and b = 5 and c = 4|Y，使用到a，b，c|
|where b = 3 或者 where b = 3 and c = 4 或者 where c = 4|N|
|where a = 3 and c = 5|使用到了a，但是c没有被使用，因为b中间断了|
|where a = 3 and b > 4 and c = 5| 使用到了a和b，c不能用在范围之后，b断了|
| where a = 3 and b like 'kk%' and c = 4 | Y，使用到a，b，c|
| where a = 3 and b like '%kk' and c = 4 |Y，只用到a|
| where a = 3 and b like '%kk%' and c = 4 |Y，只用到a|
| where a = 3 and b like 'k%kk%' and c = 4 |Y，使用到a，b，c|

案例2：
建表语句：

```
CREATE TABLE test03(
`id` INT PRIMARY KEY NOT NULL AUTO_INCREMENT,
`c1` CHAR(10),
`c2` CHAR(10),
`c3` CHAR(10),
`c4` CHAR(10),
`c5` CHAR(10)
);
```

插入数据：

```
INSERT INTO test03(c1,c2,c3,c4,c5) VALUES('a1','a2','a3','a4','a5');
INSERT INTO test03(c1,c2,c3,c4,c5) VALUES('b1','b2','b3','b4','b5');
INSERT INTO test03(c1,c2,c3,c4,c5) VALUES('c1','c2','c3','c4','c5');
INSERT INTO test03(c1,c2,c3,c4,c5) VALUES('d1','d2','d3','d4','d5');
INSERT INTO test03(c1,c2,c3,c4,c5) VALUES('e1','e2','e3','e4','e5');
```

创建复合索引：

```
create index idx_test03_c1234 on test03(c1,c2,c3,c4);
show index from test03;
```
![mysql-index-optimization-04-12](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/12.jpg?raw=true)
① 索引全值匹配时，顺序不一致
![mysql-index-optimization-04-13](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/13.jpg?raw=true)
从上图可以看出，我们所建立的索引顺序为`c1`，`c2`，`c3`，`c4`，但是当我们查询顺序与所建的索引顺序不一致时，执行计划依然是最优的索引查询，`type`为`ref`,`ref`依然为`const`。这是因为`MySQL`逻辑架构层服务层中自带查询优化器，会在`SQL`执行语句之前进行优化处理。
> 避免MySQL底层会多一层转化，在不影响业务数据查询时，建议`SQL`语句与索引建立的顺序保持一致。

② 范围之后全失效
![mysql-index-optimization-04-14](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-index-optimization-04/14.jpg?raw=true)
分析得到，第一种情况用到3个索引，是因为查询顺序与所建立的索引顺序一致，到`c3`之后变成了范围查询，`c4`失效，因此只用到三个索引`c1`，`c2`，`c3`；第二种情况用到了`c1`和`c2`，因为`c4`是范围查询，导致`c3`也失效，只用到了两个索引。第三种情况用到2个索引，因为`c4`断层，导致`c3`和`c4`没有用到索引，`c3`的作用在于排序而不是查找。第四种情况和第三种是一样的。第五种情况用到了2个索引，并且额外用到了文件排序。

优化总结口诀：
> 全值匹配我最爱，最左前缀要遵守；
带头大哥不能死，中间兄弟不能断；
索引列上少计算，范围之后全失效；
LIKE百分写最右，覆盖索引不写星；
不等空值还有or，索引失效要少用；
VAR引号不可丢，SQL高级也不难。

结论：
① 全值匹配最佳，所建立的索引字段与查询条件中的字段一一对应（个数和顺序一致）是最佳的。
② 如果索引为多列，要遵守最左前缀法则。查询从索引的最左前列开始并且不能跳过索引中的列。
③ 不在索引列上做任何操作（计算、函数、（自动或手动）类型转换），会导致索引失效而转向全表扫描。
④ 存储引擎不能使用索引中范围条件右边的列。
⑤ 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），避免`select*`。
⑥ `MySQL`在使用不等于（!=或者<>）的时候无法使用索引，会导致全表扫描。
⑦ `is null`，`is not null` 也无法使用索引。
⑧ `like`以通配符开头（`%abc...`）`MySQL`索引会失效变成全表扫描。
⑨ 字符串不加单引号索引失效。
⑩ 少用`or`，用它来连接时索引会失效。
