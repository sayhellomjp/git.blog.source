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
