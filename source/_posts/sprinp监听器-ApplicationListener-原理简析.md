---
title: sprinp监听器(ApplicationListener)原理简析
date: 2020-01-13 18:43:06
tags: 
	- spring
	- 源码
---

##一、spring监听器的使用

​	spring中监听器的使用很方便，建立一个Listener继承ApplicationListener，建立一个Event继承ApplicationEvent，需要使用时，用applicationContext上下文调用publishEvent()方法触发事件

#### 1.1 定义事件MyEvent

```java
public class MyEvent extends ApplicationEvent{
	public MyEvent(Object source) {
		super(source);
	}

	@Override
	public Object getSource() {
		return super.getSource();
	}
}
```

#### 1.2 定义监听器MyListener

```java
@Component
public class MyListener implements ApplicationListener<MyEvent>{
	
	/**
	 * 触发事件时，会执行这个方法 
	 */
	@Override
	public void onApplicationEvent(MyEvent event) {
		System.out.println("监听到事件:" + event.getClass().getSimpleName());
	}

}
```

<!-- more -->

####  1.3 触发事件

```java
public class MyTest {
	public static void main(String[] args) {
        //获取spring上下文
		AbstractApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
		MyEvent myEvent = new MyEvent("Hello");
        //触发事件
		context.publishEvent(myEvent);
	}
}
```



## 二、源码分析（触发事件——>执行监听器过程）

![监听器时序图](ApplicationListener时序图.jpg)

#### 2.1 触发事件

​	触发事件时，使用了AbstractApplicationContext.publishEvent()方法，以此为切入点，查看这个方法的源码：

```java
	/**
	 * 触发给定的事件，通知所有的监听者
	 * @param event the event to publish (may be an {@link ApplicationEvent}
	 * or a payload object to be turned into a {@link PayloadApplicationEvent})
	 * @param eventType the resolved event type, if known
	 * @since 4.2
	 */
	protected void publishEvent(Object event, ResolvableType eventType) {
		
      	...省略...

		// Multicast right now if possible - or lazily once the multicaster is initialized
		if (this.earlyApplicationEvents != null) {
			this.earlyApplicationEvents.add(applicationEvent);
		}
		else {
          	//调用这个方法，将事件广播出去
			getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);
		}

		...省略...
	}
```

​	上面这段代码重点在`getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);`，调用SimpleApplicationEventMulticaster.multicastEvent()方法，将事件广播给监听器

#### 2.2 广播事件

```java
	@Override
	public void multicastEvent(final ApplicationEvent event, ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
      	
		for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			Executor executor = getTaskExecutor();
			if (executor != null) {
				executor.execute(new Runnable() {
					@Override
					public void run() {
						invokeListener(listener, event);
					}
				});
			}
			else {
				invokeListener(listener, event);
			}
		}
	}
```

​	在这个方法中，用getApplicationListeners()方法，获取该事件的所有监听器，然后遍历监听器并用invokeListener()方法，具体的去执行各个监听器逻辑

####  2.3 获取监听器列表

```java
	protected Collection<ApplicationListener<?>> getApplicationListeners(
			ApplicationEvent event, ResolvableType eventType) {

		Object source = event.getSource();
		Class<?> sourceType = (source != null ? source.getClass() : null);
		ListenerCacheKey cacheKey = new ListenerCacheKey(eventType, sourceType);

		// 查询缓存中是否有监听器，如果命中缓存，则直接返回缓存中的监听器
		ListenerRetriever retriever = this.retrieverCache.get(cacheKey);
		if (retriever != null) {
			return retriever.getApplicationListeners();
		}

		if (this.beanClassLoader == null ||
				(ClassUtils.isCacheSafe(event.getClass(), this.beanClassLoader) &&
						(sourceType == null || ClassUtils.isCacheSafe(sourceType, this.beanClassLoader)))) {
			// Fully synchronized building and caching of a ListenerRetriever
			synchronized (this.retrievalMutex) {
				retriever = this.retrieverCache.get(cacheKey);
				if (retriever != null) {
					return retriever.getApplicationListeners();
				}1
                  
              	//未命中缓存，重新获取监听器列
				retriever = new ListenerRetriever(true);
				Collection<ApplicationListener<?>> listeners =
						retrieveApplicationListeners(eventType, sourceType, retriever);
				this.retrieverCache.put(cacheKey, retriever);
				return listeners;
			}
		}
		else {
			// No ListenerRetriever caching -> no synchronization necessary
			return retrieveApplicationListeners(eventType, sourceType, null);
		}
	}
```

​	AbstractApplicationEventMulticaster是SimpleApplicationEventMulticaster的父类，getApplicationListeners()方法主要逻辑是查询缓存中是否有该事件的监听器列表，如果没有则调用retrieveApplicationListeners()重新获取监听器列表。

#### 2.4 重新获取监听器列表

```java
private Collection<ApplicationListener<?>> retrieveApplicationListeners(
			ResolvableType eventType, Class<?> sourceType, ListenerRetriever retriever) {

		LinkedList<ApplicationListener<?>> allListeners = new LinkedList<ApplicationListener<?>>();
		Set<ApplicationListener<?>> listeners;
		Set<String> listenerBeans;
		synchronized (this.retrievalMutex) {
          	//从defaultRetriever中获取listeners和listenerBeans
			listeners = new LinkedHashSet<ApplicationListener<?>>(this.defaultRetriever.applicationListeners);
			listenerBeans = new LinkedHashSet<String>(this.defaultRetriever.applicationListenerBeans);
		}
		for (ApplicationListener<?> listener : listeners) {
			if (supportsEvent(listener, eventType, sourceType)) {
				if (retriever != null) {
					retriever.applicationListeners.add(listener);
				}
				allListeners.add(listener);
			}
		}
		if (!listenerBeans.isEmpty()) {
			BeanFactory beanFactory = getBeanFactory();
          	//遍历listenerBeans，获取所有监听器
			for (String listenerBeanName : listenerBeans) {
				try {
					Class<?> listenerType = beanFactory.getType(listenerBeanName);
					if (listenerType == null || supportsEvent(listenerType, eventType)) {
						ApplicationListener<?> listener =
								beanFactory.getBean(listenerBeanName, ApplicationListener.class);
						if (!allListeners.contains(listener) && supportsEvent(listener, eventType, sourceType)) {
							if (retriever != null) {
								retriever.applicationListenerBeans.add(listenerBeanName);
							}
							allListeners.add(listener);
						}
					}
				}
				catch (NoSuchBeanDefinitionException ex) {
					// Singleton listener instance (without backing bean definition) disappeared -
					// probably in the middle of the destruction phase
				}
			}
		}
		AnnotationAwareOrderComparator.sort(allListeners);
		return allListeners;
	}
```

​	在这个方法中，会获取AbstractApplicationEventMulticaster.defaultRetriever中的listeners(监听器列表)和listenerBeans(监听器BeanId列表)，然后会遍历listenerBeans获取所有监听器实例（所以定义一个监听器后，需要将它注册到IOC中）。_defaultRetriever的初始化在下面会介绍。_

#### 2.5 通知监听器

```java 
	private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
		try {
          	//执行监听器中的逻辑
			listener.onApplicationEvent(event);
		}
		catch (ClassCastException ex) {
			String msg = ex.getMessage();
			if (msg == null || matchesClassCastMessage(msg, event.getClass().getName())) {
				// Possibly a lambda-defined listener which we could not resolve the generic event type for
				// -> let's suppress the exception and just log a debug message.
				Log logger = LogFactory.getLog(getClass());
				if (logger.isDebugEnabled()) {
					logger.debug("Non-matching event type for listener: " + listener, ex);
				}
			}
			else {
				throw ex;
			}
		}
	}
```

​	最后回到invokeListener()方法，里面会调用doInvokeListener()，在这里可以看到`listener.onApplicationEvent(event);`这一句，即调用监听器中的onApplicationEvent()方法去执行监听器的业务逻辑

## 三、源码分析（监听器的初始化和注册）

#### 3.1 初始化spring上下文

```java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				...省略...
                
                // 初始化事件多播器
                initApplicationEventMulticaster
                  
				...省略...
                  
				// 注册监听器
				registerListeners();
              
              	...省略...
			}
			...省略...
		}
	}
```

在refresh()方法中，调用了registerListeners()方法，在这个方法中注册监听器

```java
protected void registerListeners() {
		// Register statically specified listeners first.
		for (ApplicationListener<?> listener : getApplicationListeners()) {
          	//这里的addApplicationListener()方法即初始化2.4中的defaultRetriever
			getApplicationEventMulticaster().addApplicationListener(listener);
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let post-processors apply to them!
		String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
		for (String listenerBeanName : listenerBeanNames) {
			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
		}

		// Publish early application events now that we finally have a multicaster...
		Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
		this.earlyApplicationEvents = null;
		if (earlyEventsToProcess != null) {
			for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
				getApplicationEventMulticaster().multicastEvent(earlyEvent);
			}
		}
	}
```

​	在registerListeners()中调用了`getApplicationEventMulticaster().addApplicationListener(listener);`方法，此方法即初始化了前面2.4中的defaultRetriever

