---
layout: post
title: ﻿【Java 基础】Java 泛型
date: 2019-07-28 19:19:24.000000000 +09:00
categories:
- 技术
tags:
- Java
toc: true
---

**
摘要：泛型是`JDK5`中引入的特性，它的本质是参数化类型，按照参数类型的使用范围分为泛型类、泛型方法、泛型接口。通过泛型可以定义类型安全的数据结构（类型安全），而无须使用实际的数据类型（可扩展），在程序编译期间即可发现问题并解决。本篇文章介绍了泛型类的格式及使用、泛型方法的格式及使用、泛型接口的格式和使用；最后介绍了三种类型通配符，包含类型通配符、类型通配符上限、类型通配符下限。
**
<!-- more -->
<The rest of contents | 余下全文>
 
 ### 泛型概述
 泛型是`JDK5`中引入的特性，它提供了编译时类型安全检测机制，该机制允许在编译时检测到非法的类型。
 
 它的本质是参数化类型，所操作的数据类型被指定为一个参数。所谓的参数化类型就是将类型由原来具体的类型参数化，在使用或者调用时传入具体的类型。
 
 这种参数类型可以用在类、方法和接口中，分别被称为泛型类、泛型方法、泛型接口。
 
 ### 泛型格式
 `<类型>`：指定一种类型的格式，这里的类型可以看成是形参。
 `<类型1,类型2...>`：指定多种类型的格式，多种类型之间用逗号隔开，这里的类型也看成是形参。
 注：将来具体调用时候给定的类型可以看成是实参，并且实参的类型只能是引用数据类型。
 
 ### 泛型好处
 1、使用泛型可以将运行时期的问题提前到编译期间，避免了强制类型转换。
 2、 通过泛型可以定义类型安全的数据结构（类型安全），而无须使用实际的数据类型（可扩展）。
 
 
 ### 泛型格式和示例
 泛型类格式：`修饰符 class 类名<类型> {}`
 泛型类示例：`public class Generic<T> {}`
 
 泛型方法格式：`修饰符 <类型> 返回值类型 方法名(类型 变量名) {}`
 泛型方法示例： `public <T> void show(T t) {}`
 
 泛型接口格式：`修饰符 interface 接口名<类型> {}`
 泛型接口示例：`public interface Generic<T> {}` `public class GenericImpl<T> implements Generic<T> {}`
 
 ### 泛型类
 泛型类格式：`修饰符 class 类名<类型> {}`
 泛型类示例：`public class Generic<T> {}`
 
 代码示例：
 学生类：
 ```
 public class Student {
     private String name;
 
     public String getName() {
         return name;
     }
 
     public void setName(String name) {
         this.name = name;
     }
 }
 ```
 教师类：
 ```
 public class Teacher {
     private Integer age;
 
     public Integer getAge() {
         return age;
     }
 
     public void setAge(Integer age) {
         this.age = age;
     }
 }
 ```
 泛型类：
 ```
 public class Generic<T> {
     private T t;
 
     public T getT() {
         return t;
     }
 
     public void setT(T t) {
         this.t = t;
     }
 }
 ```
 实现类：
 ```
 public class GenericDemo {
     public static void main(String[] args) {
         Student s = new Student();
         s.setName("wx001");
         // Student对象中只能设置String类型
         // s.setName(001);
         System.out.println(s.getName()); // wx001
 
         Teacher t = new Teacher();
         t.setAge(30);
         // Teacher对象只能设置Integer类型
         // t.setAge("30");
         System.out.println(t.getAge()); // 30
         System.out.println("--------");
 
         Generic<String> g1 = new Generic<String>();
         g1.setT("wx001");
         System.out.println(g1.getT()); // wx001
 
         Generic<Integer> g2 = new Generic<Integer>();
         g2.setT(30);
         System.out.println(g2.getT()); // 30
 
         Generic<Boolean> g3 = new Generic<Boolean>();
         g3.setT(true);
         System.out.println(g3.getT()); // true
     }
 }
 ```
 
 ### 泛型方法
 泛型方法格式：`修饰符 <类型> 返回值类型 方法名(类型 变量名) {}`
 泛型方法示例： `public <T> void show(T t) {}`
 代码示例
 泛型方法：
 ```
 public class Generic {
     public <T> void show(T t) {
         System.out.println(t);
     }
 }
 ```
 实现类：
 ```
 public class GenericDemo {
     public static void main(String[] args) {
         Generic g = new Generic();
         g.show("wx001"); // wx001
         g.show(30); // 30
         g.show(true); // true
         g.show(12.34); // 12.34
     }
 }
 ```
 
 ### 泛型接口
 泛型接口格式：`修饰符 interface 接口名<类型> {}`
 泛型接口示例：`public interface Generic<T> {}` `public class GenericImpl<T> implements Generic<T> {}`
 定义泛型接口：
 ```
 public interface Generic<T> {
     void show(T t);
 }
 ```
 接口的实现：
 ```
 public class GenericImpl<T> implements Generic<T> {
     @Override
     public void show(T t) {
         System.out.println(t);
     }
 }
 ```
 具体实现类：
 ```
 public class GenericDemo {
     public static void main(String[] args) {
         Generic<String> g1 = new GenericImpl<String>();
         g1.show("wx001"); // wx001
 
         Generic<Integer> g2 = new GenericImpl<Integer>();
         g2.show(30); // 30
     }
 }
 ```
 
 ### 类型通配符
 类型通配符格式：`<?>`
 类型通配符示例：`List<?>`：表示元素类型未知的`List`，它的元素可以匹配任何的类型。
     
 类型通配符上限格式：`<? extends 类型>`
 类型通配符上限示例：`List<? extends Number>`：它表示的类型是`Number`或者其子类型。
     
 类型通配符下限格式：`<? super 类型>`
 类型通配符下限示例：`List<? super Number>`：它表示的类型是`Number`或者其父类型。
 
 代码示例：
 ```
 public class GenericDemo {
     public static void main(String[] args) {
         // 类型通配符：<?>
         List<?> list1 = new ArrayList<Object>();
         List<?> list2 = new ArrayList<Number>();
         List<?> list3 = new ArrayList<Integer>();
 
         // 类型通配符上限：<? extends 类型>
         List<? extends Number> list5 = new ArrayList<Number>();
         List<? extends Number> list6 = new ArrayList<Integer>();
         // 上限只能到Number
         // List<? extends Number> list9 = new ArrayList<Object>();
 
         // 类型通配符下限：<? super 类型>
         List<? super Number> list7 = new ArrayList<Object>();
         List<? super Number> list8 = new ArrayList<Number>();
         // 下限只能到Number
         // List<? super Number> list9 = new ArrayList<Integer>();
     }
 }
 ```
