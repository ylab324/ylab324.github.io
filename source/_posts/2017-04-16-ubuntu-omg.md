---
title: Ubuntu使用记录
date:   2017-04-16
categories: Linux
tags:  [linux,ubuntu]
---

记录一次Ubuntu初始化流程。

<!-- more -->

### 显卡驱动
- 检查你的电脑有哪些显卡  
lspci -k | grep -A 2 -i "VGA"  
- 查看Ubuntu正在使用哪块显卡  
System Settings > Details > Overview  
- 查看哪一个专有驱动是推荐安装的  
sudo ubuntu-drivers devices  
- 安装Nvidia专有显卡驱动  
终端执行software-properties-gtk打开GUI或执行sudo apt-get install nvidia-375直接安装  
- 切换显卡  
Dash中打开Nvidia X Server Settings或终端执行命令nvidia-settings，在PRIME Profiles切换  

### 系统基础配置
- 卸载多余的软件  
sudo apt-get remove libreoffice-common  
sudo apt-get remove unity-webapps-common  
sudo apt-get remove totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku landscape-client-ui-install onboard deja-dup imagemagick  
- 桌面布局  
gsettings set org.compiz.unityshell:/org/compiz/profiles/unity/plugins/unityshell/ launcher-minimize-window true  
gsettings set com.canonical.Unity.Launcher launcher-position Bottom  
- 更换源  
sudo vim /etc/apt/sources.list  
sudo apt-get update  
sudo apt-get upgrade  
- 安装基础软件  
  - 通过源直接安装  
  sudo apt-get install vim  
  sudo apt-get install samba  
  sudo apt-get install chromium  
  sudo apt-get install screenfetch  
  sudo apt-get install git subversion vpnc  
  sudo apt-get install axel uget aria2  
  sudo apt-get install vlc  
  sudo apt-get install okular  
  sudo apt-get install kchmviewer  
  sudo apt-get install mediainfo mediainfo-gui  
  sudo add-apt-repository ppa:webupd8team/sublime-text-3  
  sudo apt-get update  
  sudo apt-get install sublime-text  
  - 安装deb文件  
  [微信](https://github.com/geeeeeeeeek/electronic-wechat)  
  [网易云音乐](https://music.163.com/#/download)  
  sudo apt-get install libqgsttools-p1 libqt5multimedia5-plugins libqt5multimediawidgets5 libqt5libqgtk2  
  sudo dpkg -i netease-cloud-music_1.0.0_amd64_ubuntu16.04.deb  
  [搜狗输入法](https://pinyin.sogou.com/linux/?r=pinyin)  
  sudo apt install libopencc1 fcitx fcitx-libs fcitx-libs-qt fonts-droid-fallback  
  sudo dpkg -i sogoupinyin_2.1.0.0086_amd64.deb  
  [百度云盘](https://github.com/LiuLang/bcloud-packages)  
  sudo apt-get install gnome-icon-theme-symbolic python3-crypto python3-pyinotify python3-keyring  
  sudo dpkg -i bcloud_3.8.2-1_all.deb  
  [蓝灯](https://github.com/getlantern/lantern)  
  sudo dpkg -i lantern-installer-beta-64-bit.deb  
  [坚果云](https://www.jianguoyun.com/s/downloads/linux)  
  sudo apt-get install python-gtk2 python-notify default-jre-headless  
  sudo dpkg -i nautilus_nutstore_amd64.deb  
  [WPS](https://community.wps.cn/download/)  
  sudo dpkg -i wps-office_10.1.0.5672~a21_amd64.deb  
- [Gedit中文乱码](https://wiki.ubuntu.org.cn/Gedit%E4%B8%AD%E6%96%87%E4%B9%B1%E7%A0%81)  
sudo apt-get install dconf-editor  
运行dconf-editor  
展开/org/gnome/gedit/preferences/encodings  
Candidate Encodings的value配置为['GB18030', 'UTF-8', 'CURRENT', 'ISO-8859-15', 'UTF-16']  

### Jekyll环境
- sudo apt-get install ruby ruby-dev  
- gem sources --remove https://rubygems.org/  
- gem sources -a https://ruby.taobao.org/  
- gem sources -l  
- sudo gem install jekyll bundler  
- bundle install  
- [build&run](https://jekyllcn.com/)

### Append

-  **在Linux Mint 18.3上执行sudo gem install jekyll报错：**
            
> Building native extensions.  This could take a while...
> ERROR:  Error installing jekyll:
>	ERROR: Failed to build gem native extension.
> 
>   current directory: /var/lib/gems/2.3.0/gems/http_parser.rb-0.6.0/ext/ruby_http_parser
> /usr/bin/ruby2.3 -r ./siteconf20180226-49726-1ijhpau.rb extconf.rb
> creating Makefile
> 
> current directory: /var/lib/gems/2.3.0/gems/http_parser.rb-0.6.0/ext/ruby_http_parser
> make "DESTDIR=" clean
> 
> current directory: /var/lib/gems/2.3.0/gems/http_parser.rb-0.6.0/ext/ruby_http_parser
> make "DESTDIR="
> compiling ruby_http_parser.c
> In file included from /usr/include/ruby-2.3.0/ruby/ruby.h:36:0,
>                 from /usr/include/ruby-2.3.0/ruby.h:33,
>                 from ruby_http_parser.c:1:
> /usr/include/ruby-2.3.0/ruby/defines.h:26:19: fatal error: stdio.h: No such file or directory
> compilation terminated.
> Makefile:239: recipe for target 'ruby_http_parser.o' failed
> make: *** [ruby_http_parser.o] Error 1
>
> make failed, exit code 2
> 
> Gem files will remain installed in /var/lib/gems/2.3.0/gems/http_parser.rb-0.6.0 for inspection.
> Results logged to /var/lib/gems/2.3.0/extensions/x86_64-linux/2.3.0/http_parser.rb-0.6.0/gem_make.out   

解决方法：
sudo apt-get install build-essential patch ruby-dev zlib1g-dev liblzma-dev libsqlite3-dev

- **VMware Tools工具安装失败，无法复制粘贴（平台：Mint 18.3，VMware 11）**
> Makefile:120: recipe for target 'vmhgfs.ko' failed  

解决方法：
1. Make sure the updates are done:  
sudo apt-get update

2. Make sure git is installed  
sudo apt-get install git

3. Run the command to get the tools from repository.  
  sudo git clone https://github.com/rasa/vmware-tools-patches.git
  
4. cd to vmware-tools-folder  
   cd vmware-tools-patches
   
5. Run the patch  
   sudo ./download-tools.sh
   
6. Run the following patch  
   sudo ./untar-and-patch.sh
   
7. Run the complie.sh file  
sudo ./compile.sh  
[参考此贴](https://communities.vmware.com/message/2665867?tstart=0)

- **Windows无法访问Samba服务器，访问被拒绝**  
需要在Samba中创建用户。Ubuntu系统中的用户，和Samba用户是两回事，要将资源共享给系统中的某个用户，必须将该用户添加到Samba中。[参考](https://blog.sina.com.cn/s/blog_6c9d65a10100oobp.html)  
用到的命令：smbpasswd。[参考](https://man.linuxde.net/smbpasswd)  

- **安装oh-my-zsh步骤**
1. sudo apt-get install zsh //安装
2. zsh --version //确认是否安装成功
3. sudo chsh -s $(which zsh) //设置zsh为默认shell
4. 注销重新登录
5. echo $SHELL //确认zsh是否是默认SHELL
6. 使用curl方式安装oh-my-zsh：sudo sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
7. 配置：vim ~/.zshrc
8. 若有中文乱码，再配置：export LANG=zh_CN.UTF-8
9. 使配置生效：source ~/.zshrc
