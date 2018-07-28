---
layout: post
title: 面向对象基础-1
date: 2016-10-23 08:32:24.000000000 +09:00
categories:
- 技术
tags:
- JavaScript
toc: true
---
**
摘要：面向对象基础
**
<!-- more -->
<The rest of contents | 余下全文>
---
## 1、什么是面向对象？
- [x] 面向对象的核心思想
 以对象为中心的程序设计思维。使用程序设计语言更好的描述现实的业务逻辑，将现实业务逻辑中的实体，映射成计算机编程语言中的对象，对象由属性和方法组成。
- [x] 面向对象的优势
  利于提高程序的重用性(降低冗余度)，使程序结构更加清晰
- [x] 三大特征
  封装，继承，多态
  封装(class)：隐藏内部实现，仅仅开发外部操作接口，将一类对象的特征使用类来描述。
  继承(extends)：对象去使用另外对象的成员
  多态：多种状态，一个方法，具有不同的实现方法
- [x] PHP中的典型语法
  定义类，实例化对象，操作成员(属性和方法)
## 2、接口和抽象类的区别
###2.1 接口技术
作用:用于限制一个对象(类)应该具备哪些公共(方法)操作的一种结构。
① 声明接口(interface)
```
interface I_USB{
    public function connect();
    public function sendData($data);
    public function getData();
}
```
声明的方法：没有方法体部分(抽象方法)
② 使用接口(implements)-类实现接口
```
class Mp3 implements I_USB{}
class Upan implements I_USB{}
```
如果某个类，实现了某个接口，需要将接口中所定义的所有接口方法，全部实现才可以。

###2.2 抽象类
作用:用于限定子类中必须存在，但是实现可以是不同的方法。
① abstract class:该类不能被实例化对象
```
abstract class Goods{}
$obj = new Goods();// cannot instantitate abstract class Goods
```
② abstract function:该方法不能被直接调用(需要子类重写)。该抽象方法仅仅存在方法的声明与参数列表部分，不存在方法体部分。
```
abstract public function getName();
```
注意：
1) 如果一个方法为抽象方法，则必须存在于抽象类中。(只有抽象类中才可以包含抽象方法)
2) 如果继承抽象类的子类，不是抽象类，则必须要将所继承的所有抽象方法全都实现，重写父类的抽象方法。
3) 一个抽象类可以不包含抽象方法。

###2.3 接口与抽象类区别
```
interface one{
 function fun1();
 function fun2();
}
abstract class two implements one{
 abstract function fun1(); //增加了抽象方法(也可以出现非抽象方法)
 abstract function fun2();
}
class four extents two{
 function fun1(){
  echo "fun1";
 }
 function fun2(){
  echo "fun2";
 }
}

```
```
1、抽象类中可以有非抽象的方法而接口中只能够有抽象的方法！
2、一个类可以继承多个接口，而一个类只能继承一个抽象类！
3、抽象方法仅仅一种特殊的方法而已，可以是其他访问级别的抽象方法。
4、而接口，仅仅支持公共方法，需要仅仅限制外部操作而已，与内部实现没有关系。
5、抽象类，有两个功能：a，为子类提供基础操作（普通成员）。B，限制子类的实现过程。
6、而接口，就一个功能：a，限制其实现列的外部操作。
7、接口的使用方式通过implements关键字进行，抽象类则是通过继承extends关键字进行！
```
