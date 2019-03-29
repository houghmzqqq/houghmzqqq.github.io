---
title: springboot 2.0中定义拦截器
date: 2018-04-04 09:51:23
tags:	
	- 笔记
	- 框架
	- spring
---

HandlerInterceptor 的功能跟过滤器类似，但是提供更精细的的控制能力：在request被响应之前、request被响应之后、视图渲染之前以及request全部结束之后。我们不能通过拦截器修改request内容，但是可以通过抛出异常（或者返回false）来暂停request的执行。

实现 UserRoleAuthorizationInterceptor 的拦截器有： 
ConversionServiceExposingInterceptor 
CorsInterceptor 
LocaleChangeInterceptor 
PathExposingHandlerInterceptor 
ResourceUrlProviderExposingInterceptor 
ThemeChangeInterceptor 
UriTemplateVariablesHandlerInterceptor 
UserRoleAuthorizationInterceptor

其中 LocaleChangeInterceptor 和 ThemeChangeInterceptor 比较常用。

配置拦截器也很简单，Spring 为我们提供了基础类WebMvcConfigurerAdapter ，我们只需要重写 addInterceptors 方法添加注册拦截器。

<!-- more -->

实现自定义拦截器只需要3步： 
1、创建我们自己的拦截器类并实现 HandlerInterceptor 接口。 
2、创建一个Java类继承WebMvcConfigurerAdapter，并重写 addInterceptors 方法。 
2、实例化我们自定义的拦截器，然后将对像手动添加到拦截器链中（在addInterceptors方法中添加）。

### 1、LoginInterceptor.java

新建一个Interceptor类，实现HandlerInteceptor接口

```java
public class LoginInterceptor implements HandlerInterceptor{
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("-->request请求执行前，Controller执行前。");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
        System.out.println("-->request请求执行后，Controller执行前。");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
        System.out.println("-->request请求执行后，Controller执行后。");
    }
}
```

### 2、WebAppConfigurer

​	新建WebAppConfigurer类继承WebMvcConfigurationSupport，重写addInteceptor()方法，在其中配置拦截器。（在SpringBoot2.0中，WebMvcConfigurationAdapter已过时，用WebMvcConfigurationSupport替换）

```java
@Configuration
public class WebAppConfigurer extends WebMvcConfigurationSupport {
    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor()).addPathPatterns("/**");
        super.addInterceptors(registry);
    }
}
```



拦截器配置完成，在浏览器访问该项目路径，拦截器会进行拦截

```
-->request请求执行前，Controller执行前。
-->request请求执行后，Controller执行前。
-->request请求执行后，Controller执行后。
```

