---
title: Android延迟执行的三种方式
date: 2019-01-18
categories: Android
tags: [android]
---

Android延迟执行的三种方式

<!-- more -->

* 线程  
```java
new Thread(new Runnable() {
	@Override
	public void run() {
		Thread.sleep(1000); // 休眠1秒
		/**
		 * 延时执行的代码
		 */
	}
}).start();
```

* 延时器
```java
Timer timer = new Timer();
timer.schedule(new TimerTask() {
	@Override
	public void run() {
		/**
		 * 延时执行的代码
		 */
	}
},1000); // 延时1秒
```

* Android消息处理
```java
new Handler().postDelayed(new Runnable() {
	@Override
	public void run() {
		/**
		 * 延时执行的代码
		 */
	}
},1000); // 延时1秒
```
