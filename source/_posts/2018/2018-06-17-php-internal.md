---
layout: post
title: 【PHP 核心】PHP底层运行机制和原理
date: 2018-06-17 18:02:24.000000000 +09:00
categories:
- 技术
tags:
- PHP
toc: true
---

** 
摘要：完成各种复杂的业务，写过无数代码的你，是否认真思考过PHP动态语言执行过程是如何进行的？运行某一段代码，要经过扫描(`Scanning`) 、解析(`Parsing`) 、编译(`Complication`)、执行(`Execution`)、输出(`Output Buffer`)五个主要步骤，才能完成整个执行过程。本篇文章，主要介绍整个代码执行的流程以及执行过程中每个阶段主要的任务，以便为以后剖析PHP内核先打下一个基础。
![](/hexo_blog/img/article/php-internal/01.png)
**
<!-- more -->
<The rest of contents | 余下全文>


# 执行流程

拿到一段代码后，经过词法解析、语法解析等阶段后，源程序会被翻译成一个个指令（`opcodes`）,然后`Zend`虚拟机顺次执行这些指令完成操作。

PHP本身是用C实现的，因此最终调用的也是C的函数，实际上，我们可以把PHP看做一个C开发的软件。

PHP动态语言执行过程如下所示：

* 扫描(`Scanning`) : 先进行语法分析和词法分析，将`PHP`代码转换为语言片段(`Tokens`)。
* 解析(`Parsing`) : 词法分析后, 将`Tokens`转换成简单而有意义的表达式。
* 编译(`Complication`) : 将表达式编译成中间码(`Opcodes`)。
* 执行(`Execution`) : 顺次执行`Opcodes`，每次一条，从而实现`PHP`脚本的功能。
* 输出(`Output Buffer`): 将要输出的内容输出到缓冲区。

> `Parsing`首先会丢弃`Tokens Array`中的多余的空格，然后将剩余的`Tokens`转换成一个一个的简单的表达式。

# 详细示例

## 扫描阶段

扫描(`scanning`) : 先进行语法分析和词法分析，将PHP代码转换为语言片段(Tokens)。

具体源代码内容如下：

```
$source = <<<'code'
<?php
echo 'hello wangxiong';
code;
// 对 source 源码字符进行解析，变成一个语言片段（token）
$tokens = token_get_all($source, TOKEN_PARSE);
print_r($tokens);
foreach ($tokens as $token) 
{
    if (is_array($token)) 
    {
        // 使用 Zend 引擎的语法分析器获取源码中的 PHP 语言的解析器代号
        echo token_name($token[0]) , PHP_EOL;
    }
}
```

使用[`token_get_all()`](http://php.net/manual/en/function.token-get-all.php)和[`token_name()`](http://www.php.net/manual/en/function.token-name.php)函数获取结果显示：

```
token_get_all — Split given source into PHP tokens
token_name — Get the symbolic name of a given PHP token
```

> array token_get_all （string \$source [，int \$flags= 0 ]）
> string token_name ( int \$token )


```
[root@wx www]# php test.php
Array
(
    [0] => Array
        (
            [0] => 379
            [1] => <?php
            [2] => 1
        )
    [1] => Array
        (
            [0] => 328
            [1] => echo
            [2] => 2
        )
    [2] => Array
        (
            [0] => 382
            [1] =>
            [2] => 2
        )
    [3] => Array
        (
            [0] => 323
            [1] => 'hello wangxiong'
            [2] => 2
        )
    [4] => ;
)

T_OPEN_TAG
T_ECHO
T_WHITESPACE
T_CONSTANT_ENCAPSED_STRING
```

分析这个返回结果我们可以发现，源码中的字符串，字符，空格，都会原样返回。每个源代码中的字符，都会出现在相应的顺序处。并且都是以三个部分显示：

① `Token ID` (在`Zend`内部该`Token`的对应码)，也就是"词"`ID`。
② "词"具体的内容。
③ 所在的位置,行数。


具体所代表的意思如下：

```
(

            // Token ID (在Zend内部该Token的对应码)，也就是"词"ID
            [0] => 323

            // "词"具体的内容
            [1] => 'hello wangxiong'

             // 所在的位置,行数
            [2] => 2
)

// <?php, <? or <%
T_OPEN_TAG

// echo  
T_ECHO

// \t \r\n
T_WHITESPACE

// "foo" or 'bar' string syntax
T_CONSTANT_ENCAPSED_STRING
```

## 解析阶段

解析(`Parsing`) : 词法分析后, 将`Tokens`转换成简单而有意义的表达式。

经过上面的第一阶段-扫描阶段，将`PHP`代码经过语法分析和词法分析并转换成一个一个的`Tokens`片段。接下来，就要将`Tokens`转换成简单而有意义的表达式。

解析(`Parsing`)首先会丢弃`Tokens Array`中的多于的空格，然后将剩余的`Tokens`转换成一个一个的简单的表达式:

```
1.echo a constant string
2.add two numbers together
3.store the result of the prior expression to a variable
4.echo a variable
```



## 编译阶段

编译(`Complication`) : 将表达式编译成中间码(`Opcode`)。

`Compilation`阶段，它会把`Tokens`编译成一个个`op_array`, 每个`op_arrayd`包含如下5个部分：

```
1.Opcode数字的标识，指明了每个op_array的操作类型，比如add , echo
2.结果 存放Opcode结果
3.操作数1 给Opcode的操作数
4.操作数2
5.扩展值 1个整形用来区别被重载的操作符
```

## 执行阶段

执行(`Execution`) : 顺次执行`Opcodes`，每次一条，从而实现`PHP`脚本的功能。

## 输出阶段

输出(`Output Buffer`): 将要输出的内容输出到缓冲区。

# 参考文章

[深入理解PHP原理之Opcodes](http://www.laruence.com/2008/06/18/221.html)

[PHP7内核剖析-代码的编译](https://www.kancloud.cn/nickbai/php7/363273)
