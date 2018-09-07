---
layout: post
title: 【PHP核心】PHP的运行模式
date: 2018-07-04 17:02:24.000000000 +09:00
categories:
- 技术
tags:
- PHP
toc: true
---

** 
摘要：关于`PHP`的运行模式，我想你一定经常听到`CGI`、`FastCGI`、`PHP-CGI`、`PHP-FPM`。他们之间到底是什么样的关系？本文通过详细的介绍，主要分析了`PHP`常见的四种运行模式,﻿`CGI`通用网关接口`(Common Gateway Interface)`、﻿`FastCGI`常驻型`CGI(Long-Live CGI)`、﻿`CLI`命令行模式`(Command Line Interface)`和﻿模块模式(`Apache`等`Web`服务器运行的模式) 。
![](/hexo_blog/img/article/php-operation-mode/php-cgi.jpeg)
**
<!-- more -->
<The rest of contents | 余下全文>

# PHP的四层体系

PHP的四层体系架构如下图所示：

![](/hexo_blog/img/article/php-operation-mode/01.jpeg)

从图上可以看出，PHP从上到下是一个四层体系：

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

> `CGI`是一个协议， `PHP-CGI`实现了这个协议。
   `FastCGI`是一个协议， `PHP-FPM`实现了这个协议。
  `FastCGI `是用来提高`CGI`程序性能的。
   `PHP-CGI`是用来解释`PHP`脚本的程序。

# PHP的运行模式

要想了解PHP的运行模式，首先需要先了解`SAPI`：

* SAPI:`Server Application Programming Interface` 服务器端应用编程端口。它就是PHP与其它应用交互的接口，PHP脚本要执行有很多种方式，通过Web服务器，或者直接在命令行下，也可以嵌入在其他程序中。

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
* ② 当 Web 服务器接受到用户请求后, 例如`index.php`,会通过它配置的CGI服务来执行。
* ③ 启动CGI解析器，生成一个`php-cgi.exe`的子进程处理请求，执行的返回结果交给 Web 服务器，并结束这个子进程（`fork-and-execute`模式）
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

关于[FastCGI模式](https://zh.wikipedia.org/wiki/FastCGI)，维基百科的解释是这样的：

> 快速通用网关接口（`Fast Common Gateway Interface／FastCGI`）是一种让交互程序与`Web`服务器通信的协议`FastCGI`是早期通用网关接口（`CGI`）的增强版本。`FastCGI`致力于减少网页服务器与`CGI`程序之间交互的开销，从而使服务器可以同时处理更多的网页请求。

### 执行过程
![](/hexo_blog/img/article/php-operation-mode/04.png)

* ① 用户通过 Web 浏览器向 Web 服务器发起Http请求。
* ② Web 服务器首次启动时载入`FastCGI`进程管理器，例如IIS(ISAPI)、Apache (`mod_fastcgi`)、Nginx(`ngx_http_fastcgi_module`)、Lighttpd(`mod fastcgi`)，从而管理多个`PHP-CGI`进程来准备响应用户的请求。FastCGI进程管理器自身初始化，启动多个CGI解释器进程 (在任务管理器中可见多个`php-cgi.exe`)并等待来自Web 服务器的连接。PHP的FastCGI进程管理器是PHP-FPM(`PHP-FastCGI Process Manager`)。
* ③ 当 Web 浏览器请求到达Web 服务器时，FastCGI进程管理器选择并连接到一个CGI解释器。Web 服务器将CGI环境变量和标准输入发送到FastCGI子进程`php-cgi`。
* ④  FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回`Web Server`。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器（运行在` Web Server`中）的下一个连接。而在正常的CGI模式中，`php-cgi.exe`在此便退出了。
* ⑤ Web 服务器得到标准的输出结果后通过Http响应返回给 Web 浏览器。

注：在`CGI`模式中，你可以想象`CGI`通常有多慢。每一个`Web`请求，`PHP`都必须重新解析`php.ini`、重新载入全部`dll`扩展并重初始化全部数据结构。使用`FastCGI`，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(`Persistent database connection`)可以工作。

### 应用场景

*  提供HTTP服务

### 优点

* 高稳定性。`FastCGI`是以独立的进程池运行来`CGI`,单独一个进程死掉,系统可以很轻易的丢弃,然后重新分配新的进程来运行逻辑。
* 高性能。`FastCGI`把动态逻辑的处理从`Web Server`中分离出来, 大负荷的`IO`处理还是留给宿主`Web Server`, 这样宿主`Web Server`可以一心一意作`IO`,对于一个普通的动态网页来说, 逻辑处理可能只有一小部分, 大量的图片等静态`IO`处理完全不需要逻辑程序的参与。
* 良好的安全性。`FastCGI`和宿主的`Web Server`完全独立, `FastCGI`怎么`Down`也不会把`Web Server`搞垮。
* 良好的扩展性。 `FastCGI`是一个中立的技术标准, 完全可以支持任何语言写的处理程序。

### 缺点

* 多进程,消耗较多内存。因为是多进程，所以比`CGI`多线程消耗更多的服务器内存，`PHP-CGI`解释器每进程消耗7至25兆内存，将这个数字乘以50或100就是很大的内存数。

> `Nginx 0.8.46+PHP 5.2.14`(`FastCGI`)服务器在3万并发连接下，开启的10个Nginx进程消耗150M内存（15M\*10=150M），开启的64个`php-cgi`进程消耗1280M内存（20M\*64=1280M），加上系统自身消耗的内存，总共消耗不到2GB内存。如果服务器内存较小，完全可以只开启25个`php-cgi`进程，这样`php-cgi`消耗的总内存数才500M。

## CLI模式

`Command Line Interface`的简称，即`PHP`命令行接口，在`windows`和`linux`下都支持`PHP-CLI`模式,它可以直接在命令行下运行,那就意味着完全可以不要任何`http`容器. 例如 `php test.php`。

相关详细解释，具体可参考[官方文档-PHP 的命令行模式](http://php.net/manual/zh/features.commandline.php)。

### 具体使用

① 在命令行直接运行代码：

```
[root@wx]# php -r 'print_r(get_defined_constants());'
array(6) {
    [PCNTL_ENOEXEC] => 8
    [PCNTL_ENOTDIR] => 20
    [PCNTL_ETXTBSY] => 26
    [STDIN] => Resource id #1
    [STDOUT] => Resource id #2
    [STDERR] => Resource id #3
}
```

② 在命令行中传参数：

```
[root@wx wdata]# php -r 'var_dump($argv);' -- -name wangxiong -sex male
array(5) {
  [0]=>
  string(1) "-"
  [1]=>
  string(5) "-name"
  [2]=>
  string(9) "wangxiong"
  [3]=>
  string(4) "-sex"
  [4]=>
  string(4) "male"
}
```

③ 通过脚本方式进行运行：

```
#!/usr/bin/php
<?php
    var_dump($argv);
?>
```

为该文件设置正确的运行属性（例如：`chmod +x test`):

```
[root@wx wdata]# chmod +x test
[root@wx wdata]# ./test -h -- foo
array(4) {
  [0]=>
  string(6) "./test"
  [1]=>
  string(2) "-h"
  [2]=>
  string(2) "--"
  [3]=>
  string(3) "foo"
}
```

相关参数命令：

```
[root@wx wdata]# php -h
  -a Run interactively
  -c <path>|<file> Look for php.ini file in this directory
  -n No configuration (ini) files will be used
  -d foo[=bar] Define INI entry foo with value 'bar'
  -e Generate extended information for debugger/profiler
  -f <file> Parse and execute <file>.
  -h This help
  -i PHP information
  -l Syntax check only (lint)
  -m Show compiled in modules
  -r <code> Run PHP <code> without using script tags <?..?>
  -B <begin_code> Run PHP <begin_code> before processing input lines
  -R <code> Run PHP <code> for every input line
  -F <file> Parse and execute <file> for every input line
  -E <end_code> Run PHP <end_code> after processing all input lines
  -H Hide any passed arguments from external tools.
  -S <addr>:<port> Run with built-in web server.
  -t <docroot> Specify document root <docroot> for built-in web server.
  -s Output HTML syntax highlighted source.
  -v Version number
  -w Output source with stripped comments and whitespace.
  -z <file> Load Zend extension <file>.

  args... Arguments passed to script. Use -- args when first argument
                   starts with - or script is read from stdin
  --ini Show configuration file names
  --rf <name> Show information about function <name>.
  --rc <name> Show information about class <name>.
  --re <name> Show information about extension <name>.
  --rz <name> Show information about Zend extension <name>.
  --ri <name> Show configuration for extension <name>.
```

### 应用场景

* 定时任务。
* 开发桌面应用就是使用`PHP-CLI`和`GTK`包。
* 开发`shell`脚本。

### 优点

* 利用`crontab`去跑`php`，可以给服务器减压，当然在这里有一个条件，就是实时性要求不高。比如：`sns`中的好友动态，这个实时要求不高，但是数据量比较大，这个时候定时跑的话，会给`web`服务器，数据库服务器分担不小的压力。
* 可以定时去完成某一事情，比如：我要删除一个月前，用户留言，这个时候，写的`php`脚本在`crontab`去执行，一天跑一次就行了。而不是手动去执行`php`程序。

### 缺点

* 无法为普通用户提供`http`服务。

## 模块模式

模块模式指将`php`作为`web`服务器的一个模块运行。

### 工作原理

① 在`Apache`启动时加载模块：

模块模式是以`mod_phpx`模块的形式集成，此时`mod_phpx`模块的作用是接收`Apache`传递过来的`PHP`文件请求，并处理这些请求，然后将处理后的结果返回给`Apache`。如果我们在`Apache`启动前在其配置文件中配置好了`PHP`模块（`mod_phpx`）， `PHP`模块通过注册`apache2`的`ap_hook_post_config`挂钩，在`Apache`启动的时候启动此模块以接受`PHP`文件的请求。

② 在`Apache`运行时动态加载：

这意味着对服务器可以进行功能扩展而不需要重新对源代码进行编译，甚至根本不需要停止服务器。我们所需要做的仅仅是给服务器发送信号`HUP`或者`AP_SIG_GRACEFUL`通知服务器重新载入模块。但是在动态加载之前， 我们需要将模块编译成为动态链接库。此时的动态加载就是加载动态链接库。 `Apache`中对动态链接库的处理是通过模块`mod_so`来完成的，因此`mod_so`模块不能被动态加载，它只能被静态编译进`Apache`的核心。这意味着它是随着`Apache`一起启动的。

> Apache是如何加载模块的呢？

以`mod_php7`模块为例，首先我们需要在`Apache`的配置文件`httpd.conf`中添加一行：

```
LoadModule php7_module modules/mod_php7.so
```
这里我们使用了`LoadModule`命令:

该命令的第一个参数是模块的名称，名称可以在模块实现的源码中找到。

第二个选项是该模块所处的路径。 如果需要在服务器运行时加载模块，可以通过发送信号`HUP`或者`AP_SIG_GRACEFUL`给服务器，一旦接受到该信号，`Apache`将重新装载模块， 而不需要重新启动服务器。

### 应用场景

* 提供http服务。

### 优点

* 安装配置方便，不需要安装代码解析程序。
* 支持多线程，占用资源少。
*  支持大并发。当然一定情况下不如`FastCGI`支持的大并发。

### 缺点

*  仅能与Apache一起工作。
*  增加了Apache子进程内存开销。
*  当更改`php.ini`文件后，需要重启Apache。

## PHP-CGI 执行过程

具体过程：
① 通过浏览器访问`index.php`，通过`Web Server`转发动态请求并传递相应的参数给`php-cgi`。
② `php-cgi`调用并初始化zend引擎加载并解析`index.php`后返回给`php-cgi`。
③ `php-cgi`再将返回结果返回给`Web Server`，`Web Server`再返回给浏览器。

![](/hexo_blog/img/article/php-operation-mode/php-cgi.jpeg)

* ① 初始化php的各种相关变量
* ② 调用并初始化zend虚拟机
* ③ 加载并解析php.ini
* ④ 激活zend，zend加载a.php文件，并做词法、语法的解析，编译a.php脚本为opcode，并执行输出结果后关闭zend虚拟机。
* ⑤ 返回结果给 Web 服务器。

# 参考文章

* [PHP的运行模式](https://blog.csdn.net/hguisu/article/details/7386882)
* [PHP底层的运行机制与原理](https://www.awaimai.com/509.html#i)
* [CGI、FastCGI和PHP-FPM关系图解](https://www.awaimai.com/371.html)
* [PHP设计模式教程](https://www.awaimai.com/patterns)
* [CGI，FastCGI，PHP-CGI与PHP-FPM](http://www.thinkphp.cn/topic/42338.html)

