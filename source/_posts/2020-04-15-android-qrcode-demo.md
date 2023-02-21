---
title: '安卓生成二维码demo'
date: 2020-04-15
categories: Android
tags: [android,qrcode,demo]
---

举个栗子。  

<!-- more -->

1. 需要用到`zxing`这个库。  
下载`core-3.4.0.jar`这个东西，修改`Android.mk`,将`core-3.4.0.jar`写到`LOCAL_STATIC_JAVA_LIBRARIES`和`LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES`，用于编译。  

2. 添加二维码工具类`QRCodeUtils.java`  

	```java
	package com.test.menu.util;
	
	import android.graphics.Bitmap;
	import android.graphics.Canvas;
	import android.graphics.Matrix;
	import android.support.annotation.Nullable;
	import android.text.TextUtils;
	
	import com.google.zxing.BarcodeFormat;
	import com.google.zxing.EncodeHintType;
	import com.google.zxing.WriterException;
	import com.google.zxing.common.BitMatrix;
	import com.google.zxing.qrcode.QRCodeWriter;
	
	import java.util.Hashtable;
	
	public class QRCodeUtils {
		
	    private static final String TAG = "QRCodeUtil";
		
		public static Bitmap createQRImage(String url, final int width, final int height) {
	        try {
				
	            if (url == null || "".equals(url) || url.length() < 1) {
	                return null;
	            }
	            Hashtable<EncodeHintType, String> hints = new Hashtable<EncodeHintType, String>();
	            hints.put(EncodeHintType.CHARACTER_SET, "utf-8");
	            hints.put(EncodeHintType.MARGIN, "1");
	
	            BitMatrix bitMatrix = new QRCodeWriter().encode(url,
	                    BarcodeFormat.QR_CODE, width, height, hints);
	            int[] pixels = new int[width * height];
	
	
	            for (int y = 0; y < height; y++) {
	                for (int x = 0; x < width; x++) {
	                    if (bitMatrix.get(x, y)) {
	                        pixels[y * width + x] = 0xff000000;
	                    } else {
	                        pixels[y * width + x] = 0xffffffff;
	                    }
	                }
	            }
	
	            Bitmap bitmap = Bitmap.createBitmap(width, height,
	                    Bitmap.Config.ARGB_8888);
	            bitmap.setPixels(pixels, 0, width, 0, 0, width, height);
	            return bitmap;
	        } catch (WriterException e) {
	            e.printStackTrace();
	        }
	        return null;
	    }
	}
	
	```

3. 制作菜单  
略  

4. 把二维码图片设置到菜单  
二维码图片的内容是往`createQRImage`传入的字符串。  

	```java
	String allString = "helloworld";
	mImageView.setImageBitmap(QRCodeUtils.createQRImage(allString, 128, 128));
	```
