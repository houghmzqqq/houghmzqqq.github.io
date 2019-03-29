---
title: redis学习笔记
date: 2018-01-02 19:47:27
tags:

	- redis

	- 数据库
---



## 1.什么是redis

​	redis是一种NoSql（非关系型数据库），它是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Map), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。



## 2.redis的安装

redis一般是安装在liunx上的，本人使用centOs进行安装



<!-- more -->

#### 2.1配置环境

​	redis是使用c语言开发的，使用它需要配置c语言的运行环境，需要在centOs中安装gcc-c++软件

~~~
yum install gcc-c++
~~~



​	下载redis for linux，使用SecureCRT上传到虚拟机上运行的centOs，并进行解压

~~~
tar -zxvf redis-4.0.6.tat.gz
~~~



​	解压后进入redis-4.0.6文件夹

~~~
cd redis-4.0.6
~~~

​	然后执行make命令，对它进行基本的编译

~~~
make
~~~

​	进行基本的安装

~~~
make PREFIX=/usr/local/redis install
~~~

​	然后进到/usr/local/redis/bin目录中

~~~
cd /usr/local/redis/bin
~~~

​	该目录中可以看到以下文件

![/usr/local/redis/bin下文件](bin下文件.png)]

> redis-benchmark		性能测试的工具
>
> redis-check-aof		aof文件修复工具
>
> redis-check-rdb		rdb文件检查工具
>
> redis-cli				命令行的客户端
>
> redis-sentinel -> redis-server
>
> redis-server			redis服务器启动命令

​	回到redis-4.0.6目录下，将redis.conf文件复制到redis目录

~~~
cp redis.conf /usr/local/redis
~~~

​	修改redis.conf文件

> 将daemonize yes 改为 no (表示开启)
>
> 将bind 127.0.0.1 注释掉(否则，redis只能由本机访问)
>
> 将protected-mode yes 改为no(表示关闭保护模式，否则在关闭bind的情况下，不能通过外部程序访问redis)

​	启动redis

~~~
./usr/local/redis/bin/redis-server ./usr/local/redis/redis.conf
~~~

​	查看是否启动

~~~
ps -ef | grep -i redis
~~~

​	关闭redis

~~~
./usr/local/redis/bin/redis-cli shutdown
~~~

​	执行客户端，之后就可以向redis数据库存取数据了

~~~
./usr/local/redis/bin/redis-cli
~~~

​	插入命令，插入一个key=name，value=zhangsan的数据

~~~
set name zhangsan
~~~

​	查询命令，查询key=name的数据

~~~
get name
~~~

​	查看所有key名称

~~~
keys *
~~~



## 3.使用jedis操作linux上的redis

​	jedis是redis官方推荐的一个针对java语言的客户端，使用它可以使用java程序操作redis。

#### 3.1打开redis端口

在root用户下，通过vim /etc/sysconfig/iptables 修改防火墙配置，

![iptables配置](iptables配置.png)

以上6379为默认端口，配置你自己的redis端口



#### 3.2在maven工程中导入jar包

~~~xml
<dependency>
   <groupId>commons-pool</groupId>
   <artifactId>commons-pool</artifactId>
   <version>1.6</version>
</dependency>
<dependency>
   <groupId>redis.clients</groupId>
   <artifactId>jedis</artifactId>
   <version>2.7.0</version>
</dependency>
~~~



#### 3.3测试

~~~java
public class JedisDemo1 {

  	/**
  	*连接linux上的redis
  	**/
    @Test
    public void demo1(){
        //1.设置ip地址和端口号
        Jedis jedis = new Jedis("192.168.13.153");
        //2.保存数据
        jedis.set("name","yejunfeng");
        //3.获取数据
        System.out.println(jedis.get("name"));
        //4.释放资源
        jedis.close();
    }

  	/**
  	*设置redis的连接池
  	**/
    @Test
    public void demo2(){
        //1.获得连接池配置对象
        JedisPoolConfig config = new JedisPoolConfig();
        //设置最大连接数
        config.setMaxTotal(10);
        //设置最大空闲连接数
        config.setMaxIdle(5);

        //获得连接池
        JedisPool jedisPool = new JedisPool(config,"192.168.13.153",6379);

        //获得jedis对象
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.set("handsome","yjf");
            System.out.println(jedis.get("handsome"));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if(jedis!=null){
                jedis.close();
            }
            if(jedisPool!=null){
                jedisPool.close();
            }
        }

    }
}
~~~

## 4.redis中的数据结构

redis有5种数据结构，分别是hash、String、list、sort set、set



#### 4.1字符串的基本操作

> set	key  value
>
> get  key  value
>
> incr  key(该key的value为一个可以转换为数字的字符串，否则报错)
>
> decr	  key
>
> incrby  key  4(表示该key的值加4)
>
> decrby  key  4(表示该key的值减4)
>
> append  key  aa(如果key的值为b，执行该命令后key的值为aab)



#### 4.2hash的基本操作

key为hash的名称，field为属性名，value为属性值

> hset  key  field  value  
>
> hget  key  field
>
> hdel  key  field
>
> hgetall  key
>
> hmset  key  field  value   [field  value]
>
> hincrby  key  field  increment(使hash的某个属性增加‘increment’)
>
> hexists  key  field(判断某个hash的某个属性是否存在)
>
> hlen  key(返回某个hash的属性个数)
>
> hkeys  key(返回某个hash的所有属性名称)
>
> hvals  key(返回某个hash的所有属性的值)



#### 4.3list的基本操作

> lpush  key  value  [value] (从左边将元素放入队列)
>
> lpushx  key  value  [value] (为已存在的列表添加至)
>
> rpush  key  value  [value] (从右边将元素放入队列)
>
> lrange  key  start  stop (start和stop表示查找的范围，0  -1 表示查找全部)
>
> lindex  key  index (通过索引查找列表中的元素)
>
> rpop  key (弹出列表的最后一个元素)
>
> rpoplpush  list1  list2 (将list1中的最后一个元素删除并移出，放入list2的头部)
>
> linsert  key  BEFORE|AFTER  pivot  value (在pivot之前或之后插入元素)



#### 4.4set的基本操作