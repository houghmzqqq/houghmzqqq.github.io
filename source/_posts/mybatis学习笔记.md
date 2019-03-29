---
title: mybatis学习笔记
date: 2018-01-02 09:03:25
tags:

	- 框架
---



## 1.初次使用mybatis

### 1.1通过maven导入jar包

~~~xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>1.1.1</version>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.30</version>
</dependency>

~~~



<!-- more -->

### 1.2mybatis的配置文件

mybatis需要一个全局的配置文件，在其中声明数据库、连接池等参数

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <!-- 配置数据库连接信息 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
                <property name="username" value="root" />
                <property name="password" value="XDP" />
            </dataSource>
        </environment>
    </environments>
    
  <mappers>
    <mapper resource="User.xml"/>
  </mappers>
</configuration>
~~~



## 2.3创建一个实体类User以及相应的映射文件User.xml

~~~java
public class User {
    private int id;
    private String userName;
    private String password;
	…getter and  setter…
}

~~~

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 为这个mapper指定一个唯一的namespace，namespace的值习惯上设置成包名+sql映射文件名，这样就能够保证namespace的值是唯一的
例如namespace="me.gacl.mapping.userMapper"就是me.gacl.mapping(包名)+userMapper(userMapper.xml文件去除后缀)
 -->
<mapper namespace="me.gacl.mapping.userMapper">
    <!-- 在select标签中编写查询的SQL语句， 设置select标签的id属性为getUser，id属性值必须是唯一的，不能够重复
    使用parameterType属性指明查询时使用的参数类型，resultType属性指明查询返回的结果集类型
    resultType="me.gacl.domain.User"就表示将查询结果封装成一个User类的对象返回
    User类就是users表所对应的实体类
    -->
    <!-- 
        根据id查询得到一个user对象
     -->
    <select id="getUser" parameterType="int" 
        resultType="me.gacl.domain.User">
        select * from user where id=#{id}
    </select>
</mapper>
~~~



