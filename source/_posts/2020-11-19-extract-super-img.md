---
title: 'Android10 super.img解包方法'
date: 2020-11-19
categories: Android
tags: [android,image]
---

如题。  

<!-- more -->


对于super.img，这份Android10的代码提供了system/extras/partition_tools工具，mmm system/extras/partition_tools会在out/host/linux-x86/bin目录（全编代码后，out/host/linux-x86/bin/目录下的工具基本齐全了。登录终端后，进android代码根目录执行source build/envsetup.sh;lunch后，lpunpack这些命令都可以直接像系统命令跑 了，不用加路径）生成lpdump、lpflash、lpmake、lpunpack文件。其中：  
- lpdump displays pretty-printed partition metadata.  
- lpflash writes a non-sparse image from lpmake to a block device.   
- lpmake is a command-line tool for generating a “super” partition image.  
- lpunpack is a command-line tool for extracting partition images from super.  

直接解编译出来的super.img包看看：  
执行命令：  
```console
foo@bar:~$ lpunpack super.img  
```
得到  
> lpunpack E 11-19 17:09:36 233504 233504 reader.cpp:77] [liblp]Logical partition metadata has invalid geometry magic signature.  
> lpunpack E 11-19 17:09:36 233504 233504 reader.cpp:77] [liblp]Logical partition metadata has invalid geometry magic signature.  
> This image appears to be a sparse image. It must be unsparsed to be unpacked.  

解包失败，需要将super.img转换成unsparsed格式。  

Android提供了system/core/libsparse工具，mmm system/core/libsparse可在out/host/linux-x86/bin目录得到simg2img文件，这个工具可以将sparse格式转换成unsparsed格式。  

先转换格式：  
```console
foo@bar:~$ simg2img super.img super.raw.img  
```

再解包：  
```console
foo@bar:~$ lpunpack super.raw.img  
```

得到odm.img、product.img、system.img、vendor.img这4个文件。  

顺便用lpdump工具dump一下unsparsed格式的super.img，执行命令：  
```console
foo@bar:~$ lpdump super.raw.img  
```

可以得到：  

> Metadata version: 10.0  
> Metadata size: 592 bytes  
> Metadata max size: 65536 bytes  
> Metadata slot count: 2  
> Partition table:  
> ------------------------  
>   Name: system  
>   Group: realtek_dynamic_partitions  
>   Attributes: readonly  
>   Extents:  
>     0 .. 1734399 linear super 2048  
> ------------------------  
>   Name: vendor  
>   Group: realtek_dynamic_partitions  
>   Attributes: readonly  
>   Extents:  
>     0 .. 496975 linear super 1736704  
> ------------------------  
>   Name: product  
>   Group: realtek_dynamic_partitions  
>   Attributes: readonly  
>   Extents:  
>     0 .. 664919 linear super 2234368  
> ------------------------  
>   Name: odm  
>   Group: realtek_dynamic_partitions  
>   Attributes: readonly  
>   Extents:  
>     0 .. 1295 linear super 2899968  
> ------------------------  
> Block device table:  
> ------------------------  
>   Partition name: super  
>   First sector: 2048  
>   Size: 2147483648 bytes  
>   Flags: none  
> ------------------------  
> Group table:  
> ------------------------  
>   Name: default  
>   Maximum size: 0 bytes  
>   Flags: none  
> ------------------------  
>   Name: realtek_dynamic_partitions  
>   Maximum size: 2147483648 bytes  
>   Flags: none  
> ------------------------  

确实是有这4个分区。  

看下文件属性：  
```console
foo@bar:~$ file *  
```

> odm.img:       Linux rev 1.0 ext2 filesystem data, UUID=e1cf857c-334d-40cb-aa89-7634b14a39a2, volume name "odm" (extents) (large files) (huge files)  
> product.img:   Linux rev 1.0 ext2 filesystem data, UUID=9e1933ad-3f1b-4441-a314-f95987dc6c29, volume name "product" (extents) (large files) (huge files)  
> super.img:     data  
> super.raw.img: data  
> system.img:    Linux rev 1.0 ext2 filesystem data, UUID=3dab5a90-e84d-4e0c-a1e1-cc8779c3e0b2 (extents) (large files) (huge files)  
> vendor.img:    Linux rev 1.0 ext2 filesystem data, UUID=830ef149-3c4e-4455-ad55-f80faad5c8e1, volume name "vendor" (extents) (large files) (huge files)  

如果查看各个分区的文件？  
Linux rev 1.0 ext2 filesystem data 这个类型的文件可以直接mount：  
```console
foo@bar:~$ mkdir vendor_partition; mount vendor.img vendor_partition  
```

或者在Windows平台用7zip工具直接打开（右键->7-Zip->打开压缩包）。  

至此，算是解包完成了。  

最后贴一个纯Windows平台的教程：  
<iframe width="560" height="315" src="https://www.youtube.com/embed/J5cQdzivtXk" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>  

