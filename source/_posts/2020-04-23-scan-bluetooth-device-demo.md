---
title: '蓝牙扫描demo'
date: 2020-04-23
categories: Android
tags: [bluetooth,android,demo]
---


安卓扫描附近的蓝牙设备。  


<!-- more -->


```java
//MainActivity.java
package com.example.bluetoothtest;

import android.Manifest;
import android.bluetooth.BluetoothAdapter;
import android.bluetooth.BluetoothDevice;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.net.ConnectivityManager;
import android.os.Build;
import android.os.Message;
import android.support.v4.app.ActivityCompat;
import android.support.v4.content.ContextCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "SystemInfoFragment";
    private ConnectivityManager mConnectivityManager;
    private BluetoothAdapter btAdapt;
    private List<String> mBluetoothList;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mBluetoothList = new ArrayList<>();

        btAdapt = BluetoothAdapter.getDefaultAdapter();

        if(btAdapt.getState() != BluetoothAdapter.STATE_ON) {
            btAdapt.enable();
        }

        mBluetoothList.clear();
        if (btAdapt.isDiscovering()) {
            btAdapt.cancelDiscovery();
        }

        if (Build.VERSION.SDK_INT >= 23) {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
                //需要动态申请权限
                ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.ACCESS_COARSE_LOCATION}, 10);
            }
        }

        btAdapt.startDiscovery();

        IntentFilter intent = new IntentFilter();
        intent.addAction(BluetoothDevice.ACTION_FOUND);
        intent.addAction(BluetoothAdapter.ACTION_DISCOVERY_STARTED);
        intent.addAction(BluetoothAdapter.ACTION_DISCOVERY_FINISHED);
        registerReceiver(scanBluetoothDevices, intent);
    }

    @Override
    public void onDestroy() {
        if(scanBluetoothDevices != null) {
            unregisterReceiver(scanBluetoothDevices);
        }
        if (btAdapt.isDiscovering()) {
            btAdapt.cancelDiscovery();
        }
        mBluetoothList.clear();
        super.onDestroy();
    }

    public BroadcastReceiver scanBluetoothDevices = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();

            if (action.equals(BluetoothDevice.ACTION_FOUND)) {
                //found device
                BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
                int deviceType = device.getType();
                String str = device.getName() + "(" + device.getAddress() + ")";
                Log.d(TAG, "found bluetooth device: " + deviceType + " - " + str);

                if (mBluetoothList.indexOf(str) == -1) {

                    if(deviceType == BluetoothDevice.DEVICE_TYPE_CLASSIC)
                        Log.d(TAG, "deviceType: DEVICE_TYPE_CLASSIC");
                    else if(deviceType == BluetoothDevice.DEVICE_TYPE_LE)
                        Log.d(TAG, "deviceType: DEVICE_TYPE_LE");
                    else if(deviceType == BluetoothDevice.DEVICE_TYPE_DUAL)
                        Log.d(TAG, "deviceType: DEVICE_TYPE_DUAL");
                    else if(deviceType == BluetoothDevice.DEVICE_TYPE_UNKNOWN)
                        Log.d(TAG, "deviceType: DEVICE_TYPE_UNKNOWN");

                    mBluetoothList.add(str);
                    Log.d(TAG, "add bluetooth device: " + str);
                }

            } else if (action.equals(BluetoothAdapter.ACTION_DISCOVERY_STARTED)) {
                //Scanning
                Log.d(TAG, "start scanning bluetooth devices... ");
            } else if (action.equals(BluetoothAdapter.ACTION_DISCOVERY_FINISHED)) {
                //Scan Finished
                Log.d(TAG, "scan bluetooth devices finished");
                Log.d(TAG, "mBluetoothList size = " + mBluetoothList.size());
            }
        }
    };
}
```

`AndroidManifest.xml`需要添加如下权限：  
```xml
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```


简单总结就是：先`startDiscovery()`，然后注册一个搜索监听广播`scanBluetoothDevices`。  
`startDiscovery()`这个接口能扫所有设备，包括经典蓝牙和BLE。  
网上资料说，不设定扫描周期的话，默认12秒，但是实测并不都是12秒。  
顺便一提，BLE最大的广播间隔是10.25秒。  
顺便再提，蓝牙耳机都是经典蓝牙，BLE目前的协议是不支持音频传输的。  

遇到的问题：  
1. 编译不过。"error: cannot find symbol registerReceiver/unregisterReceiver"，当时是写在`Fragment`中，但是`Fragment`是没有这个方法的。最终在`onAttach(Activity activity)`中`activity.registerReceiver(scanBluetoothDevices, intent);`，在`onDetach()`中`getActivity().unregisterReceiver(scanBluetoothDevices);`。  
2. 菜单上看到一些空白设备。当时以为是扫不到设备，检查打印才知道有扫到了，只是设备没有名字，所以就空白了，但是打印能看到有`mac`地址。  

