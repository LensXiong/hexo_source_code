---
layout: post
title: 深入理解Nginx与PHP的交互机制
date: 2017-11-12 10:32:24.000000000 +09:00
categories:
- 技术
tags:
- PHP
toc: true
---

** 
摘要：关于如何配置 Nginx 与FPM ，我想你一定看过很多文章。这些文章更多的从操作的角度出发，告诉我们怎么做，但却没有告诉我们为什么要这么做。本篇文章从 Nginx 与 FPM 的工作机制出发，探讨配置背后的原理，让我们真正理解 Nginx 与 PHP 是如何协同工作的。这篇文章会告诉你什么是CGI？ 什么是FastCGI？什么是PHP-FPM？Nginx与PHP到底是如何通信？
**
<!-- more -->
<The rest of contents | 余下全文>
要说 Nginx 与 PHP 是如何协同工作的，首先得说 CGI (Common Gateway Interface) 和 FastCGI 这两个协议。

# 什么是CGI？
CGI全称是“公共网关接口”(Common Gateway Interface)，HTTP服务器与你的或其它机器上的程序进行“交谈”的一种工具，其程序须运行在网络服务器上。

CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl,tcl等

# 什么是FastCGI？
FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次(这是CGI最为人诟病的fork-and-execute 模式)。

它还支持分布式的运算, 即 FastCGI 程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求。

FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。

众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over特性等等。

# 什么是PHP-FPM？
PHP-FPM的全称是PHP FastCGI Process Manager。

它是 PHP 针对 FastCGI 协议的具体实现，它会通过用户配置来管理一批FastCGI进程。

因此它也是PHP 在多种服务器端应用编程端口（SAPI：cgi、fast-cgi、cli、isapi、apache）里使用最普遍、性能最佳的一款进程管理器。

在PHP-FPM管理下的某个FastCGI进程挂了，PHP-FPM会根据用户配置来看是否要重启补全。

PHP-FPM更像是管理器，而真正衔接Nginx与PHP的则是FastCGI进程。

因此，CGI是通用网关协议，FastCGI则是一种常驻进程的CGI模式程序，而PHP-FPM更像是管理器，用于管理FastCGI进程。

# Nginx服务配置
Nginx配置中包含了 fastcgi_* 开头的一些配置，以及引入的 fastcgi.conf 文件。

其实在fastcgi.conf中，也是一堆fastcgi*的配置项，只是这些配置项相对不常变，通常单独文件保管可以在多处引用。

> Nginx中为何能写很多 fastcgi_* 的配置项。

这是因为Nginx的一个默认内置module实现了FastCGI的Client。

关于Module ngx_http_fastcgi_module的详细文档可以查看[这里]( http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html )。


```
  server {
        listen      80;
        server_name  localhost;
        root /wdata/www/wangxiong/apps/imtou_2.0/public;
        index index.html index.php;
        access_log  logs/host.access.log  main;
        include vhost/allow_ip;
        location ~ \.php {
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_index index.php;
                fastcgi_split_path_info ^(.+\.php)(.*)$;
                fastcgi_param PATH_INFO $fastcgi_path_info;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi.conf;
        }
        location /{
                if (!-e $request_filename){
                        rewrite  ^(.*)$  /index.php?s=$1  last;
                }
        }
```
# fastcgi.conf 文件中的内容
```
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING      $query_string;
fastcgi_param  REQUEST_METHOD    $request_method;
fastcgi_param  CONTENT_TYPE      $content_type;
fastcgi_param  CONTENT_LENGTH    $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI      $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME    $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```
在fastcgi.conf中，有很多的fastcgi_param配置，结合nginx server配置中的fastcgi_pass、fastcgi_index，Nginx与PHP之间打交道就是用的FastCGI。

但再深问FastCGI是什么？它起到衔接Nginx、PHP的什么作用？

# FastCGI的工作原理
　　
1、Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module)
　　
2、FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
　　
3、当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
　　
4、FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。
　　
# Nginx与PHP如何通信？
![](/hexo_blog/img/article/nginx-fastcgi-php/nginx-fastcgi-php.png)
如上图所示，FastCGI的下游，是CGI-APP，在我们的LNMP架构里，这个CGI-APP就是PHP程序。

而FastCGI的上游是Nginx，他们之间有一个通信载体，即图中的socket。

上图中的Pre-fork，则对应着我们PHP-FPM的启动，也就是在我们启动PHP-FPM时便会根据用户配置启动诸多FastCGI触发器（FastCGI Wrapper）。



# 总结
Nginx与PHP分工明确，Nginx负责承载HTTP请求的响应与返回，以及超时控制记录日志等HTTP相关的功能。

PHP则负责处理具体请求要做的业务逻辑，它们俩的这种合作模式也是常见的分层架构设计中的一种，在它们各有专注面的同时，FastCGI又很好的将两块衔接，保障上下游通信交互。
