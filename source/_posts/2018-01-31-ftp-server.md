---
title: Linux下搭建FTP服务器
date: 2018-01-31
categories: Linux
tags: [ftp,linux]
---

前前后后配了好几次FTP Server，每次都要重新开始的样子各种百度谷歌。先记一次留着下次用。

<!-- more -->

环境：Ubuntu 16.04 i386

### 安装软件和添加用户
1. sudo apt-get install vsftpd //安装vsftpd
2. vsftpd -version //查看vsftpd版本
3. sudo mkdir /home/ftp_root -p //创建FTP服务器工作目录ftp_root //注意目录权限
4. sudo useradd -d /home/ftp_root -s /bin/bash xxx //新建FTP用户xxx
5. sudo passwd xxx //为新建的FTP用户xxx设置密码 //可用cat /etc/passwd可以查看当前系统用户

### 配置
1. vim /etc/vsftpd.conf  
```bash
listen=YES

listen_ipv6=NO

anonymous_enable=YES
anon_root=/home/ftp_root/ftp_anonymous

local_enable=YES

write_enable=YES

local_root=/home/ftp_root

local_umask=022

dirmessage_enable=YES

use_localtime=YES

xferlog_enable=YES

connect_from_port_20=YES

xferlog_file=/var/log/vsftpd.log

ftpd_banner=Welcome to blah FTP service.

chroot_local_user=YES
chroot_list_enable=YES

chroot_list_file=/etc/vsftpd.chroot_list

secure_chroot_dir=/var/run/vsftpd/empty

#pam_service_name=vsftpd
pam_service_name=ftp

rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO

userlist_enable=YES
userlist_deny=NO
userlist_file=/etc/vsftpd.user_list
```

2. vim /etc/vsftpd.user_list
3. vim /etc/vsftpd.chroot_list
4. 其他
- 防火墙：直接关闭或者开放端口
- SeLinux：关闭
- 添加开机自启动


### 启动vsftpd服务
sudo service vsftpd start

### 登陆
方式有很多种，比如ssh、浏览器、Windows资源管理器添加FTP站点、FTP客户端工具等都可以。  
我在Windows下用客户端工具FileZilla，好处是文件共享方便的同时是能直观看到log信息。



#### 遇到的问题
1. 启动vsftpd服务后查看状态，发现启动失败
                      
> Active: failed (Result: exit-code)  
> (code=exited, status=2)  
               
原因：不明  
解决方法：将 vsftpd.conf 里面的 listen 从 NO 改成 YES  

2. 331和530错误，且匿名用户能登陆，但是用户账户不能登陆
                      
> 331 Please specify the password.  
> 530 Logiin incorrect.  
                                   
原因：“By default vsFTPd uses the file /etc/pam.d/vsftpd. This file by default requires FTP users to have a shell listed in /etc/shells and requires them not to be listed in /etc/ftpusers. If you check those 2 things your probably find what the problem is.”  
解决方法：将 vsftpd.conf 里面的 pam_service_name 从 vsftpd 改成 ftp  

3. 500错误
                       
> 500 OOPS: cannot change directory  
                
原因：未明，权限问题？普通用户不能进入到 /root 目录？  
解决方法：修改 /etc/passwd ，把FTP工作目录从 /root/ftp_root 改为 /home/ftp_root  

4. root用户能上传文件，但是xxx用户不行
                        
> 227 Entering Passive Mode  
> 553 Could not create file  
                         
原因：未明  
解决方法：无解

#### 其他
1. [Linux useradd 与 adduser的区别， /sbin/nologin 与 /bin/bash](https://blog.csdn.net/danson_yang/article/details/65629948)
2. 主动模式与被动模式
- [参考链接1](https://www.cnblogs.com/ShaYeBlog/p/5897303.html)
- [参考链接2](https://www.360doc.com/content/14/1225/15/15077656_435676206.shtml)
- [参考链接3](https://blog.csdn.net/huanggang028/article/details/41248663)
- [参考链接4](https://blog.csdn.net/u010154760/article/details/45458219)
- [参考链接5](https://www.cnblogs.com/xiaohh/p/4789813.html)
