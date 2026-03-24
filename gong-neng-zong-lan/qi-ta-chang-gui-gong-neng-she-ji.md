---
description: 本小节的功能需要蓝牙连接状态才能够正常使用。本部分指令只提供总览，具体使用办法，可以去具体章节查看
icon: rectangles-mixed
---

# 其他常规功能设计

### 主动心率测量

触发主动测量有三个可选项配置：是否上传rawdata，是否上传测量进度，是否上传RR间期结果。开发者可以根据自己的算法需求进行配置。

*   安卓相关API：

    ```java
    LmAPILite.GET_HEART_ROTA
    ```
*   IOS相关API：

    ```swift
    /// 心率测量
    /// - Parameters:
    ///   - collectTime: 采集时间(单位：秒)
    ///   - collectFrequency: 采集频率(单位：次/秒)
    ///   - waveformConfig: 波形配置(0:不上传 1:上传)
    ///   - progressConfig: 进度配置(0:不上传 1:上传)
    ///   - intervalConfig: 间期配置(0:不上传 1:上传)
    /// - Result: 测量结果
    /// - BCLHeartRateResponse: 包含测量结果的响应模型
    /// - BCLError: 错误信息
    BCLRingManager.shared.startHeartRate(collectTime: Int, collectFrequency: Int, waveformConfig: Int, progressConfig: Int, intervalConfig: Int, completion: @escaping (Result<BCLHeartRateResponse, BCLError>) -> Void)
    ```

### 主动血氧测量

触发主动测量有三个可选项配置：是否上传rawdata，是否上传测量进度。开发者可以根据自己的算法需求进行配置。

*   安卓相关API：

    ```java
    LmAPILite.GET_HEART_ROTA
    LmAPILite.STOP_HEART()
    ```
*   IOS相关API：

    ```swift
    /// 血氧测量
    /// - Parameters:
    ///   - collectTime: 采集时间(单位：秒)
    ///   - collectFrequency: 采集频率(单位：次/秒)
    ///   - waveformConfig: 波形配置(0:不上传 1:上传)
    ///   - progressConfig: 进度配置(0:不上传 1:上传)
    /// - Result: 测量结果
    /// - BCLBloodOxygenResponse: 包含测量结果的响应模型
    /// - BCLError: 错误信息
    BCLRingManager.shared.startBloodOxygen(collectTime: Int, collectFrequency: Int, waveformConfig: Int, progressConfig: Int, completion: @escaping (Result<BCLBloodOxygenResponse, BCLError>) -> Void)
    ```

### 主动心电图测量

心电图测量的UI界面设计是复杂的，我们开发了绘图方法，在demo的./sourece/文件夹下。

*   安卓相关API：

    ```java
    LmAPILite.STAR_ELEC()//开启心电测量
    LmAPILite.STOP_ELECTROCARDIOGRAM();//结束心电测量
    ```
* IOS相关API：

### 触摸手势设置

触摸手势的设置不是一直保持的，当戒指充电或者低电量关机后再次开机的时候，功能是默认关闭的。

*   安卓相关API：

    ```java
    LmAPILite.GET_HID_CODE//获取HID功能码
    LmAPILite.SET_HID;//设置戒指的HID模式
    LmAPILite.GET_HID;//获取HID模式
    ```
* IOS相关API：

### 实时语音传输

有两种传输模式： 1.APP下发实时语音的开始和停止。 2.触摸控制实时语音的采集和推送。

*   安卓相关API：

    ```java
    LmAPILite.SET_AUDIO//控制adpcm格式音频传输
    LmAPILite.CONTROL_AUDIO_ADPCM_AUDIO;//设置主动推送音频信息
    LmAPILite.GET_CONTROL_AUDIO_ADPCM;//获取主动推送音频信息
    ```
* IOS相关API：

### 血压测量

*   安卓相关API：

    ```java
    LmAPILite.BLOOD_PRESSURE_APP//开始测量血压
    LmAPILite.STOP_BLOOD_PRESSURE_APP;//停止血压测量
    ```
* IOS相关API：

### 震动闹钟设置

可以按照闹钟的方式设置戒指震动，最多设置5个闹钟

*   安卓相关API：

    ```java
    LmAPILite.ALARM_CLOCK_SETTING//闹钟设置
    LmAPILite.GET_ALARM_CLOCK;//读取闹钟设置
    ```
* IOS相关API：

### OTA无线升级

本SDK封装了OTA文件服务器的网络接口和OTA蓝牙协议。

*   安卓相关API：

    ```java
    LogicalApi.createToken //申请token
    OtaApi.otaUpdateWithCheckVersion//ota升级
    ```
* IOS相关API：

### 蓝牙名字

*   安卓相关API：

    ```java
    LmAPILite.Set_BlueTooth_Name//设置蓝牙名称
    LmAPILite.Get_BlueTooth_Name//获取蓝牙名称
    ```
* IOS相关API：

```swift
/// 设置蓝牙名称
/// - Parameter name: 蓝牙名称（字节长度1-12）
/// - Parameter completion: 设置蓝牙名称回调
/// - BCLSetBluetoothNameResponse: 包含设置结果的响应模型
func setBluetoothName(name: String, completion: @escaping (Result<BCLSetBluetoothNameResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
BCLRingManager.shared.setBluetoothName(name: "MyRing") { result in
    switch result {
    case .success(let response):
        print("设置蓝牙名称成功")
    case .failure(let error):
        print("设置蓝牙名称失败: \(error)")
    }
}
```

**iOS:**

```swift
/// 获取蓝牙名称
/// - Parameter completion: 获取蓝牙名称回调
/// - BCLReadBluetoothNameResponse: 包含获取结果的响应模型
func getBluetoothName(completion: @escaping (Result<BCLReadBluetoothNameResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
BCLRingManager.shared.getBluetoothName { result in
    switch result {
    case .success(let response):
        print("蓝牙名称: \(response.name)")
    case .failure(let error):
        print("获取蓝牙名称失败: \(error)")
    }
}
```

### 测量间隔

*   安卓相关API：

    ```java
    LmAPILite.SET_COLLECTION//设置
    LmAPILite.GET_COLLECTION//读取
    ```
* IOS相关API：

```swift
/// 设置采集周期
/// - Parameter period: 采集周期（单位：秒）最小值为60
/// - Parameter completion: 设置采集周期回调
/// - Result: 设置结果
/// - BCLSetCollectPeriodResponse: 包含设置结果的响应模型
/// - BCLError: 错误信息
BCLRingManager.shared.setCollectPeriod(period: Int, completion: @escaping (Result<BCLSetCollectPeriodResponse, BCLError>) -> Void)

/// 获取采集周期
/// - Parameter completion: 获取采集周期回调
/// - Result: 获取结果
/// - BCLGetCollectPeriodResponse: 包含采集周期的响应模型
/// - BCLError: 错误信息
BCLRingManager.shared.getCollectPeriod(completion: @escaping (Result<BCLGetCollectPeriodResponse, BCLError>) -> Void)
```

### 用户信息

*   安卓相关API：

    <pre class="language-java"><code class="lang-java">    /**
       *  设置个人信息
       * @param sex 性别，0女，1男
       * @param height 身高，单位1cm
       * @param weight 体重，单位0.1kg
       * @param age 年龄，单位月
       */
    <strong>LmAPILite.SET_USER_INFO(int sex,int height,int weight,int age) //设置用户信息
    </strong>
    LmAPILite.GET_USER_INFO//获取个人信息
    </code></pre>
* IOS相关API： **iOS:**

```swift
/// 设置个人信息
/// - Parameters:
///   - sex: 性别（0：女、1：男）
///   - age: 年龄(单位：月)
///   - height: 身高(单位：cm)
///   - weight: 体重(单位：kg)
///   - completion: 设置个人信息回调
/// - BCLSetPersonalInformationResponse: 包含设置结果的响应模型
func setPersonalInformation(sex: Int, age: Int, height: Int, weight: Int, completion: @escaping (Result<BCLSetPersonalInformationResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
BCLRingManager.shared.setPersonalInformation(sex: 1, age: 300, height: 175, weight: 70) { result in
    switch result {
    case .success(let response):
        print("设置个人信息成功")
    case .failure(let error):
        print("设置个人信息失败: \(error)")
    }
}
```

**iOS:**

```swift
/// 获取个人信息
/// - Parameter completion: 获取个人信息回调
/// - BCLReadPersonalInformationResponse: 包含个人信息的响应模型
func getPersonalInformation(completion: @escaping (Result<BCLReadPersonalInformationResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
BCLRingManager.shared.getPersonalInformation { result in
    switch result {
    case .success(let response):
        print("性别: \(response.sex), 年龄: \(response.age)月, 身高: \(response.height)cm, 体重: \(response.weight)kg")
    case .failure(let error):
        print("获取个人信息失败: \(error)")
    }
}
```
