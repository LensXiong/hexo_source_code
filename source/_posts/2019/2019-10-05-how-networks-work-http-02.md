---
layout: post
title: ﻿【网络连接】DNS解析过程
date: 2019-10-05 09:49:24.000000000 +09:00
categories:
- 技术
tags:
- NetWorks
toc: true
---

**
摘要：当我们在浏览器中输入一段`URL`并回车后，浏览器能够解析网址并生成 `HTTP` 消息，但是由于浏览器本身并不具备将消息发送到网络中的功能，浏览器需要委托操作系统向 `DNS` 服务器查询 `Web` 服务器的 `IP` 地址。应用程序（`WEB`浏览器）是如何与 `DNS` 服务器进行交互的呢？要想回答这个问题，我们需要了解一些基础的`IP`知识、`Socket` 库、`DNS` 解析器、`DNS`服务器的工作原理。通过了解这些基础知识，我们明白浏览器通过调用`Socket `库中的一个程序组件解析器，解析器通过协议栈和网卡与`DNS` 服务器进行交互，`DNS` 服务器优先查询本地缓存，然后依次从根域服务器、顶级域 `DNS` 服务器、权威 `DNS` 服务器递归查询找到最终对应域名的`IP`地址返回给浏览器。
**

![](https://raw.githubusercontent.com/LensXiong/hexo_source_code/master/img/technology/2019/networks/networks-02.jpeg)

<!-- more -->
<The rest of contents | 余下全文>


# 前言
浏览器能够解析网址并生成 `HTTP` 消息，但它本身并不具备将消息发送到网络中的功能，这一功能需要委托操作系统来实现。发送消息的功能对于所有的应用程序来说都是通用的，因此让操作系统来实现这一功能，其他应用程序委托操作系统来进行操作，是一个比较合理的做法。在委托操作系统发送消息时，必须要提供的不是通信对象的域名，而是它的 `IP` 地址。因此，本篇文章主要介绍浏览器委托操作系统如何向 `DNS` 服务器查询 `Web` 服务器的 `IP` 地址的。

# 关于DNS
域名系统（英语：`Domain Name System`，缩写：`DNS`）是一个将域名映射成`IP`地址的系统。它作为将域名和`IP`地址相互映射的一个分布式数据库，能够使用户更方便的去访问互联网而不用去记住能够被机器直接读取的`IP`地址，而通过域名，最终得到该域名对应`IP`地址的过程则是域名解析的过程。

为什么要使用`IP`地址解析表示而不直接使用域名？

> 为了提高数据在路由器之间的传输效率，减少路由器的负担，减少传输时间。

在实际的互联网中，存在无数的路由器，它们之间相互配合，根据 `IP` 地址来判断应该把数据传送到什么地方，那么如果我们不用 `IP` 地址而是改用名称会怎么样呢? `IP` 地址的长度 为 32 比特，也就是 4 字节，相对地，域名最短也要几十个字节，最长甚至 可以达到 255 字节。换句话说，使用 `IP` 地址只需要处理 4 字节的数字，而域名则需要处理几十个到 255 个字节的字符，这增加了路由器的负担，传送数据也会花费更长的时间。虽然提高路由器的性能也能达到一定的效果，但通过域名这并不是一个合理和聪明的设计。

# IP地址
在讲关于`DNS`如何解析`IP`地址之前，我们需要了解`IP`地址的基础知识。

互联网协议地址（英语：`Internet Protocol Address`，又译为网际协议地址），缩写为`IP`地址（英语：`IP Address`），是分配给网络上使用网际协议（英语：`Internet Protocol,` `IP`）的设备的数字标签。常见的`IP`地址分为`IPv4`与`IPv6`两大类，但是也有其他不常用的小分类。

当前`IPv4`技术可能使用的`IP`地址最多可有4,294,967,296个（即2^32），随着互联网的快速发展，`IPv4`的42亿个地址的分配最终于2011年2月3日用尽。相应的科研组织已研究出128位的`IPv6`，其`IP`地址数量最高可达3.402823669 × 1038个，届时每个人家居中的每件电器，每件对象，甚至地球上每一粒沙子都可以拥有自己的`IP`地址。

`IPv4`地址由 32 比特（位）二进制数组成，按照 8 比特（位，1 字节)为一组分成 4 组，分别用小于或等于255的十进制表示然后再用圆点隔开。
格式：
```
网络号（前三组）+ 主机号（第四组）/ 子网掩码
```
其中，子网掩码表示网络号与主机号之间的边界，子网掩码的作用是通过逻辑运算，将`IP`地址划分为网络标识(`Net.ID`)和主机标识(`Host.ID`)，只有网络标识相同的两台主机在无路由的情况下才能相互通信。根据`RFC950`定义，子网掩码是一个 32 位的 2 进制数， 其对应网络地址的所有位都置为1，对应于主机地址的所有位都置为0。子网掩码告知路由器，地址的哪一部分是网络地址，哪一部分是主机地址，使路由器正确判断任意`IP`地址是否是本网段的，从而正确地进行路由。网络上，数据从一个地方传到另外一个地方，是依靠`IP`寻址。从逻辑上来讲，是两步的。第一步，从`IP`中找到所属的网络，好比是去找这个人是哪个小区的；第二步，再从`IP`中找到主机在这个网络中的位置，好比是在小区里面找到这个人。

子网掩码工作过程是：将 32 位的子网掩码与 `IP` 地址进行二进制形式的按位逻辑与（`AND`）运算得到的便是网络地址，将子网掩码二进制按位取反，然后`IP`地址进行二进制的逻辑与（`AND`）运算，得到的就是主机地址。

例如：`192.168.10.10` `AND` `255.255.255.0`，结果为`192.168.10.0`，其表达的含义为：该`IP`地址属于`192.168.10.0`这个网络；`192.168.10.10` `AND` `0.0.0.255`，结果为`0.0.0.10`，其表达的含义为：该网络主机地址为10。

```
192.168.10.10/255.255.255.0
192.168.10.10/x
```

`IPv6`地址为 128 位长，但通常写作 8 组，每组四个十六进制数的形式，例如：
```
2001:0db8:85a3:08d3:1319:8a2e:0370:7344
```

下面例举`IPv4`地址的一些表示方法：
* `IPv4`地址主体的表示，`192.168.10.10`。
* `IPv4`地址主体与相同格式的子网掩码表示，`192.168.10.10/255.255.255.0`。
*  `IPv4`地址主体与网络号比特数子网掩码表示，`192.168.10.10/24`。
* 表示子网的地址，`192.168.10.0/24`。 主机号部分的比特全部为0，表示整个子网，而不是单独的一台计算机。
* 表示子网内广播的地址，`192.168.10.255/24`。 主机号部分的比特全部为1，表示对整个子网进行广播。

主机号部分的比特全部为 0 或者全部为 1 时代表两种特殊的含义。主机号部分全部为 0 代表整个子网而不是子网中的某台设备。主机号部分全部为 1 代表向子网上所有设备发送包，即广 播。

# Socket 库和 DNS 解析器

> 浏览器是如何向 DNS 服务器发出查询的呢?

根据域名查询 `IP` 地址时，浏览器会使用 `Socket` 库中的解析器，最终通过解析器向 `DNS` 服务器发出查询。

向 `DNS` 服务器发出查询，也就是向 `DNS` 服务器发送查询消息，并接收服务器返回的响应消息。这一过程是通过计算机中的 `DNS` 客户端来实现的，`DNS` 客户端的部分称为 `DNS`解析器，简称解析器。通过 `DNS` 查询 `IP` 地址的操作称为域名解析，因此负责执行解析(`resolution`)这一操作的就叫解析器(`resolver`)了。解析器实际上是一段程序，它包含在操作系统的 `Socket` 库中。

`Socket `库是用于调用网络功能的程序组件集合，可以让其他的应用程序调用操作系统的网络功能，而解析器就是这个库中的一种程序组件。`Socket`库中包含很多用于发送和接收数据的程序组件，这里我们只探究解析器这一种程序组件。

在任何应用程序（比如浏览器）中，编写一段调用解析器的代码，解析器会向 `DNS` 服务器发送查询消息，然后 `DNS` 服 务器会返回响应消息。响应消息中包含查询到的 `IP` 地址，解析器会取出 `IP` 地址，并将其写入浏览器指定的内存地址中，之后就将它与 `HTTP` 请求消息一起交给操作系统。

# 解析器的内部原理

> 当应用程序（比如浏览器）调用解析器时，解析器内部是怎样工作的？

浏览器调用解析器后，`Socket` 库中的解析器开始运行，解析器会生成要发送给 `DNS` 服务器的查询消息，由于解析器本身不具有使用网络收发数据的功能，因此解析器需要调用协议栈执行发送消息的操作，最终通过网卡将消息发送给 `DNS` 服务器。

>  协议栈:操作系统内部的网络控制软件，也叫协议驱动、`TCP/IP` 驱动等。

以下为调用解析器的工作流程：
1）应用程序（`Web`浏览器）使用代码调用`Socket `库中的一个程序组件-解析器。
2）`Socket `库中的解析器程序组件会生成发送给`DNS`服务器的查询消息。
3）解析器委托操作系统内部的协议栈向`DNS`服务器发送`UDP`消息，并将消息打包后委托网卡向`DNS`服务器查询信息。
4）`DNS`服务器将查询到的`IP`地址等信息依次通过网卡、操作系统内部的协议栈接收到相应的`UDP`消息。
5）操作系统内部的协议栈将`DNS`服务器返回的响应消息发送给`Socket `库中的解析器，解析器将消息中的`IP`地址存放到内存地址后，给应用程序（`Web`浏览器）返回应用程序消息。
6） 应用程序（`Web`浏览器）从内存中取出 `IP` 地址。


# DNS服务器的工作原理
上面介绍了浏览器通过调用`Socket `库中的一个程序组件解析器，解析器通过协议栈和网卡与`DNS` 服务器进行交互，接下来我们需要看看 `DNS` 服务器是怎么工作的？

 `DNS` 服务器的基本工作就是接收来自客户端的查询消息（域名、`Class`、记录类型），然后从保存的记录中查找，最后将域名对应的内容数据返回给客户端。

客户端的查询消息包含三种信息：
a）域名，服务器、邮件服务器(邮件地址中 @ 后面的部分)的名称。
b） `Class`，用来识别网络的信息，实际中除了互联网并没有其他的网络了，因此 `Class` 的值永远是代表互联网的 `IN`。
c）记录类型，表示域名对应何种类型的记录。

##  `DNS` 记录类型

常见 `DNS` 记录的类型：

|类型|目的|
|--|--|
|A|地址记录，用来指定域名的 IPv4 地址，如果需要将域名指向一个 IP 地址，就需要添加 A 记录。|
|AAAA|用来指定主机名(或域名)对应的 IPv6 地址记录。|
|CNAME|如果需要将域名指向另一个域名，再由另一个域名提供 ip 地址，就需要添加 CNAME 记录。|
|MX|如果需要设置邮箱，让邮箱能够收到邮件，需要添加 MX 记录。|
|NS|域名服务器记录，如果需要把子域名交给其他 DNS 服务器解析，就需要添加 NS 记录。|
|SOA|SOA 这种记录是所有区域性文件中的强制性记录。它必须是一个文件中的第一个记录。|
|TXT|可以写任何东西，长度限制为 255。绝大多数的 TXT记录是用来做 SPF 记录(反垃圾邮件)。|

> A 是 `Address` 的缩写，`MX`:`Mail eXchange`，邮件交换。

DNS 服务器的基本工作示例：

|域名|Class|记录类型|响应数据|
|--|--|--|--|
|www.wwxiong.com|IN|A|104.24.118.85|
|wwxiong.com|IN|MX|10 mail.wwxiong.com|
|mail.wwxiong.com|IN|A|104.24.119.85|
|...|...|...|...|

例如，如果要查询 `www.wwxiong.com` 这个域名对应的 `IP` 地址，客 户端会向 `DNS `服务器发送包含以下信息的查询消息：
a）域名 = `www.wwxiong.com`。
b）`Class` = `IN`。
c）记录类型 = `A`。

记录类型为`MX`、`NS`、`SOA`等的工作原理同`A`类型，`DNS` 服务器会从域名与 `IP `地址的对照表中查找相应的记录，并返回 `IP` 地址。

## 域名的层次结构
`DNS` 服务器中的所有信息都是按照域名以分层次的结构来保存的，在域名中，越靠右的位置表示其层级越高，一个域的信息是作为一个整体存放在 `DNS` 服务器中的，不能将一个域拆开来存放在多台 `DNS` 服务器中。

* 根 `DNS` 服务器 ：返回顶级域 `DNS` 服务器的 `IP` 地址。
* 顶级域 `DNS` 服务器：返回权威 `DNS` 服务器的 `IP` 地址。
* 权威 `DNS` 服务器：返回相应主机的 `IP` 地址。

比如 `www.wang.xiong.com.` 这个域名如果按照公司里的组织结构来说，大概就是`com` 事业集团 `xiong` 部 `wang` 科的 `www` 。其中，相当于一个层级的部分称为域。因此，`com` 域的下一层是 `xiong` 域，再下一层是 `wang`域，再下面才是` www` 这个名字。

如何找到 `DNS` 服务器中存放的信息?

通过上级 `DNS` 服务器查询出下级 `DNS `服务器的 `IP` 地址，从而向下级 `DNS` 服务器发送查询请求，以此类推使用递归查询。

上图来源为作者户根勤《网络是怎样连接的》一书。

比如要查询`www.lab.glasscom.com.` 这个域名的`IP`：
1）客户端计算机向最近的`DNS`服务器发起请求。
2）最近的 `DNS` 服务器中保存了根域 `DNS` 服务器的信息，因 此它会将来自客户端的查询消息转发给根域 `DNS `服务器。
3）根域服务器中也没有 `www.lab.glasscom.com` 这个域名，但根据域名结构可以判断这个域名属于 `com` 域，因此根域 `DNS` 服务器会返回它所管理的 `com `域中的 `DNS` 服务器的` IP` 地址。
4）最近的 `DNS` 服务器又会向 `com` 域的 `DNS` 服务器发送查询消息。`com` 域中也没有 `www.lab.glasscom.com `这个域名的信息，和刚才一样，`com` 域服务器会返回它下面的 `glasscom.com` 域的 `DNS` 服务器的 `IP` 地址。
5）以此类推...最终到达`www`目标服务器找到对应的`IP`地址。
6）最终将`IP`地址返回给客户端计算机。

## DNS服务器缓存
实际的互联网中，`DNS`服务器有一个缓存功能，如果一个要查询的域名和相关信息被缓存过，可以直接返回响应，相比每次从根域开始查找，可以减少查询时间。

当查询的域名不存在的时候，不存在这一响应结果也会被缓存。这样，当下次查询这个不存在的域名时，也可以快速响应。

`DNS`服务器保存的信息都会设置一个有效期，当缓存中的信息超过有效期后，数据就会从缓存中删除。而且，在对查询进行响应时，`DNS` 服务器也会告知客户端这一响应的结果是来自缓存中还是来自负责管理该域名的 `DNS` 服务器。

# DNS记录查询相关命令
使用`dig`进行统计相关信息：
```
 ~ dig wwxiong.com

; <<>> DiG 9.10.6 <<>> wwxiong.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18701
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 5

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;wwxiong.com.	IN	A

;; ANSWER SECTION:
wwxiong.com.	30	IN	A	104.24.118.85
wwxiong.com.	30	IN	A	104.24.119.85

;; AUTHORITY SECTION:
wwxiong.com.	116228	IN	NS	reza.ns.cloudflare.com.
wwxiong.com.	116228	IN	NS	thomas.ns.cloudflare.com.

;; ADDITIONAL SECTION:
reza.ns.cloudflare.com.	20418	IN	A	173.245.58.217
reza.ns.cloudflare.com.	88790	IN	AAAA	2400:cb00:2049:1::adf5:3ad9
thomas.ns.cloudflare.com. 1200	IN	A	173.245.59.238
thomas.ns.cloudflare.com. 1200	IN	AAAA	2400:cb00:2049:1::adf5:3bee

;; Query time: 244 msec
;; SERVER: 192.168.31.1#53(192.168.31.1)
;; WHEN: Sat Oct 05 09:17:55 CST 2019
;; MSG SIZE rcvd: 214
```

查找域名对应的`DNS`记录：
```
~ dig wwxiong.com ANY +noall +answer
; <<>> DiG 9.10.6 <<>> wwxiong.com ANY +noall +answer
;; global options: +cmd
wwxiong.com.	3789	IN	HINFO	"RFC8482" ""
wwxiong.com.	92	IN	A	104.24.118.85
wwxiong.com.	92	IN	A	104.24.119.85
wwxiong.com.	171813	IN	NS	reza.ns.cloudflare.com.
wwxiong.com.	171813	IN	NS	thomas.ns.cloudflare.com.
```
通过`dig +trace`我们可以很清晰的看到一个域名解析的过程：
```
dig +trace wwxiong.com

; <<>> DiG 9.10.6 <<>> +trace wwxiong.com
;; global options: +cmd
.	469783	IN	NS	l.root-servers.net.
.	469783	IN	NS	h.root-servers.net.
.	469783	IN	NS	j.root-servers.net.
.	469783	IN	NS	m.root-servers.net.
.	469783	IN	NS	g.root-servers.net.
.	469783	IN	NS	i.root-servers.net.
.	469783	IN	NS	c.root-servers.net.
.	469783	IN	NS	e.root-servers.net.
.	469783	IN	NS	k.root-servers.net.
.	469783	IN	NS	b.root-servers.net.
.	469783	IN	NS	f.root-servers.net.
.	469783	IN	NS	d.root-servers.net.
.	469783	IN	NS	a.root-servers.net.
.	469783	IN	RRSIG	NS 8 0 518400 20191017050000 20191004040000 22545 . VNhsx/yN88mF0iic7y5I3c4n34dTVTgZ/LK...
;; Received 1097 bytes from 192.168.31.1#53(192.168.31.1) in 7 ms

com.	172800	IN	NS	a.gtld-servers.net.
com.	172800	IN	NS	b.gtld-servers.net.
com.	172800	IN	NS	c.gtld-servers.net.
com.	172800	IN	NS	d.gtld-servers.net.
com.	172800	IN	NS	e.gtld-servers.net.
com.	172800	IN	NS	f.gtld-servers.net.
com.	172800	IN	NS	g.gtld-servers.net.
com.	172800	IN	NS	h.gtld-servers.net.
com.	172800	IN	NS	i.gtld-servers.net.
com.	172800	IN	NS	j.gtld-servers.net.
com.	172800	IN	NS	k.gtld-servers.net.
com.	172800	IN	NS	l.gtld-servers.net.
com.	172800	IN	NS	m.gtld-servers.net.
com.	86400	IN	DS	30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
com.	86400	IN	RRSIG	DS 8 1 86400 20191017190000 20191004180000 22545 . O8ojVwBBbOTIOcTV84euNELxMLYfXUEVvO5vnYCO69YzC6e6k4Y11Yuj fD15C2B4oXWHsvsGS1PQYyavDAEdG/NlVtESkq9ZOLTU36oYOo5lQAht jYOz6ShIV6uSVDbSpYoDsH7PkSUfFrILFqzqLOYxbrfx6ggMN98zUJQM j+szzH//MnVMnZETwwbqht136k4vT3pPmI/UzYHp+I17v6Elhl1IiZt1 jCfbi3KBYPahSfz0TO3BrLdUUPXSHyqPOO5kwyflI5Ou2HKIN2crrSwH xLNbJnRgVN8k+oOvasHsceLfLggnl/kE0QWraYgr3FO6OylhryXK7XT3 Txa62w==
;; Received 1171 bytes from 199.7.91.13#53(d.root-servers.net) in 334 ms

;; Received 29 bytes from 192.43.172.30#53(i.gtld-servers.net) in 9 ms
```

# 小结

> 浏览器委托操作系统如何向 `DNS` 服务器查询 `Web` 服务器的 `IP` 地址的？

应用程序（`WEB`浏览器等）中会有一段代码进行调用`Socket` 库中的解析器（一种程序组件），解析器会生成要发送给 `DNS` 服务器的查询消息，由于解析器本身不具有使用网络收发数据的功能，因此解析器需要通过调用协议栈执行发送消息的操作，最终通过网卡将消息发送给 `DNS` 服务器。`DNS` 服务器会优先查询本地缓存信息，如果没有找到对应的域名信息就会从依次从根域服务器、顶级域 `DNS` 服务器、权威 `DNS` 服务器递归查询，最终将查询信息返回给本地`DNS`服务器，之后通过网卡和操作系统内部的协议栈返回给`Socket` 库中的解析器，解析器将`IP`地址放入内存中，并将响应的结果返回给浏览器，最终浏览器从内存中取出域名对应的`IP`地址。

