---
layout: post
title: 【PHP-Resque】基于Redis的消息队列
date: 2018-05-22 18:02:24.000000000 +09:00
categories:
- 技术
tags:
- 队列
toc: true
---

** 
摘要：当系统中出现“生产”和“消费”的速度或稳定性等因素不一致的时候，就需要消息队列，作为抽象层，弥合双方的差异。消息队列（`Message Queue`，简称MQ）是在消息的传输过程中保存消息的容器。本篇文章从项目实际需求出发，介绍使用消息队列的原因？消息队列的应用场景有哪些？并引申出队列系统的概念，以`Redis`作为存储队列， `PHP-Resque`作为队列系统为例，介绍了相关组件`PHP` 的 `Redis` 扩展(`PHP-Redis`)的安装、 `PCNTL` 扩展的安装和补救方法，最后对队列系统最重要的`Job`任务、`Queue`队列、`Worker`执行者做了简单讲解。
![](/hexo_blog/img/article/php-redis-queue/php-redis-queue.png)
**
<!-- more -->
<The rest of contents | 余下全文>

# 什么是消息队列?

> 什么是消息？
消息是在两台计算机间传送的数据单位。消息可以非常简单，例如只包含文本字符串；也可以更复杂，可能包含嵌入对象。 消息被发送到队列中。

> 什么是消息队列？

消息队列（`Message Queue`，简称`MQ`），从字面意思上看，本质是个队列，`FIFO`先入先出，只不过队列中存放的内容是`Message`而已。实质上，`MQ`是在消息的传输过程中保存消息的容器。

> 消息队列管理器扮演的什么角色？

消息队列管理器在将消息从它的源中继到它的目标时充当中间人。

# 为什么要使用消息队列?

在实际的开发中，你一定会遇到下面的问题：

> 后台系统中需要导出成百上千万条的用户数据如何处理。

> 某个项目放款时需要给成百上千个用户生成合同文件。

> 用户注册时，需要发送注册成功的邮件，注册短信通知，平台系统操作指南，发送红包和加息券，如果是好友邀请还会发送好友奖励，分析个人信息用来做推荐使用，等等。

当系统中出现“生产“和“消费“的速度或稳定性等因素不一致的时候，就需要消息队列，作为抽象层，弥合双方的差异。

一般服务器都有设置执行的超时时间，使用`PHP`去完成一些比较耗时的后台操作就会出现超时问题。

任务处理类的系统，都可以把用户发起的任务请求接收过来存到消息队列中，然后后端开启多个应用程序从队列中取任务进行处理。

对于用户注册而言，服务端添加用户的基本信息后，用户就可以登录进去做自己想做的事情，不需要一直等到所有业务都处理才可以进入系统。

至于其他的事情，非要在这一次请求中全部完成么？值得用户浪费时间等你处理这些对他来说无关紧要的事情么？所以实际当第一步做完后，服务端就可以把其他的操作放入对应的消息队列中然后马上返回用户结果，由消息队列异步的进行这些操作。

消息队列在生活中随处可见：

> 假如你去商场中排队买奶茶，你只需要告诉店家你需要的奶茶并付款后，你便可以离开队伍，去旁边买热狗，逛花店，买电影票。之后根据等位号码回来取走你要的奶茶即可。这样，你不需要花费自己的时间排在队伍中一直等待，你可以做很多你想做的事情。

# 使用消息队列的应用场景

* 异步处理：非核心流程异步化，提高系统响应性能。
* 应用解耦：消息发送者的成功不依赖消息接受者（比如有些银行接口不稳定，但调用方并不需要依赖这些接口）。
* 广播：只需要关心消息是否送达了队列，至于谁希望订阅，是下游的事情。
* 流量削峰：通过异步处理，将短时间高并发产生的事务消息存储在消息队列中，从而削平高峰期的并发事务。
* 日志处理：将消息队列用在日志处理中，比如Kafka的应用，解决大量日志传输的问题。
* 消息通讯：消息队列一般都内置了高效的通信机制，因此也可以用于单纯的消息通讯，比如实现点对点消息队列或者聊天室等。

# 什么是队列系统？

一个队列系统包含以下三个部分：

* Job任务：负责处理对应事件的逻辑，比如合同文件的生成。在`Resque`中一个`Job`就是一个`Class`。

* Queue队列：负责`Job`的存取，` Queue`队列基于`Redis`来实现。`Resque`提供了一个简单的队列管理器，可以实现将`Job`插入/取出队列等功能。

* Worker执行者：`Worker`常驻内存，循环`POP`队列中的服务。从`Queue`中取`Job`来执行，实时监听队列中有无`Job`，在`CLI`模式下以后台守护方式运行。

具体的实现流程如下：

*  将一个后台任务编写为一个独立的`Class`，这个`Class`就是一个`Job`。
*  在需要使用后台程序的地方，系统将`Job Class`的名称以及所需参数放入队列。
*  以命令行方式开启一个`Worker`，并通过参数指定`Worker`所需要处理的队列。
*  `Worker`作为守护进程运行，并且定时检查队列。
*  当队列中有`Job`时，`Worker`取出`Job`并运行，即实例化`Job Class`并执行`Class`中的方法。

满足任务队列系统的条件：

* 必须有一个队列类型的数据结构，支持推送和拉取任务。
* 速度非常快
* 可以实现多个服务端的共享
* 支持数据持久化

你可以选择满足以上条件的队列系统，或使用第三方的程序，像`RabbitMQ`、`Gearman`、`Redis` 等。

本篇文章将会使用 `Redis`存储队列，并且使用 `PHP-Resque`作为队列系统。

`Resque`是开源在 `Github` 上的一个使用 `Ruby` 编写的队列系统，`PHP-Resque` 是 `Resque` 的 `PHP` 版本。

# 安装 PHP-Resque 

`PHP-Resque` 是依赖 `Redis` 的，所以需要先安装 `Redis` 及 `PHP` 的 `Redis` 扩展。
以下是所有需要安装的组件：

* [Redis](https://redis.io/)
* [PHP 的 Redis 扩展(PHP-Redis)](https://github.com/phpredis/phpredis/releases)
* [PHP 的 PCNTL 扩展](http://www.php.net/downloads.php)
* [PHP-Resque](https://github.com/chrisboulton/php-resque)

## 安装 Redis

`REmote DIctionary Server`(`Redis`) 是一个由`Salvatore Sanfilippo`写的`key-value`存储系统。

它通常被称为数据结构服务器，因为值（`value`）可以是字符串(`String`), 哈希(`Map`), 列表(`list`), 集合(`sets`) 和有序集合(`sorted sets`)等类型。

具体下载和安装文档可参考[Redis官方网站](https://redis.io/download)。

安装命令：

```
[root@wx ~]# cd /usr/local/
[root@wx ~]# wget http://download.redis.io/releases/redis-4.0.11.tar.gz
[root@wx ~]# tar xzf redis-4.0.11.tar.gz
[root@wx ~]# cd redis-4.0.11
[root@wx ~]# make
```

用以下命令启动`Redis`:

```
[root@wx ~]# /usr/local/redis-4.0.7/src/redis-server
```

登录客户端查看`Redis`的守护进程配置:

```
[root@wx ~]# /usr/local/redis-4.0.7/src/redis-cli
127.0.0.1:6379>config get daemonize
1) "daemonize"
2) "no"
```

将`Redis`服务修改为守护进程在后台运行：

```
[root@wx ~]# vim /usr/local/redis-4.0.11/redis.conf 136行 daemonize no 更改为 daemonize yes
```

## 安装 PHP-Redis 扩展

具体安装过程可参考Github上[安装PHP-Redis说明](https://github.com/phpredis/phpredis/blob/develop/INSTALL.markdown)。

### 安装环境

操作系统：`CentOS Linux release 7.5.1804 (Core)`
phpize目录：`/usr/local/php7/bin/phpize`
php安装目录：`/usr/local/php7`
php.ini配置文件路径：`/usr/local/php7/etc/php.ini`
php-config路径：`/usr/local/php7/bin/php-config`
nginx安装目录：`/usr/local/nginx`

> `phpize`是用来扩展`php`扩展模块的，通过`phpize`可以建立`php`的外挂模块。比如你想在原来编译好的`php`中加入`memcached`或者`ImageMagick`等扩展模块，可以使用`phpize`。

### 安装编译工具

```
[root@wx server]# yum install wget  make gcc gcc-c++ zlib-devel openssl openssl-devel pcre-devel kernel keyutils  patch perl
```

### 源码安装

相关版本可参见[Github-PHP-Redis](https://github.com/phpredis/phpredis/releases):

① 下载相关版本并解压进入到安装目录：

```
[root@wx server]# wget https://github.com/phpredis/phpredis/archive/4.0.0.tar.gz
[root@wx server]# tar xzvf 4.0.0.tar.gz && cd phpredis-4.0.0
```

② 用`phpize`生成`configure`配置文件并配置:

```
[root@wx phpredis-4.0.0]# /usr/local/php7/bin/phpize
[root@wx phpredis-4.0.0]# ./configure --with-php-config=/usr/local/php7/bin/php-config
```

③  编译和安装：

```
[root@wx phpredis-4.0.0]# make && make install
```

安装成功后会出现如下的安装路径：

```
Installing shared extensions: /usr/local/php7/lib/php/extensions/no-debug-zts-20160303/
```

### 配置php支持

```
[root@wx phpredis-4.0.0]# find / -name php.ini
/usr/local/php7/etc/php.ini
[root@wx phpredis-4.0.0]# echo 'extension=redis.so'  >>  /usr/local/php7/etc/php.ini
```

重启`php-fpm`:

```
[root@wx phpredis-4.0.0]# killall php-fpm
[root@wx phpredis-4.0.0]# php-fpm
[root@wx phpredis-4.0.0]# php -m | grep redis
redis
```

查看`phpinfo`信息，搜索`redis`即可看到到相应的扩展。

## 安装 PCNTL 扩展

### 编译时添加

在`PHP`中进程控制支持默认是关闭的。您需要使用 `--enable-pcntl` 配置选项重新编译`PHP`的` CGI`或`CLI`版本以打开进程控制支持。

```
[root@wx ~]# './configure' '--prefix=/usr/local/php7' '--with-config-file-path=/usr/local/php7/etc' '--with-pdo-mysql=mysqlnd' '--with-mysqli=mysqlnd' '--enable-fpm' '--enable-static' '--enable-maintainer-zts' '--enable-inline-optimization' '--enable-sockets' '--enable-wddx' '--enable-zip' '--enable-calendar' '--enable-bcmath' '--enable-soap' '--with-zlib' '--with-iconv' '--with-gd' '--with-xmlrpc' '--enable-mbstring' '--enable-phar' '--enable-pcntl' '--with-curl' '--with-freetype-dir=/usr/local/freetype' '--with-openssl' '--disable-fileinfo' '--with-iconv=/usr/local/libiconv' '--enable-ftp' '--enable-session' '--with-mysql-sock=/tmp/mysql.sock'
```



### 编译后补救

编译的时候没有带`--enable-pcntl`参数的补救办法:

```
[root@wx server]# wget http://cn2.php.net/get/php-7.1.15.tar.gz/from/this/mirror
[root@wx server]# tar xzf mirror && mv php-7.1.15 php7
[root@wx server]# cd /wdata/server/php7/ext/pcntl
[root@wx pcntl]# find / -name phpize
/usr/local/php7/bin/phpize
[root@wx pcntl]# find / -name php-config
/usr/local/php7/bin/php-config
[root@wx pcntl]# find / -name php.ini
/usr/local/php7/etc/php.ini
[root@wx pcntl]# /usr/local/php7/bin/phpize
[root@wx pcntl]# ./configure --with-php-config=/usr/local/php7/bin/php-config
[root@wx pcntl]# make && make install
Installing shared extensions: /usr/local/php7/lib/php/extensions/no-debug-zts-20160303/
[root@wx pcntl]#  echo 'extension=pcntl.so' >> /usr/local/php7/etc/php.ini
[root@wx pcntl]# killall php-fpm
[root@wx pcntl]# php-fpm
[root@wx pcntl]# php -m | grep pcntl
pcntl
```

## 下载 PHP-Resque

下载`PHP-Resque`库可参考以下任意一种方式：

* Github下载：[连接地址](https://github.com/chrisboulton/php-resque)
* 直接下载：[连接地址](https://github.com/chrisboulton/php-resque/zipball/master)
* Git方式克隆:

```
[root@wx ~]# git clone git://github.com/chrisboulton/php-resque.git
```

* Composer下载：

```
[root@wx ~]# composer require chrisboulton/php-resque
```

将以上下载的`PHP-Resque`库文件放在项目中合理位置，并引入该文件。

# 启动Worker

`php-resque `的环境变量有:
*  `QUEUE`： 必选，会决定 worker 要执行什么任务，重要的在前，例如 QUEUE=notify,mail,log 。也可以设定為 QUEUE=* 表示执行所有任务。
* `APP_INCLUDE` ：可选，加载文件用的。可以设成 APP_INCLUDE=require.php ，在 require.php 中引入所有 Job 的 Class即可。
* `COUNT`：设定 worker 数量，预设是1，比如 COUNT=5 。
* `REDIS_BACKEND`： 设定 Redis 的 ip, port。如果没设定，预设是连 localhost:6379 。
* `VERBOSE`：啰嗦模式，设置 1 为启用，会输出基本的调试信息
* `VVERBOSE`：设置“1”启用更啰嗦模式，会输出详细的调试信息
* `VVERBOSE` ：比较详细的 log， VVERBOSE=1 debug 的时候可以开出来看。
* `INTERVAL` ：worker 检查 queue 的间隔，预设是五秒 INTERVAL=5 。
* `PIDFILE` ： 如果你是开单 worker，可以指定 PIDFILE 把 pid 写入，例如 PIDFILE=/var/run/resque.pid 。
* `BACKGROUND` ：可以把 resque 丢到背景执行。或者使用 php resque.php &就可以了。
* `PREFIX`：前缀。在 Redis 数据库中为队列的 KEY 添加前缀，以方便多个 Worker 运行在同一个Redis 数据库中方便区分。默认为空。

`Resque_Job_Status`四种状态：

* `Resque_Job_Status::STATUS_WAITING` = 1; (等待)
* `Resque_Job_Status::STATUS_RUNNING` = 2; (正在执行)
* `Resque_Job_Status::STATUS_FAILED` = 3; (失败)
* `Resque_Job_Status::STATUS_COMPLETE` = 4; (结束)

Job类：

```
class Job
{

    // 导出文件表
    private $exportTable = null;

    public function setUp()
    {
       // .. Set up environment for this job
    }

    public function perform()
    {
        // .. Run job
     }

    public function tearDown()
    {
        // ... Remove environment for this job
    }
}
```

Resque类：

```
 class Resque
 {
     /**
     * Create a new job and save it to the specified queue.
     *
     * @param string $queue The name of the queue to place the job in.
     * @param string $class The name of the class that contains the code to execute the job.
     * @param array $args Any optional arguments that should be passed when the job is executed.
     * @param boolean $trackStatus Set to true to be able to monitor the status of a job.
     *
     * @return string|boolean Job ID when the job was created, false if creation was cancelled due to beforeEnqueue
     */

    public static function enqueue($queue, $class, $args = null, $trackStatus = false)
        {
            $id = Resque::generateJobId();
            $hookParams = array(
                'class' => $class,
                'args' => $args,
                'queue' => $queue,
                'id' => $id,
            );
            try {
                Resque_Event::trigger('beforeEnqueue', $hookParams);
            }
            catch(Resque_Job_DontCreate $e) {
                return false;
            }
            Resque_Job::create($queue, $class, $args, $trackStatus, $id);
            Resque_Event::trigger('afterEnqueue', $hookParams);
            return $id;
        }
 }
```

加入队列：

```
Resque::setBackend('redis://user:pass@127.0.0.1:6379');
$jobId = Resque::enqueue('default', 'Job', $args, true);
```