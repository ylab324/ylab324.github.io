---
title: 'Linux下Thread使用demo'
date: 2020-04-15
categories: Coding
tags: [linux,thread,demo]
---


这里的场景是串口自发自收。  


<!-- more -->


先贴代码：  
```c++
int readRet = 0;
void *CH_Factory_Uart_Selfcheck_Reading_Thread(void *arg) {
	SINT32 count = 0;
	SINT32 port_id = 0;// use uart 0
	SINT32 read_len = 0;
	char buf[] = "changhong";
	memset(buf,0,sizeof(buf));
	while(1)
	{
		RHAL_UART_ReadUart(port_id,(UINT8 *)buf,&read_len,sizeof(buf));
		count++;
		if(read_len > 0 && strcmp(buf,"changhong") == 0) {
			ALOGD("[%s] [%d]: count = %d, buf = %s, read_len = %d\n",__func__,__LINE__,count,buf,read_len);
			ALOGD("[%s] [%d]: OOOOOOOOOOOOOOOOK!\n",__func__,__LINE__);
			readRet = 1;
			pthread_exit(0);
		}
		if(count > 20) {
			ALOGD("[%s] [%d]: ERRORRRRRRRRRRRR!\n",__func__,__LINE__);
			readRet = 0;
			pthread_exit(0);
		}
	}
	return NULL;
}
int CSystemControl::CH_Factory_Uart_Check(void)
{
	ALOGD("[%s] [%d]\n",__func__,__LINE__);
    int ret = 0;
    SINT32 read_len = 0;
	
    char buf[] = "changhong";
    SINT32 port_id = 0;// use uart 0
    pthread_t readThreadId;

	system("stop console");

    if(RHAL_UART_SelectUart(port_id)!=0)
	{
		system("start console");
		return false;
    }

	pthread_create(&readThreadId, NULL, CH_Factory_Uart_Selfcheck_Reading_Thread, NULL);
	
    ret = RHAL_UART_WriteUart(port_id,(UINT8 *)buf,sizeof(buf));
    if(0 != ret) 
    {
        ALOGD("[%s] [%d]: write uart fail\n",__func__,__LINE__);
		pthread_join(readThreadId, NULL);
		RHAL_UART_SelectUart(1); //switch back to UART1
		system("start console");
        return false;
    }
	
	pthread_join(readThreadId, NULL);

	RHAL_UART_SelectUart(1); //switch back to UART1
	system("start console");

	if(!readRet)
		return false;

	return true;
}
```

以上代码线程的点在于：  
1. 创建一个`ID`  
> pthread_t readThreadId;  

2. 创建了线程，`ID`与实际的线程函数匹配了  
> pthread_create(&readThreadId, NULL, CH_Factory_Uart_Selfcheck_Reading_Thread, NULL);  

	`CH_Factory_Uart_Selfcheck_Reading_Thread`就是被创建的线程，`ID`是`readThreadId`  

3. 退出线程  
> pthread_join(readThreadId, NULL);  

	`pthread_join`会阻塞主线程，等待`CH_Factory_Uart_Selfcheck_Reading_Thread`结束。`CH_Factory_Uart_Selfcheck_Reading_Thread`是一个`while(1)`循环，当串口读到需要的数据时，通过`pthread_exit(0)`使线程内部结束，这个时候跳到主线程的`pthread_join`，`pthread_join`终于等到了`CH_Factory_Uart_Selfcheck_Reading_Thread`结束，主线程就可以继续跑下去了。  
