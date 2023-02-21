---
title: 'Android系统属性'
date: 2020-05-27
categories: Android
tags: [android]
---


来源网络

<!-- more -->


### 属性的分类
ro开头的属性，表明该属性是只读的，一旦设置，不能更改。  
persist开头的属性，表明该属性是可修改的，以persist开始的属性会在/data/property存一个副本。也就是说，如果程序调property_set设了一个以persist为前缀的属性，系统会在/data/property/persistent_properties记录这个属性，重启之后这个属性还会存在。  
ctl.start和ctl.stop属性。用来启动和停止init.rc中定义的服务。  
其他格式的属性都可修改，重启不保存，因为属性是在内存里存的，所以重启后这个属性就没有了。  

### 属性的设置和获取
Java  
```java
import android.os.SystemProperties;

SystemProperties.set("persist.sys.miracast.hdcp2",”1”);

if(SystemProperties.get("persist.sys.miracast.hdcp2").equals("1")) {
    Log.d(TAG, "666");
}
```

Native  
```c
#include <cutils/properties.h>

char value[PROPERTY_VALUE_MAX];
property_get("persist.sys.miracast.hdcp2", value, "1");
property_set("persist.sys.miracast.hdcp2", value);
```

Shell  
```shell
setprop persist.sys.miracast.hdcp2 0
getprop persist.sys.miracast.hdcp2
```

### 属性的权限
程序要正确获取/设置权限，需要有system权限。  
1. 在AndroidManifest.xml中，在manifest加入android:sharedUserId="android.uid.system"  
2. 在Android.mk中，將LOCAL_CERTIFICATE := XXX修改成LOCAL_CERTIFICATE := platform  

运行时候可能还会有“libc access denied finding property android”的问题，需要把相关avc denied的报错信息修掉  

