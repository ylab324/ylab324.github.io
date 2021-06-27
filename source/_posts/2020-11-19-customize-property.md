---
title: '定义新的Property'
date: 2020-11-19
categories: Android
tags: [property]
---

自定义新属性。<dd23>  

<!-- more -->


:point_down:  
### 步骤
希望新增自定义的属性，单纯使用  
```java
android.os.SystemProperties.set("json.smarttv.config.order.backlight.home","80");
```
会报错：  
> libc    : Unable to set property "json.smarttv.config.order.backlight.home" to "80": error code: 0x18  

需要在device/realtek/common/sepolicy/property_contexts定义自己的属性  
```
json.                           u:object_r:json_prop:s0  
```

在device/realtek/common/sepolicy/property.te定义新增加的json_prop域  
```
type json_prop, property_type;  
```

重新编译运行，可能还会有错。  
可能还需要补充权限，比如：  
修改device/realtek/common/sepolicy/hal_smarttv_default.te：  
```
set_prop(hal_smarttv_default, json_prop);  
get_prop(hal_smarttv_default, json_prop);  
```
修改device/realtek/common/sepolicy/system_app.te：  
```
allow system_app json_prop:file { read open getattr map };  
```

### 参考链接  
[system.prop新增自己的欄位](https://www.itread01.com/content/1546437184.html)  
[Android coredomain 如何使用自定义的 property type？](https://my.oschina.net/u/4339087/blog/3306403)  

### 问题
编译`user`版本的image，添加的`property`属性无法在console通过`getprop`获得，但是查看out/target/product/RealtekATV/system/build.prop文件却有这些属性，可能与`selinux`规则有关，把这些属性修改到system/sepolicy/private/property_contexts
```bash
ro.yndx.build           u:object_r:system_prop:s0
yndx.config             u:object_r:system_prop:s0
```
OK，但是不知道为什么：  
1. [编写 SELinux 政策](https://source.android.google.cn/security/selinux/device-policy?hl=zh-cn)  
2. [SELinux 安全上下文](https://lishiwen4.github.io/android/selinux-security-context)  
3. [permissive domains not allowed in user builds](https://gaozhipeng.me/posts/permissive-domain-in-userbuild/)  
4. [Android 9 SELinux](https://www.jianshu.com/p/e95cd0c17adc)  
5. [自定义 SELinux](https://source.android.com/security/selinux/customize?hl=zh-cn%5C)  


### 深入理解SELinux SEAndroid  
* [第一部分](https://blog.csdn.net/Innost/article/details/19299937)  
* [第二部分](https://blog.csdn.net/Innost/article/details/19641487)  
* [第三部分](https://blog.csdn.net/Innost/article/details/19767621)  
