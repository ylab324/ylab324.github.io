---
title: '[转载]Android System Server大纲之LightsService'
date: 2019-03-19
categories: Android
tags: [android,service]
---

## 前言
从功能机以来，手机等设备就配备了Led闪光灯，当有未接电话、未读短信和通知等等，Led就会闪烁。和振动器类似，都是给用户一种人机交互的反馈，哪怕这种人机交互是那么的简单。既然它是一个Led，也就是一个硬件，Android系统上层驱动这个硬件的服务就是LightsService，所以，这个文章也是描述一个软硬件结合的功能。但是上层APP不能直接驱动Led硬件，LightsService是系统所使用。    


<!-- more -->


既然是软硬结合的一个功能，那么在Android系统里，从上到下实现这个功能的架构如下：  
![FamilyYuan.png](https://i.loli.net/2019/03/19/5c90af777f197.png)  



## LightsService初始化
回顾《[Android系统之System Server大纲](https://blog.csdn.net/myfriend0/article/details/55098173)》一文，LightsService在frameworks/base/services/java/com/android/server/SystemServer.java中的启动过程是：  
```java
public final class SystemServer {
        // Manages LEDs and display backlight so we need it to bring up the display.
        mSystemServiceManager.startService(LightsService.class);
}
```
从上面的代码中，LightsService是通过SystemServiceManager.startService()的方式启动，在《[Android系统之System Server大纲](https://blog.csdn.net/myfriend0/article/details/55098173)》一文中提到的Android系统的各种服务的启动方式可知，LightsService是继承了SystemService，LightsService启动后会回调onStart()方法。代码如下：  
```java
public class LightsService extends SystemService {
    @Override
    public void onStart() {
        publishLocalService(LightsManager.class, mService);
    }
}
```
上述代码在frameworks/base/services/core/java/com/android/server/lights/LightsService.java中。  
在《[Android系统之System Server大纲](https://blog.csdn.net/myfriend0/article/details/55098173)》一文中提到，publishLocalService()的作用，这里是把mService推进LocalServices中，所以外部引用LightsService时，实际是拿到mService这个对象。mService的代码如下：  
```java
public class LightsService extends SystemService {
    private final LightsManager mService = new LightsManager() {
        @Override
        public Light getLight(int id) {
            if (id < LIGHT_ID_COUNT) {
                return mLights[id];
            } else {
                return null;
            }
        }
    };
}
```
mService的实质是一个LightsManager，LightsManager只提供一个方法getLight(int id)，从数组mLights中匹配一个返回值，返回值是什么类型呢？看mLights[]的本质，如下：  
```java
public class LightsService extends SystemService {
    final LightImpl mLights[] = new LightImpl[LightsManager.LIGHT_ID_COUNT];
}
```
mLights[]的实质一个LightImpl的数组，所以LightsManager的getLight(int id)返回的是一个LightImpl的实例，LightImpl的代码如下：  
```java
public class LightsService extends SystemService {
    private final class LightImpl extends Light {

        ......

        @Override
        public void setFlashing(int color, int mode, int onMS, int offMS) {
            synchronized (this) {
                setLightLocked(color, mode, onMS, offMS, BRIGHTNESS_MODE_USER);
            }
        }

       ......
    }
}
```
LightImpl实际是Light的子类，而Light实质是一个led硬件的抽象，那么一个led就被抽象成一个Light对象。因此，要控制led，只要拿到led的抽象对象Light即可实现驱动led。  


回头再看看LightsService的初始化，LightsService的构造方法如下：  
```java
public class LightsService extends SystemService {
    public LightsService(Context context) {
        super(context);

        mNativePointer = init_native();

        for (int i = 0; i < LightsManager.LIGHT_ID_COUNT; i++) {
            mLights[i] = new LightImpl(i);
        }
    }
}
```
上文提到的LightImpl数组mLights就是在LightsService中被初始化了，这里会根据LightsManager.LIGHT_ID_COUNT的总数循环生成LightImpl的实例保存在mLights数组中。这里的LightsManager.LIGHT_ID_COUNT的值是8，代码如下：  
```java
public abstract class LightsManager {
    public static final int LIGHT_ID_BACKLIGHT = 0;
    public static final int LIGHT_ID_KEYBOARD = 1;
    public static final int LIGHT_ID_BUTTONS = 2;
    public static final int LIGHT_ID_BATTERY = 3;
    public static final int LIGHT_ID_NOTIFICATIONS = 4;
    public static final int LIGHT_ID_ATTENTION = 5;
    public static final int LIGHT_ID_BLUETOOTH = 6;
    public static final int LIGHT_ID_WIFI = 7;
    public static final int LIGHT_ID_COUNT = 8;

    public abstract Light getLight(int id);
}
```
从LIGHT_ID_BACKLIGHT到LIGHT_ID_WIFI共8个，这里是否表示8个led呢，后文在论述清楚。回到LightsService的构造方法，调用了init_native()方法，当然，熟悉Android架构的都清楚，其它硬件的服务和LightsService一样，都有这么一个过程。目的就是LightsService启动时，调用init_native()初始化硬件，也就是软硬件的通道在init_native()这个方法中打通。init_native()是一个native方法，对应的JNI接口是：  
```java
static jlong init_native(JNIEnv* /* env */, jobject /* clazz */)
{
    int err;
    hw_module_t* module;
    Devices* devices;

    devices = (Devices*)malloc(sizeof(Devices));

    err = hw_get_module(LIGHTS_HARDWARE_MODULE_ID, (hw_module_t const**)&module);
    if (err == 0) {
        ......
        devices->lights[LIGHT_INDEX_NOTIFICATIONS]
                = get_device(module, LIGHT_ID_NOTIFICATIONS);
        ......
    } else {
        memset(devices, 0, sizeof(Devices));
    }

    return (jlong)devices;
}
```
这个函数定义在文件frameworks/base/services/core/jni/com_android_server_lights_LightsService.cpp中。  


init_native()中首先调用了hw_get_module()函数，这个函数就是打开硬件设备，这个是函数的实现过程在hardware/libhardware/hardware.c中，对Android HAL熟悉的读者，应该很熟悉这个过程，本文也不往下赘述这个过程了。然后通过调用get_device()函数取得返回值赋值Devices的变量lights，get_device()的一个参数是LIGHT_ID_NOTIFICATIONS，这和上文中LightsManager的8个led id是对应的。Devices的变量lights的实质是：  
```java
struct Devices {
    light_device_t* lights[LIGHT_COUNT];
};
```
light_device_t定义在hardware/libhardware/include/hardware/lights.h文件中。回到init_native()函数，接着看get_device()这个函数：  
```java
static light_device_t* get_device(hw_module_t* module, char const* name)
{
    int err;
    hw_device_t* device;
    err = module->methods->open(module, name, &device);
    if (err == 0) {
        return (light_device_t*)device;
    } else {
        return NULL;
    }
}
```
通过module->methods->open()打开硬件设备，这个过程本文就不再赘述了。打开设备通道后，取得light_device_t，保存在Devices的数组lights中，这和LightsService上文中的LightImpl的数组mLights也是对应的关系。  



## 通过LightsService启动Led
APP虽然不能直接驱动led，但是led一般也是为Android的Notification所使用，所以APP发起一个通知，led也会闪烁。本文就从Android的notification入手，分析驱动led的过程。  

APP创建Notification时，可以通过如下方法指定这个notification需要led：  
```java
public Builder setLights(@ColorInt int argb, int onMs, int offMs) {
    mN.ledARGB = argb;
    mN.ledOnMS = onMs;
    mN.ledOffMS = offMs;
    if (onMs != 0 || offMs != 0) {
        mN.flags |= FLAG_SHOW_LIGHTS;
    }
    return this;
}
```
本文暂时不对这个方法进行过多的阐述，会在以后的文章中再详细介绍Android的notification机制。当然APP如果不调用这个方法，那么Android系统会自动为APP的notification选取默认值。  

APP的notification通知到Android系统以后，就会驱动led闪烁起来。在上文LightsService的初始化中提到，led硬件已经抽象成一个Light对象，这个对象便是LightImpl，获取到LightImpl的实例，便可驱动led。Android的notification获取LightImpl的代码如下：  
```java
public class NotificationManagerService extends SystemService {
    final LightsManager lights = getLocalService(LightsManager.class);
    mNotificationLight = lights.getLight(LightsManager.LIGHT_ID_NOTIFICATIONS);
}
```
这个类定义在文件frameworks/base/services/core/java/com/android/server/notification/NotificationManagerService.java中。  

在上文LightsService的初始化中可知，LightsService被推进到LocalService中，所以是通过getLocalService()获取到LightsManager，然后再通过getLight()方法，获取到LightImpl的实例，这个过程在上文LightsService的初始化中已经阐述的很明白。获取到了LightImpl，通过什么接口驱动led呢？先看看LightImpl的结构：  
```java
public abstract class Light {
    ......
    public abstract void setBrightness(int brightness);
    public abstract void setBrightness(int brightness, int brightnessMode);
    public abstract void setColor(int color);
    public abstract void setFlashing(int color, int mode, int onMS, int offMS);
    public abstract void pulse();
    public abstract void pulse(int color, int onMS);
    public abstract void turnOff();
}
```
这个类定义在文件frameworks/base/services/core/java/com/android/server/lights/Light.java中。  

如上面的代码，LightImpl提供setBrightness()设置亮度、setColor设置颜色、setFlashing启动led等7个方法。因此，Anroid notification驱动led的代码如下：  
```java
mNotificationLight.setFlashing(ledARGB, Light.LIGHT_FLASH_TIMED,ledOnMS, ledOffMS);
```
这个方法定义在文件frameworks/base/services/core/java/com/android/server/notification/NotificationManagerService.java中。  
setFlashing()接收四个参数，第一个当然是颜色，类型是rgb；第二参数是mode，数值分别是可以是：  
```java
public static final int LIGHT_FLASH_NONE = 0;
public static final int LIGHT_FLASH_TIMED = 1;
public static final int LIGHT_FLASH_HARDWARE = 2;
```
这些变量定义在文件frameworks/base/services/core/java/com/android/server/lights/Light.java中。  
第三个和第四个是一对相反的参数，分别表示led闪烁时亮的时间，led闪烁时灭的时间。接着看setFlashing()的实现：  
```java
private final class LightImpl extends Light {
    public void setFlashing(int color, int mode, int onMS, int offMS) {
        synchronized (this) {
            setLightLocked(color, mode, onMS, offMS, BRIGHTNESS_MODE_USER);
        }
    }
}
```
这个方法定义在文件frameworks/base/services/core/java/com/android/server/lights/LightsService.java中。  
直接调用了setLightLocked()，代码如下：  
```java
private final class LightImpl extends Light {
    private void setLightLocked(int color, int mode, int onMS, int offMS, int brightnessMode) {
    if (!mLocked && (color != mColor || mode != mMode || onMS != mOnMS || offMS != mOffMS ||
            mBrightnessMode != brightnessMode)) {
        if (DEBUG) Slog.v(TAG, "setLight #" + mId + ": color=#"
                + Integer.toHexString(color) + ": brightnessMode=" + brightnessMode);
        mLastColor = mColor;
        mColor = color;
        mMode = mode;
        mOnMS = onMS;
        mOffMS = offMS;
        mLastBrightnessMode = mBrightnessMode;
        mBrightnessMode = brightnessMode;
        Trace.traceBegin(Trace.TRACE_TAG_POWER, "setLight(" + mId + ", 0x"
                + Integer.toHexString(color) + ")");
        try {
            setLight_native(mNativePointer, mId, color, mode, onMS, offMS, brightnessMode);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_POWER);
        }
    }
}
```
这个方法定义在文件frameworks/base/services/core/java/com/android/server/lights/LightsService.java中。  

对参数整理一遍后，调用了JNI方法setLight_native()，这里的第二个参数便是LightsManager.LIGHT_ID_NOTIFICATIONS = 4；继续往下看setLight_native()：  
```cpp
static void setLight_native(JNIEnv* /* env */, jobject /* clazz */, jlong ptr,
        jint light, jint colorARGB, jint flashMode, jint onMS, jint offMS, jint brightnessMode)
{
    Devices* devices = (Devices*)ptr;
    light_state_t state;
    ......
    state.brightnessMode = brightnessMode;

    {
        ALOGD_IF_SLOW(50, "Excessive delay setting light");
        devices->lights[light]->set_light(devices->lights[light], &state);
    }
}
```
这个函数定义在文件frameworks/base/services/core/jni/com_android_server_lights_LightsService.cpp中。  

set_light()的定义是在文件hardware/libhardware/include/hardware/lights.h中，具体实现，就因led的硬件厂商不同而不同了，这里以某个led的厂商为例：  
```cpp
static int open_lights(const struct hw_module_t *module, char const *name,
               struct hw_device_t **device)
{
    struct dragon_lights *lights;
    ......
    lights->base.set_light = set_light_backlight;

    *device = (struct hw_device_t *)lights;
    return 0;
}
```
在led通道初始化过程中，set_light()函数实际调用的是set_light_backlight函数，set_light_backlight函数如下：  
```cpp
static int set_light_backlight(struct light_device_t *dev,
                   struct light_state_t const *state)
{
    struct dragon_lights *lights = to_dragon_lights(dev);
    int err, brightness_idx;
    int brightness = rgb_to_brightness(state);

    if (brightness > 0) {
        // Get the bin number for brightness (0 to kNumBrightnessLevels - 1)
        brightness_idx = (brightness - 1) * kNumBrightnessLevels / 0xff;

        // Get brightness level
        brightness = kBrightnessLevels[brightness_idx];
    }

    pthread_mutex_lock(&lights->lock);
    err = write_brightness(lights, brightness);
    pthread_mutex_unlock(&lights->lock);

    return err;
}
```
跟踪到这里，本文也就不往下跟踪了，这些代码都会因led厂商不同而不同。驱动led的过程就到此结束。  



## 总结
本文阐述了LightsService的作用，从打开硬件通道和驱动硬件led闪烁起来的过程，虽然APP是不可以直接驱动led，但是led几乎都是为android notification所用，所以，APP发布的notification也可以使用led，通知用户有新的状态。对于上文提到的led颜色，对于很多设备，是不支持设置颜色了的，设置了也只能是一种颜色，所以在这些设备上，可以说是大多数设备，这个功能基本就没有任何用处。另外，因led硬件的不同，文章中提到的8个led id，可能有，可能没有。  



- - -  
1. :point_right: [原文链接](https://blog.csdn.net/myfriend0/article/details/56482594)  
2. 网上另外一篇不错的文章：[lights从上到下的流程](https://blog.csdn.net/zhangchiytu/article/details/7958513)  
