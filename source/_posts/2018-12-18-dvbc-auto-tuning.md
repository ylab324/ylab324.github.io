---
title: 一个DVB-C自动搜台的疑问
date:   2018-12-18
categories: TV
tags:  [tv,realtek,dvb,dvb-c,dtv,si,scan]
---

DVBC搜台过程中，频点乱序。

<!-- more -->

### 背景
Mantis提交了一条问题：DVBC搜台过程中，进度条在前进，频点也在跳动，但是乱序，并非从低到高。  
实际情况是：码流机播放《610.750 Network ID 1536.ts》码流，频点设定到482MHz，DVBC搜台过程中，频率从低到高搜，但是搜到482MHz的时候，频率就乱了。  

### 材料
截取部分log信息：  
> [virtual void CTableScanner::xWorkerThread() 139] CH_SCAN_STATE_BEGIN_FREQ start  
> [virtual void CTableScanner::xWorkerThread() 145] CH_SCAN_STATE_SCANNING start  
> xStageBeginFreq:161:: m_isDeleteAll = 1, m_curFreq = 474000000  
> [virtual void CDtvFreqDetector::xStageBeginFreq() 296] isScanMode = 1, m_curFreq = 474000000, m_curBandwidth = 8000000, tunerId = 0, m_feType = 2, m_modulation = 11,  m_symbolrate = 6900000  
> [bool FrontendLib_SetTuner(UINT32, UINT32, UINT8, RT_FRONTEND_TYPE, RT_MODULATION, RT_SPECTRAL_INVERSION, bool, UINT32, SatelliteInfo *) 156] isScanMode = 1, frequency = 474000000, bandwidth = 8000000, tunerId = 0, feType = 2, modulation = 11,  symbolRate = 6900000  
> [FrontendLib_SetTuner 174] frequency=474000000 bandwidth=8000000 tvSystem=19  
> [virtual void CDtvFreqDetector::xStageBeginFreq() 298] ret = 0  
> [virtual void CTableScanner::xWorkerThread() 166] CH_SCAN_STATE_SCANNING end  
> [virtual void CTableScanner::xWorkerThread() 171] CH_SCAN_STATE_END_FREQ begin  
> [xUpdateChannelMgr:197] freq=474000000, bSave=1, bUpdateCurChIndex=0, bKeepChannel=0, curFrequency=0, modulation=11  
> xUpdateChannelMgr 211 RemoveChannel 474000000  
> [xRemoveChannel:2220]  
> [bool CDvbTableScanner::xGetNitFreqList(bool) 139] errCode = 1900551, freqCount = 0  
> [virtual void CTableScanner::xWorkerThread() 139] CH_SCAN_STATE_BEGIN_FREQ start  
> [virtual void CTableScanner::xWorkerThread() 145] CH_SCAN_STATE_SCANNING start  
> xStageBeginFreq:161:: m_isDeleteAll = 1, m_curFreq = 482000000  
> [virtual void CDtvFreqDetector::xStageBeginFreq() 296] isScanMode = 1, m_curFreq = 482000000, m_curBandwidth = 8000000, tunerId = 0, m_feType = 2, m_modulation = 11,  m_symbolrate = 6900000  
> [bool FrontendLib_SetTuner(UINT32, UINT32, UINT8, RT_FRONTEND_TYPE, RT_MODULATION, RT_SPECTRAL_INVERSION, bool, UINT32, SatelliteInfo *) 156] isScanMode = 1, frequency = 482000000, bandwidth = 8000000, tunerId = 0, feType = 2, modulation = 11,  symbolRate = 6900000  
> [FrontendLib_SetTuner 174] frequency=482000000 bandwidth=8000000 tvSystem=19  
> [virtual void CDtvFreqDetector::xStageBeginFreq() 298] ret = 1  
> virtual void CDtvFreqDetector::xStageBeginFreq()_304 signal LOCKED !!!!!!!!  
> [virtual void CDtvFreqDetector::xStageCheckFrontend() 771] m_curFreq = 482000000, m_curBandwidth = 8000000 m_modulation = 11  
> [virtual void CTableScanner::xWorkerThread() 166] CH_SCAN_STATE_SCANNING end  
> [virtual void CTableScanner::xWorkerThread() 171] CH_SCAN_STATE_END_FREQ begin  
> [xUpdateChannelMgr:197] freq=482000000, bSave=1, bUpdateCurChIndex=0, bKeepChannel=0, curFrequency=0, modulation=11  
> xUpdateChannelMgr 211 RemoveChannel 482000000  
> [xRemoveChannel:2220]  
> **[bool CDvbTableScanner::xGetNitFreqList(bool) 139] errCode = 0, freqCount = 39**  
> **xGetNitFreqList..freq:482750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:458750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:124000000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:466750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:506750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:578750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:386750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:498750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:626750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:650750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:148000000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:714750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:698750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:706750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:762750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:826750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:858750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:642750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:778750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:810750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:834750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:746750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:298750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:602750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:610750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:850750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:140000000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:738750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:530750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:818750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:690750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:164000000, symbolRate:6900000, modulation:9, polarization:0**  
> **xGetNitFreqList..freq:842750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:666750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:682750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:156000000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:554750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:490750000, symbolRate:6900000, modulation:11, polarization:0**  
> **xGetNitFreqList..freq:132000000, symbolRate:6900000, modulation:11, polarization:0**  
> [Mf_ScanInit 1000] modeEx=0, param=-1, scanMode=7 m_deleteAll=1  
> [xRemoveChannel:2288] m_chVector.m_size=39  
> ScanInit....type 2 antennaIndex 0xffffffff typeEx 0 time 304186343  
> [virtual void CTableScanner::xWorkerThread() 139] CH_SCAN_STATE_BEGIN_FREQ start  
> [virtual void CTableScanner::xWorkerThread() 145] CH_SCAN_STATE_SCANNING start  
> xStageBeginFreq:161:: m_isDeleteAll = 1, m_curFreq = 482750000  
> [virtual void CDtvFreqDetector::xStageBeginFreq() 296] isScanMode = 1, m_curFreq = 482750000, m_curBandwidth = 8000000, tunerId = 0, m_feType = 2, m_modulation = 11,  m_symbolrate = 6900000  
> [bool FrontendLib_SetTuner(UINT32, UINT32, UINT8, RT_FRONTEND_TYPE, RT_MODULATION, RT_SPECTRAL_INVERSION, bool, UINT32, SatelliteInfo *) 156] isScanMode = 1, frequency = 482750000, bandwidth = 8000000, tunerId = 0, feType = 2, modulation = 11,  symbolRate = 6900000  
> [FrontendLib_SetTuner 174] frequency=482750000 bandwidth=8000000 tvSystem=19  
> [virtual void CDtvFreqDetector::xStageBeginFreq() 298] ret = 1  
> virtual void CDtvFreqDetector::xStageBeginFreq()_304 signal LOCKED !!!!!!!!  
> [virtual void CDtvFreqDetector::xStageCheckFrontend() 771] m_curFreq = 482750000, m_curBandwidth = 8000000 m_modulation = 11  
> [virtual void CTableScanner::xWorkerThread() 166] CH_SCAN_STATE_SCANNING end  
> [virtual void CTableScanner::xWorkerThread() 171] CH_SCAN_STATE_END_FREQ begin  
> [xUpdateChannelMgr:197] freq=482750000, bSave=1, bUpdateCurChIndex=0, bKeepChannel=0, curFrequency=0, modulation=11  
> bool CDvbSiMgr::xUpdateChannelMgr(__UINT32, INT16, bool, bool, bool, __UINT32, RT_MODULATION) 218 count 6  
> AddChannel:LCN=1,chName=Nederland 1 HD,frequency=482750000  
> AddChannel:LCN=2,chName=Nederland 2 HD,frequency=482750000  
> AddChannel:LCN=3,chName=RTL Lounge,frequency=482750000  
> AddChannel:LCN=4,chName=TVE,frequency=482750000  
> AddChannel:LCN=5,chName=HZN Barker 1,frequency=482750000  
> AddChannel:LCN=6,chName=HZN Barker 2,frequency=482750000  
> bool CDvbSiMgr::xUpdateChannelMgr(__UINT32, INT16, bool, bool, bool, __UINT32, RT_MODULATION) use time 0  

使用码流分析工具查看码流：  
![NIT](https://i.loli.net/2018/12/18/5c18b1955d467.png)  


代码片段[RTD2831]：  
```cpp
bool CDvbTableScanner::xGetNitFreqList(bool bDelete)
{
	int freqCount=0;
	SI_DELIVERY_PARAM *pFreqList = NULL;
	ErrCode errCode;
	int startindex=0;
	bool bDelFollow=false;
	bool bAddCurrent=false;
	bool bIncludeExistTP=false;
	RT_FRONTEND_TYPE feType = m_pFreqScanDetector->Mf_GetFrontendType();
	SI *siHandle = (SI*)((IDtvMedia*)m_pTvMedia)->GetDtvFlow()->GetSiHandle();
	if(feType == RT_FRONTEND_DVB_CABLE && !m_bRemoveAll)
	{
		bIncludeExistTP=true;
	}

	errCode = SI_GetFreqList(siHandle, &freqCount, &pFreqList, bIncludeExistTP);
	ALOGD("[%s %d] errCode = %lu, freqCount = %d\n", __func__, __LINE__, errCode, freqCount);

	if (errCode != SI_ERR_OK || freqCount==0 || pFreqList == NULL)
	{
		return FALSE; //如果freqCount为零，就返回了。
	}

	if(feType == RT_FRONTEND_DVB_SATELLITE && m_HomeTp.count!=0)
	{
		bDelete=true;
		if(m_HomeTp.onid!=0)
		{
			bDelete=false;
			for(int i=0;i<freqCount;i++)
			{
				if(pFreqList[i].onid==m_HomeTp.onid)
				{
					bDelete=true;
					break;
				}
			}
		}
	}

	if(feType == RT_FRONTEND_DVB_SATELLITE && bDelete && m_pFreqList!=NULL)
	{
		bDelFollow=true;
		bDelete=false;
	}

	if(bDelete || m_pFreqList==NULL)
	{
		xFreeTable();
		
		m_pFreqList = (RT_PHY_CHANNEL *)malloc(sizeof(RT_PHY_CHANNEL) * (freqCount+1));
		if(m_pFreqList == 0)
			return false;
		m_freqListSize = freqCount+1;
		memset(m_pFreqList, 0, sizeof(RT_PHY_CHANNEL)* (m_freqListSize));
		m_isMallocedList=true;
		
		if(feType == RT_FRONTEND_DVB_CABLE)
		{
			bAddCurrent=true;
		}
	}
	else
	{
		RT_PHY_CHANNEL *pFreqList;
		int sizeTmp=m_freqListSize;
		pFreqList = (RT_PHY_CHANNEL *)malloc(sizeof(RT_PHY_CHANNEL)* (sizeTmp+freqCount));
		if(pFreqList==NULL)
		{
			return false;
		}
		memcpy(pFreqList, m_pFreqList, sizeof(RT_PHY_CHANNEL)* (sizeTmp));
		xFreeTable();
		m_pFreqList=pFreqList;
		if(bDelFollow)
			startindex=m_curFreqIndex;
		else
			startindex=sizeTmp;
		m_freqListSize=sizeTmp+freqCount;
		m_isMallocedList=true;
	}

	for (int i = 0; i < freqCount && startindex<m_freqListSize; i++)
	{
		m_pFreqList[startindex].channelNum = startindex;

		m_pFreqList[startindex].frequency = pFreqList[i].frequency;

		switch (pFreqList[startindex].modulation)    // Cable delivery system descriptor, defined in en300468 DVB-SI spec
		{
			case SI_DVB_16_QAM:      // 16QAM,
				m_pFreqList[startindex].modulation = RT_MOD_QAM16;
				break;
			case SI_DVB_32_QAM:      // 32QAM,
				m_pFreqList[startindex].modulation = RT_MOD_QAM32;
				break;
			case SI_DVB_64_QAM:      // 64QAM,
				m_pFreqList[startindex].modulation = RT_MOD_QAM64;
				break;
			case SI_DVB_128_QAM:      // 128QAM,
				m_pFreqList[startindex].modulation = RT_MOD_QAM128;
				break;
						default:
			case SI_DVB_256_QAM:      // 256QAM,
				m_pFreqList[startindex].modulation = RT_MOD_QAM256;
				break;
		}
		m_pFreqList[startindex].symbolRate = pFreqList[i].symbol_rate;


		if(feType == RT_FRONTEND_DVB_SATELLITE)
		{
			m_pFreqList[startindex].modulation = pFreqList[i].modulation_system == 0 ? RT_MOD_DVB_S : RT_MOD_DVB_S2;
			m_pFreqList[startindex].symbolRate = pFreqList[i].symbol_rate;
		}


		m_pFreqList[startindex].bandwidth = 8000000;
		m_pFreqList[startindex].polarization= pFreqList[i].polarization%2;
		if(bDelete || iSNewTP(m_pFreqList,startindex,&m_pFreqList[startindex]))
		{
			ALOGD("%s..freq:%d, symbolRate:%d, modulation:%d, polarization:%d\n",__FUNCTION__,m_pFreqList[startindex].frequency,m_pFreqList[startindex].symbolRate,
			m_pFreqList[startindex].modulation,m_pFreqList[startindex].polarization);
				startindex++;
		}

		if(bAddCurrent && startindex<m_freqListSize)
		{
			if(m_curFreq/1000000 == m_pFreqList[startindex].frequency/1000000)
			{
				bAddCurrent=false;
			}
		}
	}

	if(bAddCurrent)
	{
		m_pFreqList[startindex].frequency = m_curFreq;
		m_pFreqList[startindex].bandwidth = m_curBandwidth;
		m_pFreqList[startindex].channelNum = m_curChNum;
		m_pFreqList[startindex].modulation = m_curModulation;
		m_pFreqList[startindex].symbolRate = m_curSymbolRate;
		m_pFreqList[startindex].polarization = m_curPolarization;
		startindex++;
	}

	m_freqListSize=startindex;
	return true;
}

bool CDvbTableScanner::iSNewTP(RT_PHY_CHANNEL *src, int size, RT_PHY_CHANNEL *pTp)
{
	if(pTp->frequency == 0)
		return FALSE;
	for(int i=0;i<size;i++)
	{
		if(abs((int)src[i].frequency/100000 - (int)pTp->frequency/100000) < 3)
		{
			if(src[i].symbolRate != pTp->symbolRate)
				continue;

			if(src[i].polarization != pTp->polarization)
				continue;

			if(src[i].modulation != pTp->modulation)
				continue;

			return false;
		}
	}
	return true;
}
```  

### 结论
可知是NIT的原因，自动搜台的时候（如果没有指定频点）会从频点由低往高搜台，搜到台后，解析NIT时发现有FreqList，就去FreqList里面的频点搜。  

### 参考
* NIT(Network Information Table 网络信息表)。此DVB列表包含了有关网络的范围、转换器等信息。它总是位于PID 0x0010的位置。DVB定义了两种类型的NIT，NIT Actual与NIT Other。NIT Actual是一个包含有关正在被访问的网络的物理参数的一个命令列表。NIT Other包含了有关其他网络的物理参数，NIT Other是可选项。  
* NIT搜索就是从NIT表开始的搜索，NIT表中的第二个循环，给出了当前网络中的频点，调制方式，符号率。搜索NIT表知道这些消息以后，将得到的频点做一个列表，然后挨个锁频，在每个频点下搜索PAT,PMT，然后得到节目，搜索完所有的频点以后，当前网络所有的节目就得到了。  
* [【PSI/SI学习系列】2.PSI/SI深入学习3——SI信息解析1(NIT,BAT)](https://blog.csdn.net/kkdestiny/article/details/12994675)  
* [DVB-SI理解入门](https://blog.csdn.net/rongdeguoqian/article/details/40372097)  
* [现有数字电视的搜台方式包含两大类全频点搜台和利用NIT (Network Information Table,网络信息表)快速搜台。前者的做法与模拟电视相似，根据频压曲线的变化和解码器相关数据当前的状态改变步进从低频到高频逐步扫描，这种过程一般会超过20分钟，并且由于无法利用NIT表，没有节目自动更新的功能；而后者先找到NIT并从NIT中提取Frequency List Descriptor(频率列表描述子)，利用频率列表描述子所带的频率信息，快速定位到每一个有信 号的频率点，从而完成快速搜台](https://www.xjishu.com/zhuanli/62/200810067838.html)
