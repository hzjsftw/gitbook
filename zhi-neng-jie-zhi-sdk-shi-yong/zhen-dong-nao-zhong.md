---
description: 可以通过指令，设置戒指震动，测试戒指震动功能是否正常，也可以设置闹钟类型，比如仅一次，每天 ，智能节假日 ，智能工作日等
icon: clock-two
---

# 震动，闹钟

### 戒指震动

可以通过指令，设置戒指震动，测试戒指震动功能是否正常

**android:**

<pre class="language-java"><code class="lang-java"><strong>//time:时间（s），type:  1：强力振动 2：持续振动 3：渐变振动
</strong>LmAPI.SET_MOTOR(int time, int type)
</code></pre>

简化版方法名与正常版一致

```java
LmAPI或者LmAPILite 
/**
     * 设置线性马达参数
     * @param pattern 模式
     * @param dutyCycle 占空比
     * @param sequence 序列执行次数
     * @param repetitions 序列中周期重复次数
     */
    public static void SET_GOMORE_LINEAR(int pattern,int dutyCycle,int sequence,int repetitions) 

```

### 定制闹钟

接口功能：通过定制闹钟的方式，让戒指定时震动，只支持5个闹钟

简化版方法名与正常版一致

**android:**

<pre class="language-java"><code class="lang-java">/**
*设置闹钟
**/
<strong>LmAPI.ALARM_CLOCK_SETTING(List&#x3C;AlarmClockBean> alarmClockBeans);
</strong></code></pre>

```java
/**
*获取闹钟配置
**/
LmAPI.GET_ALARM_CLOCK(IAlarmClockListenerLite listenerLite);
```

AlarmClockBean说明：

```java
/**
     * time 时间戳
     * repetitiveType 重复类型 0：仅一次 1：每天 2：智能节假日 3：智能工作日
     * vibrationEffect 震动效果 0：强 1：弱 2：渐变
     * onOrOff 闹钟开关 0：关闭 1：打开
     */
    private  long time;
    private  byte repetitiveType;
    private  byte vibrationEffect;
    private  byte onOrOff;
```

```java
/**
*设置节假日日期
**/
LmAPI.HOLIDAY_SETTING(HolidayResult holidayResult, IAlarmClockListenerLite 
mIAlarmClockListenerLite);
 /**
     * 读取节假日日期（0x87）
     */
LmAPI.GET_HOLIDAY_SETTING(int year, IAlarmClockListenerLite mIAlarmClockListenerLite)
```

HolidayResult说明：

```java
public class HolidayResult {
    /**
     * 年份
     */
    private String year;
    /**
     * 全年非周末的假日天数m
     */
    private int nonWeekendHolidayCount;
    /**
     * 非周末的假日下标的列表（一年中的第几天）
     */
    private List<Integer> nonWeekendHolidayIndexList;
    /**
     * 周末中的工作天数
     */
    private int workingDaysOnWeekends;
    /**
     * 周末中的工作天数的列表（一年中的第几天）
     */
    private List<Integer> workingDaysOnWeekendsList;
    }
```

IAlarmClockListenerLite：

```java
public interface IAlarmClockListenerLite {

    /**
     * 闹钟列表
     * @param alarmClockBeanList
     */
    void result(List<AlarmClockBean> alarmClockBeanList);

    /**
     * 节假日
     * @param holidayResult
     */
    void holidayResult(HolidayResult holidayResult);
    /**
     * 设置闹钟返回结果
     */
    void setAlarmResult(boolean success);

    /**
     * 设置闹钟返回结果
     */
    void setHolidayResult(boolean success);
}
```

### 定时振动（倒计时）

**iOS:**

```swift
/// 震动马达-定时振动（倒计时）
/// - Parameters:
///   - seconds: 振动时间
///   - type: 振动类型
/// - Parameter completion: 定时振动设置回调
func linearMotorTimerVibration(seconds: Int, type: BCLVibrationMotorType, completion: @escaping (Result<BCLLinearMotorTimerVibrationResponse, BCLError>) -> Void)

/// 震动类型
public enum BCLVibrationMotorType: Int {
    /// 强力震动
    case strongVibration = 1
    /// 持续震动
    case continuousVibration = 2
    /// 渐变震动
    case gradientVibration = 3
}
```

#### 调用示例

```swift
BCLRingManager.shared.linearMotorTimerVibration(seconds: 5, type: .normal) { result in
    switch result {
    case .success(_):
        print("定时振动设置成功")
    case .failure(let error):
        print("定时振动设置失败: \(error)")
    }
}
```

### 立即振动

**iOS:**

```swift
/// 震动马达-立即振动
/// - Parameters:
///   - type: 振动类型
/// - Parameter completion: 立即振动设置回调
func linearMotorImmediateVibration(type: BCLVibrationMotorType, completion: @escaping (Result<BCLLinearMotorImmediateVibrationResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
BCLRingManager.shared.linearMotorImmediateVibration(type: .normal) { result in
    switch result {
    case .success(_):
        print("立即振动成功")
    case .failure(let error):
        print("立即振动失败: \(error)")
    }
}
```

### 设置闹钟

**iOS:**

```swift
/// 设置闹钟
/// - Parameters:
///   - items: 闹钟数据(最多5个)
///   - completion: 设置闹钟回调
/// - BCLSetAlarmClockResponse: 包含设置闹钟结果的响应模型
func setAlarmClock(items: [BCLAlarmClockData], completion: @escaping (Result<BCLSetAlarmClockResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
let alarmClock1 = BCLAlarmClockData(timestamp: "2025-06-24 12:33:00".toDate("yyyy-MM-dd HH:mm:ss", region: .local)?.date.timeIntervalSince1970 ?? 0,
                                    repeatType: .once,
                                    vibrationEffect: .strong,
                                    isEnabled: true,
                                    isMonday: true,
                                    isTuesday: true,
                                    isWednesday: true,
                                    isThursday: true,
                                    isFriday: true,
                                    isSaturday: true,
                                    isSunday: true)
let alarmClock2 = BCLAlarmClockData(timestamp: "2025-06-24 12:34:00".toDate("yyyy-MM-dd HH:mm:ss", region: .local)?.date.timeIntervalSince1970 ?? 0,
                                    repeatType: .once,
                                    vibrationEffect: .strong,
                                    isEnabled: true,
                                    isMonday: true,
                                    isTuesday: true,
                                    isWednesday: true,
                                    isThursday: true,
                                    isFriday: true,
                                    isSaturday: true,
                                    isSunday: true)
let alarmClock3 = BCLAlarmClockData(timestamp: "2025-06-24 12:35:00".toDate("yyyy-MM-dd HH:mm:ss", region: .local)?.date.timeIntervalSince1970 ?? 0,
                                    repeatType: .once,
                                    vibrationEffect: .strong,
                                    isEnabled: true,
                                    isMonday: true,
                                    isTuesday: true,
                                    isWednesday: true,
                                    isThursday: true,
                                    isFriday: true,
                                    isSaturday: true,
                                    isSunday: true)
let alarmClock4 = BCLAlarmClockData(timestamp: "2025-06-24 12:36:00".toDate("yyyy-MM-dd HH:mm:ss", region: .local)?.date.timeIntervalSince1970 ?? 0,
                                    repeatType: .once,
                                    vibrationEffect: .strong,
                                    isEnabled: true,
                                    isMonday: true,
                                    isTuesday: true,
                                    isWednesday: true,
                                    isThursday: true,
                                    isFriday: true,
                                    isSaturday: true,
                                    isSunday: true)

let alarmClock5 = BCLAlarmClockData(timestamp: "2025-06-24 12:37:00".toDate("yyyy-MM-dd HH:mm:ss", region: .local)?.date.timeIntervalSince1970 ?? 0,
                                    repeatType: .once,
                                    vibrationEffect: .strong,
                                    isEnabled: true,
                                    isMonday: true,
                                    isTuesday: true,
                                    isWednesday: true,
                                    isThursday: true,
                                    isFriday: true,
                                    isSaturday: true,
                                    isSunday: true)
BCLRingManager.shared.setAlarmClock(items: [alarmClock1, alarmClock2, alarmClock3, alarmClock4, alarmClock5]) { res in
    switch res {
    case let .success(response):
        if response.status == 1 {
            BDLogger.info("闹钟设置--成功")
        } else {
            BDLogger.error("闹钟设置--失败")
        }
    case let .failure(err):
        BDLogger.error("闹钟设置失败：\(err)")
    }
}
```

### 26.2 读取闹钟

**iOS:**

```swift
/// 读取闹钟
/// - Parameters:
///   - completion: 读取闹钟回调
/// - BCLReadAlarmClockResponse: 包含读取闹钟结果的响应模型
func readAlarmClock(completion: @escaping (Result<BCLReadAlarmClockResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
BCLRingManager.shared.readAlarmClock { res in
    switch res {
    case let .success(response):
        BDLogger.info("读取闹钟配置成功，共有\(response.items.count)个闹钟")
        for alarmClock in response.items {
            BDLogger.info("闹钟时间: \(alarmClock.timestamp)")
            BDLogger.info("重复类型: \(alarmClock.repeatType.rawValue)")
            BDLogger.info("振动效果: \(alarmClock.vibrationEffect.rawValue)")
            BDLogger.info("是否启用: \(alarmClock.isEnabled)")
            BDLogger.info("星期一: \(alarmClock.isMonday)")
            BDLogger.info("星期二: \(alarmClock.isTuesday)")
            BDLogger.info("星期三: \(alarmClock.isWednesday)")
            BDLogger.info("星期四: \(alarmClock.isThursday)")
            BDLogger.info("星期五: \(alarmClock.isFriday)")
            BDLogger.info("星期六: \(alarmClock.isSaturday)")
            BDLogger.info("星期日: \(alarmClock.isSunday)")
        }
    case let .failure(error):
        BDLogger.error("读取闹钟配置失败: \(error)")
    }
}
```

### 26.3 节假日日期设置

**iOS:**

```swift
/// 设置节假日日期
/// - Parameters:
///   - holidayData: 节假日日期数据
///   - completion: 设置节假日日期回调
func setHoliday(holidayData: BCLHolidayData, completion: @escaping (Result<BCLSetHolidayResponse, BCLError>) -> Void)

/// 读取节假日日期
/// - Parameters:
///   - year: 年份
///   - completion: 读取节假日日期回调
func readHoliday(year: Int, completion: @escaping (Result<BCLReadHolidayResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
// 注意：此处智能节假日配置仅为示例，具体可以配置内容可通过云端接口进行获取
let holidayData = BCLHolidayData(year: 2025, nonWeekendHolidayCount: 18, nonWeekendHolidayDays: [1, 28, 29, 30, 31, 34, 35, 94, 121, 122, 125, 153, 274, 275, 276, 279, 280, 281], workDaysCount: 5, workDays: [26, 39, 117, 271, 284])
BCLRingManager.shared.setHoliday(holidayData: holidayData) { res in
    switch res {
    case let .success(response):
        if response.status == 1 {
            BDLogger.info("智能节假日配置设置成功")
        } else {
            BDLogger.error("智能节假日配置设置失败")
        }
    case let .failure(err):
        BDLogger.error("智能节假日配置设置失败: \(err)")
    }
}

BCLRingManager.shared.readHoliday(year: 2025) { res in
    switch res {
    case let .success(response):
        guard let holidayData = response.holidayData else {
            BDLogger.info("智能节假日配置读取失败，数据为空")
            return
        }
        BDLogger.info("智能节假日配置读取成功")
        BDLogger.info("智能节假日数据-年份: \(holidayData.year)")
        BDLogger.info("智能节假日数据-全年非周末的假日天数: \(holidayData.nonWeekendHolidayCount)")
        BDLogger.info("智能节假日数据-非周六日的假日下nonWeekendHolidayCount标的列表（一年中的第几天）: \(holidayData.nonWeekendHolidayDays)")
        BDLogger.info("智能节假日数据-周末中的工作天数: \(holidayData.workDaysCount)")
        BDLogger.info("智能节假日数据-周末中的调休日期（是一年中的第几天): \(holidayData.workDays)")
        self.showSuccess("智能节假日配置读取成功")
    case let .failure(error):
        BDLogger.error("智能节假日配置读取失败: \(error)")
        self.showError("智能节假日配置读取失败: \(error)")
    }
}
```
