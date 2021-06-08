---
title: 'Android:一个菜单刷新很慢的问题'
date: 2020-10-30
categories: Android
tags: [android,ui,thread]
---

低端打工人之日常疑惑。  

<!-- more -->


#### 流程  
一个串口自动化测试程序：  
是一个服务：`FactoryAutomationService` （FactoryAutomationService.java文件）  
服务启动之后准备创建子线程：  
```java
private FactoryUartThread mFacUartThread;  
mFacUartThread = new FactoryUartThread(this);  
mFacUartThread.startChangHongUartThread();  
```

具体创建了子线程：（FactoryUartThread.java文件）  
```java
public Thread changhongUartThread;  
changhongUartThread = new Thread(ChangHongUartTest);  
changhongUartThread.start();  
```

子线程主要干的事情是`handleChangHongFactoryCommand`:  
```java
private Runnable ChangHongUartTest = new Runnable() {  
    @Override  
    public void run() {  
        while (RunChangHongUartThread) {  
	    //...  
	    handleChangHongFactoryCommand(ReceiveBuffer);  
	    //...  
	}  
    }  
};  
```

`handleChangHongFactoryCommand`里面拿到正常的数据后，要刷新菜单，这个时候通过`handler`切换主线程去刷新菜单，如果有一些耗时的比如保存数据等操作，则还是在原来的子线程上操作。  
如果测试完毕，要关闭`FactoryAutomationService`服务了，会关子线程，  
```java
changhongUartThread.join();  
```
`join()`会阻塞主线程，等子线程结束，保证数据存储完成，主线程可以继续跑下去。  


#### 疑问  
在子线程`handleChangHongFactoryCommand`中有如下代码：  
```java
//写法1
saveChangHongColorTemp(getcommand[2]); //刷新菜单
/*
    刷新菜单的实现如下：
    Log.d(TAG, "System.currentTimeMillis() = " + System.currentTimeMillis());
    mHandlerComands.post(new Runnable() {
        @Override
	public void run() {
	    Log.d(TAG, "AAA");
	    Log.d(TAG, "System.currentTimeMillis() = " + System.currentTimeMillis());
	    showRGBValue(cTyte, r_value, g_value, b_value);
	}
    });
*/
mHandler.post(new Runnable() {
    @Override
    public void run() {
        Log.d(TAG, "BBB");
	GetSys().SetValueInt("Misc_PRE_SAVE_TEMP_VALUE", getcommand[2]&0xFF); //数据存储。按理说不应切到主线程？
    }
});
```

以上写法，菜单要等一会才会被刷新出来。  
稍作修改：  
```java
//写法2
saveChangHongColorTemp(getcommand[2]); //刷新菜单
/*
    刷新菜单的实现如下:
    Log.d(TAG, "System.currentTimeMillis() = " + System.currentTimeMillis());
    mHandlerComands.post(new Runnable() {
        @Override
	public void run() {
	    Log.d(TAG, "AAA");
	    Log.d(TAG, "System.currentTimeMillis() = " + System.currentTimeMillis());
	    showRGBValue(cTyte, r_value, g_value, b_value);
	}
    });
*/
GetSys().SetValueInt("Misc_PRE_SAVE_TEMP_VALUE", getcommand[2]&0xFF); //数据存储
```

菜单刷新就很及时，为什么？  
正常来说，耗时操作应该放在子线程，所以要按照写法2写代码。  
但是如果按照写法1的话，为什么菜单刷新变慢了？看打印是先打印AAA，再打印BBB，刷新菜单的操作没有被存储数据的操作阻塞，那么它是堵在哪？不懂啊。  

顺便一提，无论是写法1还是写法2，System.currentTimeMillis()时间差都是25左右。  

再修改下：  
```java
//写法3
saveChangHongColorTemp(getcommand[2]); //刷新菜单
/*
    刷新菜单的实现如下：
    showRGBValue(cTyte, r_value, g_value, b_value);
*/
mHandler.post(new Runnable() {
    @Override
    public void run() {
        GetSys().SetValueInt("Misc_PRE_SAVE_TEMP_VALUE", getcommand[2]&0xFF); //数据存储
    }
});
```

这样，菜单刷新跟写法2一样快。但是风险点在于子线程刷新UI，以及把耗时操作放到主线程。  

总结起来就是：刷新菜单和存储数据不在同一个线程，菜单就能正常刷新。菜单刷新在主线程，数据存储在子线程，所以写法2较为更为合理。  

问题在于，写法1是怎么阻塞的？可能要了解一下handler原理？  


#### :rocket:  
判断是否是主线程还是子线程：  
```java
public boolean isMainThread() {
    return Thread.currentThread().getId()) == Looper.getMainLooper().getThread().getId();
}
```

