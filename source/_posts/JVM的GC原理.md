---
title: JVM的GC原理
date: 2020-03-11 15:39:34
tags:
	- JVM
	- GC
---

​	GC是Java虚拟机中至关重要的一部分，然而由于Java良好的垃圾自动回收机制，使我们平时开发过程中并没有什么感觉，我们不必像c、c++一样写一堆的析构函数，在日常的开发中不必为如何清理内存空间而烦恼。

### 什么是垃圾回收(GC)

​	垃圾回收（Garbage Collection，GC），顾名思义就是释放垃圾占用的空间，防止内存泄露。有效的使用可以使用的内存，对内存堆中已经死亡的或者长时间没有使用的对象进行清除和回收。

​	Java 语言出来之前，大家都在拼命的写 C 或者 C++ 的程序，而此时存在一个很大的矛盾，C++ 等语言创建对象要不断的去开辟空间，不用的时候又需要不断的去释放控件，既要写构造函数，又要写析构函数，很多时候都在重复的 allocated，然后不停的析构。于是，有人就提出，能不能写一段程序实现这块功能，每次创建，释放控件的时候复用这段代码，而无需重复的书写呢？

​	1960年，基于 MIT 的 Lisp 首先提出了垃圾回收的概念，而这时 Java 还没有出世呢！**所以实际上 GC 并不是Java的专利，GC 的历史远远大于 Java 的历史！**

<!-- more -->

### 怎样定义垃圾

​	既然要回收，我们首先要知道哪些东西是垃圾、是需要回收的，这里介绍两种标记垃圾的算法

#### 1.引用计数法

​	引用计数法是指，在对象头中新增一个标签，记录给对象被引用的次数，当对象被引用次数为零时，则标记该对象为可回收

~~~java
String m = new String("Jack");
m = null;
~~~

首先新建一个对象，这是m引用了"Jack"对象，所以"Jack"对象不会被标记回收

![引用计数法1](引用计数法1.png)	

将m设置为null后，"Jack"的引用次数为零，被标记为可回收对象

![引用计数法2](引用计数法2.png)

​	引用计数法虽然可以标记出可回收对象，但是却有一个很大的问题，它无法处理两个对象相互引用的情况

~~~java
public class ReferenceCountingGC {
    public Object instance;
    public ReferenceCountingGC(String name){}
}

public static void testGC(){
    ReferenceCountingGC a = new 		 ReferenceCountingGC("objA");
    ReferenceCountingGC b = new ReferenceCountingGC("objB");

    a.instance = b;
    b.instance = a;

    a = null;
    b = null;
}
~~~

> 简单描述上面代码：
>
> a.定义两个对象	
>
> b.两个对象相互引用	
>
> c.两个对象设置为null

![引用计数法3](引用计数法3.png)

​	最后由于a，b两个对象相互调用，它们的引用计数永远不会为零，这两个对象就成钉子户了，再也不会被GC回收。

#### 2.可达性分析法

​	可达性分析法的思路是，通过一些引用链的对象作为起点(GC Roots)，从这些起点向下搜索，搜索过的路径称为(Reference Chain)参考链，当一个对象无法链接到(GC Roots)时(该节点到GC Roots 不可达)，则这个对象为可回收对象

![可达性分析法1](可达性分析法1.png)

​	通过可达性算法，成功解决了引用计数所无法解决的问题-“循环依赖”，只要你无法与 GC Root 建立直接或间接的连接，系统就会判定你为可回收对象。那这样就引申出了另一个问题，哪些属于 GC Root。

在Java中，可以作为GC Root 的对象有一下四种：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中 JNI（即一般说的 Native 方法）引用的对象

###### 虚拟机栈(栈帧中的本地变量表)

~~~java
public class StackLocalParameter {
    public StackLocalParameter(String name){}
}

public static void testGC(){
    StackLocalParameter s = new StackLocalParameter("localParameter");
    s = null;
}
~~~

这里的s被压入了虚拟机栈中，可以被看做GC Root，当把s设置为null，"localParameter"对象断开了与GC Root 的引用链而被标记为可回收对象

###### 方法去中类静态属性应用的对象

~~~java
public class MethodAreaStaicProperties {
    public static MethodAreaStaicProperties m;
    public MethodAreaStaicProperties(String name){}
}

public static void testGC(){
    MethodAreaStaicProperties s = new MethodAreaStaicProperties("properties");
    s.m = new MethodAreaStaicProperties("parameter");
    s = null;
}
~~~

s 为 GC Root，s 置为 null，经过 GC 后，s 所指向的 properties 对象由于无法与 GC Root 建立关系被回收。

而 m 作为类的静态属性，也属于 GC Root，parameter 对象依然与 GC root 建立着连接，所以此时 parameter 对象并不会被回收。

###### 方法区中常量引用的对象

~~~java
public class MethodAreaStaicProperties {
    public static final MethodAreaStaicProperties m = MethodAreaStaicProperties("final");
    public MethodAreaStaicProperties(String name){}
}

public static void testGC(){
    MethodAreaStaicProperties s = new MethodAreaStaicProperties("staticProperties");
    s = null;
}
~~~

m 即为方法区中的常量引用，也为 GC Root，s 置为 null 后，final 对象也不会被回收。

###### 本地方法栈中引用的对象

![可达性分析法2](可达性分析法2.png)

任何 Native 接口都会使用某种本地方法栈，实现的本地方法接口是使用 C 连接模型的话，那么它的本地方法栈就是 C 栈。当线程调用 Java 方法时，虚拟机会创建一个新的栈帧并压入 Java 栈。然而当它调用的是本地方法时，虚拟机会保持 Java 栈不变，不再在线程的 Java 栈中压入新的帧，虚拟机只是简单地动态连接并直接调用指定的本地方法。

### 怎么回收垃圾

标记出了需要回收的垃圾之后，我们需要考虑怎么样去回收垃圾更快、更好，下面是两种常见的回收算法。

#### 1.标记--清除算法

![标记--清除算法1](标记--清除算法1.png)

标记清除算法的思路是，现在内存中将可回收对象标记出来，然后直接清除掉。

这种方法简单便捷，但是也有很大问题。我们知道，内存空间是连续的，当我们需要使用2M的内存时，必须要有一个2M以上的内存空间，2个1M的内存空间是无法使用的。标记清除算法会产生大量的内存碎片，破坏内存空间的连续性，导致我们有很多的内存空间缺无法使用。

#### 2.复制算法

![复制算法1](复制算法1.png)

复制算法是将内存空间划分为大小相等的两块，每次只使用其中一块。当使用中的内存块进行GC时，将存活对象复制到另一个内存块中，将使用中的内存块清空。

复制算法的缺陷也很明显，对内存空间的牺牲太大，合着我这140平的大三房，只能当70平米的小两房来使？

#### 3.标记整理法

![标记整理法1](标记整理法1.png)

标记整理法会将不同状态的对象进行分类并排序，将存活对象排列在内存前端，可回收对象放在其后，排列完成后再将垃圾对象进行回收清理。

标记整理算法一方面在标记-清除算法上做了升级，解决了内存碎片的问题，也规避了复制算法只能利用一半内存区域的弊端。看起来很美好，但从上图可以看到，它对内存变动更频繁，需要整理所有存活对象的引用地址，在效率上比复制算法要差很多。



JVM使用的是分代收集的方法，这是结合上面的三种方法而形成的，一般把JVM堆分为新生代和老年代，在不同区域根据实际情况使用不同的算法。



### 内存模型和回收策略

![内存模型1](内存模型1.png)

![内存模型2](内存模型2.png)

堆内存是JVM中最大的一块内存区域，GC的主要清理对象也是堆。堆主要分为新生代和老年代，新生代中又有Eden和Survivor From、Survivor To区。需要提一下，在Java1.8之后，原本方法区中的永久代改为了元空间，它在逻辑上是属于堆中的，但是又将它称为NON--Heap(非堆)，应该是为了将它和堆区分开。

##### Eden

IBM 公司的专业研究表明，有将近98%的对象是朝生夕死，所以针对这一现状，大多数情况下，对象会在新生代 Eden 区中进行分配，当 Eden 区没有足够空间进行分配时，虚拟机会发起一次 Minor GC，Minor GC 相比 Major GC 更频繁，回收速度也更快。

通过 Minor GC 之后，Eden 会被清空，Eden 区中绝大部分对象会被回收，而那些无需回收的存活对象，将会进到 Survivor 的 From 区（若 From 区不够，则直接进入 Old 区）。

##### Survivor

survivor区相当于Old区的缓冲区域，当Eden进行Minor GC后，存活的对象会转移到Survivor Form中，如果Form区满了，则直接转移到Old区中。那么Survivor To区是做什么的呢？当第二次Minor GC到来时，Eden和From的存活对象转移到To区，Eden和From区清空，这时From和To的职能调换。每经历一次这个过程，Survivor 中的存活对象年龄会+1，当年龄增长到15岁后，就会将对象转移到Old区。

**为啥需要？**

不就是新生代到老年代么，直接 Eden 到 Old 不好了吗，为啥要这么复杂。想想如果没有 Survivor 区，Eden 区每进行一次 Minor GC，存活的对象就会被送到老年代，老年代很快就会被填满。而有很多对象虽然一次 Minor GC 没有消灭，但其实也并不会蹦跶多久，或许第二次，第三次就需要被清除。这时候移入老年区，很明显不是一个明智的决定。

所以，Survivor 的存在意义就是减少被送到老年代的对象，进而减少 Major GC 的发生。Survivor 的预筛选保证，只有经历16次 Minor GC 还能在新生代中存活的对象，才会被送到老年代。

**为啥需要俩？**

设置两个 Survivor 区最大的好处就是解决内存碎片化。

我们先假设一下，Survivor 如果只有一个区域会怎样。Minor GC 执行后，Eden 区被清空了，存活的对象放到了 Survivor 区，而之前 Survivor 区中的对象，可能也有一些是需要被清除的。问题来了，这时候我们怎么清除它们？在这种场景下，我们只能标记清除，而我们知道标记清除最大的问题就是内存碎片，在新生代这种经常会消亡的区域，采用标记清除必然会让内存产生严重的碎片化。因为 Survivor 有2个区域，所以每次 Minor GC，会将之前 Eden 区和 From 区中的存活对象复制到 To 区域。第二次 Minor GC 时，From 与 To 职责兑换，这时候会将 Eden 区和 To 区中的存活对象再复制到 From 区域，以此反复。

这种机制最大的好处就是，整个过程中，永远有一个 Survivor space 是空的，另一个非空的 Survivor space 是无碎片的。那么，Survivor 为什么不分更多块呢？比方说分成三个、四个、五个?显然，如果 Survivor 区再细分下去，每一块的空间就会比较小，容易导致 Survivor 区满，两块 Survivor 区可能是经过权衡之后的最佳方案。

##### Old区

老年代占了堆内存2/3的空间，只有在Major GC(full GC)时才会清理老年代，Major GC时会发生(Stop-The-World)程序暂停现象。内存越大，STW时间就越长。所以并不是内存空间越大越好。由于老年代里的对象是存活率较高的对象，所以使用标记--整理算法是最合适的。

**大对象**

大对象指需要大量连续内存空间的对象，这部分对象不管是不是“朝生夕死”，都会直接进到老年代。这样做主要是为了避免在 Eden 区及2个 Survivor 区之间发生大量的内存复制。当你的系统有非常多“朝生夕死”的大对象时，得注意了。

**长期存活对象**

虚拟机给每个对象定义了一个对象年龄（Age）计数器。正常情况下对象会不断的在 Survivor 的 From 区与 To 区之间移动，对象在 Survivor 区中每经历一次 Minor GC，年龄就增加1岁。当年龄增加到15岁时，这时候就会被转移到老年代。当然，这里的15，JVM 也支持进行特殊设置，但是只能够比15小，因为对象的年龄是定义在对象头中的，用4位二进制表示。

**动态对象年龄**

虚拟机并不重视要求对象年龄必须到15岁，才会放入老年区，如果 Survivor 空间中相同年龄所有对象大小的总合大于 Survivor 空间的一半，年龄大于等于该年龄的对象就可以直接进去老年区，无需等你“成年”。



### GC调优

1.打印GC日志

```-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:./gc.log```

Tomcat可以直接加载JAVA_OPTS变量里

2.分析日志得到关键性指标

3.分析GC原因，调优JVM参数

主要目标是减少或消灭Full GC的次数