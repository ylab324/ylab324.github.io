---
title: '一个简单的Android功能需求'
date: 2020-03-04
categories: Android
tags: [android,code]
---

菜鸡历险记。  

<!-- more -->



需要加一个功能，包含两部分，一是工厂菜单也可以手动检索U盘中的OTA包，并且进行升级，二是插入U盘的时候，平台能自动识别U盘中的OTA包，并且可以升级。  

第一部分，具体就是，按菜单上升级选项的时候，去确认U盘是否存在指定文件名的文件，如果存在，把这个文件拷贝到系统目录`/data`，拷贝的时候弹出一个进度条`ProgressDialog`，等拷贝完成，进度条消失，最后调用系统接口`RecoverySystem.installPackage`升级OTA包。以上的动作基本都在Fragment菜单页通过Handler`sendEmptyMessageDelayed`的方式处理，验证也OK。  

第二部分：  
- 阶段A  
做插U盘识别的，在广播接收器里面，如果有U盘mount上了，就去查这个东西，然后调用Fragment的函数，最后也想通过Fragment的Handler把这些事情处理完。  
本想通过这样，可以达到Fragment上一样的效果，既能执行拷贝等动作也能刷新进度条菜单。但是发生了很多`CRASH`，比如调用的很多函数里需要用到`Context`，所以我又在`BootCompleteReceiver`（名字是这样写，但是其实里面有判断`android.intent.action.MEDIA_MOUNTED`的）把`Context`传到`Fragment`，这都还好，最麻烦的就是，我在`BootCompleteReceiver`里无论通过静态的方式还是new的方式调用`Fragment`的方法，都用不了控件`ProgressDialog`，提示`Unable to add window`，`Dialog`只能在`Activity`中显示！到这，不知道怎么在广播里更新UI了。这样的话，就只有COPY和升级的功能，但是看不到整个过程的进度条，这样看起来很奇怪。要不在`BootCompleteReceiver`先把`Activity`启动起来，再接着做事情？感觉越来越奇怪了。还有遇到的就是调用`Fragment`方法的时候，报错`Attempt to invoke virtual method on a null object reference`，感觉也是没有对象，所以一些方法用了会有问题之类的问题，麻烦啊。  

- 阶段B  
那咋搞？我就把`Fragment`里面的`Handler`处理方式差不多复制一份搞到`BootCompleteReceiver`里面，动作都写在`BootCompleteReceiver`，不通过别的地方调用了，`ProgressDialog`也是，但是操作控件的时候还是没有`Activity`啊。但是插入U盘之后`AlertDialog`是怎么出来的呢？原来`AlertDialog`有`WindowManager.LayoutParams.TYPE_SYSTEM_ALERT`属性，这应该是系统层面的东西吧，不依赖`Activity`。那有没有系统层面的进度条？好像没有。但是网上有说把`ProgressDialog`设置成`TYPE_SYSTEM_ALERT`或者`TYPE_TOAST`，一试确实OK。但是，这只是菜单而已。菜单运行良好了，但是实际上，OTA包体积很大，在`BootCompleteReceiver`跑拷贝动作的时候，发生`ANR`，最终没有跑到`RecoverySystem.installPackage`那里去，为啥，拷贝是在`BootCompleteReceiver`里面的`Handler`做的，网上查，说没有`new Thread`直接创建的`new handler`是`UI线程`，`UI线程`不能做耗时操作，但是为啥在阶段A的时候，在`Fragment`的`Handler`里面拷贝OTA包却没有问题？在广播里的`Handler`却有问题？不明白。查到说[一个进程如果正在执行BroadcastReceiver的 onReceive() 方法，就会被当做一个前台进程，不易被系统杀死。当 onReceive() 执行完毕，BroadcastReceiver 就不再活跃，其所在进程会被系统当做一个空进程，随时有可能被系统杀死。](https://www.jianshu.com/p/34c8d5384f94)可能就是这个原因吧。`onReceive`都跑完了，`Handler`的拷贝动作还没有跑完。但是`Fragment`的话，一直在前台，所以不会被杀死。  

- 阶段C  
改成`new Thread`行不行呢，没有尝试，估计不行。`Fragment`写一次，`BootCompleteReceiver`写一次，太蠢了。按照[《如何在BroadcastReceiver中执行耗时操作》](https://www.jianshu.com/p/34c8d5384f94)这里说的，用`IntentService`，很好用，但是操作`ProgressDialog`还是用了`TYPE_SYSTEM_ALERT`属性。  



但是如果启动了一个服务，这个服务没有对应的`Activity`，而服务需要刷新UI，怎么搞比较好呢？不能都用`TYPE_SYSTEM_ALERT`吧。  

纯小白，真惨啊。  
