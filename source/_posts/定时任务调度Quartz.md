---
title: 定时任务调度Quartz
date: 2018-10-25 15:50:02
tags:	
	- Java
---

### Quartz是做什么的

Quartz是一个任务调度框架。比如你遇到这样的问题

- 想每月25号，信用卡自动还款
- 想每年4月1日自己给当年暗恋女神发一封匿名贺卡
- 想每隔1小时，备份一下自己的爱情动作片 学习笔记到云盘

这些问题总结起来就是：在某一个有规律的时间点干某件事。并且时间的触发的条件可以非常复杂（比如每月最后一个工作日的17:50），复杂到需要一个专门的框架来干这个事。 Quartz就是来干这样的事，你给它一个触发条件的定义，它负责到了时间点，触发相应的Job起来干活。



### ### 一个简单例子

TestQuartz.java

~~~java
public class TestQuartz {
	public static void main(String[] args) {
		try {
            //调度工厂
			SchedulerFactory schedulerFactory = new StdSchedulerFactory();
            //调度器
			Scheduler scheduler = schedulerFactory.getScheduler();
			scheduler.start();
            //生成一个job对象,TestJob中是具体的调度内容
			JobDetail job = JobBuilder.newJob(TestJob.class)
					.withIdentity("myjob", "group1").build();
            //触发器
			Trigger trigger = TriggerBuilder.newTrigger()
					.withIdentity("myTrigger", "group1")
					.startNow()
					.withSchedule(SimpleScheduleBuilder.simpleSchedule()
							.withIntervalInSeconds(5)
							.repeatForever())
					.build();
            //开始管理该调度
			scheduler.scheduleJob(job, trigger);
		} catch (SchedulerException e) {
			e.printStackTrace();
		}
	}
}
~~~

TestJob