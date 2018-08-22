---
layout: post
title: 【Git服务器】搭建Git服务器（源码安装）
date: 2018-01-03 13:02:24.000000000 +09:00
categories:
- 技术
tags:
- Git
toc: true
---

** 
摘要：如果我们不想使用第三方托管平台托管自己的项目，可以自己搭建一台Git服务器作为私有仓库使用。搭建Git服务器需要准备一台运行Linux的机器，本文使用Centos 7.4。安装Git服务器有两种方式，因yum方式比较简单，本文重点介绍了另一种源码安装的方式，并在此基础上阐述了禁用shell登录的设置及Git钩子的使用。
**
<!-- more -->
<The rest of contents | 余下全文>


# 准备工作

## 系统环境
查看系统版本信息：
```
[root@wx ~]# lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID:    CentOS
Description:    CentOS Linux release 7.4.1708 (Core)
Release:    7.4.1708
Codename:    Core
```
查看系统内核版本号：
```
[root@wx ~]# cat /proc/version 
Linux version 3.10.0-693.2.2.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) ) #1 SMP Tue Sep 12 22:26:13 UTC 2017
[root@wx ~]# uname –a
Linux iZrj9hb9k9jtcpp85t8ryeZ 3.10.0-693.2.2.el7.x86_64 #1 SMP Tue Sep 12 22:26:13 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

## 检查并移除旧Git
安装之前需要使用yum remove git卸载（安装后卸载也可以）。

```
[root@wx ~]# git --version  ## 查看自带的版本
git version 1.8.3
[root@wx ~]# yum remove git ## 移除原来的版本
```

# 安装方式
## 源码安装
1、安装依赖包
```
[root@wx ~]# yum install  autoconf automake libtool
[root@wx ~]# yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel 
[root@wx ~]# yum install gcc-c++ perl-ExtUtils-MakeMaker
```
2、下载&解压

>[GitHub源码下载地址](https://github.com/git/git/releases)

```
[root@wx ~]# cd /wdata/server/
[root@wx ~]# wget https://github.com/git/git/archive/v2.16.0.tar.gz
[root@wx ~]# tar -zxf v2.16.0.tar.gz
[root@wx ~]# rm -f v2.16.0.tar.gz
```

3、编译&安装
```
[root@wx ~]# cd /wdata/server/git-2.16.0/
[root@wx ~]# make configure
[root@wx ~]# ./configure --prefix=/usr/local/git
[root@wx ~]# make profix=/usr/local/git
[root@wx ~]# make install
```

4、设置全局变量
```
[root@wx ~]# echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
[root@wx ~]# source /etc/profile
```
5、检查版本
```
[root@wx ~]# git --version
git version 2.16.0
```
## yum安装
运行以下两条命令，通过yum方式安装：
```
[root@wx ~]# yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel
[root@wx ~]# yum install git
```
> 注：因yum方式本文不做重点介绍，禁用shell登录及钩子的使用参照的是源码安装，相关路径与yum安装有所差别。

# 搭建私有仓库
## 创建git用户组和用户
 创建一个git用户组和用户，用来运行git服务：
```
[root@wx ~]# groupadd git
[root@wx ~]# useradd git -g git
[root@wx ~]# passwd git 
*******
```
## 创建证书目录

收集客户端所有需要登录的用户的公钥，
公钥位于id_rsa.pub文件中，将公钥导入服务器端 /home/git/.ssh/authorized_keys文件里，一行一个。
如果没有该文件创建它：
```
[root@wx ~]# cd /home/git/
[root@wx ~]# mkdir .ssh
[root@wx ~]# chmod 755 .ssh
[root@wx ~]# chown -R git:git .ssh
[root@wx ~]# touch .ssh/authorized_keys
[root@wx ~]# chmod 644 .ssh/authorized_keys
[root@wx ~]# vim authorized_keys
ssh-rsa ****** wangxiong@wangxiongdeMacBook-Pro.local
```
MacBook客户端的公钥两种方式复制 :

```
① vim /Users/wangxiong/.ssh/id_rsa.pub 进行复制
② ssh-copy-id -i /Users/wangxiong/.ssh/id_rsa.pub git@47.254.30.91
输入服务器密码
```
MacBook本地生成SSH公钥：
```
[macbook@wx ~]# cd /Users/wangxiong/.ssh/
[macbook@wx ~]# ssh-keygen
```
> ssh-keygen 会确认密钥的存储位置（默认是 .ssh/id_rsa），然后它会要求你输入两次密钥口令。如果你不想在使用密钥时输入口令，将其留空即可。

## 初始化仓库
因创建完git用户下,在git目录下输入命令：
```
[root@wx ~]# cd /home/git/
[root@wx ~]# mkdir  lensxiong.git && chown -R git:git lensxiong.git
[root@wx ~]# git init --bare lensxiong.git
```
## 克隆仓库
```
[macbook@wx ~]# git clone git@47.254.30.91:/home/git/lensxiong.git
Cloning into 'lensxiong'...
warning: You appear to have cloned an empty repository.
```

## 禁用shell登录

出于安全考虑，创建的git用户不允许登录shell。

1、查看一下git-shell的位置：
```
[root@wx ~]# which git-shell
/usr/local/git/bin/git-shell
```
2、将git-shell的路径添加到/etc/shells文件中：
```
[root@wx ~]# vim /etc/shells
/usr/local/git/bin/git-shell
```
3、修改git用户的shell：
编辑/etc/passwd文件，到类似下面的一行：
```
[root@wx ~]# vim /etc/passwd
 git:x:1002:1002::/home/git:/bin/bash
```

改为：
```
git:x:1001:1001:,,,:/home/git:/usr/local/git/bin/git-shell
```

把/bin/bash改为/usr/local/git/bin/git-shell，这样git这个账户就只能用来克隆或者推送数据到git仓库中了，而不能用它来登录到主机。
```
[macbook@wx ~]# ssh git@47.254.30.91
fatal: Interactive git shell is not enabled.
hint: ~/git-shell-commands should exist and have read and execute access.
Connection to 47.254.30.91 closed.
```

# 使用Git钩子
当我们在本地把开发好的项目文件push到服务器时，只是提交到了创建的Git服务器创建的裸仓库中。还需要进入服务器的web运行目录，通过git pull命令拉取到web目录。从本地仓库git push项目到远程仓库，让push到远程仓库中的项目能在web目录运行起来，还需要web目录进行pull拉取下。push一次就需要pull一下，操作起来很繁琐，相当不方便。为了解决这个问题就可以使用Git中的钩子来解决该问题。

以之前搭建的Git服务器为例，仓库文件地址为：
```
[root@wx ~]# git clone git@47.254.30.91:/home/git/lensxiong.git
```
1、进入其仓库目录下的hooks文件夹，新建post-receive文件:：
```
 [root@wx ~]# cd /home/git/lensxiong.git/hooks/
```

2、在post-receive写入以下内容：
```
[root@wx ~]# vim post-receive
#!/bin/bash 
git --work-tree=/wdata/www/website/ checkout -f
```
3、设置post-receive为可执行文件并给与git权限：
```
[root@wx ~]# ll | grep post-receive
-rw-r--r-- 1 root root 61 1月 5 10:16 post-receive
[root@wx ~]# chmod +x post-receive
[root@wx ~]# chown -R git:git /home/git/lensxiong.git/
[root@wx ~]# ll | grep post-receive
-rwxr-xr-x 1 git git 61 1月 5 10:16 post-receive
```

> 说明：/wdata/www/website/需要同步的站点目录

4、修改web站点目录的权限：
文件post-receive的主要目的是当远程仓库发现有用户执行了push操作，就会执行一个脚本post-receive（钩子）。需要修改你的web站点目录的权限，修改所属用户与用户组为git，否则钩子的权限可能会不足而导致执行失败。设置好钩子后，当你本地再次执行push的时候，会发现web目录的文件也同步的更新了。

```
[root@wx ~]# chown -R git:git /wdata/www/website/
```

# 问题解决
## make configure问题
make configure出现如下错误：
```
[root@wx ~]# make configure
GIT_VERSION = 2.16.0
    GEN configure
/bin/sh: autoconf: 未找到命令
make: *** [configure] 错误 127
```
解决办法，需要安装autoconf：
```
[root@wx ~]# yum install  autoconf automake libtool
```

## libiconv.so.2问题
git安装完后，执行命令的时候出现：
```
git: error while loading shared libraries: libiconv.so.2: cannot open shared object file: No such file or directory
```
解决办法如下：
```
1.在/etc/ld.so.conf中加一行/usr/local/lib，
2.然后运行/sbin/ldconfig
```

## git-upload-pack问题
>[Linux git clone 报错：git-upload-pack: command not found](https://www.cnblogs.com/love-snow/articles/7542267.html)

执行 git clone 出现如下问题：
```
bash: git-upload-pack: command not found
fatal: The remote end hung up unexpectedly
```
原因:原来代码服务器上的git安装路径是/usr/local/git，不是默认路径，根据提示，在git服务器上， 建立链接文件：
```
[root@wx ~]#  ln -s /usr/local/git/bin/git-upload-pack /usr/bin/git-upload-pack 
```

执行 git clone 出现如下问题：
```
bash: git-receive-pack: command not found
fatal: Could not read from remote repository
```
原因:原来代码服务器上的git安装路径是/usr/local/git，不是默认路径，根据提示，在git服务器上， 建立链接文件：
```
[root@wx ~]# ln -s /usr/local/git/bin/git-receive-pack /usr/bin/git-receive-pack
```
## 权限问题
本地向远程推送时，出现如下问题：
```
[macbook@wx ~]# git push
Counting objects: 8732, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (8310/8310), done.
error: remote unpack failed: unable to create temporary object directory
error: failed to push some refs to 'git@47.254.30.91:/home/git/lensxiong.git'
```
检查仓库目录下的权限，已经变为可执行权限：
```
[root@wx ~]# ll  /home/git/lensxiong.git
drwxr-xr-x 2 root root 4096 1月 5 10:16 hooks
```
解决办法：
```
[root@wx ~]# chown -R git:git /home/git/lensxiong.git/
```

# 参考文档
>[官方文档](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)

>[GitHub源码](https://github.com/git/git/releases)

>[Kernel.org网站](https://mirrors.edge.kernel.org/pub/software/scm/git/)
