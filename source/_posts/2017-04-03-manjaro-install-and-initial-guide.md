---
title:  How to install and initialize Manjaro OS
date:   2017-04-03
categories: Linux
tags: [linux,manjaro]
---


记录一次Manjaro安装流程。

<!-- more -->

### Installation
一、安装系统  
1. df -->确认U盘名称和路径  
2. umount sdb1 -->卸载已挂载的U盘  
3. dd if=/home/manjaro-xfce-17.0-stable-x86_64.iso of=/dev/sdb -->dd命令中的目标是sdb，没有标号  
4. install boot non-free -->对于双显卡驱动，安装的时候boot选择non-free，如果boot default的话，装好系统以后也可以在设置里安装bumblebee  

### Initialization
二、系统初始配置  
1. sudo pacman-mirrors -g -->排列源  
2. sudo pacman-optimize&&sync -->同步，碎片整理。固态硬盘不能用pacman-optimize  
3. sudo pacman -Syyu -->升级系统  
4. sudo nano /etc/pacman.conf -->添加ArchLinuxCN源  
5. sudo pacman -Syyu -->同步  
6. sudo pacman -S archlinuxcn-keyring -->导入GPG Key  

### Basic Tools
三、安装基础软件  
1. sudo pacman -Syu yaourt -->Archlinux的软件包管理工具是Pacman，而yaourt工具可以安装AUR(Archlinux官方不支持)的软件  
2. yaourt screenfetch -->一个能够在终端显示系统/主题信息的命令行脚本  
3. yaourt vim -->文本编辑工具  
4. yaourt lantern -->科学上网工具  
5. yaourt sogou -->搜狗输入法  
	进入fcitx config gui异常，还需安装fcitx-configtool；  
	vim ~/.xprofile,添加  
	export GTK_IM_MODULE=fcitx  
	export QT_IM_MODULE=fcitx  
	export XMODIFIERS="@im=fcitx"  
	注销重新登录系统  
	还是不行，执行fcitx -diagnose命令诊断，然后执行对应的修复  
6. yaourt uget -->Firefox可以通过添加FlashGot插件把uget设置为其默认下载管理器  
7. yaourt qq -->安装QQ  
8. yaourt netease -->安装网易云音乐  
9. yaourt wechat -->安装微信  
10. 安装zsh  
	安装 zsh: sudo pacman -S zsh  
	配置 oh-my-zsh: sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"  
	更换默认的 shell: chsh -s /bin/zsh  
	重启  
