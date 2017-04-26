---
layout: post
title: JavaScript基础-3
date: 2017-03-20 19:32:24.000000000 +09:00
---
## <span id="0">目录</span>
>* [**作用域链**](#1)
>* [作用域链作用](#1.1)
>* [AO活动对象](#1.2)
>* [相关变量](#1.3)
>* [**闭包**](#2)
>* [闭包定义](#2.1)
>* [闭包计数器](#2.2)
>* [闭包生成数组](#2.3)
>* [闭包高亮显示](#2.4)
>* [**基于（面向）对象**](#3)
>* [创建对象](#3.1)
>* [return关键字](#3.2)
>* [构造函数](#3.3)
>* [内存分配](#3.4)
>* [函数执行](#3.5)
>* [this关键字](#3.6)
>* [函数对象](#3.7)

<embed src="//music.163.com/style/swf/widget.swf?sid=406000649&type=2&auto=1&width=320&height=66" width="340" height="86"  allowNetworking="all"></embed>

## [<span id="1">作用域链</span>](#0)

在当前环境定义的变量，其在当前环境、内部环境、内部深层环境都可以使用。
变量在许多环境里边可以使用的现象类似一个链条的体现，称之为“作用域链”。

```
<script type="text/javascript">
var addr = '北京'; //全局变量

function f1(){
    //可以调用addr
    console.log('f1:'+addr);
    function f2(){
        //可以调用addr
        console.log('f2:'+addr);
        function f3(){
            //可以调用addr
            console.log('f3:'+addr);
        }
        f3();
    }
    f2();
}
f1();
</script>
```

## [<span id="1.1">作用域链作用</span>](#0)
① 内部环境可以访问外部环境定义的变量,外部环境不可以访问内部环境的变量。
```
<script type="text/javascript">
var addr = '北京'; //全局变量
function f1(){
    var age = 23;
    function f2(){
        function f3(){
            //内部环境可以访问外部环境的变量
            console.log('f3:'+addr+"---"+age);
        }
        f3();
    }
    f2();
}
f1();
console.log(age);//错误，禁止“外部环境”访问“内部环境”变量
</script>
```
② 保证变量按照一定的顺序访问(先声明、后使用)。
a.变量必须“先声明、后使用”

```
<script type="text/javascript">
function f1(){
    function f2(){
        console.log(age); //25
    }
    f2();
}
var age = 25; //声明一个变量
f1();         //函数调用
</script>

```

b.同名的“变量”会覆盖“函数”。

```
<script type="text/javascript">
var getInfo = "昌平";
function getInfo(){
    console.log('1234');
}
getInfo();//getInfo is not function
</script>

```
c.如果变量在函数里边使用，则函数的调用必须在变量的声明之后。
d.函数可以“先使用、后声明”。
条件：在同一个标签里边，函数可以先使用、后声明。
函数有“预加载”过程，函数的声明会先于其他普通的代码被优先放入内存，因此，函数可以先调用、后声明。

```
<script type="text/javascript">
f1();
function f1(){
    console.log('abcd');
}
</script>
```

③ 变量作用域是声明时候就决定好的，而不是运行时。
示例1：
```
<script type="text/javascript">
var addr = "海淀";
function f1(){
    console.log(addr);
}
function f2(){
    var addr = "朝阳";
    f1();
}
f2();   //海淀
</script>
```

示例2：
```
<script type="text/javascript">
var addr = "海淀";
function f1(){
    console.log(addr);
}
addr = "丰台";
function f2(){
    var addr = "朝阳";
    f1();
}
addr = "崇文";
f2();          //崇文
addr='大兴';
</script>
```

## [<span id="1.2">AO活动对象</span>](#0)
执行环境
a. 在javascript语言里边，js代码执行是有环境的，每个函数都是一个执行环境。
b. 该环境定义了其有权访问的其他数据。
c. 环境有一个与之关联的“活动对象”。
d. 环境中所有的变量和函数都是活动对象的属性。
e. 全局环境是最外围的执行环境，活动对象是window对象。
f. 执行环境中的代码执行完毕后就被销毁。

AO-ActiveObject活动对象：
每个执行环境内部都有AO，内部定义了当前环境可以访问的变量信息，是“固态信息”。实际程序在真正执行的时候不是动态过程而是一个固态过程。
```
<script type="text/javascript">
var addr = "北京";
var area = 200;
window.addr
window.area              //window环境
function f1(){
    AO.addr
    AO.area              //f1环境
    console.log('f1:'+addr+"--"+area);
    var weather = "晴朗";
    AO.weather
    function f2(){
        AO.addr
        AO.area           //f2环境
        AO.weather
        console.log('f2:'+addr+"--"+area);
        function f3(){
            AO.addr
            AO.area        //f3环境
            AO.weather
            console.log('f3:'+addr+"--"+area);
        }
    }
}
</script>
```
执行动态过程：
在f3内部环境获得变量信息addr/area，此时本身f3环境没有该变量，这样就到外部环境f2寻找，没有找到就去f1环境寻找，也没有,最后跑到window环境，获得了addr/area两个变量信息。
执行固态过程：
每个环境内部可以使用一个变量，直接找其AO即可，代码一旦执行，每个环境都把自己可以访问的变量信息封装在了AO里边，本质上没有“动态”寻找外部环境变量的过程。

## [<span id="1.3">相关变量</span>](#0)
执行环境内部变量类型：局部变量、形参、内部函数、外部环境变量。
总顺序关系：局部变量  >  内部函数 > 形参  > 外部变量。

```
<script type="text/javascript">
// 总顺序关系：局部变量  >  内部函数 > 形参  > 外部变量
function f1(){
    function f2(){
        var addr = "shanghai";           //外部变量
        function f3(addr){               //形参
            var addr = "chongqing";      //局部变量
            console.log(addr);
            function addr(){             //内部函数
                alert('123');
            }
        }
        f3('guangzhou');
    }
    f2();
}
f1();          //chongqing
</script>
```
注：从相对概念上看，局部变量 也是”全局变量”，其在当前本身环境、内部环境、内部深层环境都会起作用。因此其是特殊的“全局变量”。
全局变量，其上级对象是window，其作用域是全部任何环境都会起作用。
## [<span id="2">闭包</span>](#0)
## [<span id="2.1">闭包定义</span>](#0)
定义:两个彼此嵌套的函数，内部的函数就是一个“闭包”,内部函数需要return返回。
作用：闭包有权对其外部环境变量进行访问。
```
<script type="text/javascript">
function f1(){
    var age = 23;
    var addr = '北京';
    function f2(){
        console.log(AO.age+"--"+AO.addr);
    }
    return f2;
}
var ff = f1(); // ff和f2是两个不同引用，指引到同一个function

/**
f1()函数执行完毕，age和addr两个变量就被销毁
f2原则上也是要被销毁的，但是有新的引用ff指引它，因此f2函数的主体function没有销毁
但是f2变量引用要被销毁。
*/

/**
ff()函数调用执行，依然可以获得“23-北京”信息，原因是f2引用被销毁,但是f2 function主体并没有被销毁,因此其内部的AO和其属性都有保留,索引
ff()执行后依然可以获得信息。
*/
ff(); //23--北京
</script>
```
## [<span id="2.2">闭包计数器</span>](#0)
① 传统计数器会出现变量污染

```
<script type="text/javascript">
var num = 0;
function addnum(){
    return ++num;
}
console.log(addnum());  //1
console.log(addnum());  //2
console.log(addnum());  //3
console.log(addnum());  //4
console.log(addnum());  //5
num = 200;
console.log(addnum());  //201
console.log(addnum());  //202
</script>
```
② 采用闭包计数器方式，内部变量不会被污染

```
<script type="text/javascript">
function f1(){
    var num = 0;
    function f2(){
        return ++num;
    }
    return f2;
}
var addnum = f1();
console.log(addnum());  //1
console.log(addnum());  //2
num = 1000; //全局变量与闭包内部的num不发生影响
console.log(addnum());  //3
</script>
```

## [<span id="2.3">闭包生成数组</span>](#0)
a.利用同一个嵌套函数生成多个闭包，这些闭包是系统里边存在的多个独立函数。
调用函数同时生成多个闭包，这些闭包的生成主体是一样的，但是大家在执行的值，各走各的，互相之间没有影响,也可以简单的理解成，生成几个闭包就生成几个彼此独立的函数。
```
<script type="text/javascript">
function f1(){
    var num = 0;
    function f2(){
        return ++num;
    }
    return f2;
}
var addnum = f1();
console.log(addnum());  //1
console.log(addnum());  //2
console.log(addnum());  //3

var getinfo = f1();
//getinfo与addnum一样，都是闭包,都是函数的体现,但他们是独立的函数。
console.log(getinfo());  //1
console.log(addnum());   //4
console.log(getinfo());  //2
</script>
```
b.闭包数组生成(不成立的示例)
```
<script type="text/javascript">
var fruit = new Array();
/*
fruit[0] = function(){console.log(0);}
fruit[1] = function(){console.log(1);}
fruit[2] = function(){console.log(2);}
fruit[3] = function(){console.log(3);}
fruit[4] = function(){console.log(4);}
*/
for(var i=0; i<5; i++){
    fruit[i] = function(){
        console.log(i);
    }
}
console.log(fruit); //[function(), function(), function(), function(), function()]
//以下访问数组元素获得到的结果都是5，原因：数组每个元素都是一个函数，他们访问到了系统里边的全局变量i,这个i的值为5。
fruit[3]();  //5
fruit[1]();  //5
fruit[0]();  //5
fruit[2]();  //5
fruit[4]();  //5
</script>
```
c.闭包数组生成(成立的示例)
```
<script type="text/javascript">
/*
fruit[0] = function(){console.log(0);}
fruit[1] = function(){console.log(1);}
fruit[2] = function(){console.log(2);}
fruit[3] = function(){console.log(3);}
fruit[4] = function(){console.log(4);}
*/
var fruit = new Array();
console.log(fruit);
for(var i=0; i<5; i++){
    //f1()函数被在循环里边调用了5次，这样系统要生成5个独立的函数,每个函数的输出信息由"n"来决定,分别是0 1 2 3 4
    //function(){console.log(0)}
    //function(){console.log(1)}
    //function(){console.log(2)}
    //function(){console.log(3)}
    //function(){console.log(4)}
    fruit[i] = f1(i);
}
function f1(n){
    function f2(){
        console.log(n);
    }
    return f2;
}
fruit[4]();  //4
fruit[0]();  //0
fruit[1]();  //1
fruit[2]();  //2
fruit[3]();  //3
</script>
```

## [<span id="2.4">闭包高亮显示</span>](#0)
```
<script type="text/javascript">
window.onload = function(){
    var lis = document.getElementsByTagName('li');
    for(var i=0; i<lis.length; i++){
        //over()获得闭包函数，共调用了4次，这样系统要生成4个函数,函数内部的n分别为：0 1 2 3
        lis[i].onmouseover = over(i);
        lis[i].onmouseout = out(i);
    }
    //鼠标移入
    function over(n){
        function fs(){
            lis[n].style.backgroundColor = "lightblue";
        }
        return fs;
    }
    //鼠标移出
    function out(n){
        function fs(){
            lis[n].style.backgroundColor = "";
        }
        return fs;
    }
}
</script>
```
## [<span id="3">基于（面向）对象</span>](#0)
在javascript里边可以直接声明一个对象出来(而不是php里边，先找一个类，才有对象)。
## [<span id="3.1">创建对象</span>](#0)
① 字面量方式创建对象
方式1：
```
<script type="text/javascript">
var animal = {};
console.log(animal); //Object {}

//给对象丰富相关的成员
animal.color = 'yellow';
animal.run = function(){
    console.log('run');
}
console.log(animal);       //Object { color="yellow", run=function()}
console.log(animal.color); //yellow
animal.run();              //run
</script>
```

方式2：
```
<script type="text/javascript">
var dog = {name:'dog',"age":6, color:'yellow', hobby:function(){console.log('watchdog');}};
console.log(dog);       //Object {name: "dog", age: 6, color: "yellow"}
console.log(dog.name);  //dog
console.log(dog.age);   //6
dog.hobby();            //watchdog
</script>
```
② 构造函数创建对象
在javascript里边可以通过 new 函数名() 方式创建一个对象,对象在被创建的过程中，要执行这个函数，该函数的作用类似php里边的构造函数。
方式1：
```
<script type="text/javascript">
//var cat = new 类名(); //php
//var cat = new 函数名(); //javascript
function Animal(){
    var age = 6;
}
var cat = new Animal();
cat.color = "white";
cat.eat = function(){
    console.log('fish');
}
console.log(cat);//Animal {color: "white"}
</script>
```

方式2：
```
<script type="text/javascript">
function Animal(cr){
    var age = 6;
    this.color = cr;
    this.name = 'dog';
    this.eat = function(){
        console.log('fish');
    }
}
//把当前对象的相关属性值给当作参数传递给构造函数
var dog = new Animal('black');
console.log(dog); // Animal {color: "black", name: "dog"}
/**
以上方式创建对象内部执行过程
① 创建this关键字对象，此时this指引null
② 创建dog对象
③ 使得dog指引this
④ 给this对象丰富成员信息，也就是间接给dog丰富成员信息
*/
</script>
```
③ Object创建对象
方式1：
```
<script type="text/javascript">
var cat = {};
console.log(cat);//Object {}
//字面量方式创建对象走的构造函数是Object，也可以直接使用Object创建对象。
var dog = new Object();
dog.age = 7;
console.log(dog); // Object {age: 7}
</script>
```
## [<span id="3.2">return关键字</span>](#0)
return关键字不影响对象接收，但是后边的代码就不能再继续执行。

```
<script type="text/javascript">
function Animal(){
    this.name = 'dog';
    this.color = 'yello';
    // return关键字
    return "beijing";
    this.run = function(){
        console.log('run');
    }
}
var dog = new Animal();
console.log(dog);//Animal {name: "dog", color: "yello"}
</script>
```

## [<span id="3.3">构造函数</span>](#0)
普通函数与构造函数本质上没有任何区别，主要是看调用的方式。
new 函数名();   ----------->构造函数
函数名();       ----------->普通函数
```
<script type="text/javascript">
function Animal(){
    //this------>window,this在函数内部就代表调用该函数的对象
    this.color = "yellow";
    this.age = 7;
    this.run = function(){
        console.log('run');
    }
}
Animal();//【普通函数调用】 会给系统生成三个全局变量:color/age/run
console.log(color); //yellow
console.log(age);   //7
run();              //run

function Person(){
    var height=170;
    var color = "yellow";
}
var tom = new Person();//【构造函数调用】
console.log(tom); // Person {}
</script>
```
## [<span id="3.4">内存分配</span>](#0)
```
<script type="text/javascript">
function Animal(){
    this.color = "yellow";
    this.age = 23;
    this.hobby = function(){
        console.log('watchdog');
    }
}
var black = new Animal();
console.log(black.color); //yellow

var white = black;//引用赋值
white.color = "white";
console.log(black.color); //white

var gold = new Animal();
console.log(gold.color); //yellow
</script>
```
## [<span id="3.5">函数执行</span>](#0)
① 作为普通函数执行(函数调用、匿名函数自调用)
② 作为“构造函数”执行(实例化对象)
③ 作为对象的成员方法使用
④ call和apply执行
被调用函数.call(函数内部this指引,实参1，实参2。。。)。
被调用函数.apply(函数内部this指引,[实参1，实参2。。。])。
```
<script type="text/javascript">
function f1(name,color){
    console.log(name+"---"+color+"=="+this.height);
}

function Person(){
    this.height = 170;
}
// call(函数内部this指引,实参1，实参2。。。)
var tom = new Person();
f1.call(tom,'call','black'); // call---black==170

// apply(函数内部this指引,[实参1，实参2。。。])
var cat = {height:20, addr:'beijing'}
f1.apply(cat,['apply','yellow']); // apply---yellow==20
</script>
```
## [<span id="3.6">this关键字</span>](#0)
① 代表window对象。
② 代表调用该方法的当前对象。
③ 代表页面上具体的元素节点对象。
```
elenode.onclick = function(){this}
```
④ 可以任意代表其他对象
例如：函数.call(内部this指引)
## [<span id="3.7">函数对象</span>](#0)
① 对象.constructor可以获得创建该对象的构造函数信息。
```
<script type="text/javascript">
function Animal(){
    this.color = "yellow";
}
var cat = new Animal();
console.log(cat.constructor);      //Animal()

var dog = {name:'black'};
console.log(dog.constructor);      //Object()

function getInfo(){
    console.log('hello beijing');
}
console.log(getInfo.constructor);  //Function()
</script>
```
② 利用Function实例化一个函数对象
```
<script type="text/javascript">
//function f1(name,age){console.log(name+'--'+age);}
//var f1 = new Function(形参1,形参2,形参。。。,形参n,函数实体);
var f1 = new Function('name','age',"console.log(name+'--'+age)");
f1('tom',25);  //tom--25
</script>
```