---
layout: post
title: 【MySQL高级】存储过程批量插入数据
date: 2019-02-15 08:09:24.000000000 +09:00
categories:
- 技术
tags:
- MySQL
toc: true
---

**
摘要：本篇文章主要介绍了如何使用存储过程批量插入大量数据，其中包括创建函数、创建存储过程、调用存储过程、删除存储过程。关于`MySQL`的存储过程，将在后期的学习中不断归纳和总结，具体细节暂时不做过多说明。这里只需关注是如何利用存储过程批量插入大量的数据即可。
![存储过程插入批量数据-01]()
**
<!-- more -->
<The rest of contents | 余下全文>

# 存储过程和函数

> 什么是存储过程和函数？

过程化`SQL`主要有两种类型，即命名块和匿名块。

所谓匿名块即是我们在程序中所直接写的`SQL`语句， 每次执行都要编译，它不会存储到数据库中，也不能在其他过程化`SQL`块中调用。

存储过程和函数则是命名块，它们被编译后保存在数据库中，称为持久性存储模块。可以被反复调用，并且运行速度较快。

# 建表语句

```
# 新建库
create database bigData;
use bigData;

# 建表dept
CREATE TABLE dept(
id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
dname VARCHAR(20) NOT NULL DEFAULT "",
loc VARCHAR(13) NOT NULL DEFAULT ""
)ENGINE=INNODB DEFAULT CHARSET=GBK;



# 建表emp

CREATE TABLE emp(
id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
empno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '编号',
ename VARCHAR(20) NOT NULL DEFAULT "" COMMENT '名字',
job VARCHAR(9) NOT NULL DEFAULT "" COMMENT '工作',
mgr MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '上级编号',
hiredate DATE NOT NULL COMMENT '入职时间',
sal DECIMAL(7,2) NOT NULL COMMENT '薪水',
comm DECIMAL(7,2) NOT NULL COMMENT '红利',
deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '部门编号'
)ENGINE=INNODB DEFAULT CHARSET=GBK;
```

# 创建函数

如果在创建或者修改函数出现以下错误时，需要将变量`log_bin_trust_function_creators`设置为1：

>  This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)

```
mysql> show variables like 'log_bin_trust_function_creators';
mysql> set global log_bin_trust_function_creators=1;
```

这是因为该参数如果设置为0（默认值），用户不得创建或修改存储函数，除非它们具有除`CREATE ROUTINE`或`ALTER ROUTINE`特权之外的`SUPER`权限。 

如果变量设置为1，`MySQL`不会对创建存储函数实施这些限制。 此变量也适用于触发器的创建。

## 创建随机字符串

```
# 创建随机字符串
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
 DECLARE char_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
 DECLARE return_str VARCHAR(255) DEFAULT '';
 DECLARE i INT DEFAULT 0;
 WHILE i < n DO
 SET return_str=CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
 SET i = i+1;
 END WHILE;
 RETURN return_str;
END $$
```
## 创建随机数

```
# 创建随机数
DELIMITER $$
CREATE FUNCTION rand_num( )
RETURNS INT(5)
BEGIN
 DECLARE i INT DEFAULT 0;
 SET i = FLOOR(100+RAND()*10);
RETURN i;
END $$
```

# 创建存储过程

创建`emp`表中插入数据的存储过程：

```
# 创建emp表中插入数据的存储过程
DELIMITER $$
CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit = 0;
REPEAT
SET i = i+1;
INSERT INTO emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) VALUES((START+i),rand_string(6),'SALESMAN',0001,CURDATE(),2000,400,rand_num());
UNTIL i = max_num
END REPEAT;
COMMIT;
END $$
```

创建`dept`表中插入数据的存储过程：

```
# 创建dept表中插入数据的存储过程
DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit = 0;
REPEAT
SET i = i+1;
INSERT INTO dept(deptno,dname,loc) VALUES ((START+i),rand_string(10),rand_string(8));
UNTIL i = max_num
END REPEAT;
COMMIT;
END $$
```

# 调用存储过程

先插入`ID`从100开始自增的10个部门：

```
DELIMITER;
CALL insert_dept(100,10);
```

再插入`ID`从100001开始的50万条员工数据：

```
DELIMITER;
CALL insert_emp(100001,500000);
```

>  50万条数据创建用时29秒，使用的配置为`MacBook Pro`顶配。

# 删除函数或存储过程

```
DROP FUNCTION rand_num;
DROP FUNCTION rand_string;
DROP PROCEDURE insert_dept;
DROP PROCEDURE insert_emp;
```

# 附录

完整语句如下：

```
# 新建库
create database bigData;
use bigData;
# 建表dept
CREATE TABLE dept(
id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
dname VARCHAR(20) NOT NULL DEFAULT "",
loc VARCHAR(13) NOT NULL DEFAULT ""
)ENGINE=INNODB DEFAULT CHARSET=GBK;

# 建表emp

CREATE TABLE emp(
id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
empno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '编号',
ename VARCHAR(20) NOT NULL DEFAULT "" COMMENT '名字',
job VARCHAR(9) NOT NULL DEFAULT "" COMMENT '工作',
mgr MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '上级编号',
hiredate DATE NOT NULL COMMENT '入职时间',
sal DECIMAL(7,2) NOT NULL COMMENT '薪水',
comm DECIMAL(7,2) NOT NULL COMMENT '红利',
deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '部门编号'
)ENGINE=INNODB DEFAULT CHARSET=GBK;

# 创建随机字符串

DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
 DECLARE char_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
 DECLARE return_str VARCHAR(255) DEFAULT '';
 DECLARE i INT DEFAULT 0;
 WHILE i < n DO
 SET return_str=CONCAT(return_str,SUBSTRING(char_str,FLOOR(1+RAND()*52),1));
 SET i = i+1;
 END WHILE;
 RETURN return_str;
END $$

# 创建随机数

DELIMITER $$
CREATE FUNCTION rand_num( )
RETURNS INT(5)
BEGIN
 DECLARE i INT DEFAULT 0;
 SET i = FLOOR(100+RAND()*10);
RETURN i;
END $$

# 创建emp表中插入数据的存储过程

DELIMITER $$
CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit = 0;
REPEAT
SET i = i+1;
INSERT INTO emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) VALUES((START+i),rand_string(6),'SALESMAN',0001,CURDATE(),2000,400,rand_num());
UNTIL i = max_num
END REPEAT;
COMMIT;
END $$

# 创建dept表中插入数据的存储过程

DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit = 0;
REPEAT
SET i = i+1;
INSERT INTO dept(deptno,dname,loc) VALUES ((START+i),rand_string(10),rand_string(8));
UNTIL i = max_num
END REPEAT;
COMMIT;
END $$

# 调用存储过程

CALL insert_dept(100,10);
-- 50万条数据 macbook pro实际用时29秒
CALL insert_emp(100001,500000);

# 删除函数和存储过程

DROP FUNCTION rand_num;
DROP FUNCTION rand_string;
DROP PROCEDURE insert_dept;
DROP PROCEDURE insert_emp;
```
