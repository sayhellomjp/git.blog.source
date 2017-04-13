---
title: 谈谈javascript语法里一些难点问题（二）
date: 2017-04-13 10:44:21
tags: JavaScript
categories: JavaScript
---

> 转载自伯乐在线http://blog.jobbole.com/81011/

## 3)    作用域链相关的问题

作用域链是javascript语言里非常红的概念，很多学习和使用javascript语言的程序员都知道作用域链是理解javascript里很重要的一些概念的关键，这些概念包括this指针，闭包等等，它非常红的另一个重要原因就是作用域链理解起来太难，就算有人真的感觉理解了它，但是碰到很多实际问题时候任然会是丈二和尚摸不到头脑，例如上篇引子里讲到的例子，本篇要讲的主题就是作用域链，再无别的内容，希望看完本文的朋友能有所收获。

讲作用域链首先要从作用域讲起，下面是百度百科里对作用域的定义：

> 作用域在许多程序设计语言中非常重要。 通常来说，一段程序代码中所用到的名字并不总是有效/可用的，而限定这个名字的可用性的代码范围就是这个名字的作用域。 作用域的使用提高了程序逻辑的局部性，增强程序的可靠性，减少名字冲突。

在我最擅长的服务端语言java里也有作用域的概念，java里作用域是以{}作为边界，不过在纯种的面向对象语言里我们没必要把作用域研究的那么深，也没必要思考复杂的作用域嵌套问题，因为这些语言关于作用域的深度运用并不会给我们编写的代码带来多大好处。但是在javascript里却大不相同，如果我们不能很好的理解javascript的作用域我们就没办法使用javascript编写出复杂的或者规模宏大的程序。

由百度百科里的定义，我们知道作用域的作用是保证变量的名字不发生冲突，用现实的场景来理解有个人叫做张三，张三虽然只是一个名字，但是认识张三的人根据名字就能唯一确认这个人到底是谁，但是这个世界上叫做张三的人可不止一个，特别是两个叫张三的人有交集的时候我们就要有个办法明确指定这个张三绝不是另外一个张三，这时我们可能会根据两大张三年龄的差异来区分：例如一个张三叫大张三，相对的另外一个张三叫小张三了。编程语言里的作用域其实就是为了做类似的标记，作用域会设定一个范围，在这个范围里我们是不会弄错变量的真实含义。

前面我讲到在java里通过{}来设置作用域，在{}里面的变量会得到保护，这种保护就是不让{}里的变量被外部变量混淆和污染。那么{}的方式适合于javascript吗？我们看看下面的例子：

{% codeblock lang:javascript %}
var s1 = "sharpxiajun";

function ftn(){

    var s2 = "xtq";

    console.log(this);// 运行结果： window

    console.log("s1:" + this.s1 + ";s2:" + this.s2);//运行结果：s1:sharpxiajun;s2:undefined

    console.log("s1:" + this.s1 + ";s2:" + s2);// 运行结果：s1:sharpxiajun;s2:xtq

}

ftn();
{% endcodeblock %}

在javascript世界里有一个大的作用域环境，这个环境就是window，window环境不需要我们自己使用什么方式构建，页面加载时候页面会自动构造的，上面代码里有一个大括号,这个大括号是对函数的定义，运行之，我们发现函数作用域内部定义的s2变量是不能被window对象访问的，因此s2变量是被{}保护起来了，它的生命周期和这个函数的生命周期有关。

由这个例子是不是说明在javascript里，变量也是被{}保护起来了，在javascript语言里还有非函数的{}，我们再看看下面的例子：

{% codeblock lang:javascript %}
if (true){

    var a = "aaaa";

}

console.log(a);// 运行结果：aaaa
{% endcodeblock %}

我们发现javascript里{}有时是起不到定义作用域的功能。这也说明javascript里的作用域定义是和其他语言例如java不同的。

在javascript里作用域有一个专门的定义execution context，有的书里把这个名字翻译成执行上下文，有的书籍里把它翻译成执行环境，我更倾向于后者执行环境，下文我提到的执行环境就是execution context。这个命名非常形象，这个形象体现在execution这个单词，execution含义就是执行，我们来想想javascript里那些情况是执行：

**情况一：当页面加载时候在script标签下的javascript代码会按顺序执行，而这些能被执行的代码都是属于window的变量或函数；**

**情况二：当函数的名字后面加上小括号()，例如ftn()，这也是在执行，不过它执行的是函数。**

**如此说来，javascript里的执行环境有两类一类是全局执行环境，即window代表的全局环境，一类是函数代表的函数执行环境，这也就是我们常说的局部作用域。**

执行环境在javascript语言里并非是一个抽象的概念，而是有具体的实现，这个实现其实是个对象，这个对象也有个名字叫做variable object，这个变量有的书里翻译为变量对象，这是直译，有的书里把它称为上下文变量，这里我还是倾向于后者上下文变量，下文里提到的上下文变量就是指代variable object。上下文变量存储的是上下文变量所处执行环境里定义的所有的变量和函数。

全局执行环境的上下文变量是可以访问到的，它就是window对象，所以我们说window能代表全局作用域是有道理的，但是局部作用域即函数的执行环境里的上下文变量是代码不能访问到的，不过javascript引擎在处理数据时候会使用到它。

**在javascript语言里还有一个概念，它的名字叫做execution context stack，翻译成中文就是执行环境栈，每个要被执行的函数都会先把函数的执行环境压入到执行环境栈里，函数执行完毕后，这个函数的执行环境就会被执行环境栈弹出，例如上面的例子：函数执行时候函数的执行环境会被压入到执行环境栈里，函数执行完毕，执行环境栈会把这个环境弹出，执行环境栈的控制权就会交由全局环境，如果函数后面还有代码，那么代码就是接着执行。如果函数里嵌套了函数，那么嵌套函数执行完毕后，执行环境栈的控制权就交由了外部函数，然后依次类推，最后就是全局执行环境了。**

讲到这里我们大名鼎鼎的作用域链要登场了，函数的执行环境被压入到执行环境栈里后，函数就要执行了，函数执行的第一步不是执行函数里的第一行代码而是在上下文变量里构造一个作用域链，作用域链的英文名字叫做**scope chain**，作用域链的作用是保证执行环境里有权访问的变量和函数是有序的，这个概念里有两个关键意思：有权访问和有序，我们看看下面的代码：

{% codeblock lang:javascript %}
var b1 = "b1";

function ftn1(){

    var b2 = "b2";

    var b1 = "bbb";

    function ftn2(){

        var b3 = "b3";

        b2 = b1;

        b1 = b3;

        console.log("b1:" + b1 + ";b2:" + b2 + ";b3:" + b3);// 运行结果：b1:b3;b2:bbb;b3:b3

    }

    ftn2();

}

ftn1();

console.log(b1);// 运行结果：b1
{% endcodeblock %}

有这个例子我们发现，ftn2函数可以访问变量b1，b2，这个体现了有权访问的概念，当ftn1作用域里改变了b1的值并且把b1变量重新定义为ftn1的局部变量，那么ftn2访问到的b1就是ftn1的，ftn2访问到b1后就不会在全局作用域里查找b1了，这个体现了有序性。

下面我要总结下上面讲述的知识：

**本篇的小标题是：作用域链的相关问题，这个标题定义的含义是指作用域链是大名鼎鼎了，但是作用域链在广大程序员的理解里其实包含的意义已经超越了作用域链在javascript语言本身的定义。广大程序员对作用域链的理解有两块一块是作用域，而作用域在javascript语言里指的是执行环境execution context，执行环境在javascript引擎里是通过上下文变量体现的variable object，javascript引擎里还有一个概念就是执行环境栈execution context stack，当某一个函数的执行环境压入到了执行环境栈里，这个时候就会在上下文变量里构造一个对象，这个对象就是作用域链scope chain，而这个作用域链就是广大程序员理解的第二块知识，作用域链的作用是保证执行环境里有权访问的变量和函数是有序的，作用域链的变量只能向上访问，变量访问到window对象即被终止，作用域链向下访问变量是不被允许的。**

很多人常常认为作用域链是理解this指针的关键，这个理解是不正确的的，this指针构造是和作用域链同时发生的，也就是说在上文变量构建作用域链的同时还会构造一个this对象，this对象也是属于上下文变量，而this变量的值就是当前执行环境外部的上下文变量的一份拷贝，这个拷贝里是没有作用域链变量的，例如代码：

{% codeblock lang:javascript %}
var b1 = "b1";

function ftn1(){

    console.log(this);// 运行结果： window

    var b2 = "b2";

    var b1 = "bbb";

    function ftn2(){

        console.log(this);// 运行结果： window

        var b3 = "b3";

        b2 = b1;

        b1 = b3;

        console.log("b1:" + b1 + ";b2:" + b2 + ";b3:" + b3);// 运行结果：b1:b3;b2:bbb;b3:b3

    }

    ftn2();

}

ftn1();
{% endcodeblock %}

我们看到函数ftn1和ftn2里的this指针都是指向window，这是为什么了？因为在javascript我们定义函数方式是通过function xxx(){}形式，那么这个函数不管定义在哪里，它都属于全局对象window，所以他们的执行环境的外部的执行上下文都是指向window。

但是我们都知道现实代码很多this指针都不是指向window，例如下面的代码：

{% codeblock lang:javascript %}
var obj = {

    name:"sharpxiajun",

    ftn:function(){

        console.log(this);// 运行结果: Object { name="sharpxiajun", ftn=function()}

        console.log(this.name);//运行结果: sharpxiajun

    }

}

obj.ftn();// ：
{% endcodeblock %}

运行之，我们发现这里this指针指向了Object，这就怪了我前文不是说javascript里作用域只有两种类型：一个是全局的一个是函数，为什么这里Object也是可以制造出作用域了，那么我的理论是不是有问题啊？那我们看看下面的代码：

{% codeblock lang:javascript %}
var obj1 = new Object();

obj1.name = "xtq";

obj1.ftn = function(){

    console.log(this);// 运行结果: Object { name="xtq", ftn=function()}

    console.log(this.name);//运行结果: xtq

}

obj1.ftn();
{% endcodeblock %}

这两种写法是等价的，第一种对象的定义方法叫做字面量定义，而第二种写法则是标准写法，Object对象的本质也是个function，所以当我们调用对象里的函数时候，函数的外部执行环境就是obj1本身，即外部执行环境上下文变量代表的就是obj1，那么this指针也是指向了obj1。