---
title: 'DVB-T自动搜台流程跟踪'
date: 2020-01-16
categories: TV
tags: [tv,dvb,dvb-t,scan,si]
---


已经有点烦了。  

<!-- more -->



`kernel/android/pie/vendor/realtek/app/dtvinput/src/com/realtek/dtv/digitalsetup/dvb/DVBScanChannelFragment.java`，菜单上的最后一步，是创建一个`DigitalScanTask`：  
```java
mDigitalScanTask = new DigitalScanTask((DigitalChannelSetupActivity)getActivity());
//...
 mDigitalScanTask.sendMessageDelayed(DigitalScanTask.MSG_CHANNEL_SACN, scanParam, 500);
```
```java
case MSG_CHANNEL_SACN:
    //...
    mHandler.sendEmptyMessage(MSG_CHANNEL_TABLE_SACN);
```
```java
case MSG_CHANNEL_TABLE_SACN:
    //...
    startTableScan();
```
```java
success = mTvServiceHelper.startAutoScan(mScanChannelList,mModeType);//auto table scan
```
向下传进一个频点表`mScanChannelList`。  

调到了`kernel/android/pie/vendor/realtek/app/dtvinput/src/com/realtek/dtv/TvServiceHelper.java`的`startAutoScan`  
```java
public boolean startAutoScan(ArrayList<FreqTable> freqtable,String type){
	if (mTv != null) {
		return mTv.StartDtvAutoScan(freqtable.size(),freqtable,-1);
	}
	return false;
}
```

接着通过JNI，HIDL（依次调用了以下文件）  
`kernel/android/pie/vendor/realtek/frameworks/base/core/java/com/realtek/tv/Tv.java`  
`kernel/android/pie/vendor/realtek/frameworks/base/core/jni/com_realtek_tv_Tv.cpp`  
`kernel/android/pie/vendor/realtek/hardware/interfaces/tv/1.0/IRtkTv.hal`  
`kernel/android/pie/vendor/realtek/hardware/interfaces/tv/1.0/default/RtkTv.cpp`  
`kernel/android/pie/vendor/realtek/frameworks/native/appclass/tv/libtvsystem/tv/TvClient.cpp`  
`kernel/android/pie/vendor/realtek/frameworks/native/appclass/tv/libtvservice/TvService.cpp`  
`kernel/android/pie/vendor/realtek/frameworks/native/appclass/tv/libtvservice/Tv.cpp`  
`kernel/android/pie/vendor/realtek/frameworks/native/appclass/tvapi/dtvcontrol/CDTVControl.cpp`  
`kernel/android/pie/vendor/realtek/frameworks/native/appclass/mediacontrol/component/channel/scan/TableScanner.cpp`  

终于创建一个`ScanThread`线程  
```c++
bool CTableScanner::AutoScanStart(int tvsubtype, int satelliteIndex,bool bSkipPMT,bool bSSUScan)
{
	RT_FRONTEND_TYPE feType = m_pFreqScanDetector->Mf_GetFrontendType();

	if (m_threadIsRunning == true || (feType != RT_FRONTEND_DVB_SATELLITE && m_freqListSize == 0))
		return false;

	if (m_scanMode != CH_SCAN_MODE_IDLE)
		return false;
	m_bRemoveAll = true;

#ifdef ENABLE_SUPPORT_DVB_S
	if(feType == RT_FRONTEND_DVB_SATELLITE && m_pFreqList == NULL)
	{
		xLoadTable(satelliteIndex);
	}
#endif

	m_serviceID=0;
	m_scanMode = CH_SCAN_MODE_AUTO;
	m_bSSUScan = bSSUScan;
	m_bDelNosignalMux = true;
	m_satelliteIndex = satelliteIndex;
	m_bQuickScan=false;
	m_bSkipPMT=bSkipPMT;
	m_curFreqIndex = 0;
	m_uiScanProgress = 0;
	m_lastScanedFreqIndex =0;
	m_curFreq = m_pFreqList!=NULL?m_pFreqList[0].frequency:0;
	m_curBandwidth = m_pFreqList!=NULL?m_pFreqList[0].bandwidth:0;
	m_curChNum = m_pFreqList!=NULL?m_pFreqList[0].channelNum:0;
	m_curModulation = m_pFreqList!=NULL?m_pFreqList[0].modulation:RT_MOD_UNKNOWN;
	m_curSymbolRate = m_pFreqList!=NULL?m_pFreqList[0].symbolRate:0;
	m_curPolarization = m_pFreqList!=NULL?m_pFreqList[0].polarization:0;

	pthread_create(&m_thread, NULL, ScanThread, (void *)this);

	// Quick sleep to cause thread to be created.
	while (m_threadIsRunning == false)
		usleep(100000);

	return true;
}
```

### 准备搜台
`kernel/android/pie/vendor/realtek/frameworks/native/appclass/mediacontrol/component/channel/scan/ChScanner.cpp`  
`ScanThread`线程首先将`CH_SCAN_STATE`设定为`CH_SCAN_STATE_INIT`，然后进入一个`xWorkerThread`循环  
```c++
void* CChScanner::ScanThread(void *pParam)
{
	CChScanner *pThis = (CChScanner *)pParam;
	pThis->m_scanState = CH_SCAN_STATE_INIT;
	pThis->m_threadIsRunning = true;
	pThis->m_runThread = true;

	while (pThis->m_runThread == true)
		pThis->xWorkerThread();

	// Manual/seekscan needs to set this variable before ending scan message is sent out.
	pThis->m_threadIsRunning = false;
	pThis->m_cancelScan = false;

	return 0;
}
```

`kernel/android/pie/vendor/realtek/frameworks/native/appclass/mediacontrol/component/channel/scan/TableScanner.cpp`  
`xWorkerThread`循环中首先处理了`case``CH_SCAN_STATE_INIT`  
```c++
void CTableScanner::xWorkerThread()
{
	// Verify scan mode is valid.
	if ((m_scanMode == CH_SCAN_MODE_IDLE) || (m_scanMode >= CH_SCAN_MODE_MAX))
		goto EXIT_SCANNING;

	switch (m_scanState)
	{
	case CH_SCAN_STATE_INIT:
		xStageInit();
		break;

	case CH_SCAN_STATE_BEGIN_FREQ:
		ALOGD("[%s %d] CH_SCAN_STATE_BEGIN_FREQ start\n", __func__, __LINE__);
		xStageBeginFreq();
		break;

	case CH_SCAN_STATE_SCANNING:
		// wait scan done
		ALOGD("[%s %d] CH_SCAN_STATE_SCANNING start\n", __func__, __LINE__);
		while (!m_pFreqScanDetector->Mf_IsScanDone()) {
			if (m_cancelScan == true)
				break;

			m_pFreqScanDetector->RunStateProc();
			if (m_cancelScan != true && !m_pFreqScanDetector->Mf_IsScanDone()) {
				if(m_scanMode == CH_SCAN_MODE_BLIND_SCAN||m_scanMode == CH_SCAN_MODE_BLIND_SCAN_NETWORK)
				{
					int Index = 0, Total = 0;
					UINT32 freq = m_pFreqScanDetector->Mf_GetFrequency(&Index,&Total);
					if(freq>0)
					{
						m_curFreqIndex = Index;
						m_freqListSize = Total;
						m_curFreq = freq;
					}
				}
				usleep(SCAN_THREAD_CS_TIME*1000);
			}
		}
		ALOGD("[%s %d] CH_SCAN_STATE_SCANNING end\n", __func__, __LINE__);
		m_scanState = CH_SCAN_STATE_END_FREQ;
		break;

	case CH_SCAN_STATE_END_FREQ:
		ALOGD("[%s %d] CH_SCAN_STATE_END_FREQ begin\n", __func__, __LINE__);
		xStageEndFreq();
		break;

	case CH_SCAN_STATE_SCAN_FAILED:
	case CH_SCAN_STATE_EXIT:
		xStageExit();
		goto EXIT_SCANNING;
		break;

	case CH_SCAN_STATE_IDLE:
	default:
		goto EXIT_SCANNING;
	}

	if (m_cancelScan == true)
		m_scanState = CH_SCAN_STATE_EXIT;

	return;


EXIT_SCANNING:
	m_runThread = false;
	return;
}
```

跑的是`xStageInit()`函数，主要做了一件事  
```c++
m_pFreqScanDetector->Mf_ScanInit(scanMode, param, m_bDelNosignalMux,scanModeEx);
```
这件事会跳到`kernel/android/pie/vendor/realtek/frameworks/native/appclass/mediacontrol/component/channel/scan/DtvFreqDetector.cpp`  
```c++
void CDtvFreqDetector::Mf_ScanInit(CH_SCAN_MODE scanMode, UINT32 param, bool bDelNoSignalMux, CH_SCAN_MODE_EX modeEx)
{
    //...
    m_pIDtvMedia->GetSiMgr()->ScanInit(getSiScanType(scanMode), param, typeEx);
    //...
}
```
接着跳到`kernel/android/pie/vendor/realtek/frameworks/native/appclass/mediacontrol/component/si/DvbSiMgr.cpp`  
```c++
void CDvbSiMgr::ScanInit(SI_MGR_SCAN_TYPE type, int antennaIndex, SI_MGR_SCAN_TYPE_EX typeEx)
{
    //...
    if (type == SI_MGR_SCAN_TYPE_AUTO||type==SI_MGR_SCAN_TYPE_AUTO_QUICK) {
	SI_SetState(siHandle, SI_STATE_AUTOSCAN);
	SI_AutoScanInit(siHandle,antennaIndex,type==SI_MGR_SCAN_TYPE_AUTO_QUICK?TRUE:FALSE,FALSE);
#if (defined ENABLE_CI && defined ENABLE_CIPLUS_1_4)
	if(m_CiCallbackObj.NotifyChannelErase != NULL) {
		m_CiCallbackObj.NotifyChannelErase();
	}
#endif
    }
    //...
}
```

`xStageInit`最后把`CH_SCAN_STATE`状态设定为`CH_SCAN_STATE_BEGIN_FREQ`  

### 开始搜台
`xWorkerThread``case``CH_SCAN_STATE_BEGIN_FREQ`执行`CTableScanner::xStageBeginFreq()`  
执行`m_pFreqScanDetector->Mf_ScanFrequency(param)`  
这里`FREQ_SCAN_STATE`被设置成状态`FREQ_SCAN_STATE_BEGIN_FREQ`  
`CTableScanner::xStageBeginFreq()`的最后，`CH_SCAN_STATE`被设置成`CH_SCAN_STATE_SCANNING`  

### 正在搜台
搜台过程就是`CTableScanner::xWorkerThread()``case``CH_SCAN_STATE_SCANNING`，是一个`while`循环  
先判断是否搜台结束  
```c++
m_pFreqScanDetector->Mf_IsScanDone()
```
否则，跑  
```c++
m_pFreqScanDetector->RunStateProc();
```
因为刚才`FREQ_SCAN_STATE`被设置为`FREQ_SCAN_STATE_BEGIN_FREQ`,所以跑`CDtvFreqDetector::xStageBeginFreq()`。在某一频点的搜台基本都是在`CDtvFreqDetector`这里处理啦。  

`m_pFreqScanDetector->RunStateProc()`做了什么？  
1. Stop SI  
2. Set tuner param  
3. Notify demod to lock  
4. Check frontend(Update modulation, bandwidth, and symbol rate)  
5. Check SI  

#### `FREQ_SCAN_STATE_BEGIN_FREQ`
跟下去能看到`tuner->tune(param)`和`tuner->isLocked()`都是在`kernel/android/pie/vendor/realtek/frameworks/native/appclass/driverbaseddtvapp/TunerMgr.cpp`处理的  
```c++
bool TunerMgr::tune(UINT32 frequency, UINT32 bandwidth, RT_FRONTEND_TYPE feType, RT_MODULATION modulation, RT_SPECTRAL_INVERSION inversion, bool isScanMode, UINT32 symbolRate, SatelliteInfo *pSatelliteInfo)
{
	mFreq = frequency;
	mBandwidth = bandwidth;
	return FrontendLib_SetTuner(frequency, bandwidth, mTunerId, feType, modulation, inversion, isScanMode, symbolRate, pSatelliteInfo);
}
```

```c++
bool TunerMgr::isLocked()
{
	uint8_t lock = false;

	if (TunerControlGetLockStatus(mTunerId, &lock) == TUNER_CTRL_OK)
		return lock;
	else
		return false;
}
```

#### `FREQ_SCAN_STATE_CHECK_FRONTEND`
```c++
void CDtvFreqDetector::xStageCheckFrontend()
{
    //...
    m_modulation = getRtModVal(m_feType, info);
    //...
    m_pIDtvMedia->GetSiMgr()->ScanStart(m_curFreq, m_modulation, m_curBandwidth, symbolrate,  m_curPhyChNum, tuner->getRFStrength(), tuner->getSignalSNR(), m_serviceID);
    //...
}
```
```c++
void CDvbSiMgr::ScanStart(UINT32 frequency, RT_MODULATION modulation, UINT32 bandwidth, UINT32 symbolrate, UINT32 phyChNum,UINT32 strength,float snr, UINT16 serviceID)
{
    SI* siHandle = (SI*)m_pTvMedia->GetDtvFlow()->GetSiHandle();
    SI_DVB_MODULATION siModulation;
    switch(modulation)
    {
    case RT_MOD_QAM16:
        siModulation=SI_DVB_16_QAM;
        break;
    case RT_MOD_QAM32:
        siModulation=SI_DVB_32_QAM;
        break;
    case RT_MOD_QAM64:
        siModulation=SI_DVB_64_QAM;
        break;
    case RT_MOD_QAM128:
        siModulation=SI_DVB_128_QAM;
        break;
    default:
    case RT_MOD_QAM256:
        siModulation=SI_DVB_256_QAM;
        break;
    }

    SI_SetState(siHandle, SI_STATE_SCAN);
    SI_ScanChannelEx(siHandle, frequency, 0, bandwidth,siModulation,symbolrate, strength,snr,serviceID,(unsigned char)phyChNum);
}
```
`SI_ScanChannelEx`路径在`kernel/android/pie/vendor/realtek/frameworks/native/appclass/si/livetv_sidvb/librtd/si4/api/SI_Api.c`！  

#### `FREQ_SCAN_STATE_CHECK_SI`
`CDtvFreqDetector::xStageCheckSi()`：Wait for SI to finish gathering tables  
```c++
void CDtvFreqDetector::xStageCheckSi()
{
    //...
RETRY:
	if (m_pIDtvMedia->GetSiMgr()->ScanIsDone(&m_bIsGetService) == false)
	{
		m_pIDtvMedia->GetSiMgr()->SetSignalInfo(tuner->getRFStrength(), tuner->getSignalSNR());
		usleep(10000);
                //...
		return;
	}
    //...
    m_pIDtvMedia->GetSiMgr()->SetModulation(m_curFreq, m_modulation);
    m_scanState = FREQ_SCAN_STATE_END_FREQ;
    //...
    return;
}
```
```c++
bool CDvbSiMgr::ScanIsDone(bool *bGetCh)
{
    SI_MESSAGE message;
    UINT32 data = 0;
    DriverBasedDtvApp *pDvbApp = m_pTvMedia->GetDtvFlow();

    if (pDvbApp->GetSiMessage(&message, &data) == false)
        return false;

    if (message == SI_MESSAGE_RESET_CHANNEL && data>0) {
        xResetChannel(data);
        return false;
    }
    else if(message == SI_MESSAGE_CHANNEL_UPDATE && data>0) {
        xUpdateChannelMgr(data);
        return false;
    }
    else if (message == SI_MESSAGE_CH_INFO_READY)
    {
        if(bGetCh!=NULL)
        {
            *bGetCh = data == 0 ? true : false;
        }
    }
    else if (message == SI_MESSAGE_SSU_SW_NOT_FOUND||message == SI_MESSAGE_SSU_SWINFO_READY)
    {
        if(bGetCh!=NULL)
        {
            *bGetCh = message == SI_MESSAGE_SSU_SWINFO_READY ? true : false;
        }
    }
    else/* if (message != SI_MESSAGE_CH_INFO_READY)*/
        return false;

    return true;
}
```

### 完成搜台
#### `FREQ_SCAN_STATE_END_FREQ`
如果搜台完成了，`CDtvFreqDetector::xStageCheckSi()`之后会把`FREQ_SCAN_STATE`状态设定成`FREQ_SCAN_STATE_END_FREQ`，在`CTableScanner::xWorkerThread()`中，`CH_SCAN_STATE``CH_SCAN_STATE_SCANNING`会退出，变成`CH_SCAN_STATE_END_FREQ`  
接着跑`CTableScanner::xStageEndFreq()`,如果频点没搜完，接着搜下一个频点，否则，`CH_SCAN_STATE`设定为`CH_SCAN_STATE_SCAN_FAILED`，跑`CTableScanner::xStageExit()`，这里面跑`m_pFreqScanDetector->Mf_ScanDeInit()`以及把一些标志清空。  
```c++
void CDtvFreqDetector::Mf_ScanDeInit(bool bNoSaveInNoCh)
{
	m_pIDtvMedia->GetSiMgr()->SetMode(SI_MGR_MODE_INACTIVE);
	m_pIDtvMedia->GetSiMgr()->ScanEnd(bNoSaveInNoCh);

	// Flush SI queue.
	m_pIDtvMedia->GetSiMgr()->FlushMessageQueue();
	m_pIDtvMedia->GetSiMgr()->SaveChannelToFile(bNoSaveInNoCh);

	DTV_STACK::TunerMgr* tuner = DTV_STACK::TunerMgr::getInstance();
	unsigned char tunerId = -1;
	tuner->getTunerId(tunerId);
	TunerControlChannelScanModeEnable(tunerId, 0);
	tuner->reset();
}
```
