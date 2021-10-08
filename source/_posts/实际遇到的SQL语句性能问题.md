---
title: 实际遇到的SQL语句性能问题
date: 2020-11-27 16:24:54
tags:
	- 数据库
---



&emsp;&emsp;最近开发的功能上了压测环境，发现了许多有性能问题的SQL语句，就很难受。。。在这里记录一下容易产生性能瓶颈的SQL语句，以及原因，避免下次再犯同样的错误吧。

# 1.执行计划

&emsp;&emsp;在分析实际遇到的问题前，我们先了解一下mysql的执行计划，在SQL语句前面加上explain即可查看该语句的执行计划，explain出来的各参数的意义如下：

#### a). **select_type**

表示查询中每个select子句的类型，具体如下：

(1) SIMPLE(简单SELECT,不使用UNION或子查询等)

(2) PRIMARY(查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY)

(3) UNION(UNION中的第二个或后面的SELECT语句)

(4) DEPENDENT UNION(UNION中的第二个或后面的SELECT语句，取决于外面的查询)

(5) UNION RESULT(UNION的结果)

(6) SUBQUERY(子查询中的第一个SELECT)

(7) DEPENDENT SUBQUERY(子查询中的第一个SELECT，取决于外面的查询)

(8) DERIVED(派生表的SELECT, FROM子句的子查询)

(9) UNCACHEABLE SUBQUERY(一个子查询的结果不能被缓存，必须重新评估外链接的第一行)

b).**table**

&emsp;&emsp;**显示这一行的数据是关于哪张表的**，有时不是真实的表名字,看到的是derivedx(x是个数字,应该是第几步执行的结果)

c).**type**

&emsp;&emsp;表示MySQL在表中找到所需行的方式，又称“访问类型”。常用的类型有： **ALL, index, range, ref, eq_ref, const, system, NULL（从左到右，性能从差到好）**

(1) ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行

(2) index: Full Index Scan，index与ALL区别为index类型只遍历索引树

(3) range:只检索给定范围的行，使用一个索引来选择行

(4) ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

(5) eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件

(6) const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量,system是const类型的特例，当查询的表只有一行的情况下，使用system

(7) NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

d).**possible_keys**

&emsp;&emsp;**指出MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用**

&emsp;&emsp;该列完全独立于EXPLAIN输出所示的表的次序。这意味着在possible_keys中的某些键实际上不能按生成的表次序使用。
&emsp;&emsp;如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查WHERE子句看是否它引用某些列或适合索引的列来提高你的查询性能。如果是这样，创造一个适当的索引并且再次用EXPLAIN检查查询

e).**Key**

&emsp;&emsp;**key列显示MySQL实际决定使用的键（索引）**

&emsp;&emsp;如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

f).**key_len**

&emsp;&emsp;**表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）**

&emsp;&emsp;不损失精确性的情况下，长度越短越好 

g).**ref**

&emsp;&emsp;**表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值**

h).**rows**

 &emsp;&emsp;**表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数**

i).**Extra**

&emsp;&emsp;**该列包含MySQL解决查询的详细信息,有以下几种情况：**

(1) Using where:列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤

(2) Using temporary：表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询

(3) Using filesort：MySQL中无法利用索引完成的排序操作称为“文件排序”

(4) Using join buffer：改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。

(5) Impossible where：这个值强调了where语句会导致没有符合条件的行。

(6) Select tables optimized away：这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行



# 2.实际问题

a).没有加索引

&emsp;&emsp;在实际项目中遇到最多的问题就是没有加索引，因为需求文档上没有标明可能有的数据量，开发过程中我就很少会有意识的去考虑数据量的问题，就忽略的性能的问题了。

&emsp;&emsp;问题是由监控系统发现的，解决过程就是查看下常用的查询条件，给它加上索引，order by用来排序的字段也要加上索引。然后用explain查看执行计划，主要看看type和key列，查看是否使用了索引。

b).索引不生效

1).使用like模糊查询

我的索引:

![索引001](索引001.png)

sql001:

![sql001](sql001.png)

sql002:

![sql002](sql002.png)

&emsp;&emsp;可以看到，上面的第一条上去了走索引，第二条不走。因为like不使用模糊查询的时候，效果是等同于"="的。

2).使用"!="、"is not null"

sql003:

![sql003](sql003.png)

3).对索引列做运算

sql004:

![sql004](sql004.png)

&emsp;&emsp;包括+、-、or等运算，都会使索引失效。

sql005:

![sql005](sql005.png)

&emsp;&emsp;需要注意的是，想上面这种or查询是能够走索引的，这里说的or运算是指sql004中的情况。

4).复合索引不符合最左原则时

sql006:

![sql006](sql006.png)

5).范围查询时，数量过多

&emsp;&emsp;像sql006中的语句，原本可以使用INDEX_START_TIME索引的，但是由于结果数据量过大导致存储引擎放弃使用索引。

&emsp;&emsp;使用如下的语句限制结果的数据量，可以使其走索引。

sql007:

![sql007](sql007.png)