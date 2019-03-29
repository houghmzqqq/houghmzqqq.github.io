---
title: SpringMVC笔记
date: 2017-09-16 16:26:20
tags:
	- 框架
	- spring
	- 笔记
---



## 1. Spring mvc 的配置

1.1 首先，我们需要在web.xml中定义Spring mvc的servlet

~~~xml
<!-- 配置Spring应用上下文 -->
<context-param>  
  <param-name>contextConfigLocation</param-name>  
  <param-value>classpath:applicationContext.xml</param-value></context-param>
<listener>  
  <listener-class>org.springframework.web.context.ContextLoaderListener
  </listener-class>
</listener>

<!-- DispatcherServlet,Spring MVC的核心 -->
<servlet>  
  <servlet-name>mvc-dispatcher</servlet-name>  
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  <init-param>
  <!-- 对应Spring应用的上下文 -->
  <param-name>contextConfigLocation</param-name>    
    <!-- 修改配置文件的路径 -->    
    <param-value>classpath:mvc-dispatcher-servlet.xml</param-value>  
  </init-param>  
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>  
  <servlet-name>mvc-dispatcher</servlet-name>  
  <!-- 表示mvc-dispatcher拦截所有的请求 -->  
  <url-pattern>/</url-pattern>
</servlet-mapping>
~~~

<!-- more -->

1.2 配置Spring mvc 的xml文件，为 DispatcherServlet提供相应的配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"  
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
     xmlns:mvc="http://www.springframework.org/schema/mvc"
     xmlns:context="http://www.springframework.org/schema/context"  
     xsi:schemaLocation="
     http://www.springframework.org/schema/beans 
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
     http://www.springframework.org/schema/context 
     http://www.springframework.org/schema/context/spring-context-3.0.xsd
     http://www.springframework.org/schema/mvc
     http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    
    <!-- 本配置文件是名为mvc-dispatcher的DispatcherServlet使用的，为其提供相关的Spring MVC配置 -->
    
    <!-- 激活@Required等注解 -->   
	<context:annotation-config/> 
	
	<!-- DispatcherServlet上下文，只搜索@Controller注解的类 -->
	<context:component-scan base-package="com.yejunfeng.action">
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
	</context:component-scan>
	
	<!-- 扩充了注解驱动，可以将请求参数绑定到控制器参数 -->
	<mvc:annotation-driven/>
	
	<mvc:interceptors>
    <!--国际化操作拦截器 如果采用基于（请求/Session/Cookie）则必需配置-->
    	<bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" />  
    <!-- 如果不定义 mvc:mapping path 将拦截所有的URL请求 -->
    	<mvc:interceptor>
    	 <mvc:mapping path="/**/*"/>
         <mvc:exclude-mapping path="/**/fonts/*"/>
         <mvc:exclude-mapping path="/**/*.css"/>
         <mvc:exclude-mapping path="/**/*.js"/>
         <mvc:exclude-mapping path="/**/*.png"/>
         <mvc:exclude-mapping path="/**/*.gif"/>
         <mvc:exclude-mapping path="/**/*.jpg"/>
         <mvc:exclude-mapping path="/**/*.jpeg"/>
         <mvc:exclude-mapping path="/**/*login*"/>
    	<bean class="com.xyz.gym_management_sys.filter.SecurityInterceptor"></bean>
    	</mvc:interceptor>
    	
    	<mvc:interceptor>
    	 <mvc:mapping path="/**/equ/*"/>
         <mvc:exclude-mapping path="/**/fonts/*"/>
         <mvc:exclude-mapping path="/**/*.css"/>
         <mvc:exclude-mapping path="/**/*.js"/>
         <mvc:exclude-mapping path="/**/*.png"/>
         <mvc:exclude-mapping path="/**/*.gif"/>
         <mvc:exclude-mapping path="/**/*.jpg"/>
         <mvc:exclude-mapping path="/**/*.jpeg"/>
         <mvc:exclude-mapping path="/**/*login*"/>
    	<bean class="com.xyz.gym_management_sys.filter.EquManageInterceptor"></bean>
    	</mvc:interceptor>
    	<mvc:interceptor>
    	 <mvc:mapping path="/**/userManage/*"/>
         <mvc:exclude-mapping path="/**/fonts/*"/>
         <mvc:exclude-mapping path="/**/*.css"/>
         <mvc:exclude-mapping path="/**/*.js"/>
         <mvc:exclude-mapping path="/**/*.png"/>
         <mvc:exclude-mapping path="/**/*.gif"/>
         <mvc:exclude-mapping path="/**/*.jpg"/>
         <mvc:exclude-mapping path="/**/*.jpeg"/>
         <mvc:exclude-mapping path="/**/*login*"/>
    	<bean class="com.xyz.gym_management_sys.filter.UserManageInterceptor"></bean>
    	</mvc:interceptor>
	</mvc:interceptors>
	
	<!-- 静态资源处理，入css，js，image -->
	<!--<mvc:resources location="/images/" mapping="/images/**" />-->
	<mvc:resources location="/lib/" mapping="/lib/**" />
	<mvc:resources location="/stylesheets/" mapping="/stylesheets/**" />
	
	<!-- 拼接视图的名称，prefix表示前缀，suffix表示后缀 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="viewClass" value="org.springframework.web.servlet.view.JstlView"></property>
		<property name="prefix" value="/WEB-INF/jsps/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>

	<!-- 文件上传的bean配置 -->
	<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<property name="maxUploadSize" value="102400" />
		<property name="maxInMemorySize" value="514" />
		<property name="defaultEncoding" value="UTF-8" />
		<!--<property name="uploadTempDir" value="upload/temp" />-->
	</bean>
</beans>
~~~

## 2.spring mvc注解

2.1 @RequeseMapping(value="path")

​	表示访问该action的路径名，可以注解类和类方法，类上的注解和方法上的注解拼接起来可以访问该方法。

~~~java
@RequestMapping(value="user")
public class User{
  	@RequestMapping(value="login")
  	public String login(){
  		return null;
	}
}
~~~

​	以上代码中~127.0.0.1:8080//user/login~地址可以访问user.login()方法，并返回一个视图名称。



2.2 @RequestParam

​	用于接收浏览器发送到服务器的请求参数

~~~java
@RequestMapping(value="login")
public String login(@RequestParam String userName, @RequestParam(value="password") String password){
  	return null;
}
~~~

​	以上代码中，login()可以接收两个请求参数，其中第一个@RequestParam因为没有value值，所以请求参数的名字必须等于userName，否则会出错。



​	除了以上的接收参数的方法，Spring mvc可以将需要接收的请求参数直接封装在一个类中。

~~~java
public class LoginInfo{
  	private String userName;
  	private String password;
  	...setter and getter...
}
~~~

~~~java
@RequestMapping(value="login")
public String login(LoginInfo logIn){
  	return null;
}
~~~

​	当然，LoginInfo中的userName和password需要和浏览器传过来的参数名称相同。



2.3 @ResponseBody

​	将内容或对象作为HTTP响应正文返回（不返回视图，只返回内容，适合做即时校验）

~~~java
@RequestMapping(value="login")
@ResponseBody
public void login(){
  return "hello world!";
}
~~~

​	直接在方法上面加上该注解，当访问~127.0.0.1:8080//user/login~时，页面上显示_hello world!_.

### 3.对于Spring 中自定义servlet时获取IOC容器中的实例

​	在Spring中如果你自定义一个自动加载的servlet，那么当你在该servlet中获取IOC容器中的实例时，是取不到的，会得到一个null值，程序会报空指针异常。

解决方案：

​	在servlet中加载Spring配置文件，通过getBean()方法获取实例，代码

~~~java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

deviceService = context.getBean(DeviceService.class);
~~~



### 4.表单上传

spring mvc 中的表单上传需要使用到MultipartFile这个类

1.表单上需要加上enctype="multipart/form-data"，打开文件上传功能

2.通过@RequestPart注解获取文件对象

~~~java
@RequestMapping(value="/add")
public String method(@RequestPart(value="file01")MultipartFile file){
  file.getInputStream();
  file.getSize();
  ...
  ...
}
~~~



### 5.拦截器

Spring中定义一个拦截器，用于登录拦截。

第一步：定义一个拦截器。spring 中的拦截器需要继承HandlerInterceptorAdapter，它实现了java中给出的拦截器接口（具体待补充）。

~~~java
public class UserLoginInteceptorByString extends HandlerInterceptorAdapter {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if("GET".equalsIgnoreCase(request.getMethod())){
            //
        }
        System.out.println("preHandle...");
        String requestUri = request.getRequestURI();
        String contextPath = request.getContextPath();
        String url = requestUri.substring(contextPath.length());
        System.out.println("requestUri = " + requestUri);
        System.out.println("contextPath = " + contextPath);
        System.out.println("url = " + url);
        String username = (String) request.getParameter("username");
        if(username==null || username.equals("")){
            //未登录，跳转到登录界面
//            request.getRequestDispatcher("/WEB-INF/jsps/login.jsp").forward(request,response);
            System.out.println("You have not login in, turn to login page...");
            return false;
        }
        else{
            System.out.println("You are login in, turn to the page you want...");
            return true;
        }
//        return super.preHandle(request, response, handler);
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle...");
        if (modelAndView != null) {
            Map<String,String> map = new HashMap<>();
//            modelAndView.addAllObjects(map);
        }

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion...");
    }
}
~~~



第二步：在Spring mvc的配置文件中进行拦截器的注册配置

~~~xml
<mvc:interceptors>
    &lt;!&ndash; 国际化操作拦截器 如果采用基于（请求/Session/Cookie）则必需配置 &ndash;&gt;
        <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" />
     &lt;!&ndash;如果不定义 mvc:mapping path 将拦截所有的URL请求&ndash;&gt;
        <mvc:interceptor>
          <!-- 拦截的url，这里表示拦截所有请求 -->
           <mvc:mapping path="/**/*"/>
          <!-- 排除静态资源，不配置的话，拦截器会拦截静态资源 -->
           <mvc:exclude-mapping path="/**/fonts/*"/>
           <mvc:exclude-mapping path="/**/*.css"/>
           <mvc:exclude-mapping path="/**/*.js"/>
           <mvc:exclude-mapping path="/**/*.png"/>
           <mvc:exclude-mapping path="/**/*.gif"/>
           <mvc:exclude-mapping path="/**/*.jpg"/>
           <mvc:exclude-mapping path="/**/*.jpeg"/>
           <mvc:exclude-mapping path="/**/*login*"/>
           <bean class="com.yejunfeng.interceptor.UserLoginInteceptorByString"></bean>
        </mvc:interceptor>
</mvc:interceptors>

~~~



当访问localhost://user/toLogin时，控制台输出：

​	preHandle...
​	requestUri = /stu/toView
​	contextPath = 
​	url = /stu/toView
​	You have not login in, turn to login page...

当访问localhost://user/toLogin?username=yyy时，控制台输出：

​	preHandle...

​	requestUri = /stu/toView

​	contextPath = 

​	url = /stu/toView

​	You are login in, turn to the page you want...

​	postHandle...



## 6.关于Spring mvc的线程安全问题

对于SpringMVC和Struts2，我们知道SpringMVC是基于方法的拦截，而Struts2是基于类的拦截。

struts2每次处理一个请求，它会产生一个action的实例，所以它是不会产生线程安全问题的。

Spring的Controller默认是singleton的，这表示所有关于这个Action的请求都使用一个实例去进行处理，这样的话就容易产生线程问题。



我的解决方案：

1.在spring配置文件Controller中声明 scope="prototype"或者使用@Scope(“prototype”)注解，每次都创建新的controller

2.使用ThreadLocal变量

3.避免在Controller中使用实例变量



