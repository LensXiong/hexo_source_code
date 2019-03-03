---
layout: post
title: ﻿【MySQL高级】MySQL锁机制
date: 2019-03-03 17:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：`MySQL`锁机制是数据库为了保证数据的一致性而使各种共享资源在被并发访问变得有序所设计的一种规则，对于任何一种数据库来说都需要有相应的锁定机制。`MySQL`为了解决并发、数据安全的问题，使用了锁机制。本篇文章主要介绍了锁的基本概念及锁的分类。从对数据操作的粒度，通过相关案例分析了行锁(`INNODB`引擎)、表锁(`MYISAM`引擎)和页级锁(`BDB`引擎 )，对于相关问题给出一些基本的优化方案。
![mysql-lock-02](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-lock/02.jpg?raw=true)
**
<!-- more -->
<The rest of contents | 余下全文>

# 概述
锁是计算机协调多个进程或线程并发访问某一资源的机制。

> 当并发事务同时访问一个资源时，有可能导致数据不一致，因此需要一种机制来将数据访问顺序化，以保证数据库数据的一致性。锁就是其中的一种机制。

在数据库中，除传统的计算资源（如`CPU`、`RAM`、`I/O`）等以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。
> 商品秒杀时，如何保证商品数量不超出实际库存？

这里，除了必须用到事务处理，还需要使用锁对有限的商品资源进行隔离，解决隔离和并发的矛盾。

# 锁分类
从数据操作的类型来分，可分为读锁和写锁。读锁也称为共享锁，是针对同一份数据，多个读操作可以同时进行而不会互相影响。写锁也称为排它锁，当前写操作没有完成前，它会阻断其他写锁和读锁。
从对数据操作的粒度分，可分为行锁(`INNODB`引擎)、表锁(`MYISAM`引擎)和页级锁(`BDB`引擎 )。
# 表级锁
表级锁是`MySQL`中锁定粒度最大的一种锁，表示对当前操作的整张表加锁，它实现简单，资源消耗较少，被大部分`MySQL`引擎支持。最常使用的`MYISAM`与`INNODB`都支持表级锁定。下面通过手动增加表锁来进行表级锁分析。
建表语句1，使用`MYISAM`引擎：
```
create table mylock(
id int not null primary key auto_increment,
name varchar(20)
)engine myisam;
insert into mylock(name) values('a');
insert into mylock(name) values('b');
insert into mylock(name) values('c');
insert into mylock(name) values('d');
insert into mylock(name) values('e');
```
建表语句2：
```
CREATE TABLE book(
`id` INT PRIMARY KEY AUTO_INCREMENT,
`name` VARCHAR(24) NOT NULL DEFAULT '' COMMENT '姓名',
`age` INT NOT NULL DEFAULT 0 COMMENT '年龄',
`pos` VARCHAR(20) NOT NULL DEFAULT '' COMMENT '职位'
) CHARSET utf8 COMMENT '员工记录表';
INSERT INTO book(name,age,pos) VALUES('z3',22,'manager');
INSERT INTO book(name,age,pos) VALUES('July',23,'dev');
INSERT INTO book(name,age,pos) VALUES('2000',23,'dev');
```
手动增加表锁的语法：
```
lock table 表名字1 read(write),表名字2 read(write),其它：
```
查看加锁表的信息：
```
mysql> show open tables;
mysql> show OPEN TABLES where In_use > 0;
mysql> lock table mylock read,book write;
```

![mysql-lock-01](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-lock/01.jpg?raw=true)

读写锁会对数据操作和系统性能会产生一定的影响：
> 对表1`mylock`进行加读锁，`session-1`和`session-2`是否可以查看和操作。

```
mysql> lock table mylock read;
```
![mysql-lock-02](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-lock/02.jpg?raw=true)

根据查询结果可知：
① 当给`mylock`加读锁时， `session-1`只可读`mylock`表，无法写表和读取其它表。`session-2`可读任意表，但不能写`mylock`表。
② 表加读锁之后，写表时系统会产生阻塞，只有当解锁之后才会立即进行写操作。

> 对表1`mylock`进行加写锁，`session-1`和`session-2`是否可以查看和操作。

![mysql-lock-03](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-lock/03.jpg?raw=true)
① 当对`mylock`加写锁时，`session-1`可读可写`mylock`表，`session-2`不能读和写`mylock`表，但是可以读和写`book`表。
② 只有当解锁`mylock`表后，`session-2`的写操作解除阻塞，才可进行写操作。

> MySQL的读写锁有什么区别？

1、对`MyISAM`表的读操作（加读锁），不会阻塞其他进程对同一表的读请求，但会阻塞对同一表的写请求，只有当锁释放后，才会执行其他进程的写操作。
2、对`MyISAM`表的写操作（加写锁），会阻塞其他进程对同一表的读和写操作，只有当写锁释放后，才会执行其他进行的读写操作。

结论：读锁会阻塞写，但是不会阻塞读。写锁则会把所有的读和写都阻塞。

记录`MySQL`内部表级锁定的情况，两个变量说明如下：
```
mysql> show status like 'table%';
```
`Table_locks_immediate`：产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1。
`Table_locks_waited`：出现表级锁定争用而发生等待的次数（不能立即获取锁的次数，每等待一次锁值加1），此值高则说明存在着较严重的表级锁争用情况。

`MyISAM`的读写锁调度是写优先，这也是`MyISAM`不适合做写为主表的引擎，因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。

![mysql-lock-04](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-lock/04.jpg?raw=true)

# 行级锁
`行级锁`是`MySQL`中锁定粒度最细的一种锁，表示只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突。其加锁粒度最小，但加锁的开销也最大。`INNODB`引擎除了支持表级锁定，也支持行级锁，但`INNODB`引擎偏向行级锁。`INNODB`引擎和`MyISAM`引擎最大的两点不同之处，一是`INNODB`引擎支持事务，二是`INNODB`引擎采用行级锁。

> 由于行锁支持事务，需要明白什么是事务？事务的四大属性具体指的什么？

事务是由一组`SQL`语句组成的逻辑处理单元，事务具有`ACID`属性。
`Atomicity`（原子性）：原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚。 因此，事务的操作如果成功就必须要完全应用到数据库，如果操作失败则不会对数据库有任何影响，也就是说，事务是应用中不可再分的最小逻辑执行体。
`Consistent`（一致性）：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以保持数据的完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的。拿转账来说，假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性。
`Isolation`（隔离性）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的独立环境执行，这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。举例来说，对于任意两个并发的事务 T1 和 T2，在事务 T1 看来，T2 要么在 T1 开始之前就已经结束，要么在 T1 结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。
`Durable`（持久性）：事务完成之后，它对数据的修改是永久存在的，即使出现系统故障也能够保持。换句换说，事务一旦提交，对数据库所做的任何改变都要记录到永久的存储器中(通常就是保存到物理数据库)。

创建`SQL`语句，使用`INNODB`引擎：
```
create table test_innodb_lock(a int(11),b varchar(16)) engine=innodb;
insert into test_innodb_lock values(1,'b2');
insert into test_innodb_lock values(3,'3');
insert into test_innodb_lock values(4,'4000');
insert into test_innodb_lock values(5,'5000');
insert into test_innodb_lock values(6,'6000');
insert into test_innodb_lock values(7,'7000');
insert into test_innodb_lock values(8,'8000');
insert into test_innodb_lock values(9,'9000');
insert into test_innodb_lock values(1,'b1');
create index test_innodb_a_ind on test_innodb_lock(a);
create index test_innodb_lock_b_ind on test_innodb_lock(b);
```
![mysql-lock-05](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-lock/05.jpg?raw=true)
如上所示，因为没有设置自动提交，因为事务的隔离性，会出现读己之所写(左边更新表后自己读取数据为4001，但是右边在`commit`前读取的数据为4000，`commit`后才读取到4001)。

![mysql-lock-06](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-lock/06.jpg?raw=true)

当左边第二次更新为4002但未执行`commit`命令时，右边第三次更新为4003会发生阻塞（等待7.43秒），只有当左边提交`commit`后，右边阻塞才会继续执行。

> 索引失效后除了会导致系统性能下降，还会导致行锁变表锁。

间隙锁：
> 什么是间隙锁？

当我们使用范围条件而不是相等条件去检索数据，并请求共享锁或排它锁时，`InnoDB`会给符合条件的已有数据的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做间隙（`GAP`）,
`InnoDB`也会对这个间隙加锁，这种锁机制就是所谓的间隙锁（`Next-Key`锁）。

> 间隙锁带来的危害？

间隙锁有一个比较致命的弱点，就是当锁定一个范围键值后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据，在某些场景下可能会对性能造成很大的危害。

![mysql-lock-07](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-lock/07.jpg?raw=true)

由上图可以看出，当进行范围更新时，因为未`commit`，右边进行数据插入时处于阻塞状态（持续时间为19.54秒），只有当`commit`提交后，阻塞才会消失。

> 如何锁定某一行？如何为需要更新的某一行上锁？

![mysql-lock-08](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-lock/08.jpg?raw=true)
在业务需要锁定某一行的时候，可进行如上语句的操作。` select * for update`锁定某一行后，其它进行某一行的操作会被阻塞，直到锁定行的会话提交`commit`。

# 页级锁
页级锁是`MySQL`中锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级冲突少，但速度慢。

# 小结
`InnoDB`存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能会比表级锁定会要高一些，但是在整体并发处理能力方面要远远优于`MyISAM`的表级锁定。当系统并发量较高的时候，`InnoDB`的整体性能和`MyISAM`相比就会有比较明显的优势。

但是，`InnoDB`存储引擎的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会让`InnoDB`存储引擎的整体性能表现不仅不能比`MyISAM`高，甚至可能会更差，比如使用不当（索引失效等）时，行锁变表锁。

优化方案：
① 尽可能让所有的数据检索都通过索引来完成，避免无索引行锁升级为表锁（`varchar`类型不加单引号会使索引失效）。
② 合理设计索引，尽量缩小锁的范围。
③ 尽可能减少检索条件，避免间隙锁。
④ 尽量控制事务大小，减少锁定资源和时间长度。
⑤ 尽可能低级别事务隔离。

