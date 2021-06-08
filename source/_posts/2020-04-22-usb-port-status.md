---
title: 'USB Port Status'
date: 2020-04-22
categories: Linux
tags: [usb,linux,android]
---


需要检测平台的两路USB端口是否接入了设备。不太明白。  


<!-- more -->


网上翻了一遍，没找到安卓上层接口可以直接判断USB端口的状态，只能简单地获得USB设备的数量。类似这种:  
```java
public static String getFirstUsbStoragePath(Context mContext) {
    final StorageVolume[] volumes = StorageManager.getVolumeList(mContext.getUserId(), StorageManager.FLAG_FOR_WRITE);
    for (int i = 0; i < volumes.length; i++) {
        if (volumes[i].getPathFile().getAbsolutePath().contains("emulated")) {
            // ignore internal path
        } else {
            DebugLog.d("USB storage path:" + volumes[i].getPathFile() + "; index:" + i);
            return volumes[i].getPathFile().getAbsolutePath();
        }
    }
    return "";
}
```
`sda1`和`sdb1`也不能跟板卡的物理端口对应上，不知道怎么做。  

后来获悉原厂有如下做法：  
```java
public static boolean getUsb1StorageState() {
	File checkfile = null;
	checkfile = new File("/sys/bus/usb/devices/1-1:1.0");
	if(checkfile != null && checkfile.exists() && checkfile.isDirectory())
		return true;
	return false;
}

public static boolean getUsb2StorageState() {
	File checkfile = null;
	checkfile = new File("/sys/bus/usb/devices/1-2:1.0");
	if(checkfile != null && checkfile.exists() && checkfile.isDirectory())
		return true;
	return false;
}
```

实际拔插U盘：  
当板卡两路USB都没有接上U盘时：  
> console:/ $ ls sys/bus/usb/devices/ -l                                         
> total 0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-0:1.0 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-0:1.0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-3 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-3
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-3:1.0 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-3/1-3:1.0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-3:1.1 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-3/1-3:1.1
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-3:1.2 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-3/1-3:1.2
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 2-0:1.0 -> ../../../devices/platform/usb/18013400.ohci_top/usb2/2-0:1.0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 usb1 -> ../../../devices/platform/usb/18013000.ehci_top/usb1
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 usb2 -> ../../../devices/platform/usb/18013400.ohci_top/usb2

当板卡两路USB都接上U盘时：  
> console:/ $ ls sys/bus/usb/devices/ -l                                         
> total 0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-0:1.0 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-0:1.0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-1 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-1
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-1:1.0 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-1/1-1:1.0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-2 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-2
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-2:1.0 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-2/1-2:1.0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-3 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-3
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-3:1.0 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-3/1-3:1.0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-3:1.1 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-3/1-3:1.1
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 1-3:1.2 -> ../../../devices/platform/usb/18013000.ehci_top/usb1/1-3/1-3:1.2
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 2-0:1.0 -> ../../../devices/platform/usb/18013400.ohci_top/usb2/2-0:1.0
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 usb1 -> ../../../devices/platform/usb/18013000.ehci_top/usb1
> lrwxrwxrwx    1 0        0                0 Apr 22 07:54 usb2 -> ../../../devices/platform/usb/18013400.ohci_top/usb2

可看到两路USB分别对应`sys/bus/usb/devices/1-1`、`sys/bus/usb/devices/1-1:1.0`和`sys/bus/usb/devices/1-2`、`sys/bus/usb/devices/1-2:1.0`，并且能跟物理地址对应上。所以可以认为当U盘接入时，对应的文件就生成了。因此通过这个文件判断对应的USB Port是否有接上设备。  

