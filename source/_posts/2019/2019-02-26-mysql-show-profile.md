---
layout: post
title: 【MySQL高级】使用SHOW PROFILE命令分析性能
date: 2019-02-26 08:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：分析`SQL`执行带来的开销是优化`SQL`的重要手段。在`MySQL`数据库中，可以通过配置`profiling`参数来启用`SQL`剖析。该参数开启后，后续执行的`SQL`语句都将记录其资源开销，诸如`IO`，上下文切换，`CPU`，`Memory`等等。根据这些开销进一步分析当前`SQL`瓶颈从而进行优化与调整。本篇文章主要介绍了相关语法及参数说明、结合实例分析如何使用及剖析结论。
![show-profile-03](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-show-profile/03.jpg?raw=true)
**
<!-- more -->
<The rest of contents | 余下全文>

# 概述

在`MySQL`数据库中，可以通过配置`profiling`参数来启用`SQL`剖析。该参数开启后，后续执行的`SQL`语句都将记录其资源开销，诸如`IO`，上下文切换，`CPU`，`Memory`等，根据这些开销分析当前`SQL`瓶颈从而进行优化与调整。`

# 语法

```
SHOW PROFILE [type [, type] ... ]
    [FOR QUERY n]
    [LIMIT row_count [OFFSET offset]]
type: {
    ALL
  | BLOCK IO
  | CONTEXT SWITCHES
  | CPU
  | IPC
  | MEMORY
  | PAGE FAULTS
  | SOURCE
  | SWAPS
}
```

# 参数

|参数|说明|
|--|--|
|ALL|显示所有的开销信息|
|BLOCK IO|显示块IO开销信息|
|CONTEXT SWITCHES|上下文切换相关开销|
|CPU|显示CPU开销信息|
|IPC|显示发送和接收相关开销信息|
|MEMORY|显示内存开销信息|
|PAGE FAULTS|显示页面错误开销信息|
| SOURCE|显示来自源代码的函数名，以及函数发生的文件的名称和行号|
|SWAPS|显示交换次数相关开销信息|


# 使用

`profiling `是由 `profiling` 会话变量控制的，它的默认值为0（`OFF`）。通过将`profiling`设置为1或者`ON`来启用`profiling`。 

查看`profiling`的值并设置开启：

```
mysql> show variables like 'profiling%'; /*查看profiling的值*/
mysql> select @@profiling;
mysql> SET profiling = 1; /*设置为1开启*/
```
![show-profile-01](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-show-profile/01.jpg?raw=true)
基于上篇文章利用存储过程批量插入的数据，运行如下`SQL`:
```
mysql> select * from emp group by id%10 limit 150000;
mysql> select * from emp group by id%20 limit  5;
mysql> show profiles;
```
`show profiles`显示发送到服务器的最近语句的列表，列表的大小由`profiling_history_size`会话变量控制，该变量的默认值为15，最大值是100。
![show-profile-02](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-show-profile/02.jpg?raw=true)
通过以下命令进一步查看某一条资源的消耗情况，如`io`，`cpu`：
```
mysql> show profile cpu,block io for query 9; 
```
![show-profile-03](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/mysql-show-profile/03.jpg?raw=true)
由上图可知一条`SQL`的完整生命周期，包括初始化，连接前检查权限，优化语句，策略分析，预处理，创建临时表，执行语句，传送数据，删除临时表，查询结束，关闭表，释放资源等一系列工作，其中可以看出因为语句用到`group by`，会导致创建和删除临时表操作，这是导致这条`SQL`语句执行慢的最主要原因。

# 剖析结论
当使用上面的语句诊断`SQL`后，如果出现以下四种情况，会导致性能下降，需要进行语句的优化处理：
① `converting HEAP to MyISAM`，查询结果太大，内存不够使用磁盘。
② `Creating tmp table` 创建临时表，先拷贝数据到临时表，用完后在进行删除。
③ `Copying to tmp table on disk` 将内存中的临时表复制到磁盘，出现此状态非常危险。
④ `locked` 锁表。

# 参考
[官方SHOW PROFILE Syntax文档说明](https://dev.mysql.com/doc/refman/8.0/en/show-profile.html)
