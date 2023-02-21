---
title: 红外解码之Toshiba协议简单记录
date:   2018-08-22
categories: TV
tags:  [rda,tv,toshiba,remote,ir]
---

之前给创维做红外白平衡调试的几点记录。  

<!-- more -->

### 需求背景
客户出大货前需要做白平衡调试，上一年还是用串口来调试，今年全球工厂都用红外来调试了，想不清楚为什么要改用红外来代替串口，感觉用串口更可靠一点。以前的做法是直接用测试主机接到我的TV平台，测试主机通过串口直接给TV下值，现在的做法是测试主机通过串口发给一个红外发射器（看了一下是用STM32做的，不同的红外协议使用的发射器还不一样），发射器再把值通过红外的方式发给TV。  

串口调试图例:  
![串口调试图例](https://i.loli.net/2018/08/22/5b7cde4b55468.jpg)  
红外调试图例:  
![红外调试图例](https://i.loli.net/2018/08/22/5b7cde6b27243.jpg)

### 设计规范
![红外白平衡设计规范](https://i.loli.net/2018/08/22/5b7ce03d4c1d7.jpg)

### 解题思路
东芝协议除了引导码和结束位，一帧数据刚好是4个字节，包括两个字节客户码和两个字节的键值。  
![frame](https://i.loli.net/2018/08/22/5b7cf86885007.jpg)  
而目前代码中对于一帧数据的用法是，客户码2个字节有利用到，但是键值只用了一个字节，反码的那个字节没有利用。原厂的解释是反码部分只是用于验证，目前规格没有利用这部分数据，一直也不影响遥控器使用。  
我们只要把这4个字节的内容都解出来，就可以实现客户功能

### 解码算法
对于“{0x0e0e, 0x11, UI_EVENT_MENU}”这个按键，客户码等于0x0e0e，Keycode等于0x11ee。按下此按键时可以看到如下打印信息：  
>  [IR]IR_ISR	IR_len:33	reg0C:0x000010ef	reg54:0x0000e0e1  
>  [IR]0	IR:00008877	tmpData:00000000	IR_Custom:00007070  
>  [IR]1	IR:443b8000	tmp:000011ee	tmp_Custom:00000e0e  
>  [IR]1st reg_single_sts received!!  
>  [IR]Single Start Address  Address_  Cmd: 0xe,0xe, 0x11,0xee!!  
>  [IR]IR_ISR	IR_len:2	reg0C:0x00000003	reg54:0x00000000  
>  [IR]0	IR:00000001	tmpData:00000000	IR_Custom:00000000  
>  [IR]1	IR:00008000	tmp:00000080	tmp_Custom:00000000  
>  [IR]repeat key received!!  
> _APP_GUIOBJ_PopMsg_OnRelease is called  
>  [IR]Single_end received!!  

而代码中表现如下：  

```c
void sisir_toshiba_ParseData(UINT8* mmiobase, IR_IOC_IOData* dataBuf)
{	
	volatile UINT32 IR_Data, IR_Custom, IR_bit_len, tmp_data, tmp_Custom, i;
	volatile UINT16 tmp_1 = 0xffff;
	volatile UINT16 tmp_2 = 0xffff;
	datBuf = dataBuf;
	IR_Data = irdatas.Value & 0xffff;

	IR_Custom = GetMMIO_DWORD(mmiobase, 0x54);	//read from IR decoder recevied custom code

	IR_Custom = IR_Custom & 0x1ffff;

	IR_bit_len = (GetMMIO_DWORD(mmiobase, 0x50) & 0x3f00) >> 8; //read from IR decoder recevied bit length
	DBG_MSG1(DBGCFG_IR, "[IR]IR_ISR\tIR_len:%d\treg0C:0x%08x\treg54:0x%08x\n", IR_bit_len, IR_Data, IR_Custom);
	IR_Data = IR_Data >> 1;
	if ( (IR_Custom & 0x1 ) == 1 )
	{
		IR_Data = IR_Data | 0x8000;
	}
	IR_Custom = IR_Custom >> 1;
	tmp_data = 0;
	tmp_Custom = 0;
	DBG_MSG1(DBGCFG_IR, "[IR]0\tIR:%08x\ttmpData:%08x\tIR_Custom:%08x\n", IR_Data, tmp_data, IR_Custom);
//	for ( i=0;i<7;i++ )
	for ( i=0;i<15;i++ ) //fenghl
	{
		if ( IR_Data & 0x8000 )
		{
			tmp_data = tmp_data | 0x8000;
		}
		tmp_data = tmp_data >> 1;
		IR_Data = IR_Data << 1;
	}
	if ( IR_Data & 0x8000 )
	{
		tmp_data = tmp_data | 0x8000;
	}
/*****************************/
	tmp_1 = tmp_data >> 8;
	tmp_2 = tmp_data << 8;
	tmp_data = tmp_1 | tmp_2;
	tmp_data = tmp_data & 0xffff;
/*****************************/
	for ( i=0;i<15; i++ )
	{
		if ( IR_Custom & 0x8000 )
		{
			tmp_Custom = tmp_Custom | 0x8000;
		}
		tmp_Custom = tmp_Custom >> 1;
		IR_Custom = IR_Custom << 1;
	}
	if ( IR_Custom & 0x8000 )
	{
		tmp_Custom = tmp_Custom | 0x8000;
	}
/*****************************/
	tmp_1 = tmp_Custom >> 8;
	tmp_2 = tmp_Custom << 8;
	tmp_Custom = tmp_1 | tmp_2;
	tmp_Custom = tmp_Custom & 0xffff;
/*****************************/
	DBG_MSG1(DBGCFG_IR, "[IR]1\tIR:%08x\ttmp:%08x\ttmp_Custom:%08x\n", IR_Data, tmp_data, tmp_Custom);
	//IR_Data = tmp_data;
	if ( IR_bit_len == 33)
	{	// Single start
		if ( (singleKey == 1) || (rpt_cnt != 0) )
		{
			if ( ir_timer_flag )
			{
				del_timer(&IR_ContiTimer);
			}
			if ( singleKey == 1 )
			{
				DBG_MSG1(DBGCFG_IR, "[IR]2nd reg_single_sts received!!\n");
				datBuf->Databuf[datBuf->Length].ContinueKey = SINGLE_KEY_END;	// single end
			}
			else
			{
				DBG_MSG1(DBGCFG_IR, "[IR]reg_single_sts received during repeat!!\n");
				datBuf->Databuf[datBuf->Length].ContinueKey = CONTINUE_KEY_START_END;	// continue end
			}
			datBuf->Databuf[datBuf->Length].Command = tmp_irCmd;
			datBuf->Databuf[datBuf->Length].Command_ = tmp_irCmd_;
			datBuf->Databuf[datBuf->Length].Address = tmp_irAddress;
			datBuf->Databuf[datBuf->Length].Address_ = tmp_irAddress_;
			datBuf->Length++;
		}
		else
		{
			DBG_MSG1(DBGCFG_IR, "[IR]1st reg_single_sts received!!\n");
		}
		tmp_irCmd = dataBuf->Databuf[dataBuf->Length].Command = (unsigned char)(tmp_data >> 8);
		tmp_irCmd_ = dataBuf->Databuf[dataBuf->Length].Command_ = (unsigned char)(tmp_data);
		tmp_irAddress = dataBuf->Databuf[dataBuf->Length].Address = (unsigned char)(tmp_Custom >> 8);
		tmp_irAddress_ = dataBuf->Databuf[dataBuf->Length].Address_ = (unsigned char)(tmp_Custom);;
		DBG_MSG1(DBGCFG_IR, "[IR]Single Start Address  Address_  Cmd: 0x%x,0x%x, 0x%x,0x%x!!\n", tmp_irAddress, tmp_irAddress_, tmp_irCmd, tmp_irCmd_);
		dataBuf->Databuf[dataBuf->Length].ContinueKey = CONTINUE_KEY_HEAD;
		dataBuf->Length++;
		singleKey = 1;
		rpt_cnt = 0;
		init_IR_cont_timer(150);		// 150 ms
		ir_timer_flag = 1;
	}
	else if ( IR_bit_len == 2 )
	{
		DBG_MSG1(DBGCFG_IR, "[IR]repeat key received!!\n");
		if ( ir_timer_flag == 1 )
		{
			del_timer(&IR_ContiTimer);
		}
		if ( rpt_cnt == 0xff )
		{
			rpt_cnt = 1;
		}
		rpt_cnt ++;
		if ( (rpt_cnt == 4) && (singleKey == 1) )
		{
			singleKey = 0;
			dataBuf->Databuf[dataBuf->Length].Command = tmp_irCmd;
			dataBuf->Databuf[dataBuf->Length].Command_ = tmp_irCmd_;
			dataBuf->Databuf[dataBuf->Length].Address = tmp_irAddress;
			dataBuf->Databuf[dataBuf->Length].Address_= tmp_irAddress_;
			dataBuf->Databuf[dataBuf->Length].ContinueKey = CONTINUE_KEY_START;
			dataBuf->Length++;
			DBG_MSG1(DBGCFG_IR, "[IR][IR]Continue_key_start\n");
		}
		init_IR_cont_timer(150);		// 150 ms
		ir_timer_flag = 1;
	} 
}
```  

其中：  

```c
irdatas.Value= GetMMIO_DWORD(mmiobase, 0x0c);	//read from IR decoder recevied data
```

### 补充
1. Toshiba客户码的高8位和低8位都是一样的？Toshiba一定是一样的,或是反向,硬体那边应该会直接转好反向的,所以到软体这边收到的都会是一样的。  
2. 按遥控器MENU按键（custom code: 0x0E0E, data code: 0x11），示波器抓波形：  
![UI_EVENT_MENU](https://i.loli.net/2018/12/06/5c0915a008d22.jpg)  
先发0x0E(0b00001110)，再发0x0E(0b00001110)，接着发0x11(0b00010001)，最后发0xEE(0b11101110)，由每个字节的低位开始发，所以示波器上按时间从早到晚（按图上从左到右）计算，得到：  
> 01110000 01110000 10001000 01110111  
