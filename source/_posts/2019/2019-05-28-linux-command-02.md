---
layout: post
title: ﻿﻿【Linux 系统】命令大全总结（一）
date: 2019-05-28 20:09:24.000000000 +09:00
categories:
- 技术
tags:
- Linux
toc: true
---

**
摘要：开发过程中经常会使用相关的`Linux`命令,但一直没有静下心来好好分析和理解每个命令，为了能提升和拓展自身`Linux`知识，需要对`Linux`系统知识进行不断的总结。本篇文章主要例举了`top` 命令、`netstat`命令和`ps`命令，除了对相关命令的使用介绍，还重点对相关选项也进行说明，从而更好的理解命令中每个参数的意义。
**
<!-- more -->
<The rest of contents | 余下全文>

# top

作用：显示当前系统正在执行的进程的相关信息，包括进程`ID`、内存占用率、`CPU`占用率等。
英文：The top program provides a dynamic real-time view of a running system.
参数：

|参数|说明|
|:-: | :-: | :-: |
|-b| 批处理|
|-c |显示完整的治命令|
|-I |忽略失效过程|
|-s |保密模式|
|-S |累积模式|
|-i<时间> |设置间隔时间|
|-u<用户名> |指定用户名|
|-p<进程号> |指定进程|
|-n<次数> |循环显示的次数|

实例1：显示当前进程信息。

```
[root@iZrj9hb9k9jtcpp85t8ryeZ ~]# top
top - 16:55:44 up 70 days, 22:39, 2 users, load average: 0.29, 0.10, 0.07
Tasks: 83 total, 1 running, 82 sleeping, 0 stopped, 0 zombie
%Cpu(s): 0.3 us, 0.3 sy, 0.0 ni, 99.3 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
KiB Mem : 1016164 total, 142848 free, 216332 used, 656984 buff/cache
KiB Swap: 1048572 total, 881460 free, 167112 used. 594816 avail Mem
PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
4343 root 20 0 424948 14220 3016 S 0.3 1.4 72:41.60 docker-containe
10620 root 20 0 132656 11972 9068 S 0.3 1.2 90:20.35 AliYunDun
24045 root 20 0 154600 5524 4212 S 0.3 0.5 0:00.02 sshd
1 root 20 0 199092 2804 1480 S 0.0 0.3 13:01.77 systemd
```

第一行：系统运行信息，同`uptime`命令的执行结果，详细说明如下：

|参数|说明|
|:-: | :-: | :-: |
|16:55:44|当前系统时间|
|up 70 days, 22:39|当前系统已经运行的时间为70天22小时39分钟（未重启）|
|2 users|当前有两个用户登录系统|
|load average: 0.29, 0.10, 0.07|1分钟、5分钟、15分钟的负载情况为0.29,0.10,0.07|

> 注：`load average`数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑`CPU`的数量，结果高于5的时候就表明系统在超负荷运转了。

第二行：`Tasks` — 任务（进程），具体信息说明如下：

|参数|说明|
|:-: | :-: | :-: |
|83 total|系统总进程数量83个|
|1 running|处于运行状态1个|
|82 sleeping|处于睡眠状态82个|
|0 stopped|处于停止状态0个|
|0 zombie|处于僵尸状态0个|

第三行：`cpu`状态信息，具体属性说明如下：

|参数|说明|
|:-: | :-: | :-: |
|0.3 us|用户空间占比，0.3%|
|0.3 sy|内核空间占比，0.3%|
|0.0 ni|改变过优先级的进程CPU占比，0.0%|
|99.3 id|空闲CPU占比，99.3%|
|0.0 wa|IO等待CPU占比，0.00%|
|0.0 hi|硬中断（Hardware IRQ）占比，0.00%|
|0.0 si|软中断（Software Interrupts）占比，0.00%|
| 0.0 st|占比，0.00%|

第四行：内存状态统计，具体信息如下：

|参数|说明|
|:-: | :-: | :-: |
|1016164 total|物理总内存量（1G）|
|142848 free|空闲内存量（0.13G）|
|216332 used| 使用中的内存总量（0.25G）|
|656984 buff/cache|缓存的内存量（0.62G）|

第五行：`swap`交换分区信息，具体信息说明如下：

|参数|说明|
|:-: | :-: | :-: |
|1048572 total|交换区总量（1G）|
|881460 free|空闲的交换区总量（0.80G）|
|167112 used|使用的交换区总量（0.20G）|

第六行：空行

第七行：各进程（任务）的状态监控，项目列信息说明如下：

|列名称|说明|
|:-: | :-: | :-: |
|PID|进程id|
|USER|进程所有者|
|PR|进程优先级|
|NI(NICE)|负值表示高优先级，正值表示低优先级。|
|VIRT|进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES|
|RES|进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA|
|SHR|共享内存大小，单位kb。|
|S |进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程|
|%CPU | 上次更新到现在的CPU时间占用百分比。|
|%MEM |进程使用的物理内存百分比。|
|TIME| 进程使用的CPU时间总计，单位1/100秒。|
|COMMAND |进程名称（命令名/命令行）。|

# find

实例1：查找超过10MB的所有`.mp3`文件，并使用一个命令删除它们 。

```
[root@wx /]# find / -type f -name "*.mp3" -size +10M -exec rm {} \;
```

实例2：查找当前目录下所有目录名为`CVS`的子目录的命令。

```
[root@wx /]# find ./CVS -maxdepth 1 -type d -print
```

# df

作用：用于显示磁盘空间使用情况。
英文：`disk free`,report file system disk space usage.

|选项|说明|作用|
|:-: | :-: | :-: | :-: |
|-T|--print-type|显示文件系统的形式。|
|-h|--human-readable|使用人类可读的格式。|

```
[root@wx server]# df -Th
文件系统 类型 容量 已用 可用 已用% 挂载点
/dev/vda1 ext4 40G 7.4G 30G 20% /
devtmpfs devtmpfs 487M 0 487M 0% /dev
tmpfs tmpfs 497M 0 497M 0% /dev/shm
tmpfs tmpfs 497M 644K 496M 1% /run
```

# du

作用：用于显示目录或文件的大小。
英文：`disk usage`,estimate file space usage.

|选项|说明|作用|
|:-: | :-: | :-: | :-: |
|-s|--summarize|仅显示总计。|
|-h|--human-readable|以K，M，G为单位，提高信息的可读性。|

```
[root@wx server]# du -sh ./*
108K	./package.xml
165M	./php7
8.5M	./redis-4.0.11
1.7M	./redis-4.0.11.tar.gz
8.9M	./xdebug-2.6.1
```

# tar

先来弄清两个基础概念：打包和压缩。
打包：是指将一大堆文件或目录变成一个总的文件。
压缩：则是将一个大的文件通过一些压缩算法变成一个小文件。

`tar`命令可以为`linux`的文件和目录创建档案。利用`tar`，可以为某一特定文件创建档案（备份文件），也可以在档案中改变文件，或者向档案中加入新的文件。

语法：

```
tar(选项)(参数)
```

选项：

|选项|说明|作用|
|-| :- | :-: |:- | 
|-x|--extract(提取)或--get|从备份文件中还原文件。|
|-z|--gzip或--ungzip|通过gzip指令处理备份文件。|
|-v|--verbose(详细)|显示指令执行过程。|
|-f|--file|指定备份文件。|
|-c|--create|建立新的备份文件。|
|-t|--list|列出备份文件的内容。|

应用1：以 `gzip` 压缩打包`file`文件夹并命名为`file.tar.gz`（显示打包详细过程）。

```
[root@wx wdata] tar -czvf file.tar.gz file
```

应用2：解压`file.tar.gz`包并命名为`file`（显示解压详细过程）

```
[root@wx wdata] tar -xzvf file.tar.gz file
```

应用3：显示`file.tar.gz`包中的内容。

```
[root@wx wdata] tar -tzvf file.tar.gz
```

# ps

作用：显示当前系统中进程的快照。也就是说，该命令能捕获系统在某一事件的进程状态。
英文：`processes snapshot`,report a snapshot of the current processes.

选项：

|选项|说明|作用|
|-| :- | :-: |:- | 
|a|all|显示所有进程|
|-a|-all|显示同一终端下的所有进程|
|-A|Identical to -e.|显示所有进程|
|-e| Identical to -A.|Select all processes.|
|f|Do full-format listing|显示程序之间的关系|
|u|userlist|  指定用户的所有进程|
|-au|| 显示本用户的详细信息|
|-aux||显示所有包含其他使用者的行程|

实例1：使用`cpu`和内存升序排序来过滤进程，并通过管道显示前10个结果。

```
[root@wx /]# ps -aux --sort -pcpu,-pmem | head -n 10
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
root 10620 0.2 1.1 132656 11968 ? Ssl 9月12 89:52 /usr/local/aegis/aegis_client/aegis_10_51/AliYunDun
root 25 0.1 0.0 0 0 ? S 7月24 147:03 [kswapd0]
```

实例2：使用`PS`实时监控进程状态（动态显示，每秒刷新一次）

```
[root@wx /]# watch -n 1 'ps -aux --sort -pcpu,-pmem | head -n 10'
```

实例3：查找特定进程的信息

```
[root@wx /]# ps -ef | grep nginx
[root@wx /]# ps -aux|grep nginx
```

# netstat

作用：用来打印`Linux`中网络系统的状态信息，获取整个`Linux`系统的网络情况。

英文：`network statistics`, Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships.

选项：

|选项|说明|作用|
|-| :- | :-: |:- | 
|-a|--all|显示所有连线中的Socket。|
|-t|--tcp|显示TCP传输协议的连线状况。|
|-u|--udp|显示UDP传输协议的连线状况。|
|-n|--numeric|直接使用ip地址，而不通过域名服务器。|
|-p|--programs|显示正在使用Socket的程序识别码和程序名称。|
|-l|-listening|显示监控中的服务器的Socket。|
|-e|--extend|显示网络其他相关信息。|

实例1：禁用反向域名解析,只列出 TCP 或 UDP 协议的连接。

```
[root@wx /]# netstat -antu
Proto Recv-Q Send-Q Local Address Foreign Address State
tcp 0 0 0.0.0.0:80 0.0.0.0:* LISTEN
udp 0 0 0.0.0.0:68 0.0.0.0:*
```

实例2：只列出监听中的`nginx`连接，要求获取进程名(-p)、进程号(-p)以及用户 ID(-e)。

```
[root@wx /]# netstat -lnept | grep nginx
tcp 0 0 0.0.0.0:80 0.0.0.0:* LISTEN 0 30270 13332/nginx: master
```

实例3：查看端口占用情况（redis-6379，mysql-3306）

```
[root@wx /]# netstat -tunpl | grep 3306
tcp6 0 0 :::3306 :::* LISTEN 22311/mysqld
```
