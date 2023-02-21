---
title: 'dlopen failed处理'
date: 2020-07-15
categories: Android
tags: [android]
---


如题


<!-- more -->


使用的是Android7.1平台。  
应用本来是系统预置的，但是有些客户需要做推送。使用U盘安装测试的时候，闪退。AndroidRuntime报错：  
> java.lang.UnsatisfiedLinkError: dlopen failed: library "/system/lib/librtk-mediaplayer_jni.so" needed  
> or dlopened by "/system/lib/libnativeloader.so" is not accessible for the namespace "classloader-namespace"  

找到librtk-mediaplayer_jni.so在系统中的/system/lib目录，修改代码/system/core/libnativeloader/native_loader.cpp文件，把  
```cpp
static constexpr const char* kWhitelistedDirectories = "/data:/mnt/expand";
```
改成  
```cpp
static constexpr const char* kWhitelistedDirectories = "/data:/mnt/expand:/system/lib";
```
编译、升级、测试，运行时有新的报错提示：  
> java.lang.UnsatisfiedLinkError: dlopen failed: library "libutils.so" not found  

找到系统的/system/etc/public.libraries.txt文件，发现其中没有libutils.so，通过  
```console
foo@bar:~$ su
foo@bar:~$ mount -o rw,remount /
foo@bar:~$ mount -o rw,remount /system
foo@bar:~$ sed -i '1alibutils.so' /system/etc/public.libraries.txt
foo@bar:~$ sync
foo@bar:~$ reboot
```
修改测试，继续有libnativehelper.so、librtk-mediaplayer.so找不到的报错，依次添加到public.libraries.txt，最后应用能正常使用了。  
接着修改代码/system/core/rootdir/etc/public.libraries.android.txt，添加libutils.so、libnativehelper.so和librtk-mediaplayer.so，重新编译、升级、测试，运行OK。  

### 参考
- [How to modify “vendor/etc/public.libraries.txt” when building pure AOSP](https://stackoverflow.com/questions/54234905/how-to-modify-vendor-etc-public-libraries-txt-when-building-pure-aosp)  
- [【Android N兼容问题】Android N上系统预置应用调用so库失败问题的看法](https://www.cnblogs.com/Qunter/p/7485090.html)  
- [android N : java.lang.UnsatisfiedLinkError](https://www.jianshu.com/p/a4af2bdcc3c0)  
- [Framework基础：Android N 公共so库怎么定义呢？](https://www.jianshu.com/p/4be3d1dafbec)  
- [Android 7.0以后 .so link 加载链接过程中 dlopen failed 问题](https://blog.csdn.net/duan_xiaosu/article/details/81031174)  
- [Android 动态链接库隔离](https://segmentfault.com/a/1190000021461854)  
