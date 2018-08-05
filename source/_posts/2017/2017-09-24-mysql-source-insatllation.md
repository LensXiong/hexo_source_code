layout: post
title: 【MySQL 安装】MySQL 5.7源码编译安装
date: 2017-09-24 08:32:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

** 
摘要：在Linux中安装MySQL，通常为二进制、rpm、源代码编译，这里我们详细讨论源码编译的方式。基于源码安装有更多的灵活性，也就是说我们可以针对自己的硬件平台选用合适的编译器来优化编译后的二进制代码，根据不同的软件平台环境调整相关的编译参数，选择自身需要选择不同的安装组件，设定需要的字符集等一些可以根据特定应用场景所作的各种调整。本文描述了如何在源码方式下安装MySQL5.7。
**
<!-- more -->
<The rest of contents | 余下全文>


# 源码下载

[MySQL 官方源码下载地址](https://downloads.mysql.com/archives/community/)

![](/hexo_blog/img/article/mysql-source-insatllation/mysql-1.png)


复制后的链接地址，执行以下命令：

```
[root@wangxiong server]# mkdir /wdata/server
[root@wangxiong server]# cd /wdata/server
[root@wangxiong server]# wget https://downloads.mysql.com/archives/get/file/mysql-boost-5.7.17.tar.gz
[root@wangxiong server]# mv mysql-boost-5.7.17.tar.gz mysql5.7
[root@wangxiong server]# tar xzf mysql5.7
[root@wangxiong server]# rm -f mysql5.7
```

执行以上命令后，可在本目录下看到mysql-5.7.17文件夹。

# 编译安装
创建MySQL用户：
```
[root@wangxiong server]# useradd mysql -s /sbin/nologin -M
[root@wangxiong server]# id mysql
uid=1001(mysql) gid=1001(mysql) 组=1001(mysql)
```
下载MySQL编译安装所需要的依赖包:
```
[root@wangxiong server]# yum install gcc gcc-c++ ncurses ncurses-devel cmake -y 
```

>gcc：GCC是Linux下的C语言编译工具，mysql源码编译完全由C和C++编写，要求必须安装GCC4.4.6或以上版本
>ncurses：字符终端处理库。
>cmake：MySQL使用cmake跨平台工具预编译源码，用于设置mysql的编译参数。如：安装目录、数据存放目录、字符编码、排序规则等。安装最新版本即可。


进入解压后的MySQL目录，进行cmake编译：
```
[root@wangxiong server]# cd   mysql-5.7.17
[root@wangxiong mysql-5.7.17]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/wdata/server/mysql-5.7.17/boost -DSYSCONFDIR=/etc -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DWITH_FEDERATED_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_MYISAM_STORAGE_ENGINE=1 -DENABLED_LOCAL_INFILE=1 -DENABLE_DTRACE=0 -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_general_ci -DWITH_EMBEDDED_SERVER=1
```
|配置名称|配置值|说明|
|----|------|----|
|DCMAKE_INSTALL_PREFI|/usr/local/mysql|MySQL安装根目录|
|DMYSQL_DATADIR|/usr/local/mysql/data|MySQL数据库文件存放目录|
|DDOWNLOAD_BOOST|1|C++库|
|DWITH_BOOST|/wdata/server/mysql-5.7.17/boost|指向boost库所在目录|
|DSYSCONFDIR|/etc|MySQL配置文件所在目录|
|DWITH_INNOBASE..|1|添加MYISAM引擎支持|
|DWITH_INNOBASE..|1|添加lnnoDB引擎支持|
|DWITH_PARTITION..|1|安装支持数据库分区|
|DWITH_FEDERATED..|1|安装frderated存储引擎|
|DWITH_BLACKHOLE..|1|安装blackhole存储引擎|
|DWITH_MYISAM..|1| 添加MYISAM引擎支持|
|DENABLED_LOCAL_INFILE|1|启动本地的LOCAL_INFILE |
|DENABLE_DTRACE|0|不安装DTRACE|
|DDEFAULT_CHARSET|utf8mb4|设置Mysql默认字符集为utf8mb4|
|DDEFAULT_COLLATION|utf8mb4_general_ci|设置默认字符集校对规则|
|DWITH_EMBEDDED_SERVER|1|嵌入式服务器|




执行以下命令检查上一条执行命令的返回结果，命令执行成功会返回 0，失败返回 1：
```
[root@wangxiong mysql-5.7.17]# echo $?
0
```
编译程序并安装文件：
```
[root@wangxiong mysql-5.7.17]# make && make install
```
编译安装完成。

# 配置启动

配置 mysql 全局变量：
```
[root@wangxiong mysql-5.7.17]# echo 'PATH=/usr/local/mysql/bin/:$PATH' >> /etc/profile 
[root@iZv69wy0n3pc7iZ ~]# source /etc/profile
```

初始化 mysql,必须保证—datadir目录下面为空：
```
[root@wangxiong mysql-5.7.17]# mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```
编辑 mysql 配置文件,找到下面几行,配置如下所示：
```
[root@wangxiong mysql-5.7.17]# vim /etc/my.cnf
datadir=/usr/local/mysql/data 
socket=/tmp/mysql.sock 
log-error=/usr/local/mysql/logs/mysqld.log 
pid-file=/tmp/mariadb.pid
```
创建 log 日志，并授权：
```
[root@wangxiong mysql-5.7.17]# mkdir -p /usr/local/mysql/logs 
[root@wangxiong mysql-5.7.17]# touch /usr/local/mysql/logs/mysqld.log 
[root@wangxiong mysql-5.7.17]# chown -R mysql.mysql /usr/local/mysql/
```
启动 mysql:
```
[root@wangxiong mysql-5.7.17]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld 
[root@wangxiong mysql-5.7.17]# chmod +x /etc/init.d/mysqld 
[root@wangxiong mysql-5.7.17]# /etc/init.d/mysqld start
```
查看 mysql 版本信息：
```
[root@wangxiong mysql-5.7.17]# mysql --version
mysql Ver 14.14 Distrib 5.7.17, for Linux (x86_64) using EditLine wrapper
```

# 相关操作
## 修改密码
```
mysql> set password for '用户名'@localhost = password('新密码');
```
## 查询日志
```
[root@wangxiong mysql-5.7.17]# tail -n 150 /var/log/mysqld.log
```
## 授权连接权限
```
mysql> GRANT ALL PRIVILEGES ON *.* TO '用户名'@'%' IDENTIFIED BY '密码' WITH GRANT OPTION;
```
# 问题解决
## g++: 内部错误：Killed (程序 cc1plus)
错误现象：
```
c++: 编译器内部错误：已杀死(程序 cc1plus)
Please submit a full bug report,
with preprocessed source if appropriate.
```

错误原因：
>[g++: 内部错误：Killed (程序 cc1plus)](https://blog.csdn.net/razertang/article/details/45694567)

```
这个原因是内存不足， 在linux下增加临时swap空间
step 1:
　　#sudo dd if=/dev/zero of=/home/swap bs=64M count=16
　　注释：of=/home/swap,放置swap的空间; count的大小就是增加的swap空间的大小，64M就是块大小，这里是64MB，所以总共空间就是bs*count=1024MB.这里分配空间的时候需要一点时间，等待执行完毕。
step 2:
　　# sudo mkswap /home/swap (可能会提示warning: don't erase bootbits sectorson whole disk. Use -f to force，不用理会)
　　注释：把刚才空间格式化成swap格式
step 3:
　　#sudo swapon /home/swap
　　注释：使刚才创建的swap空间
```

## 建立数据库连接时出错
[mysql自动停止 Plugin FEDERATED is disabled 的完美解决方法_Mysql](https://yq.aliyun.com/ziliao/128521)
```
[root@wangxiong ~]# mysql status;
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
[root@wangxiong ~]# mysql -uroot -p
Enter password:
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
[root@wangxiong ~]# /etc/rc.d/init.d/mysqld status
mysqld 已死，但是 subsys 被锁
[root@wangxiong ~]# /etc/init.d/mysqld start
[root@wangxiong ~]# vim /etc/my.cnf
[mysqld]
federated
```

## 建立数据库失败

[mysql5.6自动停止](https://segmentfault.com/q/1010000011029199)

> 错误日志中最重要的一句："InnoDB: Initializing buffer pool, size = 128.0M
InnoDB: mmap(137363456 bytes) failed; errno 12" 
你的物理内存太小了吧
方法一.加大物理内存
方法二.调小innodb_buffer_pool_size

```
vim /etc/my.conf
innodb_buffer_pool_size=1G
```
