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

<!-- more -->

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

TestJob.java

~~~java
/**
* 任务具体实现类
*/
public class TestJob implements Job{
	public void execute(JobExecutionContext paramJobExecutionContext) throws JobExecutionException {
		System.out.println("这是一个定时任务。");
	}
}
~~~



像这样就可以启动一个定时任务了，任务每隔5秒执行一次。

JobDetail用来生成一个Job对象，对应任务，可以用它注册具体的任务实现。Trigger是触发器，它决定任务在何时被触发。Scheduler是调度器，用来管理任务，它基本可以控制quartz中的任何东西了，Scheduler在手天下我有。



### cron表达式

cron表达式是用来表示时间的一段字符串，大概长这个样子`0 0/10 * * * ?`，它表示间隔10分钟

个人认为专门学习这个表达式的语法有点麻烦，简单了解下语法就好。如果有复杂的定时要求，网上有cron生成器，直接用就是了：[cron表达式生成器]

[cron表达式生成器]: <http://cron.qqe2.com/> "cron表达式生成器"



使用cron表达式生成触发器：

```java
// 表达式调度构建器
CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("0 0/10 * * * ?");
// 按照cronExpression表达式构建一个trigger
trigger = TriggerBuilder.newTrigger().startNow()
		.withIdentity(mySche.getScheId(), mySche.getScheduleName())
		.withSchedule(scheduleBuilder).build();
```



#### 使用spring与绑定数据库

quartz对spring的亲和度