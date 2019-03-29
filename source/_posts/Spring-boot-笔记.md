---
title: Spring boot 笔记
date: 2017-09-09 09:32:32
tags:
	- 框架
	- spring
	- 笔记
---

# 1 配置
### 1.1 配置文件
配置文件可以是.properties或.yml后缀的文件，两者的区别：

![.properties](properties.jpg)

![.yml](yml.jpg)

可以看出.yml比.properties文件的配置要简便

<!-- more -->

### 1.2 属性配置
#### 1.2.1 注解
@Value(${<配置文件中的属性名>})：可以将配置文件中的一个属性取出，并赋给当前类的一个属性

@ConfigurationProperties(prefix="<前缀>")：将配置文件中的值赋给一个类

配置文件：

![con](config.jpg)

实体类：

![girl](configG.jpg)

这里将配置文件中的girl的值赋给GirlProperties

### 1.3 多个配置文件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;经常会遇到这种情况，开发时的配置文件和运行时的配置文件要求不相同，这时候就需要频繁修改配置文件。spring中可以编写多个配置文件，可以按照需求转换所需的配置文件

创建三个配置文件：

![t1](t1.jpg)

application.yml:

![t2](t2.jpg)

application-dev.yml:

![t3](t3.jpg)

application-prod.yml:

![t4](t4.jpg)

使用哪一个配置文件取决于application.yml中spring.profiles.active的定义.
> 当spring.profiles.active=prod时使用application-prod.yml为配置文件  
> 当spring.profiles.active=dev时使用application-dev.yml

# 2 Controller注解
	@Controller：定义一个Controller组件，返回一个模板
	@RestController：定义一个Controller组件，返回String，json等
	@ResquestMapping：为该类或方法定义url
	
	@PathVariable：获取url中的数据
	@RequestParam：获取请求参数
	@GetMapping：组合注解，相当于RequestMapping(value="",method="GET")

### 2.1 @PathVariable用法：

![pv](pv.jpg)

![pvt](pvt.jpg)

可以从url中提取数据，并赋给方法中的属性。

### 2.2 @RequestParam用法：

![rp](rp.jpg)

![rp1](rp1.jpg)

此处的required表示是否必须，defaultValue默认值

![rpt](rpt.jpg)

获取url中的请求参数id

# 3 Spring-Data-Jpa
jpa是定义了一系列对象持久化的标准。目前实现这一规范的数据库产品有hibernate、TopLink、JDO等。使用jpa需要导入jar包。它可以简化hibernate的配置和使用。
### 3.1 jpa的使用
1).在application.yml配置文件中配置数据库信息

![jpa1](jpa1.png)

2).创建实体类

![jpa2](jpa2.png)

3).创建一个接口，继承JpaRepository

![jpa3](jpa3.png)

4).使用jpa连接数据库

![jpa4](jpa4.png)

# 4 表单验证
使用@Valid注解

# 5 使用aop
需要导入spring-boot相应的aop包，新建一个切面类，如下：

![aop1](aop1.png)

    其中
    @Pointcut("execution(public * com.myboot.gril..controller.GirlCotroller.*(..))")
    public void log(){}
    表示定义一个公用的方法。避免在@Before和@After中重复定义相同的切入点。

### 5.1 在aop里实现日志

![aop2](aop2.png)

spring已有封装好的日志模块，在切面类中实例化org.slf4j.Logger类就可以通过里面的info方法将信息放入日志中

# 6 统一异常处理
### 6.1 统一输出的数据格式
在controller返回数据时，返回的单个实例和多个实例的集合格式是有区别的，这不利于视图层对数据的处理。所以我们可以对输出的数据制定一个统一的格式。方法是新建一个Result类对输出的结果进行封装：

![exc1](exc1.png)

工具类，方便封装减少重复代码：

![exc2](exc2.png)

结果:

![res1](res1.png)

### 6.2 异常管理
1)、首先自定义一个异常类

![exc3](exc3.png)

2)、创建一个异常信息收集类

![exc4](exc4.png)

3)、处理异常时使用自定义的异常类进行处理

![exc5](exc5.png)

### 6.3 提示信息由枚举类管理
在大型项目中，提示信息是非常繁多的，如果不统一进行管理的话很容易出现混乱。所以在这里新建一个enum对提示信息进行管理。

enum:

![enum1](enum1.png)

使用时通过枚举类型取出所需要的信息即可：

![enum2](enum2.png)

# 7 单元测试
### 7.1 controller测试

![test1](test1.png)

使用到MockMvc类，可以通过url地址对controller层进行单元测试。






