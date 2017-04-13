---
title: 谈谈javascript语法里一些难点问题（一）
date: 2017-04-12 16:42:40
tags: JavaScript
categories: JavaScript
---

> 转载自伯乐在线http://blog.jobbole.com/81010/

## 1)   先看这样一段代码
{% codeblock lang:javascript %}
var a = 2;
function test(){
    alert(a);
    var a = 3;
    alert(a);
}
{% endcodeblock %}
    
执行结果为：
//undefined 
//3

这是一个令人诧异的结果，为什么第一个弹出框显示的是undefined，而不是1呢？这种疑惑的原理我描述如下：

一个页面里直接定义在script标签下的变量是全局变量即属于window对象的变量，按照javascript作用域链的原理，当一个变量在当前作用域下找不到该变量的定义，那么javascript引擎就会沿着作用域链往上找直到在全局作用域里查找，按上面的代码所示，虽然函数内部重新定义了变量的值，但是内部定义之前函数使用了该变量，那么按照作用域链的原理在函数内部变量定义之前使用该变量，javascript引擎应该会在全局作用域里找到变量定义，而实际情况却是变量未定义，这到底是怎么回事呢？

当时群里很多人都给出了问题的解答，我也给出了我自己的解答，其实这个问题很久之前我的确研究过，但是刚被问起了我居然还是有个卡壳期，在加上最近研究javascriptMVC的写法，发现自己读代码时候对new 、prototype、apply以及call的用法任然要体味半天，所以我觉得有必要对javascript基础语法里比较难理解的问题做个梳理，其实写博客的一个很大的好处就是写出来的知识逻辑会比你在脑子里反复梳理的逻辑映像更加的深刻。

下面开始本文的主要内容，我会从基础知识一步步讲起。

## 2)   JavaScript变量
Java语言里有一句很经典的话：**在java的世界里，一切皆是对象**。

Javascript虽然跟java没有半点毛关系，但是很多会使用javascript的朋友同样认为：**在javascript的世界里，一切也皆是对象**。

其实javascript语言和java语言一样变量是分为两种类型：基本数据类型和引用类型。

基本类型是指：Undefined、Null、Boolean、Number和String；而引用类型是指多个指构成的对象，所以javascript的对象指的是引用类型。在java里能说一切是对象，是因为java语言里对所有基本类型都做了对象封装，而这点在javascript语言里也是一样的，所以提在javascript世界里一切皆为对象也不为过。

但是实际开发里如果我们对基本类型和引用类型的区别不是很清晰，就会碰到我们很多不能理解的问题，下面我们来看看下面的代码：

{% codeblock lang:javascript %}
var str = "sharpxiajun";
 
str.attr01 = "hello world";
 
console.log(str);//  运行结果：sharpxiajun
 
console.log(str.attr01);// 运行结果：undefined
{% endcodeblock %}

运行之，我们发现作为基本数据类型，我们没法为这个变量添加属性，当然方法也同样不可以，例如下面的代码：

{% codeblock lang:javascript %}
var str = "sharpxiajun";
 
str.attr01 = "hello world";
 
console.log(str);//  运行结果：sharpxiajun
 
console.log(str.attr01);// 运行结果：undefined
{% endcodeblock %}

运行之，结果如下图所示：

![1](谈谈javascript语法里一些难点问题（一）/1.jpg)

当我们使用引用类型时候，结果就和上面完全不同了，大家请看下面的代码：

{% codeblock lang:javascript %}
var obj1 = new Object();
 
obj1.name = "obj1 name";
 
console.log(obj1.name);// 运行结果：obj1 name
{% endcodeblock %}

javascript里的基本类型和引用类型的区别和其他语言类似，这是一个老调长谈的问题，但是在现实中很多人都理解它，但是却很难应用它去理解问题。

Javascript里的基本变量是存放在栈区的（栈区指内存里的栈内存），它的存储结构如下图所示：

![2](谈谈javascript语法里一些难点问题（一）/2.jpg)
![3](谈谈javascript语法里一些难点问题（一）/3.jpg)

javascript里引用变量的存储就比基本类型存储要复杂多，引用类型的存储需要内存的栈区和堆区（堆区是指内存里的堆内存）共同完成，如下图所示：

在javascript里变量的存储包含三个部分：

**部分一：栈区的变量标示符；**

**部分二：栈区变量的值；**

**部分三：堆区存储的对象。**

变量不同的定义，这三个部分也会随之发生变化，下面我来列举一些典型的场景：

**场景一：如下代码所示：**

{% codeblock lang:javascript %}
var qqq;

console.log(qqq);// 运行结果：undefined
{% endcodeblock %}

运行结果是undefined，上面的代码的标准解释就是变量被命名了，但是还未初始化，此时在变量存储的内存里只拥有栈区的变量标示符而没有栈区的变量值，当然更没有堆区存储的对象。

**场景二：如下代码所示：**

{% codeblock lang:javascript %}
var qqq;

console.log(qqq);// 运行结果：undefined

console.log(xxx);// 运行结果：xxx is not defined
{% endcodeblock %}

会提示变量未定义。在任何语言里变量未定义就使用都是违法的，我们看到javascript里也是如此，但是我们做javascript开发时候，经常有人会说变量未定义也是可以使用，怎么我的例子里却不能使用了？那么我们看看下面的代码：

{% codeblock lang:javascript %}
xxx = "outer xxx";

console.log(xxx);// 运行结果：outer xxx

function testFtn(){

     sss = "inner sss";

     console.log(sss);// 运行结果：outer sss

}

testFtn();

console.log(sss);//运行结果：inner sss

console.log(window.sss);//运行结果：inner sss
{% endcodeblock %}

**由这两个场景我们可以知道在javascript里的变量不能正常使用即报出“xxx is not defined”错误（这个错误下，后续的javascript代码将不能正常运行）只有当这个变量既没有被var定义同时也没有进行赋值操作才会发生，而只有赋值操作的变量不管这个变量在那个作用域里进行的赋值，这个变量最终都是属于全局变量即window对象。**

由上面我列举的两个场景我们来理解下引子里网友提出的问题，下面我修改一下代码，如下所示：

{% codeblock lang:javascript %}
function hehe(){

     console.log(a); //unfined

     var a = 2;

     console.log(a); //2

}
hehe();
{% endcodeblock %}

{% codeblock lang:javascript %}
function hehe(){
 
    console.log(a); //a is not defined
    console.log(a);
 
}
hehe();
{% endcodeblock %}

对比二者代码以及引子里的代码，我们发现问题的关键是var a=2所引起的。在代码一里我注释了全局变量的定义，结果和引子里代码的结果一致，这说明函数内部a变量的使用和全局环境是无关的，代码二里我注释了关键代码var a = 2，代码运行结果发生了变化，程序报错了，的确很让人困惑，困惑之处在于局部作用域里变量定义的位置在变量第一次使用之后，但是程序没有报错，这不符合javascript变量未定义既要报错的原理。

其实这个变量任然被定义即内存存储里有了标示符，只不过没有被赋值，代码一则说明，内部变量a已经和外部环境无关，怎么回事？如果我们按照代码运行是按照顺序执行的逻辑来理解，这个代码也就没法理解。

其实javascript里的变量和其他语言有很大的不同，javascript的变量是一个松散的类型，松散类型变量的特点是变量定义时候不需要指定变量的类型，变量在运行时候可以随便改变数据的类型，但是这种特性并不代表javascript变量没有类型，当变量类型被确定后javascript的变量也是有类型的。但是在现实中，很多程序员把javascript松散类型理解为了javascript变量是可以随意定义即你可以不用var定义，也可以使用var定义，其实在javascript语言里变量定义没有使用var，变量必须有赋值操作，只有赋值操作的变量是赋予给window，这其实是javascript语言设计者提升javascript安全性的一个做法。

此外javascript语言的松散类型的特点以及运行时候随时更改变量类型的特点，很多程序员会认为javascript变量的定义是在运行期进行的，更有甚者有些人认为javascript代码只有运行期，其实这种理解是错误的，**javascript代码在运行前还有一个过程就是：预加载，预加载的目的是要事先构造运行环境例如全局环境，函数运行环境，还要构造作用域链（关于作用域链和环境，本文后续会做详细的讲解），而环境和作用域的构造的核心内容就是指定好变量属于哪个范畴，因此在javascript语言里变量的定义是在预加载完成而非在运行时期。**

所以，引子里的代码在函数的局部作用域下变量a被重新定义了，在预加载时候a的作用域范围也就被框定了，a变量不再属于全局变量，而是属于函数作用域，只不过赋值操作是在运行期执行（这就是为什么javascript语言在运行时候会改变变量的类型，因为赋值操作是在运行期进行的），所以第一次使用a变量时候，a变量在局部作用域里没有被赋值，只有栈区的标示名称，因此结果就是undefined了。

不过赋值操作也不是完全不对预加载产生影响，预加载时候javascript引擎会扫描所有代码，但不会运行它，当预加载扫描到了赋值操作，但是赋值操作的变量有没有被var定义，那么该变量就会被赋予全局变量即window对象。

根据上面的内容我们还可以理解下javascript两个特别的类型：undefined和null，从javascript变量存储的三部分角度思考，当变量的值为undefined时候，那么该变量只有栈区的标示符，如果我们对undefined的变量进行赋值操作，如果值是基本类型，那么栈区的值就有值了，如果栈区是对象那么堆区会有一个对象，而栈区的值则是堆区对象的地址，如果变量值是null的话，我们很自然认为这个变量是对象，而且是个空对象，按照我前面讲到的变量存储的三部分考虑：**当变量为null时候，栈区的标示符和值都会有值，堆区应该也有，只不过堆区是个空对象，这么说来null其实比undefined更耗内存了**，那么我们看看下面的代码：

{% codeblock lang:javascript %}
var ooo = null;

console.log(ooo);// 运行结果：null

console.log(ooo == undefined);// 运行结果：true

console.log(ooo == null);// 运行结果：true

 console.log(ooo === undefined);// 运行结果：false

console.log(ooo === null);// 运行结果：true
{% endcodeblock %}

运行之，结果很震惊啊，null居然可以和undefined相等，但是使用更加精确的三等号“===”，发现二者还是有点不同，其实javascript里undefined类型源自于null即null是undefined的父类，本质上null和undefined除了名字这个马甲不同，其他都是一样的，不过要让一个变量是null时候必须使用等号“=”进行赋值了。

当变量为undefined和null时候我们如果滥用它javascript语言可能就会报错，后续代码会无法正常运行，所以javascript开发规范里要求变量定义时候最好马上赋值，赋值好处就是我们后面不管怎么使用该变量，程序都很难因为变量未定义而报错从而终止程序的运行，例如上文里就算变量是string基本类型，在变量定义属性程序还是不会报错，这是提升程序健壮性的一个重要手段，由引子的例子我们还知道，变量定义最好放在变量所述作用域的最前端，这么做也是保证代码健壮性的一个重要手段。

下面我们再看一段代码：

{% codeblock lang:javascript %}
var str;

if (undefined != str && null != str && "" != str){

    console.log("true");

}else{

    console.log("false");

}

if (undefined != str && "" != str){

    console.log("true");

}else{

    console.log("false");

}

if (null != str && "" != str){

    console.log("true");

}else{

    console.log("false");

}

if (!!str){

    console.log("true");

}else{

    console.log("false");

}

str = "";

if (!!str){

    console.log("true");

}else{

    console.log("false");

}
{% endcodeblock %}

运行之，结果都是打印出false。

使用双等号“==”，undefined和null是一回事，所以第一个if语句的写法完全多余，增加了不少代码量，而第二种和第三种写法是等价，究其本质前三种写法本质都是一致的，但是现实中很多程序员会选用写法一，原因就是他们还没理解undefined和null的不同，第四种写法是更加完美的写法，在javascript里如果if语句的条件是undefined和null，那么if判断的结果就是false，使用！运算符if计算结果就是true了，再加一个就是false，所以这里我建议在书写javascript代码时候判断代码是否为未定义和null时候最好使用！运算符。

代码四里我们看到当字符串被赋值了，但是赋值是个空字符串时候，if的条件判断也是false，javascript里有五种基本类型，undefined、null、boolean、Number和string，现在我们发现除了Number都可以使用！来判断if的ture和false，那么基本类型Number呢？

{% codeblock lang:javascript %}
var num = 0;

if (!!num){

        console.log("true");

}else{

        console.log("false");

}
{% endcodeblock %}

运行之，结果是false。

如果我们把num改为负数或正数，那么运行之的结果就是true了。

这说明了一个道理：我们定义变量初始化值的时候，如果基本类型是string，我们赋值空字符串，如果基本类型是number我们赋值为0，这样使用if语句我们就可以判断该变量是否是被使用过了。

但是当变量是对象时候，结果却不一样了，如下代码：

{% codeblock lang:javascript %}
var obj = {};

if (!!obj){

        console.log("true");

}else{

        console.log("false");

}
{% endcodeblock %}

运行之，代码是true。

所以在定义对象变量时候，初始化时候我们要给变量赋予null，这样if语句就可以判断变量是否初始化过。

其实if加上！运算判断对象的现象还有玄机，这个玄机要等我把场景三讲完才能说清楚哦。

**场景三：复制变量的值和函数传递参数**

代码1

{% codeblock lang:javascript %}
var s1 = "sharpxiajun";

var s2 = s1;

console.log(s1);//// 运行结果：sharpxiajun

console.log(s2);//// 运行结果：sharpxiajun

s2 = "xtq";

console.log(s1);//// 运行结果：sharpxiajun

console.log(s2);//// 运行结果：xtq
{% endcodeblock %}

代码2

{% codeblock lang:javascript %}
var obj1 = new Object();
 
obj1.name = "obj1 name";
 
console.log(obj1.name);// 运行结果：obj1 name
 
var obj2 = obj1;
 
console.log(obj2.name);// 运行结果：obj1 name
 
obj1.name = "sharpxiajun";
 
console.log(obj2.name);// 运行结果：sharpxiajun
{% endcodeblock %}

我们发现当复制的是对象，那么obj1和obj2两个对象被串联起来了，obj1变量里的属性被改变时候，obj2的属性也被修改。

函数传递参数的本质就是外部的变量复制到函数参数的变量里，我们看看下面的代码

{% codeblock lang:javascript %}
function testFtn(sNm,pObj){
 
        console.log(sNm);// 运行结果：new Name
 
        console.log(pObj.oName);// 运行结果：new obj
 
        sNm = "change name";
 
        pObj.oName = "change obj";
 
}
 
var sNm = "new Name";
 
var pObj = {oName:"new obj"};
 
testFtn(sNm,pObj);
 
console.log(sNm);// 运行结果：new Name
 
console.log(pObj.oName);// 运行结果：change obj
{% endcodeblock %}

这个结果和变量赋值的结果是一致的。

**在javascript里传递参数是按值传递的。**

上面函数传参的问题是很多公司都爱面试的问题，其实很多人都不知道javascript传参的本质是怎样的，如果把上面传参的例子改的复杂点，很多朋友都会栽倒到这个面试题下。

为了说明这个问题的原理，就得把上面讲到的变量存储原理综合运用了，这里我把前文的内容再复述一遍，两张图，如下所示：

![2](谈谈javascript语法里一些难点问题（一）/2.jpg)
![3](谈谈javascript语法里一些难点问题（一）/3.jpg)

这是引用类型存储的内存结构。

还有个知识,如下：

在javascript里变量的存储包含三个部分：

**部分一：栈区的变量标示符；**

**部分二：栈区变量的值；**

**部分三：堆区存储的对象。**

在javascript里变量的复制（函数传参也是变量赋值）本质是传值，这个值就是栈区的值，而基本类型的内容是存放在栈区的值里，所以复制基本变量后，两个变量是独立的互不影响，但是当复制的是引用类型时候，复制操作还是复制栈区的值，但是这个时候值是堆区对象的地址，因为javascript语言是不允许操作堆内存，因此堆内存的变量并没有被复制，所以复制引用对象复制的值就是堆内存的地址，而复制双方的两个变量使用的对象是相同的，因此复制的变量其中一个修改了对象，另一个变量也会受到影响。

原理讲完了，下面我列举一个拔高的例子，代码如下：

{% codeblock lang:javascript %}
var ftn1 = function(){
 
        console.log("test:ftn1");
 
    };
 
var ftn2 = function(){
 
        console.log("test:ftn2");
 
};
 
function ftn(f){
 
       f();
 
       f = ftn2;
 
}
 
ftn(ftn1);// 运行结果：test:ftn1
 
console.log("====================华丽的分割线======================");
 
ftn1();// 运行结果：test:ftn1
{% endcodeblock %}

这个代码是很早之前有位朋友考我的，我当时答对了，但是我是蒙的，问我的朋友答错了，其实当时我们两个都没搞懂其中缘由，我朋友是这么分析的他认为f是函数的参数，属于函数的局部作用域，因此更改f的值，是没法改变ftn1的值，因为到了外部作用域f就失效了，但是这种解释很难说明我上文里给出的函数传参的实例，其实这个问题答案就是函数传参的原理，只不过这里加入了个混淆因素函数，在javascript函数也是对象，局部作用域里f = ftn2操作是将f在栈区的地址改为了ftn2的地址，对外部的ftn1和ftn2没有任何改变。

**记住：javascript里变量复制和函数传参都是在传递栈区的值。**

栈区的值除了变量复制起作用，它在if语句里也会起到作用，当栈区的值为undefined、null、“”（空字符串）、0、false时候，if的条件判断则是为false，我们可以通过！运算符计算，因此当我们的代码如下：

{% codeblock lang:javascript %}
var obj = {};
 
    if (!!obj){
 
        console.log("true");
 
    }else{
 
        console.log("false");
 
}
{% endcodeblock %}

结果则是true，因为var obj = {}相当于var obj = new Object(),虽然对象里没什么内容，但是在堆区里，对象的内存已经分配了，而变量栈区的值已经是内存地址了，所以if语句判断就是true了。