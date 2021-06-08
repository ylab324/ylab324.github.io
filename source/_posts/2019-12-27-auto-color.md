---
title: 'Auto Color'
date: 2019-12-27
categories: TV
tags: [tv,adc]
---

Auto Color，也有Auto ADC的叫法。分量和VGA属于高清模拟通道，在TV的流程里面会经过ADC处理。为了保证每台电视的一致性，在生产过程就会有一道ADC校正工序。但是目前的IC一般都会自动校正了。    

<!-- more -->


目的：  
1. 使信号的最低电平经ADC采集后输出的数位信号是0，信号的最高电平经ADC采集后输出的数位信号是255，这样可以得到最佳的对比度。  
2. 硬件电路差异，进入ADC采集器的RGB信号可能偏离标准信号，且偏离量可能不一样，故必须对RGB通道分别做Auto Color。  

原理：  
通过调整ADC Offset设定值，使信号的最低电平经ADC采集后输出的数位信号为0，通过调整ADC Gain设定值使信号的最高电平经ADC采集后输出的数位信号为255。  

![auto_adc.png](https://i.loli.net/2019/12/27/83pyl1QRjkstuZJ.png)  

![AutoColor.jpg](https://i.loli.net/2019/12/27/mJOnClSaRdHgzBG.jpg)  


VGA ADC Adjust:  
![VGA_ADC.jpg](https://i.loli.net/2019/12/27/pQ1Un9zBgfibN54.jpg)  


YPbPr ADC Adjust:  
![YPP_ADC_1.jpg](https://i.loli.net/2019/12/27/F5oqWwOavPR6YN2.jpg)  

![YPP_ADC_2.jpg](https://i.loli.net/2019/12/27/oh5dTI2NwXJn4fa.jpg)  


