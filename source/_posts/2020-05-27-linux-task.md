---
title: 'Linux开机任务和定时任务'
date: 2020-05-27
categories: Linux
tags: [linux]
---


如题


<!-- more -->


### 开机任务
编辑`/etc/rc.local`文件，加上需要执行的命令。  

但是有的Linux版本没有这个文件，添加这个文件，依然能够生效：  
1. 添加这个文件  
    ```shell
    vim /etc/rc.local
    ```
    
    ```shell
    #!/bin/bash
    
    ifconfig ens33 192.168.52.128 netmask 255.255.255.0
    
    exit 0
    ```

2. 添加权限  
    ```shell
    chmod 755 /etc/rc.local
    ```
    
3. 设置启动  
    ```shell
    systemctl restart rc-local
    ```
    
4. 重启，开机任务有被执行  


### 定时任务
```shell
crontab -l #列出工作表里的命令
```

```shell
crontab -e #编辑定时工作表
```

`crontab`命令构成为`时间`+`动作`，时间有：  
- 分(0-59)
- 时(0-23)
- 日(1-31)
- 月(1-12)
- 周(0-6，其中0代表星期日)

操作符有：  
- \* 取值范围内的所有数字  
- / 每过多少个数字，表示频率  
- \- 表示范围  
- , 散列数字  

举个例子：  
```shell
*/360 * * * * /usr/sbin/ntpdate time.nist.gov # 时间同步，每隔六小时同步一次
2 * * * * /home/foo/test.sh # 这个cron作业会在每天各小时的第2分钟执行脚本test.sh
0 5,6,7 * * * /home/foo/test.sh # 每天的第5、6、7小时执行脚本
0 2 * * * /sbin/shutdown -h # 在每天凌晨2点钟关闭计算机
```
