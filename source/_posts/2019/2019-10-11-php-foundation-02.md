---
layout: post
title: ﻿【PHP语言】数组函数（一）
date: 2019-10-11 19:49:24.000000000 +09:00
categories:
- 技术
tags:
- PHP
toc: true
---

**
摘要：本系列数组函数文章主要总结了`PHP`数组的用法，通过参考官方手册后做的一个整理，目的在于经常回顾和思考。结合实际的工作，对一些常用的技巧也做了梳理，比如我们可以巧用`array_map()`和`array_reduce()`替代`foreach`循环；利用`array_intersect_key()`和`array_flip()`结合对关联数组进行获取指定键名；利用`array_diff_key()`和`array_flip()`配合使用移除数组中的指定键名；利用`array_flip()`两次反转实现数组的去重；利用`array_slice()`来实现数字索引的重置；使用`array_splice()`来实现数组的新增、移除元素等操作。
**

<!-- more -->
<The rest of contents | 余下全文>


# 概述

* `array_column()` — 返回多维数组中指定的单列数组，可指定数组索引键的列。
* `array_map()`  — 对数组中的每个元素执行回调方法，基于给定的数组传入函数名称或匿名函数来获取一个新数组。
*  `array_reduce()` — 用回调函数迭代地将数组简化为单一的值。
*  `list()` — 把数组中的值赋给一组变量，可以在单次操作内就为一组变量赋值。
*  `array_flip()` — 交换数组中的键和值。
*  `array_intersect_key()` - 使用键名比较计算数组的交集。
*  `array_diff_key()` — 使用键名比较计算数组的差集。
*  `array_unique()` — 移除数组中重复的值,保留第一个遇到的键名。 
*  `array_slice()` — 从数组中取出一段指定数据（可重新排序并重置数组的数字索引）。
*  `array_splice()` — 数组移除后重新拼接（去掉数组中的某一部分并用其它值取代）。


# [array_column](https://www.php.net/manual/zh/function.array-column.php)

`array_column` — 返回多维数组中指定的单列数组，可指定数组索引键的列。
格式：
```
array_column ( array $input , mixed $column_key [, mixed $index_key = null ] ) : array
```
参数：
```
array_column (多维数组，返回值的列，指定数组返回时的索引列)
```
返回：
```
从多维数组中返回单列数组。
```

示例：
```
$records = [
            [
                'id' => 2135,
                'first_name' => 'John',
                'last_name' => 'Doe'
            ],
            [
                'id' => 3245,
                'first_name' => 'Sally',
                'last_name' => 'Smith',
            ],
        ];

        $last_names = array_column($records, 'last_name', 'id');
        print_r($last_names); 
```
上述示例返回的结果：
```
Array
(
    [2135] => Doe
    [3245] => Smith
)
```

# [array_map](https://www.php.net/manual/zh/function.array-map.php)
通过使用 `array_map()`，可以对数组中的每个元素执行回调方法，也可以基于给定的数组传入函数名称或匿名函数来获取一个新数组。
格式：
```
array_map ( callable $callback , array $array1 [, array $... ] ) : array
```
参数：
```
array_map ( 匿名函数或者函数名称，指定的数组1，指定的数组2，...)
```
返回：
```
返回处理后的数组。
```
示例一：空字符截取和去空值处理
```
<?php
// 空字符截取和去空值处理
$arr = ['my', '  name', '', ' is', ' wang  ', '    ',' xiong  '];
$arrAfter = array_filter(array_map('trim', $arr));
var_dump($arrAfter); // ['my', 'name', 'is', 'wang', 'xiong']
```

示例二：转小写、首字母大写、平方、整形转换
```
<?php
// 转小写
$cities = ['Beijing', 'xiAn', 'KunmING', 'Chengdu'];
$aliases = array_map('strtolower', $cities);
print_r($aliases);// ['beijing', 'xian, 'kunming', 'chengdu']

// 首字母大写
$ucfirst = array_map('ucfirst', $aliases);
print_r($ucfirst); // ['Beijing', 'Xian, 'Kunming', 'Chengdu']

// 平方
$numbers = [1, -2, 3, -4, 5];
$squares = array_map(function ($number) {
    return $number ** 2;
}, $numbers);
print_r($squares);// [1, 4, 9, 16, 25]

// 整形转换
$arr = ['2','3','4','5'];
$arr = array_map('intval' , $arr);
var_dump($arr); // [2,3,4,5]
```

示例三：创建自定义格式的数组和多维数组
```
<?php
$a = [1, 2, 3, 4, 5];
$b = ["one", "two", "three", "four", "five"];
$c = ["uno", "dos", "tres", "cuatro", "cinco"];

// 两个数组，一个作为键，一个作为值。
$d = [];
$d = array_map(function($n, $m){
	return $d[$n] = $m;
},$a, $b);
print_r($d); // [one,two,three,four,five]

// 创建多维数组，传入 NULL 作为回调函数的名称，将创建多维数组（一个数组，内部包含数组。）
$e = array_map(null, $a, $b, $c);
print_r($e);
```
上述示例打印结果：
```
Array
(
    [0] => Array
        (
            [0] => 1
            [1] => one
            [2] => uno
        )

    [1] => Array
        (
            [0] => 2
            [1] => two
            [2] => dos
        )

    [2] => Array
        (
            [0] => 3
            [1] => three
            [2] => tres
        )

    [3] => Array
        (
            [0] => 4
            [1] => four
            [2] => cuatro
        )

    [4] => Array
        (
            [0] => 5
            [1] => five
            [2] => cinco
        )

)
```

# [array_reduce](https://www.php.net/manual/zh/function.array-reduce.php)

`array_reduce` — 用回调函数迭代地将数组简化为单一的值。
格式：
```
array_reduce ( array $array , callable $callback [, mixed $initial = NULL ] ) : mixed
```
参数：
```
array_reduce ( 输入的数组 , 回调函数（$carry上次迭代的值，$item本次迭代的值）[处理开始前使用，如果处理结束数组为空时作为最后一个结果 ] ) : mixed
```
返回：
```
返回迭代后的一个结果值。
```
示例：
```
<?php
$a = [1, 2, 3, 4, 5];
$x = [];

// 求和
var_dump(array_reduce($a, function ($carry, $item){
    $carry += $item;
    return $carry;
})); // int(15)

// 如果指定了可选参数 initial，该参数将在处理开始前使用。
var_dump(array_reduce($a, function ($carry, $item) {
    $carry *= $item;
    return $carry;
}, 10)); // int(1200), because: 10*1*2*3*4*5

// 指定了可选参数 initial，处理结束后返回的数组为空时调用该结果。
var_dump(array_reduce($x, function ($carry, $item){
    $carry += $item;
    return $carry;
}, "No data to reduce")); // string(17) "No data to reduce"
```
拼接`ID`值为（1,2,3,4,5）：
```
<?php
// 拼接ID值为（1,2,3,4,5）
$arr = [
      ['id' => 1, name => 'a'],
      ['id' => 2, name => 'b'],
      ['id' => 3, name => 'c'],
      ['id' => 4, name => 'd'],
      ['id' => 5, name => 'e'],
];

var_dump(ltrim(array_reduce($arr, function($carry,$item) {
	return $carry.','.$item['id'];
}),',')); // string(9) "1,2,3,4,5"
```

# [list](https://www.php.net/manual/zh/function.list.php)
`list` — 把数组中的值赋给一组变量。 简洁之处在于可以在单次操作内就为一组变量赋值。
格式：
```
list ( mixed $var1 [, mixed $... ] ) : array
```
参数：
```
list ( 变量1 [, 变量2... ] ) 
```
返回：指定的数组

> 类似`array()`，`list()`也是语言结构，并不是真正的函数。
`PHP 5` 里，`list()` 从最右边的参数开始赋值； `PHP 7` 里，`list()` 从最左边的参数开始赋值。
在 `PHP 7.1.0` 之前的版本，`list()` 仅能用于数字索引的数组，并假定数字索引从 0 开始。

示例一： `list()` 函数的基本使用
```
<?php
$arr = ['a', 'b', 'c'];

// 不使用 list()
$a = $arr[0];
$b = $arr[1];
$c = $arr[2];

// 使用 list() 列出所有的变量
list($a, $b, $c) = $arr;
var_dump($a, $b, $c); // string(1) "a" string(1) "b" string(1) "c"

// 使用 list() 列出其中一个变量
list(,,$d) = $arr;
var_dump($d); // string(1) "c"

// list() 返回指定的数组
$res = list(,,$d) = $arr;
var_dump($res); // ['a', 'b', 'c']

// list() 不能对字符串起作用
list($bar) = "abcde";
var_dump($bar); // NULL
?>
```
示例二：`list()` 用于 `foreach` 遍历
```
$arrays = [[1, 2], [3, 4], [5, 6]];
foreach ($arrays as list($a, $b)) {
    $c = $a + $b;
    echo $c, ', '; // 3, 7, 11, 
}
```

# [array_flip](https://www.php.net/manual/zh/function.array-flip.php)

`array_flip` — 交换数组中的键和值。

格式：
```
array_flip ( array $array ) : array
```
说明：
```
array_flip(要交换键/值对的数组)
```
返回：
```
成功时返回交换后的数组，如果失败返回 NULL
```
示例1：值重复和不合法的键名
```
<?php
$raw = ['id' => 1, 'name' => 'wangxiong', 'gender' => 1];

$flip = array_flip($raw);
// 如果同一个值出现多次，则最后一个键名将作为它的值，其它键会被丢弃。
var_dump($flip); // [1 => 'gender', 'wangxiong' => 'name']

$raw = ['id' => 1, 'name' => ['last' => 'wang']];
$flip = array_flip($raw);
// 注意 array 中的值需要能够作为合法的键名（例如需要是 integer 或者 string）。如果类型不对，将出现一个警告，并且有问题的键／值对将不会出现在结果里
var_dump($flip); // Warning: array_flip(): Can only flip STRING and INTEGER values! [ 1 => 'id']
```
示例2：数组去重，保留最后的键名
```
<?php
// 要想保留最后的键名，可使用array_flip()函数两次颠倒实现。
$arr = ['one' => 666, 'two' => 222, 'three' => 666, 'four' => 222];
$res = array_flip(array_flip($arr));
var_dump($res); // ['three' => 666, 'four' => 222]
?>
```

# [array_intersect_key](https://www.php.net/manual/zh/function.array-intersect-key.php)
`array_intersect_key` — 通过比较键名获取数组的交集。
格式：
```
array_intersect_key ( array $array1 , array $array2 [, array $... ] ) : array
```
参数：
```
array_intersect_key (数组1, 数组2, 数组3)
```
示例1：基本使用
```
<?php
$a = ["a" => "red", "b" => "green", "c" => "blue"];
$b = ["a" => "red", "c" => "blue", "d" => "pink"];
$c = ["a" => "black", "d" => "blue", "e" => "orange"];
$d = ["a " => "green","d" => "blue", "e" => "orange"];
$result=array_intersect_key($a, $b, $c); // ["a" => "red"]
print_r($result);
$result1=array_intersect_key($a, $c, $d); // []
print_r($result1);
?>
```
`array_intersect_key()`返回一个数组，该数组包含了所有出现在 `array1` 中并同时出现在所有其它参数数组中的键名的值。

> 注：在 `key => value` 对中的两个键名仅在 (`string`) `$key1` === (`string`) `$key2` 时被认为相等。换句话说，执行的是严格类型检查，因此字符串的表达必须完全一样。

因此上述示例1中的`$result1`返回空数组。

示例2：数组中取指定键名
```
<?php
// 取指定键名
$arr = ['id' => 1, 'name' => 'wangxiong', 'nick' => 'lensxiong', 'gender' => 'M'];
$allowed = ['id', 'name'];
print_r(array_flip($allowed));
// 根据键名获取数组的交集
$appoint = array_intersect_key($arr, array_flip($allowed));
print_r($appoint); // ['id' => 1, 'name' => 'wangxiong']
?>
```

# [array_diff_key](https://www.php.net/manual/zh/function.array-diff-key.php)
`array_diff_key` — 使用键名比较计算数组的差集。
格式：
```
array_diff_key ( array $array1 , array $array2 [, array $... ] ) : array
```
参数：
```
array_diff_key (数组1, 数组2, 数组3)
```
返回：
```
`array_diff_key()` 返回一个数组，该数组包括了所有出现在 `array1` 中但是未出现在任何其它参数数组中的键名的值。
```
示例1：基本使用
```
<?php
$a = ["a" => "red", "b" => "green", "c" => "blue"];
$b = ["a" => "red", "c" => "blue", "d" => "pink"];
$c = ["a" => "black", "d" => "blue", "e" => "orange"];
$d = ["a " => "green","d" => "blue", "e" => "orange"];
$result = array_diff_key($a, $b, $c); 
print_r($result); // ["b" => "green"]
$result1 = array_diff_key($a, $c, $d); 
print_r($result1); // ["b" => "green", "c" => "blue"]
?>
```

示例2：移除指定键名
```
<?php
// 移除指定键名
$arr = ['id' => 1, 'name' => 'wangxiong', 'nick' => 'lensxiong', 'gender' => 'M'];
$allowed = ['nick', 'gender'];
print_r(array_flip($allowed)); // ['nick' => 0, 'gender' => 1]
// 根据键名获取数组的差集
$appoint = array_diff_key($arr, array_flip($allowed));
print_r($appoint); // ['id' => 1, 'name' => 'wangxiong']
?>
```

# [array_unique](https://www.php.net/manual/zh/function.array-unique.php)
`array_unique` — 移除数组中重复的值,保留第一个遇到的键名。
格式：
```
array_unique ( array $array [, int $sort_flags = SORT_STRING ] ) : array
```
参数：
```
array_unique (输入的数组，[, 排序的行为默认按照字符串形式比较 ]  ) 

排序类型标记：
SORT_REGULAR - 按照通常方法比较（不修改类型）
SORT_NUMERIC - 按照数字形式比较
SORT_STRING - 按照字符串形式比较
SORT_LOCALE_STRING - 根据当前的本地化设置，按照字符串比较。
```
返回：
```
返回过滤后的数组。
```

示例1：数组去重，保留第一次出现的键名
```
<?php
$arr = ['one' => 666, 'two' => 222, 'three' => 666, 'four' => 222];
$res = array_unique($arr);
// array_unique() 先将值作为字符串排序，然后对每个值只保留第一个遇到的键名，接着忽略所有后面的键名。
var_export($res); // ['one' => 666, 'two' => 222]
?>
```
示例2：四种排序类型的基本使用
```
<?php
$input = [4, "4", "3", 4, 3, "3"];

// SORT_REGULAR - 按照通常方法比较（不修改类型）
var_dump(array_unique($input, SORT_REGULAR)); // [0 => 4, 2 => "3"]

// SORT_NUMERIC - 按照数字形式比较
var_dump(array_unique($input, SORT_NUMERIC)); // [0 => 4, 2 => "3"]

// SORT_STRING - 按照字符串形式比较
var_dump(array_unique($input, SORT_STRING)); // [0 => 4, 2 => "3"]

// SORT_LOCALE_STRING - 根据当前的本地化设置，按照字符串比较。
var_dump(array_unique($input, SORT_LOCALE_STRING)); // [0 => 4, 2 => "3"]
?>
```

# [array_slice](https://www.php.net/manual/zh/function.array-slice.php)
`array_slice` — 从数组中取出一段指定数据（可重新排序并重置数组的数字索引）
格式：
```
array_slice ( array $array , int $offset [, int $length = NULL [, bool $preserve_keys = false ]] ) : array
```
参数：
```
array_slice ( 输入的数组，起始位置，数据长度，是否重新排序和重置数组的数字索引)
```
返回：
```
返回其中一段。 如果 `offset` 参数大于 `array` 尺寸，就会返回空的 `array`。
```
示例1：重置数组
```
<?php
$arr = ['one' => 666, '1' => 222, '27' => 666, '99' => 222];
//  从数组的起始位置取出数据，默认会重新排序并重置数组的数字索引
var_dump(array_slice($arr, 0)); // ['one' => 666, 0 => 222, 1 => 666, 2 => 222] 
// 不重置数组的数字索引
var_dump(array_slice($arr, 0, 3, TRUE)); // ['one' => 666, 1 => 222, 27 => 666] 
// 起始位置从末端开始
var_dump(array_slice($arr, -2, 1, TRUE)); // [27 => 666] 
?>
```

示例2：获取指定键名之前的数组。

```
<?php
$data = ['first' => 1, 'second' => 2, 'third' => 3];
function beforeKey($array, $key) {
    $keys = array_keys($array); // [0 => 'first', 1 => 'second', 2 => 'third']
    $len = array_search($key, $keys); // 返回键名
    return array_slice($array, 0, $len); // 返回起始位置和结束位置的一段数组
}
var_dump(beforeKey($data, 'first'));// []
var_dump(beforeKey($data, 'second')); // ['first' => 1]
var_dump(beforeKey($data, 'third')); // ['first' => 1, 'second' => 2]
?>
```
示例3：获取指定键名之后的数组。

```
<?php
$data = ['first' => 1, 'second' => 2, 'third' => 3];
function afterKey($array, $key) {
    $keys = array_keys($array);
    $offset = array_search($key, $keys);
    return array_slice($array, $offset + 1);
}
var_dump(afterKey($data, 'first')); // ['second' => 2, 'third' => 3]
var_dump(afterKey($data, 'second')); // ['third' => 3]
var_dump(afterKey($data, 'third')); // []
?>
```

# [array_splice](https://www.php.net/manual/zh/function.array-splice.php)

`array_splice` — 数组移除后重新拼接（去掉数组中的某一部分并用其它值取代）

格式：
```
array_splice ( array &$input , int $offset [, int $length = count($input) [, mixed $replacement = array() ]] ) : array
```
说明：
```
array_splice ( 输入的数组，起始位置，移除单元数量，移除后的单元替代)
```
返回：

```
返回一个包含有被移除单元的数组。
```

示例1：基本使用


```
<?php
$input = ["a", "b", "c", "d"];

// 如果省略 length，则移除数组中从 offset 到结尾的所有部分，起始值也被移除。
var_dump(array_splice($input, 1)); // ["b", "c", "d"]
var_dump($input); // ["a"]

$input = ["a", "b", "c", "d"];
// 如果指定了 length 并且为正值，则移除包含offset之后的length单元。
var_dump(array_splice($input, 1, 2)); // ["b", "c"]
var_dump($input); // ["a", "d"]

$input = ["a", "b", "c", "d"];
// 如果指定了 length 并且为负值，则移除从 offset 到数组末尾倒数 length 为止中间所有的单元。
var_dump(array_splice($input, 1, -1)); // ["b", "c"]
var_dump($input); // ["a", "d"]

$input = ["a", "b", "c", "d"];
var_dump(array_splice($input, 1, -2)); // ["b"]
var_dump($input); // ["a", "c", "d"]

$input = ["a", "b", "c", "d"];
// 如果设置了 length 为零，不会移除单元。
var_dump(array_splice($input, 2, 0, 'e')); // []
// 如果 offset 和 length 的组合结果是不会移除任何值，则 replacement 数组中的单元将被插入到 offset 指定的位置。 注意替换数组中的键名不保留。
var_dump($input); // ["a", "b", "e", c", "d"]

$input = ["a", "b", "c", "d"];
// 如果给出了 replacement 数组，则被移除的单元被此数组中的单元替代。
var_dump(array_splice($input, -1, 1, ["e","f"])); // ["d"]
var_dump($input); // ["a", "b", "c", "e","f"]

$input = ["a", "b", "c", "d"];
// 当给出了 replacement 时要移除从 offset 到数组末尾所有单元时，用 count($input) 作为 length。
var_dump(array_splice($input, 1, count($input), "e")); // ["b", "c", "d"]
var_dump($input); // ["a","e"]
?>
```

示例2：添加新元素
```
<?php
// 添加两个新元素到 $input  等价于array_push($input, "x", "y");
$input = ["a", "b"];
var_dump(array_push($input, "x", "y")); // 4
var_dump($input); // ["a", "b", "x" , "y"]

$input = ["a", "b"];
var_dump(array_splice($input, count($input), 0, ["x" , "y"])); // []
var_dump($input); // ["a", "b", "x" , "y"]
?>
```
示例3：移除数组最后一个元素和开头元素

```
<?php
// 移除数组最后一个元素
$input = ["a", "b", "c"];
var_dump(array_pop($input)); // "c"
var_dump($input); // ["a", "b"]

$input = ["a", "b", "c"];
var_dump(array_splice($input, -1)); // ["c"]
var_dump($input); // ["a", "b"]

// 移除数组第一个元素
$input = ["a", "b", "c"];
var_dump(array_shift($input)); // "a"
var_dump($input); // ["b", "c"]

$input = ["a", "b", "c"];
var_dump(array_splice($input, 0, 1)); // ["a"]
var_dump($input); // ["b", "c"]
?>
```
示例4：数组开头插入元素
```
<?php
// 数组开头插入元素
$input = ["a", "b", "c"];
var_dump(array_unshift($input, "d", "e")); // 5
var_dump($input); // ["d", "e", "a", "b", "c"]

$input = ["a", "b", "c"];
var_dump(array_splice($input, 0, 0, ["d", "e"])); // []
var_dump($input); // ["d", "e", "a", "b", "c"]
?>
```