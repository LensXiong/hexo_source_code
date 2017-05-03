---
layout: post
title: 设计模式-1
date: 2017-01-23 08:32:24.000000000 +09:00
categories:
- 技术
tags:
- JavaScript
toc: true
---
**
摘要：单例模式
**
<!-- more -->
<The rest of contents | 余下全文>
### 1、单例模式

#### ①定义

单例模式，单个实例的设计模式，用于保证类有且只有一个对象的设计模式（方案）

#### ②适用场景

MySQLDB，用于操作数据服务器的工具类，仅仅需要一个对象就可以完成所有的功能。使用一个对象不仅可以减少对象的占用的空间，增加PHP的处理效率。

#### ③实现步骤(三私一公)

1.增加构造方法，并将其私有化。

```
private function __construct() {}
```

2.增加一个公共的静态方法，用于在类外静态调用该方法，进入到类内，去执行实例化调用构造方法。

```
private static $instance;//私有的静态属性保存实例化好的对象
public static function getInstance() {
//先判断对象是否被保存
if(!isset(self::$instance)){
//还没有保存，实例化并返回
self::$instance = new self;
}
return self::$instance;
}
```

3.防止对象被克隆而生成新对象

```
private function __clone() {}
```

优点：从根本上，保证类能且仅能实例化一个对象。
缺点：丧失灵活性。

### 2、架构上的单例设计效果

#### ① 实现方法

增加外部操作，用于得到类的单例对象。(仅仅是单例的设计效果，而非设计模式。)

#### ② 多个类实现单例效果

```
<?php
class MySQLDB {
}
/**
 * 获得单例对象的额外的操作
 * @param $class_name string 需要单例效果的类名
 */
function getInstance($class_name) {
	static $objects = array();//静态局部变量，函数的多次调用，静态局部变量不会被初始化
	if (!isset($objects[$class_name])) {
		$objects[$class_name] = new $class_name;//可变类名
	}
	return $objects[$class_name];
}
//需要单例
$db1 = getInstance();
//不需要单例
$db2 = new MySQLDB;
```

#### ③ 一类对象，采用一个方法类得到单例对象。

```
<?php
class MySQLDB {
}
function getIMG($class_name, $file) {
	static $objects = array();//静态局部变量，函数的多次调用，静态局部变量不会被初始化
	if (!isset($objects[$class_name])) {
		$objects[$class_name] = new $class_name($file);//可变类名
	}
	return $objects[$class_name];
}
class imgJPEG {
	public function __construct($file) {}
}
class imgPNG {
	public function __construct($file) {}
}
class imgGIF {
	public function __construct($file) {}
}
$img = getIMG('imgPNG',$file);

```

### 3、工厂模式

#### ① 定义
如果类的主要功能，是用来生产其他类的对象时，该类的设计，称之为工厂模式。也叫工厂类。
由于该类主要负责生产其他类对象，而类本身是否实例化对象不重要了，经常的将工厂类全由静态方法来实现。也可叫静态工厂。

#### ② 使用场景
1.生产单例对象

```
<?php
class MySQLDB {
}

/**
 * 工厂类F
 */
class F {
	/**
	 * 获得单例对象的额外的操作
	 * @param $class_name string 需要单例效果的类名
	 */
	public static function getInstance($class_name) {
		static $objects = array();//静态局部变量，函数的多次调用，静态局部变量不会被初始化
		if (!isset($objects[$class_name])) {
			$objects[$class_name] = new $class_name;//可变类名
		}
		return $objects[$class_name];
	}
}
$db1 = F::getInstance('MySQLDB');
```

2.依据当前的运行时数据，生成不同对象。

```
<?php
class MySQLDB {
}

/**
 * 工厂类F
 */
class F {
	public static function getIMG($file) {
		switch (strrchr($file, '.')) {
			case '.jpg':
				return new imgJPEG($file);
			case '.png':
				return new imgPNG($file);
			case '.gif':
				return new imgFIG($file);
		}
	}
}
interface I_IMG {
	public function thumb($max_w, $max_h) ;
}
class imgJPEG implements I_IMG {//处理JPEG
	public function __construct($file) {}
	public function thumb($max_w, $max_h){}
}
class imgPNG implements I_IMG  {//处理PNG
	public function __construct($file) {}
	public function thumb($max_w, $max_h){}
}
class imgGIF implements I_IMG  {//处理GIF
	public function __construct($file) {}
	public function thumb($max_w, $max_h){}
}

//得到处理图片的类
$file = 'xxx.jpg';//xxx.png, xxx.gif
$o_img = F::getIMG($file);
$o_img->thumb(100, 100);
```
