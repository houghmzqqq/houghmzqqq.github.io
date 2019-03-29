---
title: 浅析String、StringBuffer、StringBuider的区别
date: 2017-07-08 20:48:38
tags: 
	- Java
---
# 1 String浅析
## 1.1 String对象的创建
<p> 
&nbsp;&nbsp;&nbsp;&nbsp;我们先来了解一下String对象的创建过程。
&nbsp;&nbsp;&nbsp;&nbsp;要创建String对象，很普遍的一种方法是利用构造器 `String str = new String("Hello World")`，问题是这里的参数"Hello World"是什么东西？也是一个String对象？
<!-- more -->
&nbsp;&nbsp;&nbsp;&nbsp;还有一种常用的创建方式，`String str = "Hello World"`,与创建基本数据类型相似。

&nbsp;&nbsp;&nbsp;&nbsp;我们知道，Java程序运行之前，编译器会先将源代码文件编译成Class文件，然后再由JVM继续执行，在class字节流中有一个常量池，用于放置源代码中的符号信息(并且不同的符号信息放置在不同标志的常量表中)，其中有四个不同类型的常量表。__上述"Hello World"字符串在编译成Class文件时，被存放进常量池中用于存放字符串的常量表中，JVM加载Class文件时，会将该常量表中的字符串取出&nbsp;并在堆中创建新的String对象（intern字符串对象，又称拘留字符串对象）。__
&nbsp;&nbsp;&nbsp;&nbsp;简单来说，就是在运行阶段&nbsp;`String str == "Hello World"` &nbsp;中的字符串"Hello World"被转换成&nbsp;__拘留字符串__&nbsp;对象被压入&nbsp;__堆__&nbsp;中
 </p>

参照事例1：

```
//代码1
String s1 = new String("Hello");
String s2 = new String("Hello");
System.out.println(s1==s2);
//代码2
String s3 = "Hello";
String s4 = "Hello";
System.out.println(s3==s4);
```
输出为：

false 

true

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;执行代码1时，jvm先为字面值“Hello”创建了一个拘留字符串对象，再将拘留字符串的值赋予给new的对象，所以这里的s1、s2存放的是new出来的对象的地址；执行2代码时，s3、s4存储的时拘留字符串的地址。

参照事例2：

```
//代码1
String s1 = "ab";
String s2 = "cd";
String s12 = s1 + s2;
String s = "abcd";
System.out.println(s12==s);
//代码2
String s1 = "ab" + "cd";
String s2 = "abcd";
System.out.println(s1==s2);
```
输出为：

false

true

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;执行上述代码1时，JVM会在堆中先创建一个以s1所指向的拘留字符串对象完成初始化的StringBuilder对象，并使用append方法对s2进行拼接，然后用toString()方法在堆中创建一个String对象，所以s12、s指向两个不同的地址；执行代码2时，“ab”+"cd"会在编译器就合并成“abcd”，因此它指向对应的拘留字符串。


# StringBuffer和StringBuilder
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StringBuffer和StringBuilder功能一样，都是用于字符串缓冲，但是StringBuffer是线程安全的，StringBuilder是线程不安全的。
由于StringBuilder所有方面都没有被synchronized修饰，因此它的效率要比StringBuffer高。


