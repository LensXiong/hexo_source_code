---
layout: post
title: ﻿【Java 基础】Java 基础术语
date: 2019-07-15 23:19:24.000000000 +09:00
categories:
- 技术
tags:
- Java
toc: true
---

**
摘要：因为工作的原因需要用`JAVA`对项目进行开发，一边维护和开发`PHP`项目的代码，一边开发`JAVA`项目。其实，用什么语言并不重要，语言只是一个工具，重要的是能够站在更高和更广的角度去思考问题。刚接触到`JAVA`的时候，经常听见一些名词，如果不静下心来，认真梳理并总结，不仅自己糊里糊涂，当别人问起来的时候自己也是道不清讲不明，那又有何用？本篇文章，主要是对一些基本概念和术语的解释和总结，在后期的文章中也会不断的完善和补充。其中，本篇文章解释了`JPA`、`ORM`、`Hibernate`、`Spring Data JPA` 及 `JPA` 和 `Hibernate`之间的关系等。
**
<!-- more -->
<The rest of contents | 余下全文>
﻿
### 什么是JPA?

> Java` 持久化`API，（Java Persistence API，简称JPA）是`SUN`公司推出的一套基于 ORM 的规范，内部是由一系列的接口和抽象类构成。 JPA 通过 JDK 5.0 注解描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。

`JPA`是`ORM`框架标准，主流的`ORM`框架都实现了这个标准。`MyBatis`框架没有实现`JPA`,属于半自动`ORM`框架，它通过`XML`来标记及映射，需要手写`SQL`，它的理念是更拥抱`SQL`；而像`Hibernate`框架则完全实现了`JPA`，属于全自动`ORM`框架，大多数情况不需要手写`SQL`，它的理念更拥抱对象。

### 什么是ORM?

> 对象关系映射，（Object Relational Mapping ）主要实现程序对象到关系数据库数据的映射。通过建立实体类和数据库表之间的关系，从而达到操作实体类就相当于操作数据库表的目的。

在面向对象的软件开发中，通过 `ORM`，就可以把对象映射到关系型数据库中。只要有一套程序能够做到建立对象与数据库的关联，操作对象就可以直接操作数据库数据，就可以说这套程序实现了 `ORM` 对象关系映射。

### 为什么要使用ORM?

当实现一个应用程序时（不使用 `ORM`），我们可能会写特别多数据访问层的代码，从数据库保存数据、修改数据、删除数据，而这些代码都是重复的。而使用 `ORM` 则会大大减少重复性代码。

常见的 `ORM` 框架：`Mybatis`（`ibatis`）、`Hibernate`、`Jpa`

### 什么是Hibernate?

`Hibernate`是一个开放源代码的对象关系映射(`ORM`)框架，它对`JDBC`进行了非常轻量级的对象封装，并将`POJO`与数据库表建立映射关系，是一个全自动的`ORM`框架，`Hibernate`可以自动生成`SQL`语句，自动执行，通过使用对象编程思维来操纵数据库。

### 什么是JDBC?

> Java数据库连接，（Java Database Connectivity，简称JDBC）是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口，提供了诸如查询和更新数据库中数据的方法。

```java
JDBC stands for Java Database Connectivity. JDBC is a Java API to connect and execute the query with the database. It is a part of JavaSE (Java Standard Edition). JDBC API uses JDBC drivers to connect with the database.
```

### 什么是POJO？

> 普通 Java 对象，（Plain Ordinary Java Object / Pure Old Java Object，简称POJO）是指那些没有遵从特定模型，约定或框架的 Java 对象。比如在 WEB 应用程序中建立一个数据库的映射对象时，我们用 POJO 这个名字来强调它是一个普通 Java 对象，而不是一个特殊的对象。

```java
POJO stands for Plain Old Java Object. It is an ordinary Java object, not bound by any special restriction other than those forced by the Java Language Specification and not requiring any class path. POJOs are used for increasing the readability and re-usability of a program
```

### JPA与Hibernate的关系

`JPA`和`Hibernate`的关系就像`JDBC`和`JDBC`驱动的关系，`JPA`是规范，`Hibernate`除了作为`ORM`框架之外，它也是一种`JPA`规范的实现。就像`JDBC`规范不能驱动底层数据库一样，`JPA`不能取代`Hibernate`，如果使用`JPA`规范进行数据库操作，底层需要`Hibernate`作为其实现类完成数据持久化工作。

> `JPA` 实质上是一种 `ORM` 规范，注意不是 `ORM` 框架——因为 `JPA` 并未提供 `ORM` 实现，它只是制订了一些规范，提供了一些编程的 `API` 接口，但具体实现则由服务厂商来提供实现。比如像`Hibernate` ,`Toplink`等`ORM`框架的实现厂商。

什么是JTA?

> `Java` 事务 `API`（`Java Transaction API`，简称`JTA` ） 是一个`Java`企业版的应用程序接口，在`Java`环境中，允许完成跨越多个`XA`资源的分布式事务。简单来讲，它是一个分布式事务管理的解决方案。

注：`XA`是一个分布式事务协议，大致分为两部分：事务管理器和本地资源管理器。其中本地资源管理器往往由数据库实现，比如`Oracle`、`DB2`这些商业数据库都实现了`XA`接口，而事务管理器作为全局的调度者，负责各个本地资源的提交和回滚。

`Java`事务的类型有三种：`JDBC`事务、`JTA`(`Java Transaction API`)事务`、`容器事务，其中`JDBC`的事务操作用法比较简单，一个 `JDBC` 事务不能跨越多个数据库，适合于处理同一个数据源的操作。`JTA`事务相对复杂，可以用于处理跨多个数据库的事务，是分布式事务的一种解决方案。

#### 什么是Spring Data JPA?

```java
Spring Data JPA, part of the larger Spring Data family, makes it easy to easily implement JPA based repositories. This module deals with enhanced support for JPA based data access layers. It makes it easier to build Spring-powered applications that use data access technologies.
```

> Spring Data JPA 是 Spring Data 全家桶中的一部分，它使JPA基础库的实现变得更简单。该模块增强了对JPA基础访问层的支持，使得我们在构建应用时对数据层的访问变得更容易。Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据库的访问和操作。

`Spring Data JPA` 让我们解脱了`DAO`层的操作，基本上所有`CRUD`都可以依赖于它来实现，在实际的工作工程中，使用`Spring Data JPA` +` ORM`（如：`Hibernate`）完成操作，这样在切换不同的ORM框架时提供了极大的方便，同时也使数据库层操作更加简单，方便解耦。

### Spring Data JPA 与 JPA 和 Hibernate之间的关系

`JPA`是一套规范，内部是有接口和抽象类组成的。

`Hibernate`是一套成熟的`ORM`框架，而且`Hibernate`实现了`JPA`规范，所以也可以称`Hibernate`为`JPA`的一种实现方式。

`Spring Data JPA`是`Spring`提供的一套对`JPA`操作更加高级的封装，是在`JPA`规范下的专门用来进行数据持久化的解决方案。我们使用`JPA`的`API`编程，意味着站在更高的角度上看待问题（面向接口编程） 。

`Spring Data JPA`包含了`JPA`规范和`Hibernate`的实现，而`Hibernate`又封装了`JDBC`的规范和操作，最终实现数据库的操作和交互。

