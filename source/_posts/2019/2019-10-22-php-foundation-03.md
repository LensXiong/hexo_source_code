---
layout: post
title: ﻿【PHP语言】数组函数（二）
date: 2019-10-22 21:49:24.000000000 +09:00
categories:
- 技术
tags:
- PHP
toc: true
---

**
摘要：本文是对数组函数的一些实际应用的总结。实际中常用的数组`array_values`，返回输入数组中所有的值并给其建立数字索引；`array_filter`函数可以用回调函数过滤数组中的单元，我们可以指定具体的过滤值，也可以进行奇数和偶数过滤；有些情况下我们需要确认数组成员全部为真时，可以使用`array_product`计算数组中所有值的乘积来进行处理； 当我们要统计 数组中重复次数最多的值，可以使用`array_count_values`；使用`array_chunk`将一个数组分割成指定大小的多个数组；`array_diff_key`函数可以让我们使用键名比较计算数组的差集；当我们需要递归地合并一个或多个数组，可以使用`array_merge_recursive`函数来进行处理。
**

<!-- more -->
<The rest of contents | 余下全文>

# 概述

*  `array_values()` — 返回输入数组中所有的值并给其建立数字索引。
*   `array_filter()` — 用回调函数过滤数组中的单元。
*   `array_product()` — 计算数组中所有值的乘积。
*   `array_count_values()` — 统计数组中所有值出现的次数，返回一个数组，键名为值，键值为统计的次数。
*   `array_chunk()` — 将一个数组分割成指定大小的多个数组。
*  `array_combine()` — 创建一个新数组，用一个数组的值作为其键名，另一个数组的值作为其值 。
*   `array_diff()` — 计算数组的差集。
*  `array_diff_assoc()` — 带索引检查计算数组的差集。
*   `array_diff_key()` — 使用键名比较计算数组的差集。
*   `array_merge_recursive()` — 递归地合并一个或多个数组。

# [array_values](https://www.php.net/manual/zh/function.array-values.php)

`array_values` — 返回输入数组中所有的值并给其建立数字索引。

格式：

```
array_values ( array $array ) : array
```

参数：

```
输入的数组
```

返回：

```
返回含所有值的索引数组。
```

示例一：基本使用

```
<?php
$a = [ 3 => 11, 1 => 22, 2 => 33];
$a[0] = 00;
var_dump($a); // [ 3 => 11, 1 => 22, 2 => 33, 0 => 0]
var_dump(array_values($a)); // [ 0 => 11, 1 => 22, 2 => 33, 3 => 0]
```

示例二：重置关联数组索引

```
<?php
$input = ['name' => 'wangxiong', 0 => 888, 99 => 666];
// 数字索引直接重置，字符串键名先删除后重置
var_dump(array_values($input)); // [ 0 => 'wangxiong', 1 => 888, 99 => 666]
```

示例三：从多维数组中获取特定键的所有值

```
<?php
/**
* 从多维数组中获取特定键的所有值
* Get all values from specific key in a multidimensional array
*
* @param $key string
* @param $arr array
* @return null|string|array
*/
function array_value_recursive($key, array $arr){
    $val = [];
    array_walk_recursive($arr, function($v, $k) use($key, &$val){
        if($k == $key) array_push($val, $v);
    });
    return count($val) > 1 ? $val : array_pop($val);
}

$arr = [
    'php' => 'php1',
    'go' => [
        'go1' => 'go1',
        'go2' => 'go2',
        'linux' => [
            'linux1' => 'linux1',
        ]
    ],
    'java' =>'java',
    'linux' => [
            'linux1' => 'linux2',
        ],
];

var_dump(array_value_recursive('linux1', $arr)); // ['linux1' => 'linux1', 'linux1' => 'linux2']
var_dump(array_value_recursive('linux', $arr)); // null 
var_dump(array_value_recursive('go', $arr)); // null 
?>
```

# [array_filter](https://www.php.net/manual/zh/function.array-filter.php)

`array_filter` — 用回调函数过滤数组中的单元。

格式：

```
array_filter ( array $array [, callable $callback [, int $flag = 0 ]] ) : array
```

说明：

```
array_filter ( 输入的数组，回调函数，回调函数接收参数的形式（默认接受值）)

决定callback接收的参数形式:
ARRAY_FILTER_USE_KEY - callback接受键名作为的唯一参数。
ARRAY_FILTER_USE_BOTH - callback同时接受键名和键值。
```

返回：

```
返回过滤后的数组。
```

示例一：基本使用

```
<?php 
$arr = ['wangxiong', false, -1, null, '', ' ', 'null', array() , 'false'];
// 如果没有提供 callback 函数， 将删除 array 中所有等值为 FALSE 的条目。
var_dump(array_filter($arr)); // [0 => 'wangxiong',2 => -1, 5 => ' ', 6 => 'null']
var_dump((bool)' '); // true
var_dump((bool)''); // false
```

> 注意：如果不填写 `callback` 函数，0、0.0、'0'字符串等这些可能有意义的值会被删除。所以如果清除的规则有所不同还需要自行编写 `callback` 函数。

示例二：奇数和偶数过滤

```
<?php
$arr = [ "a" => 1, "b" => 2, "c" => 3, "d" => 12, "e" => 15];
$oddArr = array_filter($arr, function($val) {
    return ($val & 1);
});
var_dump($oddArr); // ["a" => 1, "c" => 3, "e" => 15]
$evenArr = array_filter($arr, function($val) {
    return !($val & 1);
});
var_dump($evenArr); // ["a" => 2, "c" => 12]
```

示例三：过滤指定值

```
<?php
$arr = [ "a" => 1, "b" => 2, "c" => 3, "d" => 12, "e" => 15];
$my_value = 3;
$filtered_array = array_filter($arr, function ($element) use ($my_value) { return ($element != $my_value); } );
print_r($filtered_array); // [ "a" => 1, "b" => 2, "d" => 12, "e" => 15];
?>
```

示例四：带 `flag` 标记的数组过滤

```
<?php
$arr = [ "a" => 1, "b" => 2, "c" => 3, "d" => 12, "e" => 15];

// 默认为0 接受值作为唯一参数
var_dump(array_filter($arr, function($v) {
    return $v == 15;
})); // ["e" => 15]

// ARRAY_FILTER_USE_KEY - callback接受键名作为的唯一参数
var_dump(array_filter($arr, function($k) {
    return $k == 'b';
}, ARRAY_FILTER_USE_KEY)); // ["b" => 2]

// ARRAY_FILTER_USE_BOTH - callback同时接受键名和键值
var_dump(array_filter($arr, function($v, $k) {
    return $k == 'b' || $v ==12;
}, ARRAY_FILTER_USE_BOTH)); // ["b" => 2, "d" => 12,]
?>
```

# [array_product](https://www.php.net/manual/zh/function.array-product.php)

`array_product` — 计算数组中所有值的乘积。

格式：

```
array_product ( array $array ) : number
```

参数：

```
array_product ( 输入的数组) : number
```

返回：

```
以整数或浮点数返回一个数组中所有值的乘积。
5.3.6及之后的版本空数组会产生 1，而之前版本此函数处理空数组会产生 0。
```

示例一：基本使用

```
<?php
$a = [2, 4, 6, 8];
var_dump(array_product($a)); // 384
// 5.3.6及之后的版本空数组会产生 1，而之前版本此函数处理空数组会产生 0。
var_dump(array_product([])); // 1
?>
```

示例二：确认数组成员全部为真

```
<?php
$a = ['a' => true, 'b' => true, 'c' => true];
var_dump((bool)array_product($a)); // true
// 5.3.6及之后的版本空数组会产生 1，而之前版本此函数处理空数组会产生 0。
$b = ['a' => true, 'b' => true, 'c' => false];
var_dump((bool)array_product($b)); // false

$c = ['a' => true, 'b' => true, 'c' => 'true'];
// 注意：数组成员转为数值类型进行运算
var_dump((int)'true'); // 0
var_dump((bool)array_product($c)); // false
?>
```

# [array_count_values](https://www.php.net/manual/zh/function.array-count-values.php)

`array_count_values` — 统计数组中所有值出现的次数，返回一个数组，键名为值，键值为统计的次数。

格式：

```
array_count_values ( array $array ) : array
```

参数：

```
输入的数组
```

返回：

```
返回一个数组： 数组的键是 array 里单元的值； 数组的值是 array 单元的值出现的次数。
```

示例1：基本使用

```
<?php
$array = [6, "wang", 1, "xiong", "wang", 6, 1, 1];
var_dump(array_count_values($array)); // [ 6 => 2, 'wang' => 2, 1 => 3, 'xiong' => 1]
?>
```

示例2：数组中重复次数最多的值

```
<?php
$data = [6, 11, 11, 2, 4, 4, 11, 6, 7, 4, 2, 11, 8];
$cv = array_count_values($data); // [6 => 2, 11 => 4, 2 => 2, 4 => 3, 7 => 1, 8 => 1]
arsort($cv); // $cv [11 => 4, 4 => 3, 6 => 2, 2 => 2, 7 => 1, 8 => 1]
var_dump($cv);
var_dump(key($cv)); // 11
?>
```

# [array_chunk](https://www.php.net/manual/zh/function.array-chunk.php)

`array_chunk` —  将一个数组分割成指定大小的多个数组。

格式：

```
array_chunk ( array $array , int $size [, bool $preserve_keys = false ] ) : array
```

参数：

```
array $array：需要操作的数组。
int $size ：分割后每个数组的单元数目。
 bool $preserve_keys = false：默认false，新的数字索引，true，保留原来数组中的键名。
```

返回：

```
返回一个索引从零开始的一个多维数组，每一维包含了指定个数的元素。
```

示例1：基本使用

```
<?php
$arr = ['a', 'b', 'c', 'key4' => 'd', 'key5' => 'e'];
print_r(array_chunk($arr, 2));
print_r(array_chunk($arr, 2, true));
?>
```

示例1打印结果：

```
Array
(
    [0] => Array
        (
            [0] => a
            [1] => b
        )
    [1] => Array
        (
            [0] => c
            [1] => d
        )
    [2] => Array
        (
            [0] => e
        )
)
Array
(
    [0] => Array
        (
            [0] => a
            [1] => b
        )
    [1] => Array
        (
            [2] => c
            [key4] => d
        )
    [2] => Array
        (
            [key5] => e
        )
)
```

# [array_combine](https://www.php.net/manual/zh/function.array-combine.php)

`array_combine` — 创建一个新数组，用一个数组的值作为其键名，另一个数组的值作为其值 。

格式：

```
array_combine ( array $keys , array $values ) : array
```

参数：

```
keys：将被作为新数组的键。
values：将被作为新数组的值。
```

返回：

```
返回一个合并后的新数组，如果两个数组的单元数不同则返回 FALSE。
```

示例1：基本使用

```
$a = ['green', 'red', 'yellow'];
$b = ['avocado', 'apple', 'banana'];
$c = array_combine($a, $b);
print_r($c);
```

示例1打印结果：

```
Array
(
    [green] => avocado
    [red] => apple
    [yellow] => banana
)
```

# [array_diff](https://www.php.net/manual/zh/function.array-diff.php)

`array_diff` — 计算数组的差集。

格式：

```
array_diff ( array $array1 , array $array2 [, array $... ] ) : array
```

参数：

```
array1：从这个数组进行比较。
array2：被比较的数组。
...：更多被比较的数组。
```

返回：

```
返回一个数组，该数组包括了所有在 array1 中但是不在任何其它参数数组中的值，键名保留不变。
```

示例1：基本使用

```
<?php 
 $arr1 = ["a" => "green", "red", "blue", "red"];
$arr2 = ["b" => "green", "yellow", "red"];
$result = array_diff($arr1, $arr2);
print_r($result);
?>
```

示例1的打印结果：

```
Array
(
    [1] => blue
)
```


# [array_diff_assoc](https://www.php.net/manual/zh/function.array-diff-assoc.php)

`array_diff_assoc` — 带索引检查计算数组的差集。

格式：

```
array_diff_assoc ( array $array1 , array $array2 [, array $... ] ) : array
```

参数：

```
array1：从这个数组进行比较
array2：被比较的数组
...：更多被比较的数组
```

返回：

```
返回一个不在其他数组中出现的数组1的值。
```

示例1：基本使用

```
<?php
$arr1 = ["a" => "green", "b" => "brown", "c" => "blue", "red"];
        $arr2 = ["a" => "green", "yellow", "red"];
        $res = array_diff_assoc($arr1, $arr2);
        print_r($res);
?>
```

示例1的结果：

```
Array
(
    [b] => brown
    [c] => blue
    [0] => red
)
```

注：因数组2中的`red`的键名是1，和要比较的数组中`red`的键名0不是同一个键名，因此 `0 => "red"`会 出现在输出中。

# [array_diff_key](https://www.php.net/manual/zh/function.array-diff-key.php)

`array_diff_key` — 使用键名比较计算数组的差集。

格式：

```
array_diff_key ( array $array1 , array $array2 [, array $... ] ) : array
```

参数：

```
array1：从这个数组进行比较。
array2：被比较的数组。
...：更多被比较的数组。
```

返回：

```
返回一个新数组，该数组包括了所有出现在数组1中的数组，但是未出现在其他数组中键名的值。
```

示例1：基本使用

```
<?php
$arr1 = ['blue' => 1, 'red' => 2, 'green' => 3, 'purple' => 4];
$arr2 = ['green' => 5, 'blue' => 6, 'yellow' => 7, 'cyan' => 8];
print_r(array_diff_key($arr1, $arr2));
?>
```

示例1的打印结果：

```
Array
(
    [red] => 2
    [purple] => 4
)
```

# [array_merge_recursive](https://www.php.net/manual/zh/function.array-merge-recursive.php)

`array_merge_recursive` — 递归地合并一个或多个数组。

格式：

```
array_merge_recursive ( array $array1 [, array $... ] ) : array
```

参数：

```
array1：要合并的初始数组。

...：数组变量列表，进行递归合并。
```

返回：

```
返回一个递归合并后的新数组。
```

示例1：基本使用

```
<?php
$arr1 = ["color" => ["favorite" => "red"], 5];
$arr2 = [10, "color" => ["favorite" => "green", "blue"]];
$result = array_merge_recursive($arr1, $arr2);
print_r($result);
?>
```

示例1的打印结果：

```
Array
(
    [color] => Array
        (
            [favorite] => Array
                (
                    [0] => red
                    [1] => green
                )

            [0] => blue
        )

    [0] => 5
    [1] => 10
)
```

示例2：与`array_merge`进行比较

```
<?php
$arr1 = ["color" => ["favorite" => "red"], 5];
$arr2 = [10, "color" => ["favorite" => "green", "blue"]];
$result = array_merge($arr1, $arr2);
print_r($result);
?>
```

示例2的打印结果：

```
Array
(
    [color] => Array
        (
            [favorite] => green
            [0] => blue
        )

    [0] => 5
    [1] => 10
)
```
