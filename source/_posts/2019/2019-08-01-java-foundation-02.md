---
layout: post
title: ﻿【Java基础】List集合
date: 2019-08-01 18:19:24.000000000 +09:00
categories:
- 技术
tags:
- Java
toc: true
---

**
摘要：本篇文章主要介绍了` List`集合的相关信息。其中主要包括`List`集合概述和特点、`List`集合的特有方法、`List`集合存储学生对象三种方式遍历、迭代器遍历（集合特有的遍历方式）；普通`for`，带有索引的遍历方式；增强`for`，最方便的遍历方式。最后介绍了`List`集合子类`ArrayList`集合的基本方法和`LinkedList`集合的基本方法。`ArrayList`集合，底层是数组结构实现，查询快、增删慢。 `LinkedList`集合，底层是链表结构实现，查询慢、增删快。
**

<!-- more -->
<The rest of contents | 余下全文>

### List集合概述和特点
`List`集合概述：`List`集合为有序集合(也称为序列)，可以精确控制列表中每个元素的插入位置，用户可以通过整数索引访问元素，并搜索列表中的元素，与`Set`集合不同，列表通常允许重复的元素。

`List`集合特点（存取有序和可重复）：
1、有索引
2、可以存储重复元素
3、元素存取有序

```
public class ListDemo01 {
    public static void main(String[] args) {
        // 创建ArrayList集合对象
        List<String> list = new ArrayList<String>();

        // 添加元素
        list.add("hello");
        list.add("world");
        list.add("java");
        list.add("world");

        // 输出集合对象
        System.out.println(list); // [hello, world, java, world]

        // 迭代器的方式遍历
        Iterator<String> it = list.iterator();
        while (it.hasNext()) {
            String s = it.next();
            System.out.println(s); // hello world java world
        }
    }
}
```
### List集合的特有方法

|方法名|描述|
|---|---|
|void add(int index,E element)|在此集合中的指定位置插入指定的元素|
|E remove(int index)|删除指定索引处的元素，返回被删除的元素|
|E set(int index,E element)|修改指定索引处的元素，返回被修改的元素|
|E get(int index)|返回指定索引处的元素|

代码实现：
```
public class Student {
    private String name;
    private int age;

    public Student() {
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
```
public class ListDemo02 {
    public static void main(String[] args) {
        // 创建集合对象
        List<String> list = new ArrayList<String>();

        // 添加元素
        list.add("hello");
        list.add("world");
        list.add("java");

        // void add(int index,E element)：在此集合中的指定位置插入指定的元素
        list.add(1, "javaee");
        System.out.println(list); // [hello，javaee，world，java]
        // IndexOutOfBoundsException
        // list.add(11,"javaee");

        // E remove(int index)：删除指定索引处的元素，返回被删除的元素
        System.out.println(list.remove(1)); // javaee
        System.out.println(list); // [hello，world，java]
        // IndexOutOfBoundsException
        // System.out.println(list.remove(11));

        // E set(int index,E element)：修改指定索引处的元素，返回被修改的元素
        System.out.println(list.set(1, "javaee")); // world
        System.out.println(list); // [hello，javaee，java]
        // IndexOutOfBoundsException
        // System.out.println(list.set(11,"javaee"));

        // E get(int index)：返回指定索引处的元素
        System.out.println(list.get(1)); // javaee
        // IndexOutOfBoundsException
        // System.out.println(list.get(11));

        // 输出集合对象
        System.out.println(list); // [hello，javaee，java]
        // 获取集合指定索引数据
        System.out.println(list.get(2)); // java

        // 普通for
        for (int i = 0; i < list.size(); i++) {
            String s = list.get(i);
            System.out.println(s); // hello javaee java
        }
        // 增强for
        for (String s : list) {
            System.out.println(s); // hello javaee java
        }
    }
}
```

### List集合存储学生对象三种方式遍历
案例需求：创建一个存储学生对象的集合，存储3个学生对象，使用程序实现在控制台遍历该集合。
遍历方式：迭代器，集合特有的遍历方式； 普通`for`，带有索引的遍历方式；增强`for`，最方便的遍历方式。
具体代码：
```
public class ListDemo {
    public static void main(String[] args) {
        // 创建List集合对象
        List<Student> list = new ArrayList<Student>();

        // 创建学生对象
        Student s1 = new Student("wx001", 27);
        Student s2 = new Student("wx002", 28);
        Student s3 = new Student("wx003", 26);

        // 把学生添加到集合
        list.add(s1);
        list.add(s2);
        list.add(s3);
        // [com.wangxiong.Student@e580929, com.wangxiong.Student@1cd072a9, com.wangxiong.Student@7c75222b]
        System.out.println(list);

        // 迭代器：集合特有的遍历方式
        Iterator<Student> it = list.iterator();
        while (it.hasNext()) {
            Student s = it.next();
            System.out.println(s.getName() + "," + s.getAge());
        }
        System.out.println("--------");

        //普通for：带有索引的遍历方式
        for (int i = 0; i < list.size(); i++) {
            Student s = list.get(i);
            System.out.println(s.getName() + "," + s.getAge());
        }
        System.out.println("--------");

        //增强for：最方便的遍历方式
        for (Student s : list) {
            // wx001,27 wx002,28 wx003,26
            System.out.println(s.getName() + "," + s.getAge());
        }
    }
}
```
### List集合的实现类
`List`集合子类的特点：
`ArrayList`集合，底层是数组结构实现，查询快、增删慢。
`LinkedList`集合，底层是链表结构实现，查询慢、增删快。

#### ArrayList集合
##### ArrayList类常用方法

|方法名|说明|
|----|----|
|public ArrayList()|创建一个空的集合对象|
|public boolean remove(Object o)|删除指定的元素，返回删除是否成功|
|public E remove(int index)|删除指定索引处的元素，返回被删除的元素|
|public E set(int index,E element)|修改指定索引处的元素，返回被修改的元素|
|public E get(int index)|返回指定索引处的元素|
|public int size()|返回集合中的元素的个数|
|public boolean add(E e)|将指定的元素追加到此集合的末尾|
|public void add(int index,E element)|在此集合中的指定位置插入指定的元素|

```
public class ArrayListDemo02 {
    public static void main(String[] args) {
        // 创建集合
        ArrayList<String> array = new ArrayList<String>();

        // 添加元素
        array.add("hello");
        array.add("world");
        array.add("wx001");
        array.add("wx002");

        // public boolean remove(Object o)：删除指定的元素，返回删除是否成功
        System.out.println(array.remove("world")); // true
        System.out.println(array.remove("wx01")); // false

        // public E remove(int index)：删除指定索引处的元素，返回被删除的元素
        System.out.println(array.remove(1)); // wx001
        System.out.println(array); // [hello, wx002]

        // IndexOutOfBoundsException: Index 3 out of bounds for length 2
        // System.out.println(array.remove(3));

        // public E set(int index,E element)：修改指定索引处的元素，返回被修改的元素
        System.out.println(array.set(1, "world")); // wx002
        System.out.println(array); // [hello, world]

        // IndexOutOfBoundsException 索引越界
        // System.out.println(array.set(3,"javaee"));

        // public E get(int index)：返回指定索引处的元素
        System.out.println(array.get(0)); // hello
        System.out.println(array.get(1)); // world
        // IndexOutOfBoundsException: Index 2 out of bounds for length 2
        // System.out.println(array.get(2));

        // public int size()：返回集合中的元素的个数
        System.out.println(array.size()); // 2

        //输出集合
        System.out.println(array); // [hello, world]
    }
}
```
##### ArrayList存储字符串并遍历
案例需求：创建一个存储字符串的集合，存储3个字符串元素，使用程序实现在控制台遍历该集合。
代码实现：
```
public class ArrayListTest01 {
    public static void main(String[] args) {
        // 创建集合对象
        ArrayList<String> array = new ArrayList<String>();

        // 往集合中添加字符串对象
        array.add("wx001");
        array.add("wx002");
        array.add("wx003");

        System.out.println(array); // [wx001, wx002, wx003]
        // 遍历集合，通过get(int index)方法实现获取集合中的每一个元素
        System.out.println(array.get(0)); // wx001
        System.out.println(array.get(1)); // wx002
        System.out.println(array.get(2)); // wx003

        // 遍历集合，其次要能够获取到集合的长度，这个通过size()方法实现
        System.out.println(array.size()); // 3

        // 普通 for 遍历集合
        for (int i = 0; i < array.size(); i++) {
            String s = array.get(i);
            System.out.println(s); // wx001 wx002 wx003
        }
        // 增强 for 遍历集合
        for (String s : array) {
            System.out.println(s); // wx001 wx002 wx003
        }
    }
}
```
##### ArrayList存储学生对象并遍历
案例需求：创建一个存储学生对象的集合，存储3个学生对象，使用程序实现在控制台遍历该集合，学生的姓名和年龄来自于键盘录入。
代码实现：
学生类
```
public class Student {
    private String name;
    private String age;

    public Student() {
    }

    public Student(String name, String age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }
}
```
实现类
```
public class ArrayListTest {
    public static void main(String[] args) {
        // 创建集合对象
        ArrayList<Student> array = new ArrayList<Student>();

        // 调用添加学生
        addStudent(array);
        addStudent(array);
        addStudent(array);
        // [com.wangxiong.Student@7f560810, com.wangxiong.Student@69d9c55, com.wangxiong.Student@13a57a3b]
        System.out.println(array);

        // 普通for遍历集合
        for (int i = 0; i < array.size(); i++) {
            Student s = array.get(i);
            System.out.println(s.getName() + "," + s.getAge()); // wx001,26 wx002,27 ...
        }

        // 增强for遍历集合
        for (Student s : array) {
            System.out.println(s.getName() + "," + s.getAge()); // wx001,26 wx002,27 ...
        }
    }

    public static void addStudent(ArrayList<Student> array) {
        // 键盘录入学生对象所需要的数据
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入学生姓名:");
        String name = sc.nextLine();
        System.out.println("请输入学生年龄:");
        String age = sc.nextLine();

        // 创建学生对象，把键盘录入的数据赋值给学生对象的成员变量
        Student s = new Student();
        s.setName(name);
        s.setAge(age);

        // 往集合中添加学生对象
        array.add(s);
    }
}
```
#### LinkedList集合

|方法名|说明|
|----|----|
|public void addFirst(E e)|在该列表开头插入指定的元素|
|public void addLast(E e)|将指定的元素追加到此列表的末尾|
|public E getFirst()|返回此列表中的第一个元素|
|public E getLast()|返回此列表中的最后一个元素|
|public E removeFirst()|从此列表中删除并返回第一个元素|
|public E removeLast()|从此列表中删除并返回最后一个元素|

代码实现：
```
public class LinkedListDemo {
    public static void main(String[] args) {
        // 创建集合对象
        LinkedList<String> linkedList = new LinkedList<String>();

        linkedList.add("hello");
        linkedList.add("world");
        linkedList.add("java");

        // public void addFirst(E e)：在该列表开头插入指定的元素
        // public void addLast(E e)：将指定的元素追加到此列表的末尾
        linkedList.addFirst("javase");
        linkedList.addLast("javaee");
        System.out.println(linkedList); // [javase, hello, world, java, javaee]

        // public E getFirst()：返回此列表中的第一个元素
        // public E getLast()：返回此列表中的最后一个元素
        System.out.println(linkedList.getFirst()); // javase
        System.out.println(linkedList.getLast()); // javase

        // public E removeFirst()：从此列表中删除并返回第一个元素
        // public E removeLast()：从此列表中删除并返回最后一个元素
        System.out.println(linkedList.removeFirst()); // javase
        System.out.println(linkedList.removeLast()); // javaee
        System.out.println(linkedList); // [hello, world, java]
    }
}
```