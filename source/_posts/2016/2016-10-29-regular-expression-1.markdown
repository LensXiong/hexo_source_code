---
layout: post
title: 正则表达式基础
date: 2016-10-29 08:32:24.000000000 +09:00
categories:
- 技术
tags:
- JavaScript
toc: true
---
**
摘要：正则表达式
**
<!-- more -->
<The rest of contents | 余下全文>
---
## 1、[Regexpal调试工具](http://cs.smu.ca/~porter/csc/355/regexpal/)

## 2、PHP中正则表达式函数
定义：
\$pattern = 正则表达式
\$subject = 匹配的目标数据

```
① preg_match ($pattern , $subject, [array &$matches]) // 进行正则表达式匹配，成功返回 1 ，否则返回 0
② int preg_match_all ($pattern, $subject, array &$matches) // 进行全局正则表达式匹配
③ preg_replace($pattern,$replacement,$subject) // 匹配并替换返回的所有结果
④ preg_filter($pattern,$replacement,$subject) // 匹配并替换返回的被替换后的结果,未被替换的结果被过滤掉
⑤ preg_grep($pattern,array($input)) // 只匹配不替换，返回被匹配后的结果(与preg_filter类似)
⑥ preg_split($pattern,$subject) // 返回与模式匹配的数组单元(与explode类似)
⑦ preg_quote($str) // 正则运算符转义
```
## 3、正则表达式语法规则
3.1 界定符
除字母、数字和反斜线“\”以外的任何字符
```
/[0-9]/
#[0-9]#
{[0-9]}
|[0-9]|
![0-9]!
```
3.2 原子
① 可见原子
中文最好转换成unicode
```
普通字符作为原子：a~z、A-Z、0~9
特殊字符和元字符:\",\',\*,\+,\.,\<br\/>
```

② 不可见原子

```
空格、\t(制表符)、\r(回车符)、\n(换行符)、\f(换页符)
```

3.3 元字符


3.4 量词

3.5 边界控制

3.6 模式单元

## 4、修正模式

4.1 贪婪匹配(u)
匹配果存在结歧义时取长(默认模式)

4.2 懒惰匹配(U)
匹配果存在结歧义时取短

4.3 忽略大小写(i)

4.4 忽略空白符(x)
包括空格和Tab制表符

4.5 原点字符.匹配所有的字符(s)
"."包含所有的字符，包括换行符。

4.6 对逆向引用做正常的替换(e)

## 5、常见正则表达式书写总结
5.1 邮箱正则
① 标准邮箱正则

```
/^[A-Za-z0-9]+([._\\-]*[A-Za-z0-9]+)*@([A-Za-z0-9][-A-Za-z0-9]+\.){1,63}[A-Za-z]{2,14}$/
```

② 普通邮箱正则

```
/^\w+([._\\-]*\w+)*@\w+(\.\w+)+$/
```

5.2 URL地址正则

```
/^((http|https|ftp|rtsp|mms)?:\/\/)?(\w+\.)+[A-Za-z]+$/
```

5.3 手机号码正则

```
/^1(3|4|5|7|8)\d{9}/
```

5.4 用户名

```
.+ // 非空的匹配
\d+\.\d{2}$ // 浮点数的匹配
1(3|4|5|7|8)\d{9}  // 手机号码的匹配
^(https?://)?(\w+\.)+[a-zA-Z]+$  //URL地址

```
