---
title: CPU、JVM内存性能分析
date: 2020-11-04 16:26:02
tags:
	- JVM	
---



&emsp;&emsp;最近开发了一个新功能，测试通过美滋滋的上线了。运行一段时间后被告知有问题，我心里一咯噔，赶紧去了解了情况。原来自从更新了我的新功能后，系统的CPU占用很高，有时会导致服务器卡死。

&emsp;&emsp;了解情况后，就是排查问题了。下面记录一下我的排查过程，避免以后遇到相同问题时手忙脚乱。

&emsp;&emsp;定位问题的方法有很多种，这里我介绍一下我使用的两种方式。

### 1.jstack生成快照

&emsp;&emsp;jstack是jdk自带的分析工具，同源的工具还有jstat -gc（查看gc情况）、jmap -heap（查看堆内存情况）等。使用jstack可以查看进程中的栈信息快照，就是各线程的具体信息。

a）首先，我们执行下面命令，查看所有进程的资源占用情况：

```shell
top -c
```

![jstack01](jstack01.png)

按下P可按照CPU使用率排序。

<!-- more -->

b）然后查询该进程中各线程的资源占用情况:

```shell
top -H p 84750
```

![jstack02](jstack02.png)

同样按下P进行排序。



c）使用下面的命令生成快照文件：

```shell
jstack 84750 > ./84750.stack
```

这里的84750是进程的pid



d）读取CPU使用率最高的线程的信息

上面我们拿到了CPU使用率最高的线程pid为85158，我们需要把它转化为16进制，即14ca6

```
cat 84750.stack |grep '14ca6' -C 8
```

![jstack03](jstack03.png)

&emsp;&emsp;这样就能够定位到出问题的代码在哪个位置了，接下来就剩下找出原因并解决。CPU升高的问题无非就是死循环嘛，我最终找到的问题是，在代码中使用了while(true)，后面使用Thread.sleep(10000)代替了。



### 2.arthas

&emsp;&emsp;使用上面的jstack是可以定位到问题线程的，但是操作还是有一些繁琐的，需要找到进程pid，再找线程pid，还要将线程pid转为16进制，整个排查过程就很不流畅，不符合我优雅居士的作风和气质。

&emsp;&emsp;所以我又找到了arthas，这是alibaba开发的一款开源工具，专门用来排查分析java进程的性能问题的。他的功能和jdk自带工具的功能相似，但是使用起来很方便。

&emsp;&emsp;下载安装教程略过，下面是官方文档：[arthas官方文档](https://arthas.aliyun.com/doc/thread.html)



a）启动arthas：

```shell
java -jar arthas-boot.jar
```

![arthas01](arthas01.png)

然后arthas会列出所有java进程，选择你想要分析的进程，输入1或2等



b）查看CPU使用率最高的10个线程：

```
thread -n 10
```

![arthas02](arthas02.png)

&emsp;&emsp;只需要这样就能够定位到占用CPU最高的线程的，比jstack方便很多。



&emsp;&emsp;arthas还有很多其他功能，比如trace可以监控具体某个方法的耗时，这在某个接口响应速度比较慢的时候，用来排查哪一步耗时较长，然后分析问题并优化。其他功能移步到arthas官网查看。