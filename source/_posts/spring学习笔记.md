---
title: spring学习笔记
date: 2017-07-26 20:53:00
tags:
	- 框架
	- spring
	- 笔记

---

## 1.IOC(控制反转)
__转移控制权——由外部容器进行对象的创建，并进行对象控制权的管理__。应用程序本身不进行对象的创建，有外部容器完成创建对象和管理，当应用程序需要使用对象是，向容器进行申请。

>	DI(依赖注入)——是IOC的一中实现方法，通过容器将程序所要依赖的对象注入。
一个例子：
>	房间中介--------IOC
	找中介----------找IOC容器
	租房子----------IOC返回对象
	入住------------程序使用对象

<!-- more -->
## 2.Spring注入方式
### 2.1设值注入
通过setter方法注入的方式

```
public class MyTest{
	private Bean bean;
	//通过setter方法注入
	public void setBean(Bean bean){
		this.bean = bean;
	}
}
```
### 2.2构造器注入
通过在构造器中声明变量进行注入

```
public class MyTest{
	private Bean bean;
	//通过构造方法注入
	public MyTest(Bean bean){
		this.bean = bean;
	}
}
```

## 3.Bean
作用域
>	singleton:单例，一个IOC容器中只存在一个
>	prototype:每次请求（每次使用）创建新的实例，destroy方法不生效
>	request:每次http请求创建一个实例且仅在当前request内有效
>	session:与上类似，每次http请求创建，当前session中有效
>	global session:基于portlet的web中有效

生命周期
	配置文件中定义，IOC容器初始化，使用，IOC容器销毁

Aware
	spring提供一些以Aware结尾的接口，实现这些接口的Bean可以使用部分资源

### 3.1Bean的装配
>	通过xml文件进行显示配置
>	在Java中进行显示配置
>	隐式的Bean发现机制和自动装配
>	>   自动装配：在配置文件的beans标签中加入default autowire=“no/byName/byType/constructor”属性（作用是，当在一个bean中调用另外一个bean时，不需要在配置文件中标明）

>	Resource和ResourceLoder
>	>	ApplicationContext默认实现了Resource接口，可以通过ApplicationContext获取Resource
>	>	Resource resource = ctx.getResource();
>	>	加载资源文件的接口

### 3.2Bean的注解
>	@Component：通用的注解，可用于所有Bean
>	@Repository：有针对性的Bean注解，Dao层
>	@Service：Service层
>	@Controller：Controller层
>	@Scope：类级别的注解，定义该Bean的作用域，可自定义。

### 3.3类的自动检测和Bean的注册
>	自动检测需要在配置文件中设置，使用<context:component-scan base-package="org.example"/>标签(其中base-package表示需要检测的包名称)和<context:annotation-config/>，因为前者（常用）包含后者的功能，所以使用前者后一般不使用后者;

### 3.4其他注解
>	@Autowired
>	>	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以在setter方法、成员变量和构造方法上进行注解
>	>	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以注解BeanFactory、ApplicationContext、ResourceLoder等接口
>	>	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;@Autowired注解是由BeanPostProcessor处理的，所以不能对BeanPostProcessor、BeanFactoryPOSTProcessor等接口进行注解，必须在xml文件中声明
>	>	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于有多个实现类的接口，使用@Autowired注解List或Map可以自动将接口的实现类都装配进入List或Map中，其中Map<String,Bean>中的String为Bean的名称

>	@Qualifier（"BeanName"）
>	>	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于有多个实现类的接口，使用@Autowired注解时，可以使用@Qualifier注解指定需要装配的实现类

### 3.5在Java中进行显示装配
```
@Configuration
public class AppConfig{
	@Bean（"BeanName"）
	public MyService myService(){
		return new MyServiceImpl();
	}
}
相当于配置文件中的\<bean/\>标签
```
>	@Bean标签有InitMethod和destoryMethod属性，定义初始化和销毁方法

>	@Scope，一般与@Bean一起使用，设置Bean的作用域（单例或多例等）


## 4.AOP
>	AOP即面向切面编程，通过__预编译方式__或__运行期动态代理__实现程序功能的统一维护的一种技术

![面向切面](AOP.PNG)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主要的应用模块是：日志记录，性能统计，安全控制，事物处理，异常处理等

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如上图所示，一个系统中有产品、订单、用户等多个模块，每一个模块都需要使用事务管理，如果在每个模块中都编写事务管理的代码，工作量将会很大；使用AOP的思想，将事务管理的功能独立出来，当某个模块需要使用时，通过预编译或动态代理的方式，赋予它该功能。


![面向切面](AOP2.PNG)

### 4.1AOP配置aspect切面
```
<aop:config>
	<aop:aspect id="myAspect" ref="myBean">
		...（<aop:ponitcut/>）
	</aop:aspect>
</aop:config>

<bean id="myBean" class="...">
	...
</bean>
```

### 4.2配置pointcut切入点

![面向切面](AOP3.PNG)

![面向切面](AOP4.PNG)

等。。。。。。

# 补充：

## 5.事务管理
### 5.1编程式事务管理
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;编程式事务管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。
### 5.2声明式事务管理
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;声明式事务管理建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__声明式事务管理也有两种常用的方式，一种是基于tx和aop名字空间的xml配置文件，另一种就是基于@Transactional注解。显然基于注解的方式更简单易用，更清爽。__

application.xml中配置声明式事务管理：

```
<!-- 配置Hibernate Session -->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.format_sql">false</prop>
				<prop key="hibernate.cache.use_second_level_cache">false</prop>
				<prop key="hibernate.cache.use_query_cache">false</prop>
				<prop key="current_session_context_class">thread</prop>
				<prop key="hibernate.hbm2ddl.auto">update</prop>
				<prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate4.SpringSessionContext</prop>
				<prop key="c3p0.min_size">5</prop>
				<prop key="c3p0.max_size">30</prop>
				<prop key="c3p0.time_out">1800</prop> 
				<prop key="c3p0.max_statement">50</prop> 
			</props>
		</property>
		<!-- 实体类的包 -->
		<property name="packagesToScan">
			<list>
				<value>com.*.po</value>
			</list>
		</property>
	</bean>
```

```
<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>
```

```
<!-- 事务advice -->
	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="get*" read-only="true" propagation="REQUIRED" />
			<tx:method name="list*" read-only="true" propagation="REQUIRED" />
			<tx:method name="find*" read-only="true" propagation="REQUIRED" />
			<tx:method name="*" read-only="false" propagation="REQUIRED" rollback-for="Exception"/>
		</tx:attributes>
	</tx:advice>
	
	<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" />
	
	<!-- 配置切面 -->
	<aop:config proxy-target-class="true">
		<aop:advisor pointcut="execution(public * com.xyz.gym_management_sys.service.*Service.*(..))" advice-ref="txAdvice" />
	</aop:config>
```