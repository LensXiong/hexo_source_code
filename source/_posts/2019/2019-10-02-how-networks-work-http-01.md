---
layout: post
title: ﻿【网络连接】浏览器-生成HTTP消息
date: 2019-10-02 23:19:24.000000000 +09:00
categories:
- 技术
tags:
- NetWorks
toc: true
---

**
摘要：所谓温故而知新，为了更加全面和系统的对网络连接有更深刻的认知，在工作多年以后更需要对相关知识进行归纳和总结。本篇文章是对浏览器生成`HTTP`消息的简单归纳，首先例举了常见的`URL`的各种形式，其中包括`HTTP URL` 、`FTP URL` 、`FILES URL`、`MAILTO URL`和`NEWS URL`，紧接着对`HTTP`常见方法进行讲解，重点介绍了`GET`和`POST`的区别，`GET` 用于获取资源，是无副作用的，是幂等的，且可缓存； `POST` 用于处理资源，有副作用，非幂等，不可缓存。最后对`HTTP` 消息的格式进行示例详细说明，重点介绍响应状态码的分类和常用状态码的含义。
**

<!-- more -->
<The rest of contents | 余下全文>

# URL的各种形式

`URL`:`Uniform Resource Locator`，统一资源定位符，是因特网的万维网服务程序上用于指定信息位置的表示方法。除了我们熟悉的`HTTP URL`以外，还包括其他很多形式的`URL`，比如：

* `HTTP URL` 方案是用来标志因特网上使用`HTTP`(`HyperText Transfer Protocol`，超文本传输协议)的可达资源。
* `FTP URL` 方案可以用来指定因特网上使用`FTP`协议（`RFC959`）的可达主机上的文件和目录。
* `FILES URL`方案被用来指定那些特定主机上的可访问的文件。
* `MAILTO URL` 方案是用来指明一个个体或者服务的因特网邮件地址的。它只代表因特网邮件地址，而不表示任何其它的附加信息。
* `NEWS URL`方案被用来查阅新闻组或者`USENET`新闻上的独立文章。

根据访问目标的不同， `URL` 的写法也会不同。例如在访问 `Web` 服务器和 `FTP` 服务器时，`URL` 中会包含服务器的域名和要访问的文件的路径名等，而发邮件的 `URL` 则包含收件人的邮件地址。此外，根据需要，`URL` 中还会包含用户名、密码、 服务器端口号等信息。以下为常见的几种 `URL`形式：

## 使用`HTTP`协议访问`Web`服务器
格式：协议://用户名:密码@Web服务器的域名:端口号/文件的路径名

```
http://user:password@www.wwxiong.com:80/dir/file1.html
协议://用户名:密码@WEB服务器的域名:端口号:文件的路径名
```

`http`代表所使用的协议；`user:password`是用户名:密码，可省略；`www.wwxiong.com`代表web服务器的域名；80代表端口号，也是可以省略的；`/dir/file1.html`代表文件的路径名。

## 使用`FTP`协议上传和下载文件

格式：ftp://用户名:密码@FTP服务器域名:端口号/文件名

```
ftp://user:password@ftp.wwxiong.com:21/dir/file1.htm
```

`ftp`代表所使用的协议；`user:password`是用户名:密码，可省略；`ftp.wwxiong.com`代表`FTP`服务器的域名；21代表端口号，也是可以省略的；`/dir/file1.html`代表文件的路径名。

## 读取客户端本地文件

格式：file://主机名/文件路径名

```
file://localhost/Users/wangxiong/home/file1.zip
```

`file`代表使用文件`URL`；localhost代表主机名；`Users/wangxiong/home/file1.zip`代表文件路径名。有一种特殊情况，就是主机名可以是`localhost`或者空字符串；表示解释这个`URL`的主机。

注：像`file:`这样的 `URL` 在访问时是不使用网络的，因此说 `URL` 的开头部分表示的是协议类型并不完全准确，也许理解为`访问方法`会更好一些。

## 发送电子邮件

格式：mailto:电子邮件地址

```
mailto:lensxiong@gmail.com
```

`mailto`协议代表因特网邮件地址，而不表示任何其它的附加信息，不像其他`URL`，`mailto`协议不代表可直接访问的数据对象。如果没有正确配置电子邮件软件，则即使在地址栏中输入`mailto:`也是无法正常工作的。

## 阅读新闻组文章

格式：news:新闻组名

```
news:comp.protocols.tcp-ip
```

# HTTP的主要方法

浏览器的第一步工作是对 `URL` 进行解析，当解析完成以后接下来就是浏览器需要告诉服务器进行怎样的操作，我们称之为方法。最常用的方法是`GET `和`POST`方法，除此之外还有很多其他的方法，比如`PUT `和 `DELETE` 等方法，我们需要认真思考一下它们的含义，以便理解 `HTTP` 协议具备的所有功能，以下为`HTTP`的主要方法及其含义：

|方法|含义|
|--|--|
|GET|获取 `URI `指定的信息。如果 `URI` 指定的是文件，则返回文件的内容；如果 `URI `指定的是 `CGI `程序，则返回该程序的输出数据。|
|POST|从客户端向服务器发送数据。一般用于发送表单中填写的数据等情况下。|
|HEAD|和 `GET` 基本相同。不过它只返回 `HTTP` 的消息头 (`message` `header`)，而并不返回数据的内容。用于获取文件最后更新时间等属性信息。|
|OPTIONS|用于通知或查询通信选项。|
|PUT|替换 `URI` 指定的服务器上的文件。如果 `URI` 指定的文件不存在，则创建该文件。|
|DELETE|删除 `URI` 指定的服务器上的文件。|
|TRACE|将服务器收到的请求行和头部(`header`)直接返回给客户端。用于在使用代理的环境中检查改写请求 的情况。|
|CONNECT|使用代理传输加密消息时使用的方法。|

>问：GET和POST 的区别？

这个问题看似很简单，其实网上的很多答案都是不正确的。从标准上来看，`GET` 和 `POST `的区别如下：
* `GET` 用于获取资源，是无副作用的，是幂等的，且可缓存。
* `POST` 用于处理资源，有副作用，非幂等，不可缓存。

可以用以下图来表示：

|幂等性|改变服务器上资源的状态|不改变服务器上资源的状态|
|--|--|--|
|幂等|PUT|GET|
|非幂等|POST|无|

首先我们需要知道`RFC7231`里定义了`HTTP`方法的几个性质：

* ① `Safe` - 安全性。这里的「安全」和通常理解的「安全」意义不同，如果一个方法的语义在本质上是「只读」的，那么这个方法就是安全的。客户端向服务端的资源发起的请求如果使用了是安全的方法，就不应该引起服务端任何的状态变化，因此也是无害的。 此`RFC`定义，`GET`、`HEAD`、 `OPTIONS` 和 `TRACE` 这几个方法是安全的。但是这个定义只是规范，并不能保证方法的实现也是安全的，服务端的实现可能会不符合方法语义，比如说可以使用`GET`修改用户信息的情况。引入安全这个概念的目的是为了方便网络爬虫和缓存，以免调用或者缓存某些不安全方法时引起某些意外的后果。`User Agent`（浏览器）应该在执行安全和不安全方法时做出区分对待，并给用户以提示。
* ② `Idempotent`- 幂等性。幂等性的概念是指同一个请求方法执行多次和仅执行一次的效果完全相同。按照`RFC`规范，`PUT`、`DELETE`都是幂等的。同样，这也仅仅是规范，服务端实现是否幂等是无法确保的。引入幂等主要是为了处理同一个请求重复发送的情况，比如在请求响应前失去连接，如果方法是幂等的，就可以放心地重发一次请求。这也是浏览器在后退或者刷新时遇到`POST`会给用户提示的原因：`POST`语义不是幂等的，重复请求可能会带来意想不到的后果。
* `Cacheable` - 可缓存性。顾名思义就是一个方法是否可以被缓存，此`RFC`里`GET`，`HEAD`和某些情况下的`POST`都是可缓存的，但是绝大多数的浏览器的实现里仅仅支持`GET`和`HEAD`。

关于`GET`和`POST` 这两种方法的语义，`RFC7231`里原文定义如下：

>The GET method requests transfer of a current selected representation for the target resource. GET is the primary mechanism of information retrieval and the focus of almost all performance optimizations. Hence, when people speak of retrieving some identifiable information via HTTP, they are generally referring to making a GET request.A payload within a GET request message has no defined semantics; sending a payload body on a GET request might cause some existing implementations to reject the request.

> The POST method requests that the target resource process the representation enclosed in the request according to the resource’s own specific semantics.

以下为错误的认识：

① `GET`方法对数据长度有限制而`POST`方法没有限制？事实上，`HTTP`协议明确地指出了，`HTTP`头和`Body`都没有长度的要求，对 `URL` 限制的大多是浏览器和服务器的原因。服务器是因为处理长 `URL` 要消耗比较多的资源，为了性能和安全（防止恶意构造长 `URL` 来攻击）考虑，会给 `URL` 长度加限制，如果有人恶意地构造几个几M大小的`URL`，并不停地访问你的服务器，服务器性能就会下降。

② `POST` 方法比` GET `方法安全？有人说`POST` 比`GET`安全，因为数据在地址栏上不可见。然而，从传输的角度来说，他们都是不安全的，因为 `HTTP` 在网络上是明文传输的，只要在网络节点上抓包，就能完整地获取数据报文，要想安全传输，就只有加密，也就是 `HTTPS`进行传输。

# HTTP 消息的格式
`HTTP`（超文本传输协议）是一个基于请求与响应模式的、无状态的、应用层的协议，常基于`TCP`的连接方式，`HTTP1.1`版本中给出一种持续连接的机制，绝大多数的`Web`开发，都是构建在`HTTP`协议之上的`Web`应用。 `HTTP` 消息分为请求消息和响应消息，请求消息由三部分组成：请求行、消息头和消息体，响应消息也是由三部分组成：状态行、消息头和消息体。

请求消息包含三部分：
a、请求行：包含请求方法、URI、HTTP版本信息
b、请求消息字段
c、请求消息实体

请求消息示例：

```
// 请求头
GET /index.php HTTP/1.1
// 请求消息头
Accept:text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cache-Control: no-cache
Connection: keep-alive
Cookie: ...
Host: www.wwxiong.com
Pragma: no-cache
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.132 Safari/537.36
// 请求消息体
```

响应消息包含三部分：
a、状态行：包含HTTP版本、状态码、状态码的原因短语
b、响应消息字段
c、响应消息实体
响应消息示例：

```
// 状态行
HTTP/1.1 200 OK
// 响应消息头
Date: Wed, 02 Oct 2019 14:33:47 GMT
Server: cloudflare
Last-Modified: Thu, 05 Sep 2019 05:00:42 GMT
Accept-Ranges: bytes
Cache-control: max-age=600
Cf-ray: 51f764b428b7977e-FRA
Content-encoding: br
Content-type: text/html; charset=utf-8
Expires: Wed, 02 Oct 2019 14:43:47 GMT
...
// 响应消息体
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"> <title></title>
</head>
<body>
...
</body>
</html>
```

# HTTP状态码

`HTTP` 响应消息中状态码的第一位数字表示状态类型，第二、三位数字表示具体的情况。下表列举了第一位数字的含义，`HTTP`状态码分类:

|分类|分类描述|
|--|--|
|1xx|告知请求的处理进度和情况，服务器收到请求，需要请求者继续执行操作|
|2xx|成功，操作被成功接收并处理|
|3xx|表示需要进一步操作|
|4xx|客户端错误，请求包含语法错误或无法完成请求|
|5xx|服务器错误，服务器在处理请求的过程中发生了错误|

常用的`HTTP`状态码：

|状态码|英文|说明|
|---|---|---|
|100|Continue|继续。客户端应继续其请求|
|200|OK|请求成功|
|300|Multiple Choices|请求的资源可包括多个位置|
|301|Moved Permanently|资源（网页等）被永久转移到其它URL|
|302|Temporarily Moved|临时重定向，暂时性转移|
|304|Not Modified|客户端已经执行了GET，但文件未变化。|
|400 |Bad Request |客户端请求的语法错误，服务器无法理解|
|403|Forbidden|服务器已经理解请求，但是拒绝执行它|
|404|Not Found|请求的资源（网页等）不存在|
|500|Internal Server Error|内部服务器错误|
|502|Bad Gateway|服务器接收到上游服务器的无效响应|
|504|Gateway Time-out|网关超时，上游服务器超时|
