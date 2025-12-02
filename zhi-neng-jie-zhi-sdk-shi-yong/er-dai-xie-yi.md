---
description: >-
  二代指令使用的前提是，蓝牙设备已经通过手机连接，是简化了设备连接以后，通过指令，获取设备信息的操作，分为绑定，连接，刷新（这些是应用上的概念，不是物理设备的绑定连接刷新），是复合指令，一个指令包含之前的多个指令的结果，大大加快了连接速度，减少了指令冲突的风险，强烈建议使用二代指令
icon: '2'
---

# 二代协议

## Android部分

### APP绑定

App绑定，是在搜索到设备，然后连接到系统蓝牙里以后，延时3s左右，再获取设备信息，戒指收到这条指令执行，恢复出厂设置（清空历史记录，清除步数)，同步时间，HID功能获取

```java
    LmAPI.APP_BIND();
```

回调：

```java
 @Override
    public void appBind(SystemControlBean systemControlBean) {
        postView("\nappBind："+systemControlBean.toString());
    }
```

### APP连接

App连接，是在下次进入app的时候，连接设备，戒指收到这条指令执行，自动上传未上传数据，同步时间，HID功能获取。timeMillis的意思是上传该时间戳以后的历史数据，一般默认为0就行，如果涉及到睡眠服务，需要设置为云端最后一条历史数据的时间戳，参考《睡眠服务》

{% content-ref url="../sheng-ji-fu-wu/shui-mian-fu-wu.md" %}
[shui-mian-fu-wu.md](../sheng-ji-fu-wu/shui-mian-fu-wu.md)
{% endcontent-ref %}

```java
    LmAPI.APP_CONNECT(long timeMillis);
```

回调：

```java
 @Override
    public void appConnect(SystemControlBean systemControlBean) {
        postView("\nappConnect："+systemControlBean.toString());
    }
```

### APP刷新

App刷新，是在app里，用户主动进行刷线，比如下拉刷新等，戒指收到这条指令执行，自动上传未上传数据，同步时间。timeMillis的意思是上传该时间戳以后的历史数据，一般默认为0就行，如果涉及到睡眠服务，需要设置为云端最后一条历史数据的时间戳，参考《睡眠服务》

{% content-ref url="../sheng-ji-fu-wu/shui-mian-fu-wu.md" %}
[shui-mian-fu-wu.md](../sheng-ji-fu-wu/shui-mian-fu-wu.md)
{% endcontent-ref %}

```java
    LmAPI.APP_REFRESH(long timeMillis);
```

回调：

```java
     @Override
    public void appRefresh(SystemControlBean systemControlBean) {
        postView("\nappRefresh："+systemControlBean.toString());
    }
```

### 历史数据处理

因为APP\_CONNECT和APP\_REFRESH会自动上传未上传数据，需要在页面进行监听，否则会报错

```java
LmAPI.READ_HISTORY_AUTO(IHistoryListener iHistoryListener)
简化版：
LmAPILite.READ_HISTORY_AUTO(IHistoryListenerLite iHistoryListener)
```

### 实体类描述

```java
public class SystemControlBean {
   private String firmwareVersion;//固件版本号
    private String hardwareVersion;//硬件版本号
    private byte battery;//电量
    private byte chargingStatus;//充电状态
    private String collectionInterval;//当前采集间隔
    private byte[] HID_CODE;//当前HID功能码
    private byte[] HID_MODE;//当前HID模式
    private byte heartRate;//心率曲线支持
    private byte blood;//血氧曲线支持
    private byte variability;//变异性曲线支持
    private byte pressure;//压力曲线支持
    private byte temperature;//温度曲线支持
    private byte womenHealth;//女性健康支持
    private byte vibration;//震动闹钟支持
    private byte electrocardiogram;//心电图功能支持
    private byte microphone;//麦克风支持
    private byte sport;//运动模式支持
    private int stepCounting;//当前计步
    private int keyTest;//自检标识
    private int bloodPressure;//血压
    private int bloodSugar;//血糖
    private int fileSystem;//文件系统
    private int gomoreSleep;//固件是否支持gomore睡眠算法,1支持0不支持
    }
    //简化版本的实体类
    public class SystemControlBean {
    private String firmwareVersion;//固件版本号
    private String hardwareVersion;//硬件版本号
    private int battery;//电量
    private int chargingStatus;//充电状态
    private String collectionInterval;//当前采集间隔
    private boolean HIDSupport;//HID功能支持
    private TouchSupport touchSupport;//触摸支持
    private GestureSupport gestureSupport;//手势支持
    private int HidTouch;// 触摸hid 模式，0：刷视频模式，1：拍照模式，2：音乐模式，3：ppt模式，4：上传实时音频，0xFF:关闭

    private int HidGesture;//手势hid 模式，0：刷视频模式，1：拍照模式，2：音乐模式，3：ppt模式，4：打响指(拍照)模式，0xFF:关闭

    private int HidSystem;//系统类型 0：安卓，1：IOS，2：WINDOS

    private byte heartRate;//心率曲线支持
    private byte blood;//血氧曲线支持
    private byte variability;//变异性曲线支持
    private byte pressure;//压力曲线支持
    private byte temperature;//温度曲线支持
    private byte womenHealth;//女性健康支持
    private byte vibration;//震动闹钟支持
    private byte electrocardiogram;//心电图功能支持
    private byte microphone;//麦克风支持
    private byte sport;//运动模式支持
    private int stepCounting;//当前计步
    private int keyTest;//自检标识
    private int bloodPressure;//血压
    private int bloodSugar;//血糖
    private int fileSystem;//文件系统
    private int gomoreSleep;//gomore睡眠算法
    }
    
 //简化版本
 public static void APP_BIND(IBindConnectRefreshListenerLite listenerLite)
 public static void APP_CONNECT(IBindConnectRefreshListenerLite listenerLite)
 public static void APP_REFRESH(IBindConnectRefreshListenerLite listenerLite)

 public interface IBindConnectRefreshListenerLite {
     void appBind(SystemControlBean systemControlBean);
     void appConnect(SystemControlBean systemControlBean);
     void appRefresh(SystemControlBean systemControlBean);
 }
```
