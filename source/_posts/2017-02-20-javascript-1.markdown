---
layout: post
title: Javascript基础-1
date: 2017-02-20 10:32:24.000000000 +09:00
categories:
- 技术
tags:
- JavaScript
toc: true
---
**
摘要：
**
<!-- more -->
<The rest of contents | 余下全文>

# js简介
定义：基于对象和事件驱动并具有安全性能的脚本语言。


# 语法规则
## 引入js语言
```
<script type=”text/javascript”>具体js代码</script>
<script type=”text/javascript” src=”js文件”></script>
```
## 大小写
大小写敏感
true/false   布尔值
TRUE/FALSE  不是
## 结束标志
    在js里边每条简单语言可以使用”;”分号结束,此分号也可以不使用(推荐使用)。
## 注释
```
// 注释单行
/* 注释多行 */
```
## 变量
命名规则
组成：字母、数字、下划线、$、汉字,开始的第一个字母不能是数字
$符号：在php里边是变量名字的第一个字母，在jquery里边是一个关键字，因此在一般的javascript代码里边不推荐使用该信息作为名字组成部分。
```
var  $ = “hello”; // 正确
var get_info =”abc”;
```
#  数据类型
## php数据类型
int、float、string、boolean、array、object、resource、null
## js数据类型：
number:数值类型，包括int和float
string：字符串类型
boolean：布尔类型true/false
undefined：未定义型，变量使用之前并没有任何声明
null：空类型，空指针、空对象类型 var obj = null; 是对象的声明，后期要使用一个具体对象对其进行赋值.
object：对象类型
在ECMAscript里边把数组归结为对象类型，是对象内部的一个组成部分。


#数值类型


## 进制表示


```
<script type="text/javascript">
//八进制数,在数字的前边设置"0"的信息
console.log(05); //5
console.log(045); //4*8+5=37
console.log(037); //3*8+7=31
console.log(0735); //7*8*8+3*8+5=477
console.log(091);  //91,恢复为十进制
console.log(029);  //29. 恢复为十进制


//十六进制,在数字的前边设置"0x"的信息
//A--10  B--11  C--12  D--13  E--14  F--15
console.log(0xA);//10
console.log(0xD);//13
console.log(0x24); //2*16+4=36
console.log(0xC4); //12*16+4 = 196
</script>
```
## 最大和最小数


最大数：Number.MAX_VALUE;
最小数：Number.MIN_VALUE;


```
<script type="text/javascript">
//最大数
console.log(Number.MAX_VALUE);//1.7976931348623157e+308  1.79*10的308次方


//最小数
console.log(Number.MIN_VALUE);//5e-324   0.0000 324个零0000 5


//科学计数法
console.log(3.1415e+7);//31415000

//Infinity console.log(Number.MAX_VALUE+Number.MAX_VALUE);//Infinity
</script>


```
## typeof关键字


```
<script type="text/javascript">
console.log(typeof 103);  //number
console.log(typeof 'tom');  //string
console.log(typeof true);  //boolean
console.log(typeof false);  //boolean
console.log(typeof TRUE);  //undefined
console.log(typeof null);  //空对象object
console.log(typeof document);  //对象object
</script>
```


# 浮点类型


```
<script type="text/javascript">
//浮点数
console.log(12.34);  //12.34
console.log(0.34);  //0.34
console.log(.34);   //0.34
console.log(12.0);   //12


//浮点数计算不准确
console.log(0.1+0.2);//0.30000000000000004
console.log((0.1e+4+0.2e+4)/(10*10*10*10));//0.3
</script>
```


# 运算符
## 算术运算符
```
   +  -   *   /   %  ++   --
```


```
<script type="text/javascript">


/**
    num++ : 先赋值(先把值返回)，再++累加操作
    ++num : 先++累加操作，再赋值(把值返回)
*/


var score = 90;
var newscore = score++;
console.log(newscore+"---"+score);  //90-----91
var newscore = ++score;
console.log(newscore+"==="+score); //91===91
</script>
```


## 比较运算符
```
>  <  ==   >=  <=  !=
===三等号： 全等于，比较值大小和类型
10==’10’  true
10===’10’  false
!==: 不全等于
102!== ’102’  true
```
## 逻辑运算符


```
&&: 两边结果都为真结果为真
|| ：两边结果只要有一个为真，最终结果为真
!： 真即假、假即真
①在php里边逻辑运算符结果为true/false
在javascript里边逻辑运算符的结果有的时候是操作数之一。
```


```
<script type="text/javascript">


var name = "linken";
var addr = "beijing";
//把操作数当作逻辑运算符的返回结果
console.log(name && addr); //beijing
console.log('tom' && 0);  //0


var a = 10;
var b = 20;
var c = 30;
//把true/false当作具体返回结果
console.log(a<b && c>b); //true




var age = 20;
var height = 170;
//以下表达式age已经可以决定整体的结果，height的变量没有被执行
//height被age给短路了
console.log(age || height); //20  age短路height


console.log("" ||  height); //170  都有执行


console.log(0 && age); //0  0短路age


console.log(age || alert('123'));  //20  alert被短路（不执行）


console.log(age && alert('hello'));//undefined  都执行


// !取反
console.log(!age);  //false
console.log(!0);   //true
</script>
```
# 流程控制
## switch分支选择
变量值：常量(字符串信息、数字信息)、变量
```
switch(条件变量){
    case 变量值:
        break;
    case 变量值:
        break;
    default:
}
```
```
<script type="text/javascript">
var age = 100;
switch(true){
    case age>0 && age<10:
        console.log('儿童');
        break;
    case age>=10 && age<20:
        console.log('青少年');
        break;
    case age>=20 && age<30:
        console.log('青年');
        break;
    case age>=30 && age<40:
        console.log('成年');
        break;
    default:
        console.log('壮年');
}
</script>
```
## continue和break关键字
continue: 跳出本次循环，进入下次循环。
break：跳出本层循环
```
<script type="text/javascript">
//break和continue可以跳转到外部标志代表的循环处
second:
for(var i=0; i<5; i++){
    document.write(i+"<br />");
    for(var m=100; m>90; m--){
        if(m==95){
            //break second;
            continue second;
        }
        document.write(m+"--");
    }
    document.write("<br />");
}
</script>
```


# 函数


## 函数封装
① 普通方式声明：
```
function 函数名称(){}
```
在javascript里边把函数可以当成是一个变量，数据类型为”对象”。
传统方式函数声明，函数有预加载过程，函数要先于普通代码被加载到内存里边，因此可以先调用、后声明。
条件：在同一个script标签里边(每个script标签单独被加载、解释、运行，并且有顺序要求)
```
<script type="text/javascript">
getInfo();
function getInfo(){
    console.log('hello world');
}
</script>
```
② 变量赋值方式声明：
```
var  函数名称 =  function(){}
```
条件：该方式必须先声明、后调用。
```
<script type="text/javascript">
var getInfo = function(){
    console.log('海淀');
}
getInfo();
</script>
```

## 函数参数
①在javascript里边函数的形式参数与实际参数没有严格的对应关系。
```
<script type="text/javascript">
//参数使用，形参与实参没有严格对应关系
function getInfo(name,addr,age){
    console.log(name+"--"+addr+"--"+age);
}
getInfo();
getInfo('tom');
getInfo('mary','tianjin',24);
getInfo('mary','tianjin',25,'100');
</script>
```


②在javascript里边函数的形式参数可以有默认值。（有默认值的形参给排在参数的后边）
```
<script type="text/javascript">
function f1(name,addr='beijing',age=22){
    console.log(name+"--"+addr+"--"+age);
}
f1('linken');
f1('xiaoming','shanghai');
f1('yuehan','jiazhou',23);
</script>
```
③通过关键字arguments获得参数的具体信息
```
<script type="text/javascript">
function f1(){
console.log(arguments.length+":::"+arguments[0]+"--"+arguments[1]);
}
f1('mary','newyork');
f1('linken','guangzhou',26);
</script>
```


## 函数返回值
return可以结束当前函数的执行。
在javascript里边可以return返回一个函数给接收者使用。(函数本身就是一个对象，可以用于返回return使用)
```
function f1(){
    return 具体返回值string number boolean object null
}
```


```
<script type="text/javascript">
function f1(){
    var name = "linken";
//在函数内部声明一个函数，函数的数据量类型为“对象”
    function f2(){
        console.log('hello world');
    }
    return f2;
}
var ff = f1();
ff();  //hello world
</script>
```


## 函数调用
① 传统方式调用：函数名();
```
<script type="text/javascript">
function f1(){
    console.log('hello');
}
f1(); //hello
</script>


```
② 匿名函数自调用：(function(){})();
```
<script type="text/javascript">
(function(){
    console.log('beijing');
})();
</script>


```
## callee关键字
callee：代表当前函数的引用，该关键字可以减低代码的耦合度
```
<script type="text/javascript">
//求n的阶乘
//!5 = 5*4*3*2*1
//!4 = 4*3*2*1
//!3 = 3*2*1
//!2 = 2*1
//!1 = 1
function jiecheng(n){
    if(n==1){
        return 1;
    } else {
        return n*arguments.callee(n-1);
    }
}
</script>
```
## caller关键字
caller： 代表调用该函数的函数
```
<script type="text/javascript">
function cat(){
    dog();
    console.log('cat');
}
cat();
function pig(){
    dog();
    console.log('pig');
}
pig();
function dog(){
    //查看都有谁调用我
    console.log(dog.caller.name);
    console.log('dog');
}
</script>
```


## 局部变量和全局变量
全局变量：
在php里边，在函数外边声明的就是全局变量。
在javascript里边，变量的直接调用对象是window，这个变量就是全局变量。
在函数外部声明的变量就是全局变量，在函数内部如果不使用var关键字声明变量，其也是全局变量。


局部变量：
在php里边，函数内部声明的就是局部变量。
在javascript里边，在函数内部声明的变量，并且其使用var关键字，就是局部变量。


```
<script type="text/javascript">
//全局变量
var addr = "beijing";


function f1(){
    //全局变量
    weight = 102;


    //局部变量
    var animal = "dog";
    var color = "blue";


    //函数内部可以直接使用全局变量，无需声明
    console.log(addr);  //beijing
}
f1();
</script>


```


# 数组
定义：把许多名称和数据类型都一样的变量集合起来就是数组。
作用：使用单独的变量名来存储一系列的值。


## 数组声明


方式一：
```
var fruit = ['apple','pear','orange','grape'];
```


方式二：
使用关键词new来创建数组对象。
```
var animal = new Array('dog','pig','tiger');
```


方式三：
使用一个整数自变量来控制数组的容量,该数组只是声明，没有限制作用。
其中丰富元素的时候[]内部需要手动设置下标,与php不同。
```
var color = new Array(3);
color[0] = "red";
color[1] = "yellow";
color[2] = "blue";
console.log(typeof color);//object
```
注意：
```
<script type="text/javascript">
var color = new Array('3'); //① 一个元素的数组
var animal = new Array(3);  //② 一个有三个元素的数组
console.log(color[0]);      //undefined
</script>
```
注意：在js里边数组都是“索引”数组。




## 数组长度


数组.length：获得数组最大下标加1的信息值。
注：如果要想获得正确数组长度，则数组下标必须为为0,1,2..规则的。
```
<script type="text/javascript">
var fruit = ['apple','pear','orange','grape'];
fruit[4] = "banana";
fruit['shanxi'] = "mihoutao";
fruit[5] = "watermelon";
fruit[10] = "fireguo";
console.log(fruit.length); //11
</script>
```


## 数组遍历
方式一：for遍历数组，条件是数组下标是0,1,2,3...规则的。
```
<script type="text/javascript">
var fruit = ['apple','pear','orange','grape'];
for(var i=0; i<fruit.length; i++){
    console.log(i+"--"+fruit[i]);
}
</script>
```


方式二：for-in遍历，数组和 对象(属性和其值)都可以遍历。
使用方法：
```
for(var 下标变量 in fruit){}
```
```
<script type="text/javascript">
var fruit = ['apple','pear','orange','grape'];
fruit['hainan'] = "tao";
fruit[10] = "bananan";
for(var k in fruit){
    console.log(k+"---"+fruit[k]);
}
</script>
```


遍历二维数组：
```
<script type="text/javascript">
var group1 = ['1','2','3'];
var group2 = ['4','5','6'];
var student = [group1,group2];


for(var k in student){
    for(var kk in student[k]){
        console.log(kk+"---"+student[k][kk]);
    }
}
</script>
```


## 数组方法
instanceof方法：判断一个对象是否是指定类给实例化出来的
```
<script type="text/javascript">
var fruit = ['apple','pear','orange','grape'];
console.log(typeof fruit); //object


if(fruit instanceof Array){
    console.log('数组');
}
</script>
```


push()在数组最后压入元素。
pop()删除数组最后一个元素。
```
<script type="text/javascript">
var fruit = ['apple','pear','orange','grape'];
fruit.push('banana');
console.log(fruit); // fruit = ['apple','pear','orange','grape','banana'];


fruit.pop();
fruit.pop();
fruit.pop();
console.log(fruit); // fruit = ['apple','pear'];
</script>
```


# 字符串
字符串声明：字符串声明和对象声明的区别，对象可以给自己声明属性,而字符串不可以声明属性。
```
<script type="text/javascript">
var name = "tom";
var addr = new String('北京');
name.color = "red";
addr.color = "blue";
console.log(name.color);//undefined
console.log(addr.color);//blue
</script>
```


字符串方法：字符串本身是不应该调用属性/方法，其实质是浏览器js解释引擎首先把字符串转变为“临时对象”，通过临时对象访问属性并返回，接着把“临时对象”给销毁。
javascript里边一切都是对象。
```
<script type="text/javascript">
var addr = "beijing";
console.log(addr.length); //7
</script>
```


# eval()方法


eval()：把参数字符串信息当作表达式在上下文环境中运行。
注意：该eval内部参数字符串必须符合js语法规则。


```
<script type="text/javascript">
console.log("10+20");  //10+20
console.log(eval("10+20")); //30


var a = 100;
var b = 200;
console.log("a+b"); //a+b
console.log(eval("a+b")); //300
console.log(eval("c+d")); //错误 不存在的变量


console.log("alert('123')");//alert('123')
eval("alert('456')");//弹出框
eval("alrt('abc')");//错误,alrt不符合js语法规则
</script>
```

