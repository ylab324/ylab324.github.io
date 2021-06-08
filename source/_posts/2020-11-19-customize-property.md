---
title: '定义新的Property'
date: 2020-11-19
categories: Android
tags: [property]
---

自定义新属性。<dd23>  

<!-- more -->


:point_down:  
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

[参考]  
[system.prop新增自己的欄位](https://www.itread01.com/content/1546437184.html)  
[Android coredomain 如何使用自定义的 property type？](https://my.oschina.net/u/4339087/blog/3306403)  
