---
description: >-
  按照智能穿戴硬件产品和APP之间的从属关系，此SDK把重要的关键操作事件进行了组合，组成了五种事件：绑定，连接，刷新，断连回调，解绑。使通讯效率有效提高。以下指令为二代指令，推荐新用户对接使用。此类指令使用的前提是戒指已经通过蓝牙进行了连接。
icon: circle-2
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# sdk二代指令简介

### 二代指令优势

以android为例，一个指令，可以获取很多配置，可以在连接或刷新的时候，省略掉获取版本号，获取电量，获取充电状态，获取采集周期，获取HID配置，一键自检，当前计步以及设备支持哪些功能的指令，提高连接速度。

```java
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
```

### 绑定指令

* 触发条件：1.在用户第一次注册后绑定设备 或者 2.用户解绑过设备，当前账户下没有绑定的设备。
* 执行操作：蓝牙扫描，设备过滤，设备连接，发送“绑定事件”请求，等待接收“绑定事件”应答。
* 说明：戒指收到绑定事件后，会自动将戒指恢复出厂设置，清空出厂时候测试的数据记录，设置当前时间时区，清空步数等。
*   安卓相关API：

    ```java
    LmAPI.APP_BIND(IBindConnectRefreshListenerLite listenerLite)
    ```
* IOS相关API：

```swift
/// APP事件-绑定戒指
/// - Parameters:
///   - date: 当前时间
///   - timeZone: 时区
/// - Parameter completion: 绑定戒指回调
BCLRingManager.shared.appEventBindRing(date: Date, timeZone: BCLRingTimeZone, completion: @escaping (Result<BCLBindRingResponse, BCLError>) -> Void)
```

### 连接指令

* 触发条件：在账户已经绑定过戒指的情况下打开APP。
* 执行操作：设备连接，发送“连接事件”请求，等待接受“连接事件”应答。
* 说明：获取设备在未连接状态下产生的数据记录，同步时间，软硬件版本号，当前步数，电量，功能配置。
*   安卓相关API：

    ```java
    /**
         *
         * @param timeMillis 获取时间戳以后的数据，默认传0
         * @param listenerLite
    */
    LmAPI.APP_CONNECT(long timeMillis,IBindConnectRefreshListenerLite listenerLite) 
    ```
* IOS相关API：

```swift
/// APP事件-连接戒指
/// - Parameters:
///   - date: 当前时间
///   - timeZone: 时区
///   - callbacks: 回调集合
/// - Parameter completion: 连接戒指回调
BCLRingManager.shared.appEventConnectRing(date: Date, timeZone: BCLRingTimeZone, callbacks: BCLDataSyncCallbacks, completion: @escaping (Result<BCLConnectRingResponse, BCLError>) -> Void)
```

### 刷新指令

* 触发条件：在APP打开的情况下，APP下拉刷新
* 执行操作：发送“刷新事件”请求，等待接收“刷新事件”应答。
* 说明：获取设备在未连接状态下产生的数据记录，同步事件，软硬件版本号，当前步数，电量，功能配置。此指令和设备连接指令类似。
*   安卓相关API：

    ```java
    /**
         *
         * @param timeMillis 获取时间戳以后的数据，默认传0
         * @param listenerLite
    */
    LmAPI.APP_REFRESH(long timeMillis,IBindConnectRefreshListenerLite listenerLite) 
    ```
* IOS相关API：

```swift
  /// APP事件-刷新戒指
  /// - Parameters:
  ///   - date: 当前时间
  ///   - timeZone: 时区
  ///   - callbacks: 回调集合
  /// - Parameter completion: 刷新戒指回调
  BCLRingManager.shared.appEventRefreshRing(date: Date, timeZone: BCLRingTimeZone, callbacks: BCLDataSyncCallbacks, completion: @escaping (Result<BCLRefreshRingResponse, BCLError>) -> Void)
```
