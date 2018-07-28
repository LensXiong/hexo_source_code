---
layout: post
title: Javascript基础-2
date: 2017-03-15 15:32:24.000000000 +09:00
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

## [<span id="1">DOM简介

DOM：document  object  model  文档对象模型
BOM：browser   object  model  浏览器对象模型
在php和javascript里边都有dom，他们的标准都是一样的。
在php中，dom是php与xml/html之间沟通的桥梁。
在javascript中，dom是javascript与html/xml之间沟通的桥梁。

html文档的各个节点：
元素节点:每个HTML元素是一个元素节点。如html、head、body、input、div。
属性节点:每个HTML属性是一个属性节点。如type、name。
文本节点:每个HTML元素内的文本是一个文本节点。如this is  javascript。
注释节点:每个HTML注释是一个注释节点。
文档节点:整个document文档是一个文档节点。

```
<html>
    <head>

    </head>
    <body>
        <!注释节点-->
        <input  type=”text” name=”username”>
        <div>this is  javascript</div>
    </body>
</html>
```

nodeType:元素节点1   属性节点2  文本节点3  文档节点9  注释节点8

```
<body>
    <h2>nodeType</h2>
    <input type="text" name="username" value="tom"  class="orange" id="userone"><br />
</body>
<script type="text/javascript">
    var it = document.getElementById('userone');//获得元素节点
    var atts = it.attributes;                   //获得“数组列表”属性节点
    console.log(atts.name.nodeType);            //2
</script>
```

## [<span id="2">DOM操作

DOM操作：对HTML页面内部的各个节点进行“增、删、改、查”的操作。

### [<span id="2.1">获得元素节点
① document.getElementById(id属性值)
通过getElementById()获得元素节点,该方法每次获得一个节点。
```
<body>
    <h2>通过属性id获得元素节点</h2>
    <div id="apple">this is apple div</div>
    <div id="orange">this is orange div</div>
</body>
<script type="text/javascript">
    var dv = document.getElementById('apple');
    console.log(dv);
</script>
```

② document.getElementsByTagName(tag标签名称)
通过tag标签获得元素节点,该方式以“数组”形式返还具体结果(即使只有一个元素)。

```
<body>
    <h2>通过tag标签获得元素节点</h2>
    <div id="apple">this is apple div</div>
    <div id="orange">this is orange div</div>
</body>
<script type="text/javascript">
    var dv = document.getElementsByTagName('div');
    console.log(dv);//HTMLCOLLECTION
    console.log(dv[1]);
</script>
```


③ document.getElementsByName(name属性值)
通过name属性值元素节点,该方式以“数组”形式返还具体结果(即使只有一个元素)。
该方式一般在form表单里边使用，如果放到表单外部使用，有浏览器兼容问题，因此不推荐使用。

```
<body>
    <h2>通过name属性获得元素节点</h2>
    <input type="text" name="hobby" value="篮球"><br />
    <input type="text" name="hobby" value="足球"><br />
</body>
<script type="text/javascript">
    var it = document.getElementsByName('hobby');
    console.log(it);//NodeList[input 属性(attribute)值 = "篮球"]
    console.log(it[0]);
</script>
```

# 获取属性节点
先获得元素节点，再通过attributes获得“数组列表”的属性节点。
元素节点.attributes：  以数组列表形式返还属性节点
属性列表.属性名称：    获得具体属性节点

```
<body>
    <h2>获得属性节点</h2>
    <input type="text" name="username" value="tom"  class="orange" id="userone"><br />
</body>
<script type="text/javascript">
    var it = document.getElementById('userone');//获得元素节点
    var atts = it.attributes; //获得“数组列表”属性节点
    console.log(atts);        //[type="text", id="userone", class="orange", value="tom", name="username"]
    console.log(atts.name);   //获得具体name属性节点
    console.log(atts.id);     //获得具体id属性节点
    console.log(atts.class);  //获得具体class属性节点
</script>
```
属性相关操作：
①操作方式只能操作w3c规定的属性。
元素节点.属性名称;
元素节点.属性名称= 值;
②操作方式对“w3c规定属性”或“自定义属性”都可以操作。
node.getAttribute(属性名称)
node.setAttribute(属性名称，值)

```
<body>
    <h2>属性值操作</h2>
    <input type="text" name="username" value="tom"  class="orange" id="userone" animal="dog"><br />
    <input type="text" name="hobby" value="足球"><br />
</body>
<script type="text/javascript">
    var it = document.getElementById('userone');//获得元素节点
    //(一)方式操作
    console.log(it.name);       //获得对应属性信息
    console.log(it.className);  //获得对应属性信息
    //① 获得属性节点使用class
    //② 获得属性的值使用className
    console.log(it.type);   //获得对应属性信息
    console.log(it.value);  //获得对应属性信息
    console.log(it.id);     //获得对应属性信息
    console.log(it.animal); //不允许获得undefined

    it.id="usertwo";
    it.name="useremail";
    it.type="radio";   //不推荐修改
    it.animal="cat";   //不允许修改

    //(二)方式操作
    console.log(it.getAttribute('animal')); //dog
    it.setAttribute('animal','wolf');       //可以修改
</script>
```

### [<span id="2.3">获取文本节点
```
<body>
    <h2>获得文本节点</h2>
</body>
<script type="text/javascript">
    var h2 = document.getElementsByTagName('h2')[0];//获得元素节点
    console.log(h2.nodeType);                       //1 元素节点
    console.log(h2.firstChild);                     //获得文本节点
    console.log(h2.firstChild.nodeType);            //3 文本节点
</script>
```

### [<span id="2.4">创建节点
元素节点：createElement(tag标签名称div/p/input);
文本节点：createTextNode(文本信息);
属性信息：setAttribute(名称，值);
节点追加：父节点.appendChild(子节点);

```
<script type="text/javascript">
/**
    <ul>
        <li id="wang">wangxiong</li>
    </ul>
*/
var ull = document.createElement('ul');
var lii = document.createElement('li');
var litxt = document.createTextNode('wangxiong');
lii.setAttribute('id','wang'); //属性设置
lii.appendChild(litxt); //追加
ull.appendChild(lii);
//document.getElementsByTagName('body')[0].appendChild(ull);
document.body.appendChild(ull);
</script>
```

### [<span id="2.5">修改节点
父节点.replaceChild(newnode,oldnode)
节点替换操作，通过父节点进行替换,替换节点要发生物理位置移动。
```
<body>
    <ul>
        <li>刘备</li>
        <li id="zhang">张飞</li>
        <li>关羽</li>
    </ul>
    <ul>
        <li>曹操</li>
        <li id="si">司马懿</li>
        <li>典韦</li>
    </ul>
</body>
<script type="text/javascript">
    //"已有节点" <li id="si">司马懿</li>进行替换<li id="zhang">张飞</li>
    var zhang = document.getElementById('zhang');
    var si = document.getElementById('si');
    zhang.parentNode.replaceChild(si,zhang);
</script>
```

### [<span id="2.6">删除节点
父节点.removeChild(node)
```
<body>
    <ul>
        <li>刘备</li>
        <li id="zhang">张飞</li>
        <li>关羽</li>
    </ul>
</body>
<script type="text/javascript">
    var zhang = document.getElementById('zhang');
    zhang.parentNode.removeChild(zhang);
    //document.body.parentNode.removeChild(document.body);
    //document.removeChild(document.firstChild);
</script>
```

### [<span id="2.7">复制节点
父节点.appendChild(node)

```
<body>
    <ul>
        <li>刘备</li>
        <li id="zhang">张飞</li>
        <li>关羽</li>
    </ul>
</body>
<script type="text/javascript">
    var zhang = document.getElementById('zhang');
    var kzhang = zhang.cloneNode(true);   // 深层复制
    var kzhang = zhang.cloneNode(false);  // 浅层复制
    zhang.parentNode.appendChild(kzhang);
</script>
```

### [<span id="2.8">操作样式
获取样式信息：node.style.样式名称
设置样式信息：node.style.样式名称 = 值
```
<body>
    <div style="width:300px; height:200px; background-color:pink;"></div>
</body>
<script type="text/javascript">
var dv = document.getElementsByTagName('div')[0];
console.log(dv.style.width);           // 获取宽度 300px
console.log(dv.style.height);          // 获取高度 200px
console.log(dv.style.backgroundColor); // 获取背景 pink
dv.style.backgroundColor='lightblue';  // 设置背景
dv.style.width='500px';                // 设置宽度
</script>
```

## [<span id="3">事件驱动
定义：对鼠标或键盘所做的动作就事件，对事件的处理称为事件驱动，事件驱动一般都由函数担任。
例如：onclick  onmouseover  onmouseout  onfocus  onblur  onsubmit等。

### [<span id="3.1">事件设置
### [<span id="3.2">事件流
### [<span id="3.3">事件对象
### [<span id="3.4">事件加载




