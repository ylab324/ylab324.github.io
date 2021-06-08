---
title: 'Android TV 蓝牙遥控适配'
date: 2020-09-17
categories: Android
tags: [hid,bluetooth,android]
---

HID(Human Interface Device)设备，即人机交互设备。常见的有鼠标、键盘、游戏手柄等。一般有线方式都是通过USB连线连接到机器设备，作为用户输入设备。在蓝牙技术中，HID设备的接入就是无线的。所以客户的蓝牙遥控器也是HID设备，本文记录蓝牙遥控器适配方法。  

<!-- more -->


通过客户提供的文档或者`getevent`命令（遥控跟TV需要先匹配上）确定遥控的HID码值（不是红外码值），比如按下语音键会得到如下信息：    
`getevent`:  
> /dev/input/event3: 0004 0004 000c0221  
> /dev/input/event3: 0001 00d9 00000001  
> /dev/input/event3: 0000 0000 00000000  
> /dev/input/event3: 0004 0004 000c0221  
> /dev/input/event3: 0001 00d9 00000000  
> /dev/input/event3: 0000 0000 00000000  

`getevent -l`:  
> /dev/input/event3: EV_MSC       MSC_SCAN             000c0221  
> /dev/input/event3: EV_KEY       KEY_SEARCH           DOWN  
> /dev/input/event3: EV_SYN       SYN_REPORT           00000000  
> /dev/input/event3: EV_MSC       MSC_SCAN             000c0221  
> /dev/input/event3: EV_KEY       KEY_SEARCH           UP  
> /dev/input/event3: EV_SYN       SYN_REPORT           00000000  

通过上述打印可以得到如下信息：  
0221是这个按键的HID码值，page类型是000c，00d9是这个Linux的键值（对应了KEY_SEARCH这个名字，这个名字起得很疑惑，明明是语音功能:joy:）。  

KEY_SEARCH是在kernel/linux/linux-4.14/include/uapi/linux/input-event-codes.h这个文件定义的：  
> #define KEY_SEARCH              217  

十进制的217就是等于十六进制的d9。  

通过kernel/linux/linux-4.14/drivers/hid/hid-input.c这个文件:  
> case 0x221: map_key_clear(KEY_SEARCH);          break;  

把HID码值跟Linux键值匹配在一起。  

蓝牙键值有不同类型，“07”代表普通蓝牙键值，“0C”代表多媒体键值，“0C”类型需要修改hid-input.c的`HID_UP_CONSUMER`，“07”类型则需要修改`hid_keyboard[256]`。  


现在知道了HID码值（0x0221）及其对应的Linux键值(0xd9)(217)了，但是功能实现还需要Android键值。Linux键值与Android键值是通过`kl`文件匹配的。怎么知道是哪个kl文件？  

通过`cat /proc/bus/input/devices`命令确认遥控器的`Vendor`ID和`Product`ID  
> I: Bus=0005 Vendor=1d5a Product=c081 Version=0000  
> N: Name="H016A001"  
> P: Phys=  
> S: Sysfs=/devices/virtual/misc/uhid/0005:1D5A:C081.0001/input/input3  
> U: Uniq=20:19:10:23:00:f6  
> H: Handlers=event3  
> B: PROP=0  
> B: EV=12001f  
> B: KEY=1 8000000 0 3007f 0 0 0 0 403ffff 17aff32d bf540446 0 0 1 130f97 8b17e007 ffff7bfa d9415fff febeffd7 ffefffff ffffffff fffffffe  
> B: REL=40  
> B: ABS=1 0  
> B: MSC=10  
> B: LED=3ff  

所以需要的kl文件是：Vendor_1d5a_Product_c081.kl  
该文件有如下信息：  
> key 217   VOICE_ASSIST  

Android Framework会把VOICE_ASSIST转化成Android键值，实现对应的功能。  
Framework改到的文件有如下4个：  
![android_keycode.png](https://i.loli.net/2020/09/17/x5E4wOcneVz7BJ9.png)  

到此，基本适配完成了。  


另外，dumpsys input还可以得到如下信息：  
> 4: H016A001  
> Classes: 0x80000121  
> Path: /dev/input/event3  
> Enabled: true  
> Descriptor: 83b91aadd3b6814a870f118082cde683c34f199e  
> Location:  
> ControllerNumber: 0  
> UniqueId: 20:19:10:23:00:f6  
> Identifier: bus=0x0005, vendor=0x1d5a, product=0xc081, version=0x0000  
> KeyLayoutFile: /system/usr/keylayout/Vendor_1d5a_Product_c081.kl  
> KeyCharacterMapFile: /system/usr/keychars/Generic.kcm  
> ConfigurationFile:  
> HaveKeyboardLayoutOverlay: false  

