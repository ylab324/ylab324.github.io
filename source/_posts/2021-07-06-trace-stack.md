---
title: 打印函数调用堆栈
date: 2021-07-06 23:35:54
categories: Android
tags: [stack,debug]
top:
---

有时候希望知道函数被哪里调用了，但是源码太多，逐个加打印肯定是不方便的，以下记录跟踪函数调用栈信息的方法。

<!-- more -->

### Native C++

1. 头文件包含

   ```c++
   #include <utils/CallStack.h>
   ```

2. 在需要打印堆栈的地方添加

   ```c++
   android::CallStack stack("CS");
   ```

   CS是在`logcat`中输出的`TAG`。

3. Android.mk添加

   ```makefile
   LOCAL_SHARED_LIBRARIES += libutilscallstack
   ```



### Native C

通过C语言调用C++的函数，即extern C。

在项目里添加一个c++文件callstack.cpp

```c++
#include <utils/CallStack.h>
#include "callstack.h"
extern "C" {
	void dumping_callstack(void)
	{
		android::CallStack stack("CS");
	}
}
```

添加一个callstack.h

```c++
#ifndef _CALLSTACK_H
#define _CALLSTACK_H
#ifdef __cplusplus
extern "C" {
#endif
	void dumping_callstack(void);
#ifdef __cplusplus
}
#endif
#endif
```

修改Android.mk

```makefile
LOCAL_SRC_FILES += callstack.cpp
LOCAL_SHARED_LIBRARIES += libutilscallstack
```

在native C里面包含头文件

```c
#include "callstack.h"
```

然后调用

```c
dumping_callstack();
```



### Java

```java
Exception e = new Exception("CS");
e.printStackTrace();
```

log在`logcat`中可以看到。



### Linux Kernel

```c
dump_stack();
```




