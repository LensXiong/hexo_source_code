layout: post
title:  【安全】渗透攻防之XSS脚本攻击
date:  2017-06-16 22:11:14.000000000 +09:00
categories: 
- 技术
tags: 
- PHP
toc: true
reward: true
---
** 
摘要：如果说SQL注入是一把利剑直接插入目标的胸膛，犀利的锋芒毕露。那么，XSS跨站脚本攻击则是一个“温柔杀手”，一把隐藏在背后的匕首，而它跨站攻击的目标是客户端，也就是我们网站的访问者。XSS攻击是WEB渗透中常见的一种攻击方式，本文通过对XSS的介绍、危害、攻击方式、分类和应用对XSS攻击进行系统的介绍，并针对此攻击梳理出相应的几种防御方式。
**
<!-- more -->
<The rest of contents | 余下全文>
# 引言
XSS是一门又热门又不太受重视的WEB攻击手法，为什么会这样呢，原因有下：
① 耗时间
② 有一定几率不成功
③ 没有相应的软件来完成自动化攻击
④ 前期需要基本的html、js功底，后期需要扎实的html、js、actionscript2/3.0等语言的功底
⑤ 是一种被动的攻击手法
⑥ 对website有http-only、crossdomian.xml没有用

但是这些并没有影响黑客对此漏洞的偏爱，原因不需要多，只需要一个。

XSS几乎每个网站都存在，google、baidu、360等都存在。

# 什么是XSS攻击
跨站脚本（Cross Site Scripting，通常简称为XSS）是一种网站应用程序的安全漏洞攻击，是代码注入的一种。

百科上的标准解释是这样的：

> XSS是跨站脚本攻击(Cross Site Scripting)，为不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故将跨站脚本攻击缩写为XSS。
> 恶意攻击者往Web页面里插入恶意Script代码，当用户浏览该页之时，嵌入其中Web里面的Script代码会被执行，从而达到恶意攻击用户的目的。

它允许恶意用户将代码注入到网页上，其他用户在观看网页时就会受到影响。

这类攻击通常包含了HTML以及用户端脚本语言。

# XSS带来的危害
归根结底，XSS的攻击方式就是想办法“教唆”用户的浏览器去执行一些这个网页中原本不存在的前端代码。

可问题在于尽管一个信息框突然弹出来并不怎么友好，但也不至于会造成什么真实伤害啊。

的确如此，但要说明的是，这里拿信息框说事仅仅是为了举个栗子，真正的黑客攻击在XSS中除非恶作剧，不然是不会在恶意植入代码中写上:

```
alert("say something")
```

在真正的应用中，XSS攻击可以干的事情还有很多，这里举两个例子。

① 窃取网页浏览中的cookie值。

在网页浏览中我们常常涉及到用户登录，登录完毕之后服务端会返回一个cookie值。

这个cookie值相当于一个令牌，拿着这张令牌就等同于证明了你是某个用户。

如果你的cookie值被窃取，那么攻击者很可能能够直接利用你的这张令牌不用密码就登录你的账户。

如果想要通过script脚本获得当前页面的cookie值，通常会用到cookie。

试想下如果像空间说说中能够写入xss攻击语句，那岂不是看了你说说的人的号你都可以登录。

② 劫持流量实现恶意跳转。

这个很简单，就是在网页中想办法插入一句像这样的语句：
```javascript
<script>window.location.href="http://www.wwxiong.com";</script>
```

那么所访问的网站就会被跳转到我的博客首页。

早在2011年新浪就曾爆出过严重的xss漏洞，导致大量用户自动关注某个微博号并自动转发某条微博。

# XSS攻击的步骤

![](http://wwxiong.oss-cn-beijing.aliyuncs.com/blog-img/technology/safe/xss/1.png)

① 攻击者以某种方式发送xss的http链接给目标用户

② 目标用户登录此网站，在登陆期间打开了攻击者发送的xss链接

③ 网站执行了此xss攻击脚本

④ 目标用户页面跳转到攻击者的网站，攻击者取得了目标用户的信息

⑤ 攻击者使用目标用户的信息登录网站，完成攻击

# XSS攻击的方式

# XSS分类

# XSS应用

# XSS服务端防御

① 配置COOKIE的httponly属性

 XSS注入大部分危害是挟持COOKIE并伪装登录，得到更高的授权从而达到入侵的目地。
 
 针对这块，浏览器提供COOKIE的httponly属性，通过配置httponly从而限制脚本读取COOKIE的权限，从而禁止防止XSS注入的目地。
 
 ② 处理富文本
 
 有些数据因为使用场景问题，并不能直接在服务端进行转义存储。
 
 不过富文本数据语义是完整的HTML代码，在输出时也不会拼凑到某个标签的属性中，所以可以当特殊情况特殊处理。
 
 处理的过程是在服务端配置富文本标签和属性的白名单，不允许出现其他标签或属性（例如script、iframe、form等），即”XSS Filter“。
 
 然后在存储之前进行过滤（过滤原理没有去探明）。

# XSS客户端防御

① 输入检查
  
  输入检查的逻辑，必须放在服务器端代码中实现（因为用JavaScript做输入检查，很容易被攻击者绕过）。
  
  目前Web开发的普遍做法，是同时在客户端JavaScript中和服务器代码中实现相同的输入检查。
  
  客户端JavaScript的输入检查，可以阻挡大部分误操作的正常用户，从而节约服务资源。
  
  ② 输出检查
  一般就是在变量输出到HTML页面时，使用编码或转义的方式来防御XSS攻击。
  
  XSS的本质就是“HTML注入”，用户的数据被当成了HTML代码一部分来执行，从而混淆了原本的语义，产生了新的语义。

# 小结
