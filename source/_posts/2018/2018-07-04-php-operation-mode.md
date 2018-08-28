---
layout: post
title: PHP的运行模式
date: 2018-07-04 17:02:24.000000000 +09:00
categories:
- 技术
tags:
- PHP
toc: true
---

** 
摘要：关于`PHP`的运行模式，我想你一定经常听到`CGI`、`FastCGI`、`PHP-CGI`、`PHP-FPM`。他们之间到底是什么样的关系？本文通过详细的介绍，主要分析了`PHP`常见的四种运行模式,﻿`CGI`通用网关接口`(Common Gateway Interface)`、﻿`FastCGI`常驻型`CGI(Long-Live CGI)`、﻿`CLI`命令行模式`(Command Line Interface)`和﻿模块模式(`Apache`等`Web`服务器运行的模式) 。
![](/hexo_blog/img/article/php-operation-mode/01.jpeg)
**
<!-- more -->
<The rest of contents | 余下全文>



# 架构体系

从图上可以看出，`PHP`从上到下是一个四层体系：
* `Application`：这就是我们平时编写的PHP程序，通过不同的SAPI方式得到各种各样的应用模式，如通过Web Server实现Web应用、在命令行下以脚本方式运行等等。
* `SAPI`：SAPI全称是Server Application Programming Interface，也就是服务端应用编程接口，SAPI通过一系列钩子函数，使得PHP可以和外围交互数据，这是PHP非常优雅和成功的一个设计，通过SAPI成功的将PHP本身和上层应用解耦隔离，PHP可以不再考虑如何针对不同应用进行兼容，而应用本身也可以针对自己的特点实现不同的处理方式。
* `Extensions`：围绕着Zend引擎，extensions通过组件式的方式提供各种基础服务，我们常见的各种内置函数（如array系列）、标准库等都是通过extension来实现，用户也可以根据需要实现自己的extension以达到功能扩展、性能优化等目的（如贴吧正在使用的PHP中间层、富文本解析就是extension的典型应用）。
* `Zend引擎`：Zend整体用纯C实现，是PHP的内核部分，它将PHP代码翻译（词法、语法解析等一系列编译过程）为可执行opcode处理，并实现相应的处理方法，实现了基本的数据结构（如hashtable、oo）、内存分配及管理、提供了相应的api方法供外部调用，是一切的核心，所有的外围功能均围绕Zend实现。

# 关系图

![](/hexo_blog/img/article/php-operation-mode/02.png)
# 概念基础
*  `Web Server `：指Apache、Nginx、IIS、Lighttpd、Tomcat等服务器。
*  `Web Application`：指PHP、Java、Asp.net等应用程序。
*  `CGI`：是 Web Server 与 Web Application 之间数据交换的一种协议。
*  `FastCGI`：同 CGI，是一种通信协议，但比 CGI 在效率上做了一些优化。同样，SCGI 协议与 FastCGI 类似。
*  `PHP-CGI`：是 PHP （Web Application）对 Web Server 提供的 CGI 协议的接口程序。
*  `PHP-FPM`：是 PHP（Web Application）对 Web Server 提供的 FastCGI 协议的接口程序，额外还提供了相对智能一些任务管理。


总结：

> ```CGI```是一个协议， ```PHP-CGI```实现了这个协议。

   ```FastCGI```是一个协议， ```PHP-FPM```实现了这个协议。

  ```FastCGI ```是用来提高```CGI```程序性能的。

   ```PHP-CGI```是用来解释```PHP```脚本的程序。


# 运行模式

要想了解PHP的运行模式，首先需要先了解```SAPI```：



* SAPI:```Server Application Programming Interface``` 服务器端应用编程端口。它就是PHP与其它应用交互的接口，PHP脚本要执行有很多种方式，通过Web服务器，或者直接在命令行下，也可以嵌入在其他程序中。

* SAPI提供了一个和外部通信的接口, 常见的有五大运行模式：



>1、CGI通用网关接口(Common Gateway Interface)

2、FastCGI常驻型CGI(Long-Live CGI)

3、CLI命令行模式(Command Line Interface)

4、模块模式(Apache等Web服务器运行的模式)

5、~~ISAPI（Internet Server Application Program Interface）~~

注：在PHP5.3以后，PHP不再有ISAPI模式，安装后也不再有php5iSAPI.dll这个文件。

##  CGI模式

关于[CGI模式](https://zh.wikipedia.org/wiki/%E9%80%9A%E7%94%A8%E7%BD%91%E5%85%B3%E6%8E%A5%E5%8F%A3)，维基百科的解释是这样的：

>通用网关接口（Common Gateway Interface/CGI）是一种重要的互联网技术，可以让一个客户端，从网页浏览器向执行在网络服务器上的程序请求数据。CGI描述了服务器和请求处理程序之间传输数据的一种标准。


CGI就是专门用来和Web服务器打交道的。Web服务器收到用户请求，就会把请求提交给CGI程序（如php-CGI），CGI程序根据请求提交的参数作应处理（解析php），然后输出标准的html语句，返回给Web服务器，WEB服务器再返回给客户端，这就是普通CGI的工作原理。


### 执行过程
![](/hexo_blog/img/article/php-operation-mode/03.png)

* ① 用户通过 Web 浏览器向 Web 服务器发起Http请求。
* ② 当 Web 服务器接受到用户请求后, 例如```index.php```,会通过它配置的CGI服务来执行。

* ③ 启动CGI解析器，生成一个```php-cgi.exe```的子进程处理请求，执行的返回结果交给 Web 服务器，并结束这个子进程（```fork-and-execute```模式）

* ④  Web 服务器得到标准的输出结果后通过Http响应返回给 Web 浏览器。



### 应用场景

*  提供HTTP服务

### 优点

* 跨平台,几乎可以在任何操作系统上实现。
* 将 Web 服务器和具体的程序处理独立开来，结构清晰，可控性强。

### 缺点


* 性能低下，高资源消耗，当用户请求数量非常多时，会大量挤占系统的资源如内存，CPU时间等。
* 用CGI方式的服务器有多少连接请求就会有多少CGI子进程，每个子进程都需要启动CGI解释器、加载配置、连接其他服务器等初始化工作，子进程反复加载是CGI性能低下的主要原因。

## FastCGI模式

![](/hexo_blog/img/article/php-operation-mode/04.png)

### 工作原理


* ① 用户通过 Web 浏览器向 Web 服务器发起Http请求。
* ②   Web 服务器首次启动时载入```FastCGI```进程管理器，例如IIS(ISAPI)、Apache (mod_fastcgi)、Nginx(ngx_http_fastcgi_module)、Lighttpd(mod fastcgi)，从而管理多个PHP-CGI进程来准备响应用户的请求。FastCGI进程管理器自身初始化，启动多个CGI解释器进程 (在任务管理器中可见多个```php-cgi.exe```)并等待来自Web 服务器的连接。PHP的FastCGI进程管理器是```PHP-FPM```(PHP-FastCGI Process Manager)。

* ③ 当 Web 浏览器请求到达Web 服务器时，FastCGI进程管理器选择并连接到一个CGI解释器。Web 服务器将CGI环境变量和标准输入发送到FastCGI子进程```php-cgi```。

* ④  FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器（运行在 Web Server中）的下一个连接。在正常的CGI模式中，```php-cgi.exe```在此便退出了。

* ⑤ Web 服务器得到标准的输出结果后通过Http响应返回给 Web 浏览器。



注：在CGI模式中，你可以想象 CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部dll扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。



### PHP-CGI 执行过程

![](/hexo_blog/img/article/php-operation-mode/05.png)

* ① 初始化php的各种相关变量
* ② 调用并初始化zend虚拟机
* ③ 加载并解析php.ini
* ④ 激活zend，zend加载a.php文件，并做词法、语法的解析，编译a.php脚本为opcode，并执行输出结果后关闭zend虚拟机。
* ⑤ 返回结果给 Web 服务器。


### 优点

* 从稳定性上看, FastCGI是以独立的进程池运行来CGI,单独一个进程死掉,系统可以很轻易的丢弃,然后重新分配新的进程来运行逻辑。
* 从安全性上看,FastCGI支持分布式运算，FastCGI和宿主的server完全独立, FastCGI怎么down也不会把 Web 服务器搞垮。
* 从性能上看, FastCGI把动态逻辑的处理从 Web 服务器中分离出来, 大负荷的IO处理还是留给宿主 Web 服务器，这样宿主 Web 服务器可以一心一意作IO,对于一个普通的动态网页来说, 逻辑处理可能只有一小部分。

### 缺点

* 多进程,消耗较多内存



## CLI模式



## 模块模式



# 参考文章

* [PHP的运行模式](https://blog.csdn.net/hguisu/article/details/7386882)

* [PHP底层的运行机制与原理](https://www.awaimai.com/509.html#i)

* [CGI、FastCGI和PHP-FPM关系图解](https://www.awaimai.com/371.html)

* [PHP设计模式教程](https://www.awaimai.com/patterns)

* [CGI，FastCGI，PHP-CGI与PHP-FPM](http://www.thinkphp.cn/topic/42338.html)

