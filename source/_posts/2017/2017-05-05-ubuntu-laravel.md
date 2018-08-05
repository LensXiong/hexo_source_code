layout: post
title:  Ubuntu上部署Laravel项目
date:  2017-05-05 09:17:24.000000000 +09:00
categories: 
- 技术
tags: 
- LNMP
toc: true
reward: true
---
**
摘要：如果你不亲自去编译安装lnmp,你就无法知道这些分开之后的产品有什么特性。更多是你需要根据项目的需求去配置他们，nginx的配置文件、mysql的配置文件、PHP的配置文件，然后根据基本的配置，能不能再做优化，比如加缓存、加速的一些辅助工具。
**
<!-- more -->
<The rest of contents | 余下全文>

# 购买VPS服务器
详情可点击[vultr官网](http://www.vultr.com/?ref=6959672)进行注册。具体教程这里不做讲解，如果有需要可参照[Vultr账号注册及购买使用教程](https://www.jiloc.com/41805.html)进行注册和购买。在选择服务器时最好选择Ubuntu 14.04 LTS版本。
# 安装LNMP环境（PHP7.1）
## 安装基础工具
先进行软件源的更新操作：
```
sudo apt-get update
```
设置语言安装支持中文：
```
sudo apt-get install -y language-pack-en-base
```
命令设置语言环境：
```
locale-gen en_US.UTF-8 locale
```
安装vim编辑器、htop系统监控和进程管理软件、zip压缩文件的解压缩软件：
```
sudo apt-get install -y vim htop git unzip
```
安装 add-apt-repository工具，ubuntu 14下面需要安装这个包：
```
sudo apt-get install software-properties-common
```
将ppa/ondrej/php这个源添加到apt-get的源列表里面。加上LC_ALL=C.UTF-8是因为非utf-8下面会有一个bug：
```
sudo LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php
```
再一次更新软件源：
```
sudo apt-get update
```
## 安装PHP相关
安装php7.1：
```
sudo apt-get -y install php7.1
```
安装php7.1-mysql：
```
sudo apt-get -y install php7.1-mysql
```
安装PHPFastCGI管理器：
```
sudo apt-get install php7.1-fpm
```

安装php7.1的其它扩展：
```
apt-get install php7.1-curl php7.1-xml php7.1-mcrypt php7.1-json php7.1-gd php7.1-mbstring
```

## 安装Nginx服务器
运行以下命令：
```
sudo apt-get -y install nginx
```

## 安装Mysql数据库
运行以下命令：
```
sudo apt-get -y install mysql-server-5.6
```

# 配置环境，运行Laravel
## 编辑PHP的配置文件

```
sudo vim /etc/php/7.1/fpm/php.ini
```
vim命令输入/fix_pathinfo搜索，将cgi.fix_pathinfo=1改为cgi.fix_pathinfo=0:

> 注：Why ? 假设有如下的 URL：http://phpvim.net/foo.jpg，当访问 http://phpvim.net/foo.jpg/a.php 时，foo.jpg 将会被执行，如果 foo.jpg 是一个普通文件，那么 foo.jpg 的内容会被直接显示出来，但是如果把一段 php 代码保存为 foo.jpg，那么问题就来了，这段代码就会被直接执行。这对一个 Web 应用来说，所造成的后果无疑是毁灭性的。具体参看：http://www.cnblogs.com/buffer/archive/2011/07/24/2115552.html

## 编辑Fpm的配置文件

```
sudo vim /etc/php/7.1/fpm/pool.d/www.conf 
```
找到listen = /run/php/php7.1-fpm.sock修改为listen = /var/run/php/php7.1-fpm.sock。当然，你也可以不修改，但必须前后一致，后面会用到这个配置。紧接着运行以下代码重启：

```
sudo service php7.1-fpm restart
```
## 编辑Nginx的配置文件

```
sudo vim /etc/nginx/sites-available/default
```

```
listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /var/www/laravel-ubuntu/public;
        index index.php index.html index.htm;

        # Make site accessible from http://localhost/
        server_name localhost;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ /index.php?$query_string;
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
        }
        location ~ \.php$ {
                try_files $uri /index.php =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php7.1-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
```
> 注：root：是你的项目的public目录，也就是网站的入口
    index：添加了index.php，告诉Nginx先解析index.php文件
    server_name：你的域名，没有的话填写localhost
    location / :  try_files修改为了try_files $uri $uri/ /index.php?$query_string;
    location ~ .php$:告诉Nginx怎么解析Php，原封不动复制即可，但注意：fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;的目录要和之前fpm的配置文件中的listen一致。



## 创建网站目录
如果你还没有/var/www目录，运行mkdir /var/www，然后将Nginx的用户名和用户组www-data分配给它：

```
sudo chown -R www-data:www-data   /var/www/laravel-ubuntu/
```

## 创建Laravel项目
### Git方式创建
进入项目存放位置:
```
cd /var/www/
```
git 克隆：

```
git clone  你的代码仓库路径
```
再次给予目录权限，运行（在/var/www/laravel-ubuntu下面）：
```
sudo chmod -R 775 storage/
sudo chown -R www-data:www-data /var/www/laravel-ubuntu
```
重点事项：
检查nginx配置的root位置是否正确：
```
root /var/www/var/www/laravel-ubuntu/public;
```
storage 文件夹权限是否正确，否则在项目文件夹laravel-ubuntu下运行以下命令：
```
sudo chmod -R 775 storage/
```
注意 laravel-ubuntu 这个目录的所有者为: www-data:www-data，否则执行以下命令：
```
sudo chown -R www-data:www-data   /var/www/laravel-ubuntu/
```
重启fpm:
```
sudo service php7.1-fpm restart
```
重启nginx：
```
sudo service nginx restart
```
### Composer方式创建
composer的安装：
```
 curl -sS https://getcomposer.org/installer | php
```

全局使用composer:
```
sudo mv composer.phar /usr/local/bin/composer
```

> 注意：由于被墙的原因，第一条指令通常会失败,可以直接在composer官网下载composer.phar压缩包,然后再移动到 /usr/local/bin/composer

composer来安装larvavel项目：
```
cd /var/www
composer create-project laravel/laravel larave-ubuntu --prefer-dist "5.1.*"
```
# 配置和域名解析

# 配置SSL证书

# 参考文章

[一步一步教你部署自己的 Laravel 应用&程序到服务器](https://laravel-china.org/topics/2914/step-by-step-to-teach-you-to-deploy-your-laravel-applications-and-programs-to-the-server)

[从零开始部署 Laravel 项目](https://www.laravist.com/discuss/laravel/laravel-project-from-scratch-deployment-752)
