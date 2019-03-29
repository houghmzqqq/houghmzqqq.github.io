---
title: java多态
date: 2017-12-18 11:26:23
tags:

	- Java
---



## 什么是多态

面向对象的三大特性：

​	1.封装：通常认为封装是把数据和操作数据的方法绑定起来，对数据的访问只能通过已定义的接口。

​	2.继承：一个类可以继承另一个类，子类可以访问父类的属性和方法。

​	3.多态：多态性是指允许不同子类型的对象对同一消息作出不同的响应。简单的说就是用同样的对象引用调用同样的方法但是做了不同的事情。



举个栗子：

​	有一个Person类，它封装了walk()和talk()两个行为。有一个Teacher类继承了Person，Teacher也能够walk()和talk()，在此基础上他还能够teach()。

​	现在有~Person  per  =  new Teacher();~，它表示Person的引用指向Teacher对象，per能够使用Person中的方法但不能使用Student中独有的teach(),能够访问Person中的__非私有(private)属性__，如果Student中重写了Person中的walk()方法，则per.walk()实际上是使用了重写后的walk()。

​	总的来说，java中的多态有三个条件：继承，重写，父类引用指向子类对象



<!-- more -->

第二个栗子：

~~~java
class A ...{  
         public String show(D obj)...{  
                return ("A and D");  
         }   
         public String show(A obj)...{  
                return ("A and A");  
         }   
}   
class B extends A...{  
         public String show(B obj)...{  
                return ("B and B");  
         }  
         public String show(A obj)...{  
                return ("B and A");  
         }   
}  
class C extends B...{}   
class D extends B...{}  
~~~



~~~java
A a1 = new A();  
A a2 = new B();  
B b = new B();  
C c = new C();   
D d = new D();  

System.out.println(a1.show(b));   1
System.out.println(a1.show(c));   2
System.out.println(a1.show(d));   3
System.out.println(a2.show(b));   4
System.out.println(a2.show(c));   5
System.out.println(a2.show(d));   6
System.out.println(b.show(b));    7
System.out.println(b.show(c));    8
System.out.println(b.show(d));    9
~~~

结果：

~~~
1   A and A
2   A and A
3   A and D
4   B and A
5   B and A
6   A and D
7   B and B
8   B and B
9   A and D
~~~

