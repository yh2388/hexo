---
title: 线程池技术（一）
date: 2017-07-10 20:40:50
tags: 多线程
---
在java5中，开始为我们提供了线程池技术。接下来我们就简单的了解一下线程池的相关问题。

首先，我们可以创建线程池，并且有多种类型的线程池。
``` java
Executors.newFixedThreadPool(3);//固定数量的线程池
Executors.newCachedThreadPool();//缓存线程池，即按需提供线程数
Executors.newSingleThreadExecutor();//单例的线程池，始终保持有且只有一个线程
```
其中，我们可以使用单例线程池的特性，来解决当一个线程死掉后再创建一个线程的问题。

<!-- more -->

除了上述三个以外，还有一个具有定时器功能的线程池，我们可以看下下面的实例代码：
``` java
Executors.newScheduledThreadPool(3).scheduleAtFixedRate(
	new Runnable(){
	@Override
	public void run() {
		System.out.println("bombing!");
	}},6,2,TimeUnit.SECONDS);
```
这段代码就实现了，任务在多久时间后开始启动，并且每个多久重复运行。但是，这个方法存在一个小问题，就是它
不能指定绝对时间，所以我们需要根据需求，来转换成相对时间。