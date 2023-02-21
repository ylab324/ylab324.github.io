---
title: '电视信号大致流程'
date: 2019-12-30
categories: TV
tags: [tv]
---


如下：  

<!-- more -->



图像：  
一般称Demod及其之前的部分为前端，Decoder及其之后的部分为后端。  
![video_block_diagram.jpg](https://i.loli.net/2019/12/30/obZB2aPhHzECQAp.jpg)  
个人理解：  
RF信号给到Tuner，Tuner调谐成中频信号给到Demod，Demod解调成TS流给到Decoder，如果是隔行信号，还会经过De-Interlacer去交错，然后经过Scaler缩放，输出到Panel。  
- - - 

声音：  
由前端输入模组，中间处理模组，后端输出模组构成：  
前端输入一般叫做Audio_Rx模块，此模块与各通道HW连接，通过设定MUX，接收各个通道送过来的资料；  
中间处理模组对收进来的资料处理、编解码动作及各种postprocess动作等等；  
后端输出模组一般叫做Audio_Tx单元，会去执行Vol、PeakControl、输出通道选择等动作。  
![Audio Processor Flow Chart.png](https://i.loli.net/2020/09/09/7hkbSzi3nql5XRc.png)  
![audio_block_diagram.jpg](https://i.loli.net/2019/12/30/NxED3dr7sCqSRp1.jpg)  
个人理解：  
声音由通道输入，MUX后，经由PreScale处理声音额外的正Gain，然后经过AVL、EQ、Treble/Bass等处理后，输出到后端，执行Volume、Power Limiter、Mute等，最后经过功放放大，输出到喇叭。  
