---
title: 关于Chromium的使用两则
date:  2017-08-20
categories: Love
tags: [google,chrome]
---

Chromium在WIN/Linux平台的一个配置。

<!-- more -->

#### 1、解决 Windows 中 Chromium “缺少 GOOGLE API 密钥” 和无法登陆账户的问题（转载）

一打开 Chromium，地址栏下方就提示 “缺少 Google API 密钥，因此 Chromium 的部分功能将无法使用”。这直接导致了无法在 Chromium 登录 Google 账户并同步各种信息。  

打开 windows 的 CMD 命令提示符，依次输入以下命令： 
 
```
setx GOOGLE_API_KEY "no"
setx GOOGLE_DEFAULT_CLIENT_ID "no"
setx GOOGLE_DEFAULT_CLIENT_SECRET "no"
``` 

其实就是设置这样三个环境变量，值均为“no”。然而这样只是消除了哪行提示而已（对于没有 Google 账户的“良民”们，或许有用），Google 账户还是无法登录。  

在 Debian Jessie 为 Chromium 设置 PepperFlashPlayer 的时候，在 /etc/chromium.d 目录中看到一个 apikeys 文件。打开它，看到里面的内容如下： 
 
```
# API keys assigned to Debian by Google for access to their services like sync and gmail.
export GOOGLE_API_KEY=”AIzaSyCkfPOPZXDKNn8hhgu3JrA62wIgC93d44k”
export GOOGLE_DEFAULT_CLIENT_ID=”811574891467.apps.googleusercontent.com”
export GOOGLE_DEFAULT_CLIENT_SECRET=”kdloedMFGdGla2P1zacGjAQh”
```

这不就是设置环境变量吗？于是将上述文章中提到的环境变量按照这个 apikeys 文件中的值进行设置，即在 CMD 中执行： 
 
```
setx GOOGLE_API_KEY AIzaSyCkfPOPZXDKNn8hhgu3JrA62wIgC93d44k
setx GOOGLE_DEFAULT_CLIENT_ID 811574891467.apps.googleusercontent.com
setx GOOGLE_DEFAULT_CLIENT_SECRET kdloedMFGdGla2P1zacGjAQh
```

再尝试打开 Chromium，发现提示消失了，Google 账户也能登录了。


#### 2、解决 Mint 中 Chromium 弹窗 “Enter password for keyring 'Default keyring' to unlock” 的问题

桌面版 Mint 使用 Chromium 会弹窗提示 “Enter password for keyring 'Default keyring' to unlock” ，输入 Mint 密码弹窗消失，怎么才能在每次使用 Chromium 的时候让这个弹窗消失？  

关闭 Chromium ，命令行下运行 seahorse ，找到 Passwords -> Default keyring ，右键选 Change Password ，把新密码改为空，重启系统。  

版本：Version 59.0.3071.109 (Developer Build) Built on Ubuntu , running on LinuxMint 18.2 (32-bit)
