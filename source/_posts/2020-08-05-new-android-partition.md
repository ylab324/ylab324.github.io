---
title: '新建分区'
date: 2020-08-05
categories: Android
tags: [android,linux]
---


可能有遗漏。  

<!-- more -->


添加分区，修改kernel/android/android-10/vendor/realtek/tool/image_file_creator/configs/Frigga_2851a.cfg，添加  
```
part14 = smarttv /smarttv ext4 package7/smarttv.tar.bz2 67108864 0 0 0
```

编译系统升级测试，这个时候，执行cat /proc/partitions应该能看到新添加的分区，但是df -h却看不到这个分区有挂载上来；  


需要一个挂载点，文件系统根目录需要一个smarttv目录，用于挂载，  

修改system/core/rootdir/Android.mk文件，添加如下：  
```makefile
LOCAL_POST_INSTALL_CMD += ; mkdir -p $(TARGET_ROOT_OUT)/smarttv
```

添加目录权限，修改device/realtek/common/sepolicy/file_contexts，添加如下：  
```
#smarttv partition
/smarttv(/.*)?          u:object_r:rtk_data_file:s0
```

光有挂载点还不够，挂载规则呢？需要修改device/realtek/common/root/rtd2851a/fstab.gsi  
添加如下：  
```
/dev/block/by-name/smarttv    /smarttv      ext4 noatime,defaults               defaults
```

还需要挂载动作，这个在device/realtek/common/root/rtd2851a/init.gsi.rc已经做好了：  
```
on fs
    mount_all /system/etc/fstab.gsi
```

系统起来需要执行一次restorecon_recursive /smarttv，用于重新加载sepolicy context，否则分区可能无法正常读写，会有  
> [2020/11/9 星期一 9:27:02] 01-01 00:00:19.826   338   338 D SmartTVService: SmartTvInit::SmartTvInit  
> [2020/11/9 星期一 9:27:02] 01-01 00:00:19.809   338   338 W vendor.realtek.: type=1400 audit(0.0:15): avc: denied { search } for name="/" dev="mmcblk0p14" ino=2 scontext=u:r:hal_smarttv_default:s0 tcontext=u:object_r:unlabeled:s0 tclass=dir permissive=0  

这种报错信息。  

修改device/realtek/common/root/rtd2851a/init.rtd2851a.rc  
```
on late-fs
    restorecon_recursive /tvconfigs
    restorecon_recursive /smarttv
```
注意restorecon_recursive这个动作要做在mount之后。  

编译系统升级测试，df -h应该能看到新建的分区挂载信息。  


完。  


[参考]  
[[AOSP]Android 9.0添加分区unlabeled的原因分析及解决办法](https://blog.csdn.net/ZC_25/article/details/104027721)
