---
layout: post
title: 【Nginx安装】Nginx 1.12 源码编译安装（CentOS7）
date: 2017-12-04 22:58:04.000000000 +09:00
categories:
- 技术
tags:
- Nginx
toc: true
---

** 
![](/hexo_blog/img/article/nginx-source -code-installation/nginx-banner.png)
**
<!-- more -->
<The rest of contents | 余下全文>

# 编译环境版本
CentOS 版本:
```
[root@wangxiong ~]# cat /etc/redhat-release  # Linux查看版本当前操作系统发行版信息
[root@wangxiong ~]# CentOS Linux release 7.4.1708 (Core)
```

>[Nginx官网下载地址](https://nginx.org/en/download.html)

![](/hexo_blog/img/article/nginx-source -code-installation/nginx-download.jpeg)
# 下载Nginx源码并解压
创建源码下载解压的目录：
```
[root@wangxiong ~]# mkdir /wdata/server
```
进入目录：
```
[root@wangxiong ~]# cd /wdata/server
```
运行下面的命令，下载nginx的1.12.2稳定版本：
```
[root@wangxiong ~]# wget https://nginx.org/download/nginx-1.12.2.tar.gz
```
解压Nginx的源码包：
```
[root@wangxiong ~]# tar xzf nginx-1.12.2.tar.gz
```
重新命令文件名：
```
[root@wangxiong ~]# mv nginx-1.12.2 nginx
```
删除下载的压缩包：
```
[root@wangxiong ~]# rm -f nginx-1.12.2.tar.gz
```
# 下载Nginx编译安装所需的依赖包
```
[root@wangxiong ~]# yum install gcc gcc-c++ pcre pcre-devel zlib-devel openssl-devel jemalloc-devel -y
```
以下是关于各个依赖包的详细说明(如果想进一步研究，可自行Google)：
① gcc 编译器
```
gcc为GNU Compiler Collection的缩写，可以编译C和C++源代码等，它是GNU开发的C和C++以及其他很多种语言的编译器。
gcc 在编译C++源代码的阶段，只能编译 C++ 源文件，而不能自动和 C++ 程序使用的库链接（编译过程分为编译、链接两个阶段，注意不要和可执行文件这个概念搞混，相对可执行文件来说有三个重要的概念：编译（compile）、链接（link）、加载（load）。源程序文件被编译成目标文件，多个目标文件连同库被链接成一个最终的可执行文件，可执行文件被加载到内存中运行）。
因此，通常使用 g++ 命令来完成 C++ 程序的编译和连接，该程序会自动调用 gcc 实现编译。
```
②  gcc-c++ 编译器
```
gcc-c++也能编译C源代码，只不过把会把它当成C++源代码，后缀为.c的，gcc把它当作是C程序，而g++当作是c++程序；后缀为.cpp的，两者都会认为是c++程序，注意，虽然c++是c的超集，但是两者对语法的要求是有区别的。
```
③ pcre-devel
```
在Nginx编译需要 PCRE(Perl Compatible Regular Expression)，因为Nginx 的Rewrite模块和HTTP 核心模块会使用到PCRE正则表达式语法。
```
④ zlib-devel 
```
nginx启用压缩功能的时候，需要此模块的支持。
```
⑤ openssl-devel
```
开启SSL的时候需要此模块的支持。
```
⑥ jemalloc-devel
```
为了避免内存碎片与并发扩展
```

#  创建Nginx的使用用户www
```
[root@wangxiong ~]# useradd www -s /sbin/nologin -M
```
useradd -s：指定用户登入后所使用的shell；
useradd -M：不要自动建立用户的登入目录；

显示www真实有效的用户ID(UID)和组ID(GID):
```
[root@wangxiong ~]# id www
[root@wangxiong ~]# uid=1000(www) gid=1000(www) groups=1000(www)
```

# 编译安装
① 进入nginx源码下载解压的目录：
```
[root@wangxiong ~]# cd /wdata/server/nginx
```
② 编译参数设置：
```
[root@wangxiong ~]# ./configure --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module --with-http_gzip_static_module  --with-http_sub_module --with-ld-opt='-ljemalloc'
```
编译参数解释：
```
--user=www    # worker进程运行的用户
--group=www  # worker进程运行的组
--prefix=/usr/local/nginx # nginx安装的根路径,所有其它路径都要依赖该选项
--with-http_stub_status_module #  启用ngx_http_stub_status_module支持（获取nginx自上次启动以来的工作状态）
--with-http_stub_status_module #支持nginx状态查询
--with-http_ssl_module  # 使用https协议模块。默认情况下，该模块没有被构建。前提是openssl与openssl-devel已安装
--with-http_v2_module # 支持HTTP/2协议
--with-http_gzip_static_module # 开启GZIP功能
--with-http_sub_module # nginx替换网站响应内容
--with-ld-opt='-ljemalloc' # 优化Nginx内存管理
```
./configure作用：

* 检查机器的一些配置和环境，系统的相关依赖。如果缺少相关依赖，脚本会停止执行，软件安装失败。
* 根据之前检查环境和依赖的结果，生产Makefile文件（main job）

③ 执行安装命令：
```
[root@wangxiong ~]# make && make install
```
关于make的解释说明：
* make是Unix系统下的一个包。执行make命令需Makefile文件。make会根据Makefile文件中指令来安装软件。
* Makefile文件中有许多标签，来表示不同的section。一般的，make会编译源代码并生成可执行文件，其实Makefile主要就是描述文件编译的相互依赖关系

关于make install的解释说明：
* 当执行make命令不加任何参数，程序就会按照Makefile的指令在相应的section间跳转并且执行相应的命令
* 加上install参数即执行make install时，程序只会执行install section处的命令。install section的指令会将make阶段生产的可执行文件拷贝到相应的地方，例如/usr/local/bin
* make clean 会删除上次make生产的obj文件以及可执行文件


# Nginx全局环境变量设置
```
[root@wangxiong ~]# /usr/local/nginx/sbin/nginx -v
```
打开系统的环境变量文件：
```
[root@wangxiong ~]# vim /etc/profile
```
在profile文件的末尾加上：
```
[root@wangxiong ~]# PATH=/usr/local/nginx/sbin/:$PATH
```
>注：一定要加上:$PATH,否则系统的命令会失效。

保存后重新连接便可使用：
```
[root@wangxiong ~]# nginx -v
```

# 配置访问网站的IP或者域名：
```
http {
    include      mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen      80;
        server_name  XX. XX. XX. XX.;
        location / {
            root  html;
            index  index.html index.htm;
        }
        error_page  500 502 503 504  /50x.html;
        location = /50x.html {
            root  html;
        }
    }
```

# 实例安全组规则
如果访问IP失败，请添加安全组的规则：
找到服务的实例，点击管理实例，添加安全组规则：
```
规则方向：入方向
授权策略：允许
协议类型：地址段访问
端口范围：-1/1
授权对象：0.0.0.0/0
优先级：1
```

# 相关常用命令
查看进程：
```
[root@wangxiong ~]# ps -ef | grep nginx  
```
停止进程：
```
[root@wangxiong ~]# kill -QUIT 2072 # 从容停止
[root@wangxiong ~]# kill -TERM 2072 # 快速停止
[root@wangxiong ~]# pkill -9 nginx # 强制停止
```
验证nginx配置文件是否正确：
```
[root@wangxiong ~]# nginx -t
```
重启nginx:
```
[root@wangxiong ~]# nginx -s reload
```
端口查看：
```
[root@wangxiong ~]# netstat -lntp
```
# 错误处理
## nginx.pid无效问题：
```
[root@wangxiong ~]# nginx -s reload
nginx: [error] invalid PID number "" in "/usr/local/nginx/logs/nginx.pid"
```
解决办法：
```
[root@wangxiong ~]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```
## 访问IP无响应：

可能的原因是阿里云的安全规则组受限，解决办法新增如下安全规则组：

> 云服务器ECS->我的资源云服务器->更多->网络和安全组->安全组配置->配置规则->添加安全组规则

|名称|对应值|
|-----|-------|
|网卡类型|内网|
|规则方向|入方向|
|授权策略|允许|
|协议类型|全部|
|端口范围|-1/-1|
|优先级||
|授权类型|地址段访问|
|授权对象|0.0.0.0/0|

## nginx启动出错

```
[root@wangxiong ~]# nginx -s reload
nginx: [error] invalid PID number "" in "/var/run/nginx.pid"
```
解决办法：

需要执行启动命令：

启动代码格式：nginx安装目录地址 -c nginx配置文件地址

```
[root@wangxiong ~]# nginx -c /usr/local/nginx/conf/nginx.conf
[root@wangxiong ~]# nginx -s reload
```
nginx.conf文件的路径可以从nginx -t的返回中找到。
```
[root@wangxiong ~]# nginx -s reload
```
## nginx 502 错误
错误1：
```
[crit] 12828#0: *148 connect() to unix:/dev/shm/php-cgi.sock failed (2: No such file or directory) while connecting to upstream, client: 71.192.130.32, server: 47.95.216.27, request: "POST / HT
```
解决办法：
[解决办法](https://www.jianshu.com/p/f4048b2922d0)

错误2：
查看php的端口和服务：
```
[root@wangxiong ~]# netstat -lntup |grep php
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                  LISTEN      25971/php-fpm  
```
如果无法看到php相关的端口和服务，运行php-fpm进行重新启动。
```
[root@wangxiong ~]# php-fpm
```

# 访问网站速度提升
是使用“gzip”方法压缩响应的过滤器，有助于将传输数据的大小减少一半甚至更多。

>[模块ngx_http_gzip_module](https://nginx.org/en/docs/http/ngx_http_gzip_module.html)

```
# Gzip Compression
  gzip on;
  gzip_buffers 16 8k;
  gzip_comp_level 6;
  gzip_http_version 1.1;
  gzip_min_length 256;
  gzip_proxied any;
  gzip_vary on;
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";
```
# nginx 强制http跳转到https的配置文件
先监听80端口，为http请求，然后强制转发到https监听的443端口上面：
```
listen      80;
server_name www.wwxiong.com;
rewrite ^/(.*) https://$server_name$1 permanent;
```


