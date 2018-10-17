---
layout: post
title: 【MySQL 优化】性能优化概览
date: 2018-10-15 23:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

** 
摘要：本篇文章是`MySQL`性能优化系列的开篇，该系列文章以`MySQL`官方英文文档为基础，通过中英文翻译的形式来学习和整理相关知识，所有的内容都是基于自身的认知和理解来翻译，除了基本的翻译，文章中会加入自己的思考，也会有相关知识点的归纳和整理。希望通过不断的坚持和复盘，从中更深入的理解`MySQL`相关知识。本篇文章从数据库自身层面和硬件层面，介绍了一些优化思路。数据库自身设计层面的优化主要从表结构是否合理、是否建立了正确索引、存储引擎选择`InnoDB`还是`MyISAM`、锁机制策略的选择、内存区域的选择进行说明。硬件层面的优化主要从磁盘查询、磁盘读写、`CPU`周期及内存带宽四个方面进行说明。
**
<!-- more -->
<The rest of contents | 余下全文>

# 前言

Database performance depends on several factors at the database level, such as tables, queries, and configuration settings. 

数据库的性能取决于数据库自身层面的几个因素，比如数据表，数据查询和数据库配置的设置。



These software constructs result in CPU and I/O operations at the hardware level, which you must minimize and make as efficient as possible. 

在硬件层面对`CPU`和`I/O`操作时，软件结构本身也会对数据库性能产生一定的影响，因此要尽可能的使数据库性能高效，你必须降低这些操作带来的影响。

As you work on database performance, you start by learning the high-level rules and guidelines for the software side, and measuring performance using wall-clock time.

在处理数据库性能时，你首先要了解软件方面的高级规则和指南，并使用挂钟时间（系统调用中需要指定的时间）来衡量性能。

 As you become an expert, you learn more about what happens internally, and start measuring things such as CPU cycles and I/O operations.

当你成为一个数据库专家时，你应该要了解数据库内部发生的更多信息，并开始测量`CPU`周期和`I/O`操作等内容。

Typical users aim to get the best database performance out of their existing software and hardware configurations. 

一般典型用户的目标是从现有的软件和硬件配置中获得最佳的数据库性能。

Advanced users look for opportunities to improve the MySQL software itself, or develop their own storage engines and hardware appliances to expand the MySQL ecosystem.

而高级用户则是寻找机会去改进`MySQL`软件本身，或者去开发自己的存储引擎和硬件设备来扩展`MySQL`的生态系统。

# 优化层面

官方文档中优化原文如下：

* Optimizing at the Database Level
* Optimizing at the Hardware Level
* Balancing Portability and Performance

也就是主要针对这三个方面进行性能优化：

* 数据库层面的优化
* 硬件层面的优化
* 移植性和性能的平衡

## 数据库层面

The most important factor in making a database application fast is its basic design:

影响数据库性能快慢最重要的因素是数据库本身的基本设计原理：

* Are the tables structured properly? In particular, do the columns have the right data types, and does each table have the appropriate columns for the type of work? For example, applications that perform frequent updates often have many tables with few columns, while applications that analyze large amounts of data often have few tables with many columns.

表的结构是否合理？尤其是，列是否有正确的数据类型，是否建立合适的表或者列来满足实际的需求？例如，执行频繁更新的表通常应该具有少量列，而用于分析大量数据的表尽可能的少，但可以具有大量的列。

* Are the right indexes in place to make queries efficient?

是否有建立合适的索引来提高查询效率？

* Are you using the appropriate storage engine for each table, and taking advantage of the strengths and features of each storage engine you use? In particular, the choice of a transactional storage engine such as InnoDB or a nontransactional one such as MyISAM can be very important for performance and scalability.

是否为每个表使用合适的存储引擎，并且充分利用每种存储引擎的优势和功能？诸如，选择支持事物的存储引擎`InnoDB`还是选择不支持事物的存储引擎`MyISAM`，对性能和可伸缩性的影响都是非常重要的。

>> InnoDB is the default storage engine for new tables. In practice, the advanced InnoDB performance features mean that InnoDB tables often outperform the simpler MyISAM tables, especially for a busy database.

建立一个新表时，`InnoDB`是默认的存储引擎。事实上，尤其对于繁忙的数据库而言，先进的`InnoDB`性能特征意味着`InnoDB`表更优于`MyISAM`表。

* Does each table use an appropriate row format? This choice also depends on the storage engine used for the table. In particular, compressed tables use less disk space and so require less disk I/O to read and write the data. Compression is available for all kinds of workloads with InnoDB tables, and for read-only MyISAM tables.

是否为每个表使用合适的行格式？这个选择也依赖于所使用表的存储引擎。压缩表能够使用较少的磁盘空间，在磁盘`I/O`读写数据也会占用很少的空间，压缩表适用于`InnoDB`表的各种工作情况，`MyISAM`表的只读方式。

* Does the application use an appropriate locking strategy? For example, by allowing shared access when possible so that database operations can run concurrently, and requesting exclusive access when appropriate so that critical operations get top priority. Again, the choice of storage engine is significant. The InnoDB storage engine handles most locking issues without involvement from you, allowing for better concurrency in the database and reducing the amount of experimentation and tuning for your code.

是否使用正确的锁机制策略？例如，大多数情况下通过允许共享访问使数据库操作可以正常进行，但是当请求关键的数据库操作时应该给与优先的权限以便获得独享的访问。同样，存储引擎的选择也非常重要。`InnoDB`引擎不需要你的参与便可处理许多锁定问题，从而在数据库中允许更好的并发性，也可以减少代码中大量的测试和调优。

* Are all memory areas used for caching sized correctly? That is, large enough to hold frequently accessed data, but not so large that they overload physical memory and cause paging. The main memory areas to configure are the InnoDB buffer pool, the MyISAM key cache, and the MySQL query cache.

是否正确使用了用于缓存的所有内存区域？也就是说，较大的内存用于容纳频繁访问的数据，但也不能太大会导致物理内存超载并造成溢出。主内存区可以配置成`InnoDB`的缓存池（`buffer pool`）、`MyISAM`的内存缓冲区（`key cache`）、`MySQL`的查询缓存(`query cache`)。

## 硬件层面

Any database application eventually hits hardware limits as the database becomes more and more busy. A DBA must evaluate whether it is possible to tune the application or reconfigure the server to avoid these bottlenecks, or whether more hardware resources are required. System bottlenecks typically arise from these sources:

随着数据库系统业务越来越繁忙，任何数据库应用程序最终都会受到硬件条件的限制。为了避免这些硬件瓶颈，一个DBA工程师必须调整应用程序，重新配置服务器，或者评估是否需要更多的硬件资源。而系统的瓶颈通常来自以下这些方面：

* Disk seeks. It takes time for the disk to find a piece of data. With modern disks, the mean time for this is usually lower than 10ms, so we can in theory do about 100 seeks a second. This time improves slowly with new disks and is very hard to optimize for a single table. The way to optimize seek time is to distribute the data onto more than one disk.

磁盘查询。每查询一段数据就需要花费一段时间。对于现代的磁盘而言，就意味着花费的时间低于10ms，因此理论上可以说一秒钟大概可进行100次的磁盘查询。使用新磁盘改善时间是非常缓慢的，并且很难去针对单个表进行优化。优化查询时间的唯一方式是分发数据到多个磁盘上。

* Disk reading and writing. When the disk is at the correct position, we need to read or write the data. With modern disks, one disk delivers at least 10–20MB/s throughput. This is easier to optimize than seeks because you can read in parallel from multiple disks.

磁盘读写。当这个磁盘位于正确的位置，我们需要读取或者写入数据。就目前磁盘而言，一个磁盘可提供至少10–20MB/s的吞吐量。因为你可以从多个磁盘并行读取，显然这比查询更容易优化。

* CPU cycles. When the data is in main memory, we must process it to get our result. Having large tables compared to the amount of memory is the most common limiting factor. But with small tables, speed is usually not the problem.

CPU周期。当数据处于主内存中时，我们必须处理它以获得我们想要的结果。拥有大量数据的表相比于内存是

最常见的限制因素，但是对于小量数据的表而言，速度通常都不是问题。

* Memory bandwidth. When the CPU needs more data than can fit in the CPU cache, main memory bandwidth becomes a bottleneck. This is an uncommon bottleneck for most systems, but one to be aware of.

内存带宽。当CPU需要的数据量超过CPU缓存容量时，主内存带宽就成为了瓶颈。对于大多数系统而言这是一个不常见的瓶颈，但是值得我们重视。

# 参考文档

[MySQL 5.7 Reference Manual](https://dev.mysql.com/doc/refman/5.7/en/optimize-overview.html)
