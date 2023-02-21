---
title: 'WiFi扫描demo'
date: 2020-04-23
categories: Android
tags: [wifi,android,demo]
---


安卓扫描WiFi。  


<!-- more -->


```java
//MainActivity.java
package com.example.wifitest;

import android.Manifest;
import android.content.Context;
import android.content.pm.PackageManager;
import android.net.wifi.ScanResult;
import android.net.wifi.WifiManager;
import android.os.Build;
import android.support.v4.app.ActivityCompat;
import android.support.v4.content.ContextCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";
    private WifiManager mWifiManager;
    private List<ScanResult> mWifiList = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (Build.VERSION.SDK_INT >= 23) {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
		//需要动态申请权限
                ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.ACCESS_COARSE_LOCATION}, 10);
            }
        }

        mWifiManager = (WifiManager) this.getSystemService(Context.WIFI_SERVICE);

        if (!mWifiManager.isWifiEnabled()) {
            mWifiManager.setWifiEnabled(true);
        }
        mWifiList.clear();
        mWifiManager.startScan();
        mWifiList = mWifiManager.getScanResults();

        Log.d(TAG, "mWifiList.size(): " + mWifiList.size());
        if(mWifiList.size() > 0) {
            for (int i = 0; i < mWifiList.size(); ++i) {
                Log.d(TAG, "mWifiList[" + i + "].SSID : " + mWifiList.get(i).SSID);
            }
        }
    }
}
```

`AndroidManifest.xml`需要添加如下权限：  
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

以上。  

