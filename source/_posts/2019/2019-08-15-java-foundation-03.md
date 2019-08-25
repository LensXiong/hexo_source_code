---
layout: post
title: ﻿【Java基础】Set集合
date: 2019-08-15 19:59:24.000000000 +09:00
categories:
- 技术
tags:
- Java
toc: true
---

**
摘要：本篇文章主要介绍了`Set`集合相关的基础知识。其中包括`Set`集合的概述和特点（元素存取无序、没有索引、只能通过迭代器或增强`for`循环遍历、不能存储重复元素）；介绍了哈希值的相关知识；`HashSet`集合概述和特点；介绍了哈希表的数据结构；`HashSet`集合存储学生对象并遍历的应用案例分析；`LinkedHashSet`集合概述和特点；`TreeSet`集合概述和特点；通过案例分析了两种排序（自然排序和比较器排序）的应用；用`TreeSet`集合进行成绩排序的案例应用；使用`Set`集合完成不重复的随机数案例。
**

<!-- more -->
<The rest of contents | 余下全文>

### Set集合概述和特点
`Set`集合概述:
`Collection`是单列集合类的根接口，用于存储一系列符合某种规则的元素，它有两个重要的子接口，分别是`List`和`Set`。其中，`List`的特点是元素有序、元素可重复。`Set`的特点是元素无序并且不可重复。`List`接口的主要实现类有`ArrayList`和`LinkedList`，`Set`接口的主要实现类有`HashSet`和`TreeSet`。

`Set`集合特点：
1、元素存取无序 。
2、没有索引，只能通过迭代器或增强`for`循环遍历 。
3、不能存储重复元素。

`Set`集合基本使用：
```
public class SetDemo {
    public static void main(String[] args) {
        // 创建集合对象
        Set<String> set = new HashSet<String>();
        System.out.println(set); // []

        //添加元素
        set.add("hello");
        set.add("world");
        set.add("java");
        //不包含重复元素的集合
        set.add("world");
        System.out.println(set); // [world, java, hello]

        //遍历
        for (String s : set) {
            System.out.println(s); // world java hello
        }

    }
}
```

### 哈希值
哈希值概述：哈希值是`JDK`根据对象的地址或者字符串或者数字算出来的`int`类型的数值。

哈希值获取：通过`Object`类中的`public int hashCode()`，返回对象的哈希码值。

哈希值特点：
1、同一个对象多次调用`hashCode()`方法返回的哈希值是相同的。
2、默认情况下，不同对象的哈希值是不同的，通过重写`hashCode()`方法，可以实现让不同对象的哈希值相同。

具体代码：
学生类
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
实现类：
```
public class HashDemo {
    public static void main(String[] args) {
        // 创建学生对象
        Student s1 = new Student("wx001",30);

        // 同一个对象多次调用hashCode()方法返回的哈希值是相同的
        System.out.println(s1.hashCode()); // 240650537
        System.out.println(s1.hashCode()); // 240650537
        System.out.println("--------");
        Student s2 = new Student("wx002",30);

        // 默认情况下，不同对象的哈希值是不相同的
        // 通过方法重写，可以实现不同对象的哈希值是相同的
        System.out.println(s2.hashCode()); // 483422889
        System.out.println("--------");

        System.out.println("hello".hashCode()); // 99162322
        System.out.println("world".hashCode()); // 113318802
        System.out.println("java".hashCode()); // 3254818

        System.out.println("world".hashCode()); // 113318802
        System.out.println("--------");

        System.out.println("汉字".hashCode()); // 882734
        System.out.println("中国".hashCode()); // 642672
    }
}
```

### HashSet集合
#### HashSet集合概述和特点
`HashSet`集合的特点:
1、底层数据结构是哈希表。
2、对集合的迭代顺序不作任何保证，也就是说不保证存储和取出的元素顺序一致。
3、没有带索引的方法，所以不能使用普通`for`循环遍历。
4、由于是`Set`集合，所以是不包含重复元素的集合。

`HashSet`集合的基本使用：
```
public class HashSetDemo01 {
    public static void main(String[] args) {
        // 创建集合对象
        HashSet<String> hs = new HashSet<String>();

        // 添加元素
        hs.add("hello");
        hs.add("world");
        hs.add("java");

        hs.add("world");
        hs.add("php");
        System.out.println(hs); // [world, java, php, hello]

        //遍历
        for (String s : hs) {
            System.out.println(s); // world java php hello
        }
    }
}
```
#### HashSet集合存储学生对象并遍历
案例需求：创建一个存储学生对象的集合，存储3个学生对象，使用程序实现在控制台遍历该集合。
案例要求：学生对象的成员变量值相同，我们就认为是同一个对象。
学生类：
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

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Student student = (Student) o;

        if (age != student.age) return false;
        return name != null ? name.equals(student.name) : student.name == null;
    }

    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        return result;
    }
}
```
实现类：
```
public class HashSetDemo02 {
    public static void main(String[] args) {
        // 创建HashSet集合对象
        HashSet<Student> hs = new HashSet<Student>();

        // 创建学生对象
        Student s1 = new Student("wx001", 30);
        Student s2 = new Student("wx002", 35);
        Student s3 = new Student("wx003", 33);
        Student s4 = new Student("wx003", 33);

        // 把学生添加到集合
        hs.add(s1);
        hs.add(s2);
        hs.add(s3);
        hs.add(s4);
        // [com.wangxiong.Student@d1c24292, com.wangxiong.Student@d1c2426e, com.wangxiong.Student@d1c242af]
        System.out.println(hs);

        // 遍历集合(增强for)
        for (Student s : hs) {
            // wx002,35 wx001,30 wx003,33
            System.out.println(s.getName() + "," + s.getAge());
        }
    }
}
```
### Set集合排序
#### LinkedHashSet集合概述和特点
`LinkedHashSet`集合特点:
1、哈希表和链表实现的`Set`接口，具有可预测的迭代次序。
2、由链表保证元素有序，也就是说元素的存储和取出顺序是一致的。
3、由哈希表保证元素唯一，也就是说没有重复的元素。

`LinkedHashSet`集合基本使用：
```
public class LinkedHashSetDemo {
    public static void main(String[] args) {
        // 创建集合对象
        LinkedHashSet<String> linkedHashSet = new LinkedHashSet<String>();

        // 添加元素
        linkedHashSet.add("hello");
        linkedHashSet.add("world");
        linkedHashSet.add("java");

        // 重复添加
        linkedHashSet.add("world");
        System.out.println(linkedHashSet); // [hello, world, java]

        // 遍历集合
        for (String s : linkedHashSet) {
            // hello world java
            System.out.println(s);
        }
    }
}
```

#### TreeSet集合概述和特点
`TreeSet`集合概述和特点：
1、元素有序，可以按照一定的规则进行排序，具体排序方式取决于构造方法。

> TreeSet()：根据其元素的自然排序进行排序。
TreeSet(Comparator comparator) ：根据指定的比较器进行排序。

2、没有带索引的方法，所以不能使用普通`for`循环遍。
3、由于是`Set`集合，所以不包含重复元素的集合。

`TreeSet`集合基本使用：
```
public class TreeSetDemo01 {
    public static void main(String[] args) {
        //创建集合对象
        TreeSet<Integer> ts = new TreeSet<Integer>();

        //添加元素
        ts.add(10);
        ts.add(40);
        ts.add(30);

        ts.add(30);
        System.out.println(ts); // [10, 30, 40]

        //遍历集合
        for(Integer i : ts) {
            System.out.println(i); // 10 30 40
        }
    }
}
```

#### 自然排序Comparable的使用
案例需求：存储学生对象并遍历，创建集合使用无参构造方法。
案例要求：按照年龄从小到大排序，年龄相同时，按照姓名的字母顺序排序。

代码实现：
学生类
```
public class Student implements Comparable<Student> {
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

    @Override
    public int compareTo(Student s) {
        // 按照年龄从小到大排序
        int num = this.age - s.age;
        // 年龄相同时，按照姓名的字母顺序排序
        int num2 = num == 0 ? this.name.compareTo(s.name) : num;
        return num2;
    }
}
```
实现类：
```
public class TreeSetDemo02 {
    public static void main(String[] args) {
        // 创建集合对象
        TreeSet<Student> ts = new TreeSet<Student>();

        // 创建学生对象
        Student s1 = new Student("wx001", 29);
        Student s2 = new Student("wx002", 28);
        Student s3 = new Student("wx003", 30);
        Student s4 = new Student("wx004", 33);

        Student s5 = new Student("wx005",33);
        Student s6 = new Student("wx005",33);

        // 把学生添加到集合
        ts.add(s1);
        ts.add(s2);
        ts.add(s3);
        ts.add(s4);
        ts.add(s5);
        ts.add(s6);
        // [com.wangxiong.Student@48140564, ...]
        System.out.println(ts);

        // 遍历集合
        for (Student s : ts) {
            System.out.println(s.getName() + "," + s.getAge());
        }
        /* 输出结果
            wx002,28
            wx001,29
            wx003,30
            wx004,33
            wx005,33
         */
    }
}
```

#### 比较器排序Comparator的使用
案例需求：存储学生对象并遍历，创建TreeSet集合使用带参构造方法。
案例要求：按照年龄从小到大排序，年龄相同时，按照姓名的字母顺序排序。

代码实现：
学生类：
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
实现类：
```
public class TreeSetDemo {
    public static void main(String[] args) {
        // 创建集合对象
        TreeSet<Student> ts = new TreeSet<Student>(new Comparator<Student>() {
            @Override
            public int compare(Student s1, Student s2) {
                int num = s1.getAge() - s2.getAge();
                int num2 = num == 0 ? s1.getName().compareTo(s2.getName()) : num;
                return num2;
            }
        });

        // 创建学生对象
        Student s1 = new Student("wx001", 29);
        Student s2 = new Student("wx002", 28);

        Student s3 = new Student("wx003", 33);
        Student s4 = new Student("wx005", 33);

        // 把学生添加到集合
        ts.add(s1);
        ts.add(s2);
        ts.add(s3);
        ts.add(s4);
        // [com.wangxiong.Student@58ceff1, com.wangxiong.Student@7c30a502 ...]
        System.out.println(ts);

        //遍历集合
        for (Student s : ts) {
            // wx002,28 wx001,29 wx003,33 wx005,33
            System.out.println(s.getName() + "," + s.getAge());
        }
    }
}
```



#### 学生成绩排序案例
案例需求：用`TreeSet`集合存储多个学生信息(姓名，语文成绩，数学成绩)，并遍历该集合。
案例要求：按照总分从高到低出现。
代码实现：
学生类：
```
public class Student {
    private String name;
    private int chinese;
    private int math;

    public Student() {
    }

    public Student(String name, int chinese, int math) {
        this.name = name;
        this.chinese = chinese;
        this.math = math;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getChinese() {
        return chinese;
    }

    public void setChinese(int chinese) {
        this.chinese = chinese;
    }

    public int getMath() {
        return math;
    }

    public void setMath(int math) {
        this.math = math;
    }

    public int getSum() {
        return this.chinese + this.math;
    }
}
```
实现类：
```
public class TreeSetDemo {
    public static void main(String[] args) {
        // 创建TreeSet集合对象，通过比较器排序进行排序
        TreeSet<Student> ts = new TreeSet<Student>(new Comparator<Student>() {
            @Override
            public int compare(Student s1, Student s2) {
                // 主要条件
                int num = s2.getSum() - s1.getSum();
                // 次要条件
                int num2 = num == 0 ? s1.getChinese() - s2.getChinese() : num;
                int num3 = num2 == 0 ? s1.getName().compareTo(s2.getName()) : num2;
                return num3;
            }
        });

        // 创建学生对象
        Student s1 = new Student("wx001", 98, 100);
        Student s2 = new Student("wx002", 95, 95);
        Student s3 = new Student("wx003", 100, 93);
        Student s4 = new Student("wx004", 100, 97);
        Student s5 = new Student("wx005", 98, 98);

        Student s6 = new Student("wx006", 97, 99);
        Student s7 = new Student("wx007", 97, 99);

        // 把学生对象添加到集合
        ts.add(s1);
        ts.add(s2);
        ts.add(s3);
        ts.add(s4);
        ts.add(s5);
        ts.add(s6);
        ts.add(s7);
        // [com.wangxiong.Student@58ceff1, com.wangxiong.Student@7c30a502]
        System.out.println(ts);

        // 遍历集合
        for (Student s : ts) {
            System.out.println(s.getName() + "," + s.getChinese() + "," + s.getMath() + "," + s.getSum());
        }

        /* 输出结果
                wx001,98,100,198
                wx004,100,97,197
                wx006,97,99,196
                wx007,97,99,196
                wx005,98,98,196
                wx003,100,93,193
                wx002,95,95,190
        */
    }
}
```
#### 不重复的随机数案例
案例需求：编写一个程序，获取10个1-20之间的随机数，要求随机数不能重复，并在控制台输出。

代码实现：
```
public class SetDemo {
    public static void main(String[] args) {
        // 创建Set集合对象
        Set<Integer> set = new TreeSet<Integer>();

        // 创建随机数对象
        Random r = new Random();

        // 判断集合的长度是不是小于10
        while (set.size() < 10) {
            // 产生一个随机数，添加到集合
            int number = r.nextInt(21);
            set.add(number);
        }

        // 遍历集合
        for (Integer i : set) {
            // 4 5 6 7 8 12 ....
            System.out.println(i);
        }
    }
}
```
> 注：`public int nextInt(int n)`，生成一个随机的int值，该值介于[0,n)的区间，也就是0到n之间的随机int值，包含0而不包含n。