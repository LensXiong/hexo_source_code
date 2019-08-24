---
layout: post
title: ﻿【Java基础】Map集合
date: 2019-08-24 13:19:24.000000000 +09:00
categories:
- 技术
tags:
- Java
toc: true
---

**
摘要：本篇文章主要介绍了`Map`集合相关的内容，其中包括`Map`集合的概述和特点、`Map`集合的七种基本功能和四种获取方式、`Map`集合遍历的两种方式（键找值、键值对对象找键和值）、最后通过`Map`集合的五个应用案例，详细说明`Map`集合的实际使用场景。案例一创建了一个键是学号(`String`)，值是学生对象(`Student`)的`HashMap`集合。案例二创建了一个键是学生对象(`Student`)，值是居住地 (`String`)的`HashMap`集合。案例三创建了一个元素为`HashMap`，每一个`HashMap`的键和值都是`String`的`ArrayList`集合。案例四创建了一个键值对元素的键是`String`，值是`ArrayList`，每一个`ArrayList`的元素是`String`的`HashMap`集合。案例五通过键盘录入一个字符串，统计字符串中每个字符串出现的次数。
**

<!-- more -->
<The rest of contents | 余下全文>
### Map集合的概述和特点
应用场景：如果程序中存储了几百万个学生，而且经常需要使用学号来搜索某个学生，那么这个需求有效的数据结构就是`Map`。
`Map`集合格式：`interface Map<K,V>` K：键的类型；V：值的类型。
`Map`集合特点：
* 键值对映射关系，一个键对应一个值
* 键不能重复，值可以重复
* 元素存取无序

`Map`集合的基本使用：
```
public class MapDemo01 {
 public static void main(String[] args) {
     // 创建集合对象，具体的实现类HashMap
     Map<String,String> map = new HashMap<String,String>();

     // V put(K key, V value) 将指定的值与该映射中的指定键相关联
     map.put("001","wx1");
     map.put("002","wx2");
     map.put("003","wx3");
     map.put("004","wx4");

     // 输出集合对象
     System.out.println(map); // {001=wx1, 002=wx2, 003=wx3, 004=wx4}
 }
```

### Map集合的基本功能
基本方法：

|方法名| 说明|
|-----|----|
|V put(K key,V value) |添加元素|
|V remove(Object key) |根据键删除键值对元素|
|boolean containsKey(Object key) |判断集合是否包含指定的键|
|boolean containsValue(Object value) |判断集合是否包含指定的值|
|boolean isEmpty() |判断集合是否为空|
|int size() |集合的长度，集合中键值对的个数|
|void clear() |移除所有的键值对元素|

基本使用：
```
public class MapDemo02 {
 public static void main(String[] args) {
     // 创建集合对象，具体的实现类HashMap
     Map<String,String> map = new HashMap<String,String>();

     // V put(K key,V value)：添加元素
     map.put("001","wx1");
     map.put("002","wx2");
     map.put("003","wx3");
     map.put("004","wx4");

     // V remove(Object key)：根据键删除键值对元素
     System.out.println(map.remove(004)); // null
     System.out.println(map.remove("004")); // wx4
     System.out.println(map); // {001=wx1, 002=wx2, 003=wx3}

     // boolean containsKey(Object key)：判断集合是否包含指定的键
     System.out.println(map.containsKey("003")); // true
     System.out.println(map.containsKey("005")); // false

     // boolean isEmpty()：判断集合是否为空
     System.out.println(map.isEmpty()); // false

     // int size()：集合的长度，集合中键值对的个数
     System.out.println(map.size()); // 3

     //输出集合对象
     System.out.println(map); // {001=wx1, 002=wx2, 003=wx3}

     // void clear()：移除所有的键值对元素
     map.clear();
 }
}
```
### Map集合的获取功能
基本方法：

|方法名|说明|
|-----|----|
|V get(Object key)|根据键获取值|
|Set keySet()|获取所有键的集合|
|Collection values()|获取所有值的集合|
|Set<Map.Entry<K,V>> entrySet()|获取所有键值对对象的集合|

基本使用：
```
public class MapDemo03 {
 public static void main(String[] args) {
     // 创建集合对象，具体的实现类HashMap
     Map<String, String> map = new HashMap<String, String>();

     //添加元素
     map.put("001","wx1");
     map.put("002","wx2");
     map.put("003","wx3");
     map.put("004","wx4");

     // V get(Object key):根据键获取值
     System.out.println(map.get(001)); // null
     System.out.println(map.get("001")); // wx1

     // Set<K> keySet():获取所有键的集合
     Set<String> keySet = map.keySet();
     System.out.println(keySet); // [001, 002, 003, 004]
     for(String key : keySet) {
         System.out.println(key); // 001 002 003 004
     }

     // Collection<V> values():获取所有值的集合
     Collection<String> values = map.values();
     System.out.println(values); // [wx1, wx2, wx3, wx4]
     for(String value : values) {
         System.out.println(value); // wx1 wx2 wx3 wx4
     }

     // Set<Map.Entry> entrySet():获取所有键值对对象的集合
     Set<Map.Entry<String, String>> entrySet = map.entrySet();
     System.out.println(entrySet); // [001=wx1, 002=wx2, 003=wx3, 004=wx4]
     for(Map.Entry<String, String> es:entrySet) {
         System.out.println(es); // 001=wx1 002=wx2 003=wx3 004=wx4
     }
 }
}
```

### Map集合的两种遍历方式
方式一（键找值）：
① 获取所有键的集合，用`keySet()`方法实现。
② 遍历键的集合，获取到每一个键，用增强`for`实现。
③ 根据键去找值，用`get(Object key)`方法实现。
示例：
```
public class MapDemo01 {
 public static void main(String[] args) {
     // 创建集合对象
     Map<String, String> map = new HashMap<String, String>();

     // 添加元素
     map.put("001","wx1");
     map.put("002","wx2");
     map.put("003","wx3");

     // 获取所有键的集合。用keySet()方法实现
     Set<String> keySet = map.keySet();
     System.out.println(keySet); // [001, 002, 003]
     // 遍历键的集合，获取到每一个键。用增强for实现
     for (String key : keySet) {
         // 根据键去找值。用get(Object key)方法实现
         String value = map.get(key);
         System.out.println(key + "," + value); // 001,wx1 002,wx2 003,wx3
     }
 }
}
```

方式二（键值对对象找键和值）：
① 获取所有键值对对象的集合，用`Set<Map.Entry<K,V>> entrySet()`方法实现。
② 遍历键值对对象的集合，得到每一个键值对对象，用增强`for`实现，得到每一个`Map.Entry`。
③ 根据键值对对象获取键和值，用`getKey()`得到键，用`getValue()`得到值。
示例：
```
public class MapDemo02 {
 public static void main(String[] args) {
     // 创建集合对象
     Map<String, String> map = new HashMap<String, String>();

     // 添加元素
     map.put("001","wx1");
     map.put("002","wx2");
     map.put("003","wx3");
     System.out.println(map); // {001=wx1, 002=wx2, 003=wx3}

     // 获取所有键值对对象的集合
     Set<Map.Entry<String, String>> entrySet = map.entrySet();
     System.out.println(entrySet); // [001=wx1, 002=wx2, 003=wx3]
     // 遍历键值对对象的集合，得到每一个键值对对象
     for (Map.Entry<String, String> es : entrySet) {
         // 根据键值对对象获取键和值
         String key = es.getKey();
         String value = es.getValue();
         System.out.println(key + "," + value); // 001,wx1 002,wx2 003,wx3
     }
 }
}
```
### Map集合的应用案例

#### 键是学号(String)，值是学生对象(Student)

案例需求：创建一个`HashMap`集合，键是学号(`String`)，值是学生对象(`Student`)，存储三个键值对元素，并遍历。
```
{001=com.wangxiong.Student@e580929, 002=com.wangxiong.Student@1cd072a9, 003=com.wangxiong.Student@7c75222b}
```
实现思路：
1、 定义学生类。
2、创建HashMap集合对象。
3、 创建学生对象。
4、把学生添加到集合。
5、 两种方式遍历集合（键找值、键值对对象找键和值）。
具体代码：
* 学生类
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
*  实现类
```
public class HashMapDemo {
 public static void main(String[] args) {
     // 创建HashMap集合对象
     HashMap<String, Student> hm = new HashMap<String, Student>();

     // 创建学生对象
     Student s1 = new Student("wx001", 20);
     Student s2 = new Student("wx002", 25);
     Student s3 = new Student("wx003", 23);

     // {001=com.wangxiong.Student@e580929}
     System.out.println(s1);

     // 把学生添加到集合
     hm.put("001", s1);
     hm.put("002", s2);
     hm.put("003", s3);
     // {001=com.wangxiong.Student@e580929, 002=com.wangxiong.Student@1cd072a9, 003=com.wangxiong.Student@7c75222b}
     System.out.println(hm);

     // 方式1：键找值
     Set<String> keySet = hm.keySet();
     for (String key : keySet) {
         Student value = hm.get(key);
         // com.wangxiong.Student@e580929 ...
         System.out.println(value);
         // 001,wx001,20 002,wx002,25 003,wx003,23
         System.out.println(key + "," + value.getName() + "," + value.getAge());
     }
     System.out.println("--------");

     // 方式2：键值对对象找键和值
     Set<Map.Entry<String, Student>> entrySet = hm.entrySet();
     // [001=com.wangxiong.Student@e580929, 002=com.wangxiong.Student@1cd072a9, 003=com.wangxiong.Student@7c75222b]
     System.out.println(entrySet);
     for (Map.Entry<String, Student> me : entrySet) {
         String key = me.getKey();
         Student value = me.getValue();
         // 001,wx001,20 002,wx002,25 003,wx003,23
         System.out.println(key + "," + value.getName() + "," + value.getAge());
     }
 }
}
```

#### 键是学生对象(Student)，值是居住地 (String)

案例需求：创建一个`HashMap`集合，键是学生对象(`Student`)，值是居住地 (`String`)，存储多个键值对元素，并遍历。
```
{com.wangxiong.Student@d1c24264=西安, com.wangxiong.Student@d1c24288=云南, com.wangxiong.Student@d1c242a9=北京}
```
要求保证键的唯一性，如果学生对象的成员变量值相同，我们就认为是同一个对象。
实现思路：
1、定义学生类，在学生类中重写两个方法`hashCode()`和`equals()`。
2、创建`HashMap`集合对象。
3、创建学生对象。
4、把学生添加到集合。
5、遍历集合，通过键找值的方式。
具体代码：
* 学生类
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
* 实现类
```
public class HashMapDemo {
 public static void main(String[] args) {
     // 创建HashMap集合对象
     HashMap<Student, String> hm = new HashMap<Student, String>();
     // {}
     System.out.println(hm);

     // 创建学生对象
     Student s1 = new Student("wx001", 20);
     Student s2 = new Student("wx002", 25);
     Student s3 = new Student("wx003", 27);
     // com.wangxiong.Student@e580929
     System.out.println(s1);

     // 把学生添加到集合
     hm.put(s1, "西安");
     hm.put(s2, "云南");
     hm.put(s3, "北京");
     // {com.wangxiong.Student@d1c24264=西安, com.wangxiong.Student@d1c24288=云南, com.wangxiong.Student@d1c242a9=北京}
     System.out.println(hm);

     // 获取集合的所有键
     Set<Student> keySet = hm.keySet();
     // [com.wangxiong.Student@d1c24264, com.wangxiong.Student@d1c24288, com.wangxiong.Student@d1c242a9]
     System.out.println(keySet);

     // 遍历集合的键
     for (Student key : keySet) {
         // 通过键获取值
         String value = hm.get(key);
         // wx001,20,西安
         // wx002,25,云南
         // wx003,27,北京
         System.out.println(key.getName() + "," + key.getAge() + "," + value);
     }
 }
}
```

#### 集合嵌套之ArrayList嵌套HashMap

需求案例：创建一个`ArrayList`集合，存储三个元素，每一个元素都是`HashMap`，每一个`HashMap`的键和值都是`String`，并遍历。
```
[{001=wx001, 002=wx002}, {003=wx003, 004=wx004}, {005=wx005, 006=wx006}]
```
代码实现：
```
public class ArrayListIncludeHashMapDemo {
 public static void main(String[] args) {
     // 创建ArrayList集合
     ArrayList<HashMap<String, String>> array = new ArrayList<HashMap<String, String>>();
     // []
     System.out.println(array);

     // 创建HashMap集合，并添加键值对元素
     HashMap<String, String> hm1 = new HashMap<String, String>();
     // {}
     System.out.println(hm1);

     hm1.put("001", "wx001");
     hm1.put("002", "wx002");
     // 把HashMap作为元素添加到ArrayList集合
     array.add(hm1);
     // {001=wx001, 002=wx002}
     System.out.println(hm1);

     HashMap<String, String> hm2 = new HashMap<String, String>();
     hm2.put("003", "wx003");
     hm2.put("004", "wx004");
     // 把HashMap作为元素添加到ArrayList集合
     array.add(hm2);

     HashMap<String, String> hm3 = new HashMap<String, String>();
     hm3.put("005", "wx005");
     hm3.put("006", "wx006");
     // 把HashMap作为元素添加到ArrayList集合
     array.add(hm3);

     // [{001=wx001, 002=wx002}, {003=wx003, 004=wx004}, {005=wx005, 006=wx006}]
     System.out.println(array);

     // 遍历ArrayList集合
     for (HashMap<String, String> hm : array) {
         // {001=wx001, 002=wx002} {003=wx003, 004=wx004} {005=wx005, 006=wx006}
         System.out.println(hm);
         Set<String> keySet = hm.keySet();
         for (String key : keySet) {
             String value = hm.get(key);
             // 001,wx001 002,wx002 ...
             System.out.println(key + "," + value);
         }
     }
 }
}
```

#### 集合嵌套之HashMap嵌套ArrayList

案例需求：创建一个`HashMap`集合，存储三个键值对元素，每一个键值对元素的键是`String`，值是`ArrayList`，每一个ArrayList的元素是`String`，并遍历。
```
{水浒传=[武松, 鲁智深], 三国演义=[诸葛亮, 赵云], 西游记=[唐僧, 孙悟空]}
```
代码实现：
```
public class HashMapIncludeArrayListDemo {
 public static void main(String[] args) {
     // 创建HashMap集合
     HashMap<String, ArrayList<String>> hm = new HashMap<String, ArrayList<String>>();
     // {}
     System.out.println(hm);

     // 创建ArrayList集合，并添加元素
     ArrayList<String> sgyy = new ArrayList<String>();
     sgyy.add("诸葛亮");
     sgyy.add("赵云");
     // 把ArrayList作为元素添加到HashMap集合
     hm.put("三国演义",sgyy);
     // [诸葛亮, 赵云]
     System.out.println(sgyy);

     ArrayList<String> xyj = new ArrayList<String>();
     xyj.add("唐僧");
     xyj.add("孙悟空");
     // 把ArrayList作为元素添加到HashMap集合
     hm.put("西游记",xyj);

     ArrayList<String> shz = new ArrayList<String>();
     shz.add("武松");
     shz.add("鲁智深");
     // 把ArrayList作为元素添加到HashMap集合
     hm.put("水浒传",shz);

     // {水浒传=[武松, 鲁智深], 三国演义=[诸葛亮, 赵云], 西游记=[唐僧, 孙悟空]}
     System.out.println(hm);

     // 遍历HashMap集合
     Set<String> keySet = hm.keySet();
     for(String key : keySet) {
         System.out.println(key);
         ArrayList<String> value = hm.get(key);
         for(String s : value) {
             System.out.println("\t" + s);
         }
     }
       /*
        水浒传
             武松
             鲁智深
        三国演义
             诸葛亮
             赵云
         西游记
             唐僧
             孙悟空
      */
 }
}
```

#### 统计字符串中每个字符出现的次数

案例需求：键盘录入一个字符串，要求统计字符串中每个字符串出现的次数。

案例举例：键盘录入`aababcabcdabcde`	在控制台输出：`a(5)b(4)c(3)d(2)e(1)`。

实现思路：

1、键盘录入一个字符串，用`Scanner`对象的`nextLine`方法。
2、创建`HashMap`集合，键是`Character`，值是`Integer`。
3、遍历字符串，得到每一个字符。
4、拿得到的每一个字符作为键到`HashMap`集合中去找对应的值，看其返回值。如果返回值是`null`，说明该字符在`HashMap`集合中不存在，就把该字符作为键，1作为值存储，如果返回值不是`null`，说明该字符在`HashMap`集合中存在，把该值加1，然后重新存储该字符和对应的值。
5、遍历`HashMap`集合，得到键和值，按照要求进行拼接。
6、输出结果。

代码实现：

```

public class HashMapDemo {

 public static void main(String[] args) {
     // 键盘录入一个字符串
     Scanner sc = new Scanner(System.in);
     System.out.println("请输入一个字符串：");
     String line = sc.nextLine();

     // 创建HashMap集合，键是Character，值是Integer
     HashMap<Character, Integer> hm = new HashMap<Character, Integer>();

     // 遍历字符串，得到每一个字符
     for (int i = 0; i < line.length(); i++) {
         char key = line.charAt(i);
         // 拿得到的每一个字符作为键到HashMap集合中去找对应的值，看其返回值
         Integer value = hm.get(key);
         if (value == null) {
             // 如果返回值是null：说明该字符在HashMap集合中不存在，就把该字符作为键，1作为值存储
             hm.put(key,1);

         } else {
             // 如果返回值不是null：说明该字符在HashMap集合中存在，把该值加1，然后重新存储该字符和对应的值
             value++;
             hm.put(key,value);
         }
     }

     // 遍历HashMap集合，得到键和值，按照要求进行拼接
     StringBuilder sb = new StringBuilder();
     Set<Character> keySet = hm.keySet();
     for(Character key : keySet) {
         Integer value = hm.get(key);
         sb.append(key).append("(").append(value).append(")");
     }

     String result = sb.toString();
     //输出结果
     System.out.println(result);
 }

}

```

> 注：`HashMap`输出的顺序是无序的，如果需要输出有序请使用`TreeMap`。