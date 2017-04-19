---
title: 理解java反射机制
date: 2017-04-19 17:20:15
tags: [Java,Java基础]
categories: Java
---

## 一、什么是反射？

反射(Reflection)是Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。
Oracle官方对反射的解释是

> Reflection enables Java code to discover information about the fields, methods and constructors of loaded classes, and to use reflected fields, methods, and constructors to operate on their underlying counterparts, within security restrictions.
The API accommodates applications that need access to either the public members of a target object (based on its runtime class) or the members declared by a given class. It also allows programs to suppress default reflective access control.

 简而言之，通过反射，我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息。
程序中一般的对象的类型都是在编译期就确定下来的，而Java反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。
 反射的核心是JVM在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。

Java反射框架主要提供以下功能：

1. 在运行时判断任意一个对象所属的类；
2. 在运行时构造任意一个类的对象；
3. 在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）；
4. 在运行时调用任意一个对象的方法
**重点：是运行时而不是编译时**

## 二、反射的用途
很多人都认为反射在实际的Java开发应用中并不广泛，其实不然。
 当我们在使用IDE(如Eclipse，IDEA)时，当我们输入一个对象或类并想调用它的属性或方法时，一按点号，编译器就会自动列出它的属性或方法，这里就会用到反射。
 反射最重要的用途就是开发各种通用框架。
 很多框架（比如Spring）都是配置化的（比如通过XML文件配置JavaBean,Action之类的），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象。
 举一个例子，在运用Struts 2框架的开发中我们一般会在struts.xml里去配置Action，比如：

{% codeblock lang:xml %}
<action name="login"
        class="org.ScZyhSoft.test.action.SimpleLoginAction"
        method="execute">
    <result>/shop/shop-index.jsp</result>
    <result name="error">login.jsp</result>
</action>
{% endcodeblock %}
配置文件与Action建立了一种映射关系，当View层发出请求时，请求会被StrutsPrepareAndExecuteFilter拦截，然后StrutsPrepareAndExecuteFilter会去动态地创建Action实例。
——比如我们请求login.action，那么StrutsPrepareAndExecuteFilter就会去解析struts.xml文件，检索action中name为login的Action，并根据class属性创建SimpleLoginAction实例，并用invoke方法来调用execute方法，这个过程离不开反射。
对与框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心。而对于一般的开发者来说，不深入框架开发则用反射用的就会少一点，不过了解一下框架的底层机制有助于丰富自己的编程思想，也是很有益的。

## 三、反射的基本运用