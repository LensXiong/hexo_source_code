---
layout: post
title: Xdebug-PHP的调试和分析工具
date: 2018-08-09 10:46:24.000000000 +09:00
categories:
- 技术
tags:
- PHP
toc: true
---

** 
摘要：`Xdebug`是一个开放源代码的`PHP`程序调试器(即一个`Debug`工具)，可以用来跟踪，调试和分析`PHP`程序的运行状况。它作为`PHP`调试工具，提供了丰富的调试函数，可将`Xdebug`安装配置为`Zend studio`、`PHPStrom`调试PHP的第三方插件，通过开启自动跟踪(`auto_trace`)和分析器功能，可以直观的看到`PHP`源代码的性能数据，以便优化`PHP`代码。本文主要讲解在`CentOS7`中如何进行`Xdebug`的源码安装，并简单介绍了传值赋值和传引用赋值时，使用`xdebug_debug_zval()`函数显示`refcount`和`is_ref`的值。
![](/hexo_blog/img/article/xdebug-tool/PHPxdebug.png)
**
<!-- more -->
<The rest of contents | 余下全文>
# 关于Xdebug

`Xdebug`是一个开放源代码的`PHP`程序调试器(即一个`Debug`工具)，可以用来跟踪，调试和分析`PHP`程序的运行状况。

它提供了各种自带的函数，并对已有的某些`PHP`函数进行覆写，可以方便地用于调试排错；`Xdebug`还可以跟踪程序的运行，通过对日志文件的分析，我们可以迅速找到程序运行的瓶颈所在，提高程序效率，从而提高整个系统的性能。

# 安装环境

先检查`Liux`版本：

```
[root@wx ~]# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core)
```

查看`PHP`版本：

```
[root@wx ~]# php -v
PHP 7.1.15 (cli) (built: Jul 25 2018 20:18:54) ( ZTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
```

所需相关信息如下所示：

操作系统：`CentOS Linux release 7.5.1804 (Core)`
phpize目录：`/usr/local/php7/bin/phpize`
php安装目录：`/usr/local/php7`
php.ini配置文件路径：`/usr/local/php7/etc/php.ini`
php-config路径：`/usr/local/php7/bin/php-config`

# 安装配置

这部分描述如何进行源码安装`Xdebug`。

## 下载地址

方法一：[官方下载地址](https://xdebug.org/download.php)
方法二：[Github下载地址](https://github.com/xdebug/xdebug)
方法三：Git 地址：
```
git clone git://github.com/xdebug/xdebug.git
```

## 源码下载

根据安装的`PHP`版本，选择合适的`Xdebug`版本。这里选择`Xdebug 2.6.1`的版本进行安装。

下载命令如下：

```
[root@wx server]# wget https://xdebug.org/files/xdebug-2.6.1.tgz
[root@wx server]# tar -xzf xdebug-2.6.1.tgz
[root@wx server]# rm -rf xdebug-2.6.1.tgz
```

## 编译安装

```
[root@wx server]# cd xdebug-2.6.1
[root@wx xdebug-2.6.1]# /usr/local/php7/bin/phpize
Configuring for:
PHP Api Version: 20160303
Zend Module Api No: 20160303
Zend Extension Api No: 320160303
[root@wx xdebug-2.6.1]# ./configure --with-php-config=/usr/local/php7/bin/php-config --enable-xdebug
[root@wx xdebug-2.6.1]# make && make install
Installing shared extensions: /usr/local/php7/lib/php/extensions/no-debug-zts-20160303/
  +----------------------------------------------------------------------+
  | |
  | INSTALLATION INSTRUCTIONS |
  | ========================= |
  | |
  | See http://xdebug.org/install.php#configure-php for instructions |
  | on how to enable Xdebug for PHP. |
  | |
  | Documentation is available online as well: |
  | - A list of all settings: http://xdebug.org/docs-settings.php |
  | - A list of all functions: http://xdebug.org/docs-functions.php |
  | - Profiling instructions: http://xdebug.org/docs-profiling2.php |
  | - Remote debugging: http://xdebug.org/docs-debugger.php |
  | |
  | |
  | NOTE: Please disregard the message |
  | You should add "extension=xdebug.so" to php.ini |
  | that is emitted by the PECL installer. This does not work for |
  | Xdebug. |
  | |
  +----------------------------------------------------------------------+
```

> 注，请留意安装完成后`Installing shared extensions: /usr/local/php7/lib/php/extensions/no-debug-zts-20160303/`

需要记录下`xdebug.so`的路径，后续配置`PHP`时需要。

## 配置PHP

```
[root@wx xdebug-2.6.1]# echo 'zend_extension="/usr/local/php7/lib/php/extensions/no-debug-zts-20160303/xdebug.so"' >> /usr/local/php7/etc/php.ini
[root@wx xdebug-2.6.1]# killall php-fpm
[root@wx xdebug-2.6.1]# php-fpm
[root@wx xdebug-2.6.1]# php -m | grep xdebug
xdebug
```

# 简单使用

每个`php`变量存在一个叫`zval`的变量容器中。一个`zval`变量容器，除了包含变量的类型和值，还包括两个字节的额外信息。

第一个额外字节是`refcount`，用以表示指向这个`zval`变量容器的变量(也称符号即`symbol`)个数。所有的符号存在一个符号表中，其中每个符号都有作用域(`scope`)，那些主脚本(比如：通过浏览器请求的的脚本)和每个函数或者方法也都有作用域。

第二个是`is_ref`，是个`bool`值，用来标识这个变量是否是属于引用集合(`reference set`)。通过这个字节，`php`引擎才能把普通变量和引用变量区分开来，由于`php`允许用户通过使用&来使用自定义引用，`zval`变量容器中还有一个内部引用计数机制，来优化内存使用。

具体可参照[引用计数相关基本知识](http://www.php.net/manual/zh/features.gc.refcounting-basics.php#features.gc.refcounting-basics)。

关于传值赋值和传引用赋值，可以使用`xdebug_debug_zval()`函数显示`refcount`和`is_ref`的值。

## 传值赋值

新建文件`test1.php`：

```
[root@wx www]# vim test1.php
<?php
$a = 'wangxiong';
$c = $b = $a;
xdebug_debug_zval( 'a' );
unset( $b, $c );
xdebug_debug_zval( 'a' );
?>
```

运行如下命令后：

```
[root@wx www]# php test1.php
a: (refcount=4, is_ref=0)='wangxiong'
a: (refcount=2, is_ref=0)='wangxiong'
```

新建文件`test2.php`：

```
[root@wx www]# vim test2.php
<?php
$a = 'wangxiong';
$c = $b = $a;
$c = $b = 'lensxiong';
xdebug_debug_zval('a');
xdebug_debug_zval('b');
xdebug_debug_zval('c');
```

运行如下命令后：

```
[root@wx www]# php test2.php
a: (refcount=2, is_ref=0)='wangxiong'
b: (refcount=3, is_ref=0)='lensxiong'
c: (refcount=3, is_ref=0)='lensxiong'
```

COW特性：

> 写时复制` copy on write`。刚开始` $a` ，` $b` 都指向同一个结构体，当有一方修改值的时候，比如` $b` ，才会进行分裂。

## 传引用赋值

新建文件`test3.php`：
```
[root@wx www]# vim test3.php
<?php
$a = 3;
$b = &$a;
xdebug_debug_zval('a');
xdebug_debug_zval('b');
```
运行如下命令后：
```
[root@wx www]# php test3.php
a: (refcount=2, is_ref=1)=3
b: (refcount=2, is_ref=1)=3
```
新建文件`test4.php`：
```
[root@wx www]# vim test4.php
<?php
$a = 3;
$b = &$a;
$b = 5;
xdebug_debug_zval('a');
xdebug_debug_zval('b');
```
运行如下命令后：
```
[root@wx www]# php test4.php
a: (refcount=2, is_ref=1)=5
b: (refcount=2, is_ref=1)=5
```
