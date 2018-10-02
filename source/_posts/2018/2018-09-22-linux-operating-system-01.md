---
layout: post
title: 【Linux 系统】目录结构、命令全称、命令技巧
date: 2018-09-30 20:09:24.000000000 +09:00
categories:
- 技术
tags:
- Linux
toc: true
---

** 
摘要：既然选择用`Linux`，为何不用的深入一些。作为一名`PHP`开发者，掌握基础的`Linux`命令是远远不够的，本系列文章将重新回顾和思考`Linux`相关知识，其中也包括基础知识，后期也会逐步深入`Linux`更多底层的相关知识。本篇文章重新整理了`Linux`的目录结构说明；汇总了基本命令的英文全称，主要是方便自身用于理解并掌握命令，而不是一味的死记硬背命令缩写，何况死记硬背的知识容易忘；最后也汇总了目前在运用`Linux`命令时能提高效率的一些技巧，方便自身学习和回顾。下图给出了基本文件系统的总体概念（图片是在 `Paul Gardner` 的 `CC BY-SA` 许可下提供的）。
![](/hexo_blog/img/article/linux-operation-sysytem/dir.jpeg)
**
<!-- more -->
<The rest of contents | 余下全文>


# 目录结构

|目录|英文|作用|说明|
|-| :- | :-: | -: |
|/bin|User Binaries|用户二进制可执行文件|系统所有用户使用的命令都设在这里，例如：ps，ls，ping，grep，cp等。|
|/boot|Boot Loader Files|引导系统加载所需的文件|例如：内核的initrd、vmlinux、grub文件。|
|/dev|Device Files|设备文件|包括终端设备、USB或连接到系统的任何设备。|
|/etc|Etcetera(诸如此类) Everything To Configure|包含程序所需的系统配置文件| 例如，包含系统名称、用户及其密码、网络上计算机名称以及硬盘上分区的安装位置和时间的文件都在这里。|
|/home|Home Directiories|用户的家目录|包含保存的文件、个人设置等，一般为单独的分区。|
|/lib|System Libraries|系统库文件|/bin/ and /sbin/中二进制文件必要的库文件。|
|/media|Removable Devices|可移动媒体设备文件|用于挂载可移动设备的临时目录。例如：挂载CD-ROM的/media/cdrom，挂载软盘驱动器的/media/floppy。|
|/mnt|Mount Directory|挂载目录|临时安装目录，系统管理员可以挂载文件系统。例如：cdrom,u盘等，直接插入光驱无法使用，要先挂载后使用|
|/opt|Optinal add-on Apps|可选的附加应用程序|包含从个别厂商的附加应用程序，附加应用程序应该安装在/opt/或者/opt/的子目录下。|
|/proc|Process Information|系统进程的相关信息|一个虚拟的文件系统，包含有关正在运行的进程的信息。例如：/proc/{pid}目录中包含的与特定pid相关的信息。|
|/root|Root|超级用户的家目录||
|/run|Run|存储临时数据|系统进程出于自己不可告人的原因使用它来存储临时数据。|
|/srv|Service Data|包含服务器特定服务相关的数据||
|/sys|System|文件系统|包含连接到计算机的设备的信息|
|/tmp|Temporary Files|包含系统和用户创建的临时文件|当系统重新启动时，这个目录下的文件都将被删除。|
|/usr|User Programs|用户程序|包含二进制文件、库文件、文档和二级程序的源代码。|
|/var|Variable Files|变量文件|包括 - 系统日志文件（/var/log）;包和数据库文件（/var/lib）|

# 英文缩写

|命令|英文全称|中文释义|
|-| :- | :-: | -: |
|man|Manual|手册|
|pwd|Print Working Directory|打印工作目录|
|su|Switch User|切换用户|
|cd|Change Directory|切换目录|
|ls|List Files|列出目录下的文件| 
|ps|Process Status|进程状态|
|mkdir|Make Directory|建立目录|
|rmdir| Remove Directory|移动目录|
|mkfs|Make File System|建立文件系统|
|fsck|File System Check|文件系统检查|
|cat|Concatenate|串联|
|uname|Unix name|系统名称|
|df|Disk Free|磁盘空间信息|
|du|Disk Usage|磁盘使用率|
|lsmod|List Modules|列表模块|
|mv|Move File|移动文件|
|rv|Remove File|删除文件|
|cv|Copy File|复制文件|
|ln|Link File|链接文件|
|fg|Foreground|前景|
|bg|Background|背景|
|chown|Change Owner|改变所有者|
|chgrp|Change Group|改变用户组|
|chmod|Change Mode|改变模式|
|umount|Unmount|卸载|
|cc|C Complier|C 编译|
|dd|Convert an copy||
|tar|Tape Archive|tape,胶带，系； archive，把...存档，解压文件|
|ldd|List Dynamic Dependencies|列出动态相依|
|insmod|Install Module|安装模块|
|rmmod|Remove Module|移除模块|
|lsmod|List Module|列表模块|
|sudo|Superuser Do|超级用户操作|
|yum| Yellow dog Updater, Modified|在Fedora和RedHat以及SUSE中的Shell前端软件包管理器|
|rpm|Redhat Package Manager|Redhat操作系统的包管理器|
|dpkg|Debian Package Manager|Debian 操作系统的包管理器|
|apt|Advanced Package Tool|高级打包工具|
|ln -s|Link Soft|创建一个软连接（快捷方式）|
|.rc|Resource Configuration|资源配置文件结尾，如.bashrc、.xinitrc等|
|.o|Object File|目标文件|
|.so|Shared Object|用于动态链接的共享库|
|.a|Archive|静态库，多个.o合在一起,用于静态连接 |

# 命令技巧

巧妙的` Linux` 命令行技巧能让你节省时间、避免出错，还能让你记住和复用各种复杂的命令，专注在需要做的事情本身，而不是你要怎么做。

## 命令编辑


|命令|说明|作用|
|-| :- | :-: | -: |
|ctrl + a| |光标跳到行首|
|ctrl + e| |光标跳到行尾|
| ^ ||对上一个命令的文本替换并重新执行命令|

例如：执行如下命令，打印磁盘的使用信息。

```
[root@wx /]# du -TH
du：无效选项 -- T
Try 'du --help' for more information.
[root@wx /]# ^du^df^
df -TH
文件系统 类型 容量 已用 可用 已用% 挂载点
/dev/vda1 ext4 43G 7.9G 33G 20% /
devtmpfs devtmpfs 511M 0 511M 0% /dev
```

```
[root@wx sys]# df -th
df: 未处理文件系统
[root@wx sys]# ^-t^-T
df -Th
文件系统 类型 容量 已用 可用 已用% 挂载点
/dev/vda1 ext4 40G 7.4G 30G 20% /
devtmpfs devtmpfs 487M 0 487M 0% /dev
tmpfs tmpfs 497M 0 497M 0% /dev/shm
```

## 复用命令

|命令|作用|
|-| :- | -: |
|!!|复用上一条命令|
|!df|复用上一条以 “df” 开头的命令|
|history|查看历史命令|
|history  [n] |列出最近的n条历史命令|
|![n] |复用命令历史中的 [n]  号命令|

## 查看文件

|命令|作用|
|-| :- | :-: | -: |
|tail -f /var/log/syslog|实时显示日志文件中增加的内容|
|tail -5 test.php|查看最后五行日志信息|
|tail -n +10 test.php|从第10行开始显示文件|

## 查询文件

|命令|作用|
|-| :- | :-: | -: |
|find / -name "php.ini"|查找php.ini文件的位置|
|find /home -iname "*.txt"|在/home目录下忽略大小写，查找以.txt结尾的文件名|
|find . -type f -size +10M|搜索大于10M的文件|
|find . -type f -size -1k|搜索小于1k的文件|
|find . -type f -size 10M|搜索等于10M的文件|
|find . -name "*.txt" -o -name "*.pdf" |当前目录及子目录下查找所有以.txt和.pdf结尾的文件|

## 设置别名

|命令|说明|
|-| :- | :-: | 
|alias rm="rm -i"|更加安全地执行删除操作|
|alias df="df -h"|以 MB 或 G 为单位查看磁盘的空间|
|alias nginxconf="cd /usr/local/nginx/conf/"|Nginx长路径别名|
|unalias nginxconf|取消别名|
|alias b612="ssh -v -l root 47.254.30.91"|SSH别名设置|

## ctrl快捷键
|命令|说明|作用|
|-| :- | :-: | -: |
|ctrl + r|reverse-i-search|反向搜索执行过的命令|
|ctrl + a| head|光标跳到行首|
|ctrl + e|end |光标跳到行尾|
|ctrl + b|backward|光标后退一个字符|
|ctrl + f|forward |光标前进一个字符|
|ctrl+d|delete|删除光标后一个字符|
|ctrl+h|head|删除光标前一个字符|
|ctrl+k||清除光标后至行尾的所有内容|
|ctrl+u||清除光标前至行首的所有内容|
|ctrl + c| clear|杀死当前进程，另起一行|
|ctrl + l| line|清屏|
|ctrl + w|words| 移除光标前的一个单词|
|ctrl + t||交换光标位置前的两个字符|

# 参考文章

[Linux目录结构详细介绍](http://blog.51cto.com/yangrong/1288072)

[Linux常用命令英文全称以及中文解释](http://www.aizhuanji.com/a/8xX9Ogaw.html)

[Find命令](http://man.linuxde.net/find)


