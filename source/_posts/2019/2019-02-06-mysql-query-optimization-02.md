---
layout: post
title: ﻿【MySQL高级】查询截取分析-慢查询日志
date: 2019-02-06 08:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：慢查询日志是`MySQL`提供的一种日志记录，它是数据库调优的一个主要依据，通过分析慢查询日志我们可以对相关SQL语句变慢的原因进行进一步的分析。由于日志跟踪出来的文件是一个文本文件，查看起来费时费力，`MySQL`也提供了一个工具便于从文本文件里面查找的工具`mysqldumpslow`。本篇文章除了介绍慢查询日志的概念和具体参数，也重点介绍了`MySQL`分析日志工具`mysqldumpslow`。
![慢查询日志-03](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-slow-query/03.jpg?raw=true)
**
<!-- more -->
<The rest of contents | 余下全文>


# 什么是慢查询日志

`MySQL`的慢查询日志是`MySQL`提供的一种日志记录，它用来记录在`MySQL`中响应时间超过阈值的语句，具体指运行时间超过`long_query_time`值的`SQL`，都会被记录到慢查询日志中。

`long_query_time`的默认值为10，意思是运行10秒以上的语句称为慢`SQL`。我们也可以更改这个`long_query_time`的值，比如改为3，表示超过3秒的语句称为慢`SQL`，收集到这些语句，再结合之前的`EXPLIAN`可进行全面的分析。

> 可以通过设置`long_query_time`为0来捕获所有的查询。

# 慢查询日志参数

默认情况下，`MySQL`数据库没有开启慢查询日志，需要手动设置参数`slow_query_log`。当然，如果不是调优需要的话，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。慢查询日志支持将日志记录写入文件。

查看是否开启：

```
mysql> SHOW VARIABLES LIKE '%slow_query_log%';
```

设置开启：

```
mysql> set global slow_query_log = 1;
```
![慢查询日志-01](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-slow-query/01.jpg?raw=true)
> 第一次设置以后查看没有生效，先关闭数据库连接，再重新连接，再次查询就可以看到实际上是修改了。

像`slow_query_log`这样的参数，`MySQL`官方并不推荐长时间开启，因为开启该参数会有一定的性能开销，上述方式开启只是临时开启，如果想永久生效，就必须修改配置文件`my.cnf`。

修改`my.cnf`永久生效参数配置：

```
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/slow-query-log.log
long_query_time=3
log_output=FILE
```

关于慢查询的日志文件参数`slow_query_log_file `，它指定慢查询日志文件的存放路径，如果没有指定参数的话，系统默认会给一个缺省的文件`host-name-slow.log`。

查看并设置`long_query_time`参数，默认值为10秒：

```
mysql> SHOW VARIABLES LIKE 'long_query_time%'
```

假如运行时间正好等于`long_query_time`的值，并不会被记录下来。在`MySQL`源码中，判断的是大于

`long_query_time`的值，而不是大于等于。

设置慢`SQL`的阈值时间：

```
mysql> set global long_query_time = 3;
```

![慢查询日志-02](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-slow-query/02.jpg?raw=true)

> 修改无效，需要重新连接或新开一个会话才能看到修改值。

查看慢查询的条数：
```
mysql> show global status like '%Slow_queries%';
```

# 日志分析工具
在生产环境中，如果要手工分析日志，查找、分析`SQL`，显然是个体力活。`MySQL`提供了日志分析工具`mysqldumpslow`。

|参数|说明|
|--|--|
|s|表示按照何种方式排序|
|c|访问次数|
|l|锁定时间|
|r|返回记录|
|t|查询时间|
|al|平均锁定时间|
|ar|平均返回记录数|
|at|平均查询时间|
|-t|是top n的意思，即为返回前面多少条的数据|
|-g|后边可以写一个正则匹配模式，大小写不敏感的|

![慢查询日志-03](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-slow-query/03.jpg?raw=true)

得到返回记录集最多的10个`SQL`：
```
mysqldumpslow -s r -t 10 /var/lib/mysql/slow-query.log
```
得到访问次数最多的10个`SQL`：
```
mysqldumpslow -s c -t 10 /var/lib/mysql/slow-query.log
```
得到按照时间排序的前10条里面含有左连接的查询语句：
```
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/slow-query.log
```
另外，建议在使用这些命令时结合 | 和 `more`使用，否则有可能出现爆屏的情况：
```
mysqldumpslow -s r -t 10 /var/lib/mysql/slow-query.log | more
```
