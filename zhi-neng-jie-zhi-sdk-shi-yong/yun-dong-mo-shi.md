---
description: 一些戒指支持运动模式，可以开启运动模式，然后去运动，戒指会实时上传，如果戒指与app断连，下次连接时候，也可以通过断点续传的方式，将数据上传到app
icon: tennis-ball
---

# 运动模式

### android

```java
    /**
     * 开启运动模式
     */
    public static void START_SPORT(ISportListenerLite listenerLite) {
        iSportListenerLite = listenerLite;
        SEND_CMD(convertToBytes(0x38, new byte[]{0x1,2}));
    }

    /**
     * 获取历史运动模式，断点续传，手机和戒指断连的时候，戒指还在记录运动信息，
     下次打开app，获取运动历史，会把里边积攒的数据传到app里
     */
    public static void HIS_SPORT(ISportListenerLite listenerLite) {
        iSportListenerLite = listenerLite;
        SEND_CMD(convertToBytes(0x38, new byte[]{0x2}));
    }

    /**
     * 结束运动模式
     */
    public static void STOP_SPORT(ISportListenerLite listenerLite) {
        iSportListenerLite = listenerLite;
        SEND_CMD(convertToBytes(0x38, new byte[]{0x3}));
    }

```

返回结果是：

```java

public interface ISportListenerLite {

    /**
     * 运动模式
     * @param sportType 运动类型(1.主动推送，2断点续传)
     * @param step 步数
     * @param heart 心率
     * @param calorie 卡路里(暂不支持，默认为0)
     * @param time 时间
     */
    void onSport(int sportType,int step , int heart,int calorie,long time);
}
```

### iOS

```swift
/// 运动模式-开始
/// - Parameters:
///   - sportType: 运动类型 1:走路、2:跑步、3:游泳、4:待定、5:待定
///   - recordDuration: 记录时长（可选），时间单位-秒（定制参数，非定制项目不要使用）
///   - sliceDuration: 切片时长（可选），时间单位-秒（定制参数，非定制项目不要使用）
///   - completion: 开始回调
///   注意：此方法不再支持 sportDataCallback 和 passiveStopCallback 回调，请使用 BCLRingManager.shared.sportDataBlock 和 BCLRingManager.shared.passiveStopBlock 进行接收运动数据和被动停止回调
func startSportMode(sportType: Int,
                    recordDuration: UInt32? = nil,
                    sliceDuration: UInt32? = nil,
                    completion: @escaping (Result<BCLStartSportModeResponse, BCLError>) -> Void) 

/// 运动模式-主动停止
/// - Parameter completion: 停止回调
func stopSportMode(completion: @escaping (Result<BCLSportModeStopResponse, BCLError>) -> Void) 

/// 运动模式-请求漏点续传
/// - 注意：此方法不再支持回调，请使用 BCLRingManager.shared.sportDataMissingPointsBlock 进行接收运动数据，指令默认发送成功
func requestMissingPointsSportMode()

/// 运动数据回调
public var sportDataBlock: ((BCLSportDataResponse) -> Void)?
/// 被动停止回调
public var passiveStopBlock: ((BCLSportModePassiveStopResponse) -> Void)?
/// 运动数据漏传回调
public var sportDataMissingPointsBlock: ((BCLMissingPointsResponse) -> Void)?
```
