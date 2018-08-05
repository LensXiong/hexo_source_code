---
layout: post
title: 【PHP安装】PHP7.1源码编译安装
date: 2017-11-12 23:12:04.000000000 +09:00
categories:
- 技术
tags:
- PHP
toc: true
---

** 
![](/hexo_blog/img/article/php-source-code-install/php7.1.jpeg)
**
<!-- more -->
<The rest of contents | 余下全文>

# 编译环境及参数
先检查Linu环境：
```
[root@wangxiong ~]# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 
```
运行PHP版本信息：
> 未安装PHP前可先不查看

```
[root@wangxiong ~]# php -v
PHP 7.0.17 (cli) (built: Mar 17 2017 16:15:28) ( ZTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
```
运行PHP编译参数查看命名：
> 未安装PHP前可先不查看

```
[root@wangxiong ~]# php -i |grep configure  
Configure Command =>  './configure'  '--prefix=/usr/local/php7' '--with-config-file-path=/usr/local/php/etc' '--with-pdo-mysql=mysqlnd' '--with-mysqli=mysqlnd' '--enable-fpm' '--enable-static' '--enable-maintainer-zts' '--enable-inline-optimization' '--enable-sockets' '--enable-wddx' '--enable-zip' '--enable-calendar' '--enable-bcmath' '--enable-soap' '--with-zlib' '--with-iconv' '--with-gd' '--with-xmlrpc' '--enable-mbstring' '--with-curl' '--with-freetype-dir=/usr/local/freetype' '--with-openssl' '--disable-fileinfo' '--with-iconv=/usr/local/libiconv' '--enable-ftp' '--enable-session' '--with-mysql-sock=/tmp/mysql.sock'
```

# PHP 源码下载

>[PHP官网下载地址](http://www.php.net/downloads.php)

选择相应的PHP版本后获取下载链接，例如运行以下命令下载7.1.15这个版本的源码包：

```
[root@wangxiong ~]# wget http://cn2.php.net/get/php-7.1.15.tar.gz/from/this/mirror
```

解压下载包并重新命名为php7：

```
[root@wangxiong ~]# tar xzf mirror
[root@wangxiong ~]# mv php-7.1.15 php7
```

# 编译安装准备
下载PHP编译安装所需的依赖包：

```
[root@wangxiong ~]# yum install libxml2-devel curl-devel libpng-devel freetype-devel -y
```

下载PHP编译安装所依赖的libiconv库

```
[root@wangxiong ~]# wget https://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.15.tar.gz
[root@wangxiong ~]# tar xf libiconv-1.15.tar.gz
[root@wangxiong ~]# cd libiconv-1.15
[root@wangxiong ~]# ./configure 
[root@wangxiong ~]# make && make install
```

# 源码安装
① 进入php源码目录指定编码参数：
```
[root@wangxiong ~]# './configure'  '--prefix=/usr/local/php7' '--with-config-file-path=/usr/local/php/etc' '--with-pdo-mysql=mysqlnd' '--with-mysqli=mysqlnd' '--enable-fpm' '--enable-static' '--enable-maintainer-zts' '--enable-inline-optimization' '--enable-sockets' '--enable-wddx' '--enable-zip' '--enable-calendar' '--enable-bcmath' '--enable-soap' '--with-zlib' '--with-iconv' '--with-gd' '--with-xmlrpc' '--enable-mbstring' '--with-curl' '--with-freetype-dir=/usr/local/freetype' '--with-openssl' '--disable-fileinfo' '--with-iconv=/usr/local/libiconv' '--enable-ftp' '--enable-session' '--with-mysql-sock=/tmp/mysql.sock'
```
② 执行命令 
```
[root@wangxiong ~]# make && make install
```
# 配置PHP全局变量

运行 php -v 时需要此命令：

```
[root@wangxiong ~]# echo 'PATH=/usr/local/php7/bin/:$PATH' >>/etc/profile
```

运行php-fpm时需要执行此命令：

```
[root@wangxiong ~]# echo 'PATH=/usr/local/php7/sbin/:$PATH' >>/etc/profile
```

执行环境变量文件：

```
[root@wangxiong ~]# source /etc/profile 
```

查看PHP版本：
```
[root@wangxiong]# php -v
PHP 7.1.15 (cli) (built: July 19 2017 19:05:16) ( ZTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2017 Zend Technologies
```

查看PHP安装的模块：
```
[root@wangxiong ~]# php -m
```

# 配置并启动PHP服务
① 复制并重新命名php-fpm配置文件：
```
[root@wangxiong ~]# cp /usr/local/php7/etc/php-fpm.conf.default /usr/local/php7/etc/php-fpm.conf
```
② 复制并重新命名www.conf配置文件：
```
[root@wangxiong ~]# cp /usr/local/php7/etc/php-fpm.d/www.conf.default /usr/local/php7/etc/php-fpm.d/www.conf 
```
启动php-fpm服务：
```
[root@wangxiong ~]# php-fpm
```
查看网络端口情况：
```
[root@wangxiong php-fpm.d]# netstat -lntup | grep php
tcp        0      0 127.0.0.1:9000          0.0.0.0:*              LISTEN      14792/php-fpm: mast 
```

# php-fpm启动失败
```
[root@wangxiong etc]# php-fpm
ERROR: unable to bind listening socket for address '127.0.0.1:9000': Address already in use (98)
ERROR: FPM initialization failed
[root@iZ2zehy7gff0ksoydtymuvZ etc]#
```
解决办法：
```
[root@wangxiong]# killall php-fpm
[root@wangxiong]# service php-fpm start
```

# nginx 502 错误
错误1：
```
[crit] 12828#0: *148 connect() to unix:/dev/shm/php-cgi.sock failed (2: No such file or directory) while connecting to upstream, client: 71.192.130.32, server: 47.95.216.27, request: "POST / HT
```
解决办法：
[解决办法](https://www.jianshu.com/p/f4048b2922d0)

错误2：
查看php的端口和服务：
```
[root@wangxiong]# netstat -lntup |grep php
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                  LISTEN      25971/php-fpm  
```
如果无法看到php相关的端口和服务，运行php-fpm进行重新启动。
```
[root@wangxiong]# php-fpm
```
