---
description: >-
  复合指令使用的前提是，蓝牙设备已经通过手机连接，是简化了设备连接以后，通过指令，获取设备信息的操作，分为绑定，连接，刷新（这些是应用上的概念，不是物理设备的绑定连接刷新），是复合指令，一个指令包含之前的多个指令的结果，大大加快了连接速度，减少了指令冲突的风险，强烈建议使用符合指令
icon: sparkle
---

# 复合指令

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

App刷新，是在app里，用户主动进行刷线，比如下拉刷新等，戒指收到这条指令执行，自动上传未上传数据，同步时间。timeMillis的意思是上传该时间戳以后的历史数据，一般默认为0就行，如果涉及到睡眠服务，需要设置为云端最后一条历史数据的时间戳，参考《睡眠服务》，这个可以在应用中，随时进行刷新

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
    private byte chargingStatus;//充电状态(0未充电，1充电中，充满电)
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

### 绑定戒指

**iOS:**

```swift
/// APP事件-绑定戒指
/// - Parameters:
///   - date: 时间
///   - timeZone: 时区
/// - Parameter completion: 绑定戒指回调
func appEventBindRing(date: Date, timeZone: BCLRingTimeZone, completion: @escaping (Result<BCLBindRingResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
// 该指令适用于用户首次绑定戒指时使用，提交当前时间和时区信息，以确保戒指与APP的时间同步，同时获取戒指的基本信息和功能支持情况。
// 注意：该指令执行时，戒指内部会根据情况清除历史数据。适用场景如：用户首次使用App绑定戒指，或用户更换新的戒指时使用。避免戒指初始状态下存在旧数据影响使用体验。

// 此处提交当前时间和时区信息，以确保戒指与APP的时间同步,功能同时间同步接口一致
BCLRingManager.shared.appEventBindRing(date: Date(), timeZone: BCLRingTimeZone.getCurrentSystemTimeZone()) { res in
    switch res {
    case let .success(response):
        BDLogger.info("绑定戒指成功: \(response)")
        //  此处可根据业务需求，将数据更新到相关UI中
        BDLogger.info("固件版本: \(response.firmwareVersion)")
        BDLogger.info("硬件版本: \(response.hardwareVersion)")
        BDLogger.info("电量: \(response.batteryLevel)")
        BDLogger.info("充电状态: \(response.chargingState)")
        BDLogger.info("采集间隔: \(response.collectInterval)")
        BDLogger.info("计步: \(response.stepCount)")
        //  自检相关信息如果有错误信息，可以根据信息进行一下提醒处理。
        BDLogger.info("自检标志：\(response.selfInspectionFlag)")
        BDLogger.info("自检是否有错误：\(response.hasSelfInspectionError)")
        BDLogger.info("自检错误描述：\(response.selfInspectionErrorDescription)")

        //  可用于检查蓝牙设备是否支持HID相关功能，并根据支持情况启用相应功能
        BDLogger.info("HID功能支持：\(response.isHIDSupported)")
        if response.isHIDSupported {
            BDLogger.info("HID模式-触摸功能-拍照：\(response.isTouchPhotoSupported)")
            BDLogger.info("HID模式-触摸功能-短视频模式：\(response.isTouchShortVideoSupported)")
            BDLogger.info("HID模式-触摸功能-控制音乐：\(response.isTouchMusicControlSupported)")
            BDLogger.info("HID模式-触摸功能-控制PPT：\(response.isTouchPPTControlSupported)")
            BDLogger.info("HID模式-触摸功能-控制上传实时音频：\(response.isTouchAudioUploadSupported)")
            BDLogger.info("HID模式-手势功能-捏一捏手指拍照：\(response.isPinchPhotoSupported)")
            BDLogger.info("HID模式-手势功能-手势短视频模式：\(response.isGestureShortVideoSupported)")
            BDLogger.info("HID模式-手势功能-空中手势音乐控制：\(response.isGestureMusicControlSupported)")
            BDLogger.info("HID模式-手势功能-空中手势PPT模式：\(response.isGesturePPTControlSupported)")
            BDLogger.info("HID模式-手势功能-打响指拍照模式：\(response.isSnapPhotoSupported)")
            BDLogger.info("当前HID模式-触摸模式：\(response.touchHIDMode.description)")
            BDLogger.info("当前HID模式-手势模式：\(response.gestureHIDMode.description)")
            BDLogger.info("当前HID模式-系统类型：\(response.systemType.description)")
        }

        //  可用于检查心率、血氧等曲线是否支持，并根据支持情况启用相应功能
        BDLogger.info("血氧曲线支持：\(response.isOxygenCurveSupported)")
        BDLogger.info("变异性曲线支持：\(response.isVariabilityCurveSupported)")
        BDLogger.info("压力曲线支持：\(response.isPressureCurveSupported)")
        BDLogger.info("温度曲线支持：\(response.isTemperatureCurveSupported)")
        BDLogger.info("女性健康支持：\(response.isFemaleHealthSupported)")
        BDLogger.info("震动闹钟支持：\(response.isVibrationAlarmSupported)")
        BDLogger.info("心电图功能支持：\(response.isEcgFunctionSupported)")
        BDLogger.info("麦克风支持：\(response.isMicrophoneSupported)")
        BDLogger.info("运动模式支持：\(response.isSportModeSupported)")
        BDLogger.info("血压测量支持：\(response.isBloodPressureMeasurementSupported)")
        BDLogger.info("血糖测量支持:\(response.isBloodGlucoseMeasurementSupported) ")
        BDLogger.info("文件支持:\(response.isFileSystemSupported) ")
    case let .failure(error):
        switch error {
        case let .responseParsing(reason):
            BDLogger.error("绑定戒指响应解析失败: \(reason.localizedDescription)")
        default:
            BDLogger.error("绑定戒指指令执行-失败: \(error)")
        }
    }
}
```

### 连接戒指

**iOS:**

```swift
/// APP事件-连接戒指
/// - Parameters:
///   - date: 时间戳
///   - timeZone: 时区
///   - filterTime: 过滤时间（可选）
///   - callbacks: 回调集合
/// - Parameter completion: 连接戒指回调
func appEventConnectRing(date: Date, timeZone: BCLRingTimeZone, filterTime: Date? = nil, callbacks: BCLDataSyncCallbacks, completion: @escaping (Result<BCLConnectRingResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
// 该指令适用于App启动后连接戒指成功后使用，提交当前时间和时区信息，以确保戒指与APP的时间同步，功能同时间同步接口一致，同时获取戒指的基本信息和功能支持情况。且戒指会主动将历史数据同步到APP。
// App在接收到数据后，可以根据业务需求将数据存储到本地数据库或上传到服务器进行进一步处理和分析。

// 当前指令默认会将戒指内未上传数据全部同步到APP。(后续会根据FilterTime参数，支持只同步某个时间点之后的数据)
// 创建回调结构体
let callbacks = BCLDataSyncCallbacks(
    onProgress: { totalNumber, currentIndex, progress, model in
        BDLogger.info("连接戒指-历史数据同步进度：\(currentIndex)/\(totalNumber) (\(progress)%)")
        BDLogger.info("连接戒指-当前数据：\(model.localizedDescription)")
    },
    onStatusChanged: { status in
        BDLogger.info("连接戒指-历史数据同步状态变化：\(status)")
        switch status {
        case .syncing:
            BDLogger.info("同步中...")
        case .noData:
            BDLogger.info("没有历史数据")
        case .completed:
            BDLogger.info("同步完成")
        case .error:
            BDLogger.error("同步出错")
        }
    },
    onCompleted: { models in
        BDLogger.info("连接戒指-历史数据同步完成，共获取 \(models.count) 条记录")
        BDLogger.info("\(models)")
        // 此处数据同步过程中会通过异步方式将数据存储到SDK的本地数据库中，APP可根据业务需求将数据存储到自己的本地数据库或上传到服务器进行进一步处理和分析。
        // 如果使用了云端睡眠算法，此处可调用数据上传接口将数据提交到云端服务，并获取睡眠分析结果进行展示。
        self.historyData.append(contentsOf: models)
    },
    onError: { error in
        BDLogger.error("连接戒指-历史数据同步出错：\(error.localizedDescription)")
    }
)

// 设置过滤时间（可选）(如果不需要过滤时间，可以传nil，表示不过滤) (传入时间则只会同步过滤时间之后的数据)
// 这里设置为2025-01-01 00:00:00，表示只同步该时间之后的数据
// 此处注意该过滤时间参数目前仅部分固件支持
let filterTime = "2025-01-01 00:00:00".toDate("yyyy-MM-dd HH:mm:ss", region: BCLRingTimeZone.getCurrentSystemTimeZone().region)?.date
BDLogger.info("APP事件-连接戒指-过滤时间: \(String(describing: filterTime))")
BCLRingManager.shared.appEventConnectRing(date: Date(), timeZone: BCLRingTimeZone.getCurrentSystemTimeZone(), filterTime: filterTime, callbacks: callbacks) { res in
    switch res {
    case let .success(response):
        //  此处可根据业务需求，将数据更新到相关UI中
        BDLogger.info("App复合指令-连接指令执行-成功: \(response)")
        BDLogger.info("固件版本: \(response.firmwareVersion)")
        BDLogger.info("硬件版本: \(response.hardwareVersion)")
        BDLogger.info("电量: \(response.batteryLevel)")
        BDLogger.info("充电状态: \(response.chargingState)")
        BDLogger.info("采集间隔: \(response.collectInterval)")
        BDLogger.info("计步: \(response.stepCount)")

        //  自检相关信息如果有错误信息，可以根据信息进行一下提醒处理。
        BDLogger.info("自检标志：\(response.selfInspectionFlag)")
        BDLogger.info("自检是否有错误：\(response.hasSelfInspectionError)")
        BDLogger.info("自检错误描述：\(response.selfInspectionErrorDescription)")

        //  可用于检查蓝牙设备是否支持HID相关功能，并根据支持情况启用相应功能
        BDLogger.info("HID功能支持：\(response.isHIDSupported)")
        if response.isHIDSupported {
            BDLogger.info("HID模式-触摸功能-拍照：\(response.isTouchPhotoSupported)")
            BDLogger.info("HID模式-触摸功能-短视频模式：\(response.isTouchShortVideoSupported)")
            BDLogger.info("HID模式-触摸功能-控制音乐：\(response.isTouchMusicControlSupported)")
            BDLogger.info("HID模式-触摸功能-控制PPT：\(response.isTouchPPTControlSupported)")
            BDLogger.info("HID模式-触摸功能-控制上传实时音频：\(response.isTouchAudioUploadSupported)")
            BDLogger.info("HID模式-手势功能-捏一捏手指拍照：\(response.isPinchPhotoSupported)")
            BDLogger.info("HID模式-手势功能-手势短视频模式：\(response.isGestureShortVideoSupported)")
            BDLogger.info("HID模式-手势功能-空中手势音乐控制：\(response.isGestureMusicControlSupported)")
            BDLogger.info("HID模式-手势功能-空中手势PPT模式：\(response.isGesturePPTControlSupported)")
            BDLogger.info("HID模式-手势功能-打响指拍照模式：\(response.isSnapPhotoSupported)")
            BDLogger.info("当前HID模式-触摸模式：\(response.touchHIDMode.description)")
            BDLogger.info("当前HID模式-手势模式：\(response.gestureHIDMode.description)")
            BDLogger.info("当前HID模式-系统类型：\(response.systemType.description)")
        }

        //  可用于检查心率、血氧等曲线是否支持，并根据支持情况启用相应功能
        BDLogger.info("心率曲线支持：\(response.isHeartRateCurveSupported)")
        BDLogger.info("血氧曲线支持：\(response.isOxygenCurveSupported)")
        BDLogger.info("变异性曲线支持：\(response.isVariabilityCurveSupported)")
        BDLogger.info("压力曲线支持：\(response.isPressureCurveSupported)")
        BDLogger.info("温度曲线支持：\(response.isTemperatureCurveSupported)")
        BDLogger.info("女性健康支持：\(response.isFemaleHealthSupported)")
        BDLogger.info("震动闹钟支持：\(response.isVibrationAlarmSupported)")
        BDLogger.info("心电图功能支持：\(response.isEcgFunctionSupported)")
        BDLogger.info("麦克风支持：\(response.isMicrophoneSupported)")
        BDLogger.info("运动模式支持：\(response.isSportModeSupported)")
        BDLogger.info("血压测量支持：\(response.isBloodPressureMeasurementSupported)")
        BDLogger.info("血糖测量支持:\(response.isBloodGlucoseMeasurementSupported) ")
        BDLogger.info("文件支持:\(response.isFileSystemSupported) ")
    case let .failure(error):
        BDLogger.error("连接戒指指令执行-失败: \(error)")
    }
}
```

### 刷新戒指

**iOS:**

```swift
/// APP事件-刷新戒指
/// - Parameters:
///   - date: 时间戳
///   - timeZone: 时区
///   - filterTime: 过滤时间（可选）
///   - callbacks: 回调集合
/// - Parameter completion: 刷新戒指回调
func appEventRefreshRing(date: Date, timeZone: BCLRingTimeZone, filterTime: Date? = nil, callbacks: BCLDataSyncCallbacks, completion: @escaping (Result<BCLRefreshRingResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
// 该指令适用于需要刷新页面场景，提交当前时间和时区信息，以确保戒指与APP的时间同步，功能同时间同步接口一致，同时获取戒指的基本信息和功能支持情况。且戒指会主动将历史数据同步到APP。
// App在接收到数据后，可以根据业务需求将数据存储到本地数据库或上传到服务器进行进一步处理和分析。

// 当前指令默认会将戒指内未上传数据全部同步到APP。(后续会根据FilterTime参数，支持只同步某个时间点之后的数据)
// 创建回调结构体
let callbacks = BCLDataSyncCallbacks(
    onProgress: { totalNumber, currentIndex, progress, model in
        BDLogger.info("刷新戒指-历史数据同步进度：\(currentIndex)/\(totalNumber) (\(progress)%)")
        BDLogger.info("刷新戒指-当前数据：\(model.localizedDescription)")
    },
    onStatusChanged: { status in
        BDLogger.info("刷新戒指-历史数据同步状态变化：\(status)")
        switch status {
        case .syncing:
            BDLogger.info("同步中...")
        case .noData:
            BDLogger.info("没有历史数据")
        case .completed:
            BDLogger.info("同步完成")
        case .error:
            BDLogger.error("同步出错")
        }
    },
    onCompleted: { models in
        BDLogger.info("刷新戒指-历史数据同步完成，共获取 \(models.count) 条记录")
        BDLogger.info("\(models)")
        // 此处数据同步过程中会通过异步方式将数据存储到SDK的本地数据库中，APP可根据业务需求将数据存储到自己的本地数据库或上传到服务器进行进一步处理和分析。
        // 如果使用了云端睡眠算法，此处可调用数据上传接口将数据提交到云端服务，并获取睡眠分析结果进行展示。
        self.historyData.append(contentsOf: models)
    },
    onError: { error in
        BDLogger.error("刷新戒指-历史数据同步出错：\(error.localizedDescription)")
    }
)

// 设置过滤时间（可选）(如果不需要过滤时间，可以传nil，表示不过滤) (传入时间则只会同步过滤时间之后的数据)
// 这里设置为2025-01-01 00:00:00，表示只同步该时间之后的数据
// 此处注意该过滤时间参数目前仅部分固件支持
let filterTime = "2025-01-01 00:00:00".toDate("yyyy-MM-dd HH:mm:ss", region: BCLRingTimeZone.getCurrentSystemTimeZone().region)?.date
BDLogger.info("APP事件-刷新戒指-过滤时间: \(String(describing: filterTime))")
BCLRingManager.shared.appEventRefreshRing(date: Date(), timeZone: BCLRingTimeZone.getCurrentSystemTimeZone(), filterTime: filterTime, callbacks: callbacks) { res in
    switch res {
    case let .success(response):
        BDLogger.info("刷新戒指指令执行-成功: \(response)")

        //  此处可根据业务需求，将数据更新到相关UI中
        BDLogger.info("固件版本: \(response.firmwareVersion)")
        BDLogger.info("硬件版本: \(response.hardwareVersion)")
        BDLogger.info("电量: \(response.batteryLevel)")
        BDLogger.info("充电状态: \(response.chargingState)")
        BDLogger.info("采集间隔: \(response.collectInterval)")
        BDLogger.info("计步: \(response.stepCount)")

        //  自检相关信息如果有错误信息，可以根据信息进行一下提醒处理。
        BDLogger.info("自检标志：\(response.selfInspectionFlag)")
        BDLogger.info("自检是否有错误：\(response.hasSelfInspectionError)")
        BDLogger.info("自检错误描述：\(response.selfInspectionErrorDescription)")

        //  可用于检查蓝牙设备是否支持HID相关功能，并根据支持情况启用相应功能
        BDLogger.info("HID功能支持：\(response.isHIDSupported)")
        if response.isHIDSupported {
            BDLogger.info("HID模式-触摸功能-拍照：\(response.isTouchPhotoSupported)")
            BDLogger.info("HID模式-触摸功能-短视频模式：\(response.isTouchShortVideoSupported)")
            BDLogger.info("HID模式-触摸功能-控制音乐：\(response.isTouchMusicControlSupported)")
            BDLogger.info("HID模式-触摸功能-控制PPT：\(response.isTouchPPTControlSupported)")
            BDLogger.info("HID模式-触摸功能-控制上传实时音频：\(response.isTouchAudioUploadSupported)")
            BDLogger.info("HID模式-手势功能-捏一捏手指拍照：\(response.isPinchPhotoSupported)")
            BDLogger.info("HID模式-手势功能-手势短视频模式：\(response.isGestureShortVideoSupported)")
            BDLogger.info("HID模式-手势功能-空中手势音乐控制：\(response.isGestureMusicControlSupported)")
            BDLogger.info("HID模式-手势功能-空中手势PPT模式：\(response.isGesturePPTControlSupported)")
            BDLogger.info("HID模式-手势功能-打响指拍照模式：\(response.isSnapPhotoSupported)")
            BDLogger.info("当前HID模式-触摸模式：\(response.touchHIDMode.description)")
            BDLogger.info("当前HID模式-手势模式：\(response.gestureHIDMode.description)")
            BDLogger.info("当前HID模式-系统类型：\(response.systemType.description)")
        }

        //  可用于检查心率、血氧等曲线是否支持，并根据支持情况启用相应功能
        BDLogger.info("心率曲线支持：\(response.isHeartRateCurveSupported)")
        BDLogger.info("血氧曲线支持：\(response.isOxygenCurveSupported)")
        BDLogger.info("变异性曲线支持：\(response.isVariabilityCurveSupported)")
        BDLogger.info("压力曲线支持：\(response.isPressureCurveSupported)")
        BDLogger.info("温度曲线支持：\(response.isTemperatureCurveSupported)")
        BDLogger.info("女性健康支持：\(response.isFemaleHealthSupported)")
        BDLogger.info("震动闹钟支持：\(response.isVibrationAlarmSupported)")
        BDLogger.info("心电图功能支持：\(response.isEcgFunctionSupported)")
        BDLogger.info("麦克风支持：\(response.isMicrophoneSupported)")
        BDLogger.info("运动模式支持：\(response.isSportModeSupported)")
        BDLogger.info("血压测量支持：\(response.isBloodPressureMeasurementSupported)")
        BDLogger.info("血糖测量支持:\(response.isBloodGlucoseMeasurementSupported) ")
        BDLogger.info("文件支持:\(response.isFileSystemSupported) ")
    case let .failure(error):
        BDLogger.error("刷新戒指指令执行-失败: \(error)")
    }
}
```
