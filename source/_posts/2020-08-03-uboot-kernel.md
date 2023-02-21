---
title: 'boot到kernel的传递'
date: 2020-08-03
categories: Linux
tags: [linux,boot,kernel,cmdline]
---


理解不深，简单记录一下。


<!-- more -->


新IC出来了，要移植配置，有个问题是，增减遥控器，无作用。遥控解码是在内核处理的，debug发现：配置里没有选上的遥控器，内核也有在解析，因为ir_table用的是内核里配置的，新ir_table不是在内核里面配置的，所以我们分离出来的定制配置没有效果，所以要把内核的ir_table改成由我们定制的配置的ir_table，那要怎么做呢？定制的新ir_table在上电的时候，在boot被解析，然后被填到`arch/arm/include/asm/arch-rtk/system.h`的这个地址`POWER_ON_IR_TABLE_ADDR`。这个地址出干嘛的：  
```c
/* Memory for power on music, video, and image. Include stream buffer and decode buffer */
// ...
#define POWER_ON_IR_TABLE_ADDR                  (0x1ffff000)    // 0x1ffff000 ~ 0x20000000
#define POWER_ON_IR_TABLE_SIZE                  (0x1000 - 128 - 8)              // 4k
#define POWER_ON_KEYPAD_TABLE_ADDR              (POWER_ON_IR_TABLE_ADDR + POWER_ON_IR_TABLE_SIZE)
#define POWER_ON_KEYPAD_TABLE_SIZE              (128 + 8)
// ...
```
这个是共享内存的物理地址，内核可以从这里拿到东西。  

但是内核怎么知道地址是多少？强制定义也可以（感觉不会有问题）。这里是通过`cmdline`把东西传给内核，看看boot的启动参数：  
```console
console:/ # cat /proc/cmdline                                                  
androidboot.console=ttyS1 console=ttyS1,115200 androidboot.dtbo_idx=0 bufsize=80000  envp=e4100 wdt=<NULL>  flashtype=emmc mmcparts=rtkemmc:304927k,2097152k(/super),4718592k(/userdata),281808k(/cache),1024k(/persist),1024k(/misc),16384k(/metadata),32768k(/boot),32768k(/recovery),8192k(/dtbo),16384k(/tvconfigs),32768k(/tvdata),24576k(/impdata),1024k(/vbmeta),65536k(/smarttv)  androidboot.boot_devices=18010800.emmc loop.max_part=7 buildvariant=userdebug initrd=0x10000000,0xbd000 keepinitrd reclaim=54M@40M last_image=8M@160M VIP=2M@766M OD=8M@768M bootcode_git_version=c422e6b no_console_suspend androidboot.vbmeta.device=179:13 androidboot.vbmeta.avb_version=1.1 androidboot.vbmeta.device_state=locked androidboot.vbmeta.hash_alg=sha256 androidboot.vbmeta.size=3968 androidboot.vbmeta.digest=7d27ba8f742e398b5827d4a1ab61aec26e4f0ee5192105877cad1e8ebb960359 androidboot.vbmeta.invalidate_on_error=yes androidboot.veritymode=enforcing androidboot.verifiedbootstate=green androidboot.hardware.sku=ATV00003919R01 loglevel=4 earlyprintk androidboot.bootreason=watchdog chip=RTD2851A chip_model=4K androidboot.serialno=001020304050 irda=1-hk irda_powerup=74,221,189,16e,2f5,2f6 irda1=7-hk dvfs_low=0xC6 dvfs_high=0xFD rtk_rcs=0x4eff000 ir_table=0x1ffff000
```

可以找到ir_table=0x1ffff000，内核怎么做？  
通过`early_param`解析拿到地址：  
```c
static int __init venus_ir_input_table_addr_parse(char *options)
{
    unsigned long ir_table_phy_address = 0;
    if(options == NULL)
        return 0;

    if (sscanf(options, "%lx", &ir_table_phy_address) != 1)
        return 0;

    g_ir_boot_memory_address = ir_table_phy_address;
    IR_INFO("g_ir_boot_memory_address == %lx\n", g_ir_boot_memory_address);
        return 0;
}

early_param("ir_table", venus_ir_input_table_addr_parse);
```

把物理地址转换成虚拟地址，并拿到ir_table：  
```c
void __init venus_ir_input_early_init(void)
{
    unsigned int *ir_boot_table = NULL;

    //carvedout_buf_query(CARVEDOUT_IR_TABLE, &ir_boot_table);
    if(g_ir_boot_memory_address) {
        ir_boot_table = (unsigned int *)phys_to_virt (g_ir_boot_memory_address);
	if(!ir_boot_table) {
	    g_ir_boot_memory_address = 0;
	    return;
	}
	IR_INFO("venus_ir_input__table_parse: %px, %x,%d\n", ir_boot_table, ir_boot_table[0], ir_boot_table[1]);
	rwlock_init(&g_ir_user_key_table.lock);
	g_ir_user_key_table.keys = (IR_USER_KEY *)(ir_boot_table + 2);
	if(ir_boot_table[0] == 0x49525442 &&  ir_boot_table[1] > 0 && ir_boot_table[1] <= MAX_IR_USER_KEY_NUM) {
	    g_ir_user_key_table.size = ir_boot_table[1];
	} else {
	    g_ir_user_key_table.size = 0;
	}
	g_ir_user_key_table.is_init = 1;
    }
}
```

### 参考  
- [高通平台aboot通过shared memory保存uart log到kernel](https://blog.csdn.net/RyanLiu_/article/details/78925515)  
- [linux驱动——cmdline原理及利用](https://blog.csdn.net/sgmenghuo/article/details/41251739?utm_source=copy)  
- [linux kernel的cmdline參数解析原理分析](https://www.cnblogs.com/tlnshuju/p/6851812.html)  
- [谈高端内存和低端内存](https://blog.csdn.net/YuZhiHui_No1/article/details/46711601)  
- [ioremap 和 phys_to_virt区别](https://blog.csdn.net/tienham/article/details/9493615)  
- [Linux驱动修炼之道-内存映射 mmap()/phys_to_virt()](https://blog.csdn.net/angle_birds/article/details/8804033)  
- [虚拟地址转换为物理地址](https://wenku.baidu.com/view/0319a8c408a1284ac85043d4.html)  

