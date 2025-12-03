---
description: 这部分指令，为厂家定制指令，仅供定制厂家使用
icon: industry
---

# 厂家定制指令

* &#x20;**实时PPG血压测量**
* **6轴协议**
* **寿世PPG波形传输**

### **实时PPG血压测量**

接口功能：实时测量血压值和500hz的原始波形

**android：**\
接口声明：

```java
LmAPI.GET_REAL_TIME_BP（byte time,byte isWave,byte isProgress,IRealTimePPGBpListener iRealTimePPGBpListener）
```

注意事项：戒指固件必须支持，否则无法使用。 参数说明：\
time：采集时间,byte类型，默认30s\
isWave:是否上传波形。0：不上传，1：上传\
isProgress：是否上传进度。0：不上传，1：上传

```java
LmAPI.GET_REAL_TIME_BP((byte) 0x30, (byte) 1, (byte) 1, new IRealTimePPGBpListener() {
                    @Override
                    public void progress(int progress) {
                        //进度
                    }

                    @Override
                    public void bpResult(byte type) {
                        //[0]:舒张压
                        //[1]:收缩压
                    }

                    @Override
                    public void resultData(String bpData) {
                        //bpData包含红外值
                    }
             });
```

简化版本

```java
   public static void GET_REAL_TIME_BP(int time,int isWave,int isProgress,IRealTimePPGBpListenerLite iRealTimePPGBpListener)
   public interface IRealTimePPGBpListenerLite {
    void progress(int progress);
    /**
     * 血压响应
     * @param bloodPressureType 0：舒张压，1：收缩压
     */
    void bpResult(int bloodPressureType);
    /**
     * 血压算法响应
     * @param bpData 响应数据
     */
    void resultData(String bpData);

    void  stopRealTimeBP();
}
```

**iOS:**
```Swift
/// 血压测量
/// - Parameters:
///   - collectTime: 采集时间(单位：秒)
///   - waveformConfig: 波形配置(0:不上传 1:上传)
///   - progressConfig: 进度配置(0:不上传 1:上传)
/// - BCLBloodPressureResponse: 包含测量结果的响应模型
func startBloodPressure(collectTime: Int, waveformConfig: Int, progressConfig: Int, completion: @escaping (Result<BCLBloodPressureResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.startBloodPressure(collectTime: 30, waveformConfig: 1, progressConfig: 1) { result in
    switch result {
    case .success(let response):
        print("血压: \(response.systolic)/\(response.diastolic) mmHg")
        // 此处波形数据需要提交到服务端进行血压计算后获取结果
    case .failure(let error):
        print("血压测量失败: \(error)")
    }
}
```

接口功能：停止采集

**android：**\
接口声明：

```java
LmAPI.STOP_REAL_TIME_BP()
```

注意事项：戒指固件必须支持，否则无法使用。 参数说明：无\
回调：

```java
 @Override
    public void stopRealTimeBP(byte isSend) {
        if(isSend == (byte)0x01){
            Logger.show("TAG","停止采集已发送");
        }
  }
```

简化版本

```java
  public static void STOP_REAL_TIME_BP(IRealTimePPGBpListenerLite iRealTimePPGBpListener)
  public interface IRealTimePPGBpListenerLite {
    void progress(int progress);
    /**
     * 血压响应
     * @param bloodPressureType 0：舒张压，1：收缩压
     */
    void bpResult(int bloodPressureType);
    /**
     * 血压算法响应
     * @param bpData 响应数据
     */
    void resultData(String bpData);

    void  stopRealTimeBP();
}
```

**iOS:**
```Swift
/// 停止血压测量
/// - Parameter completion: 停止血压测量回调
/// - BCLStopBloodPressureResponse: 包含停止血压测量结果的响应模型
func stopBloodPressure(completion: @escaping (Result<BCLStopBloodPressureResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.stopBloodPressure { result in
    switch result {
    case .success(_):
        print("停止血压测量成功")
    case .failure(let error):
        print("停止血压测量失败: \(error)")
    }
}
```

### **6轴传感器协议**

```java
 LmAPI.TURN_OFF_6_AXIS_SENSORS( I6axisListener listenerLite) //关闭6轴传感器数据上报
 LmAPI.READ_6_AXIS_SENSORS(I6axisListener listenerLite);//读6轴传感器加速度数据（单次）
 LmAPI.READ_6_AXIS_ACCELERATION(I6axisListener listenerLite);//请求：读6轴传感器实时加速度数据（开启后一直上传直至接收到停止指令）
```

回调：

```java
public interface I6axisListener {
    /**
     * 关闭传感器
     */
    void turnOff();

    /**
     * 传感器数据

     */
    void sensorsData(String bpData);

}
```

### 设置六轴传感器工作频率

**iOS:**
```Swift
/// 设置六轴传感器工作频率 (暂不支持分开设置，需保证加速度、陀螺仪频率一致)
/// - Parameters:
///   - accelerationFrequency: 加速度频率 (25hz，50hz，100hz，150hz，200hz)
///   - gyroscopeFrequency: 陀螺仪频率 (25hz，50hz，100hz，150hz，200hz)
/// - BCLSetWorkFrequencyResponse: 包含设置结果的响应模型 status: 1成功，0失败
func setSixAxisWorkFrequency(accelerationFrequency: Int, gyroscopeFrequency: Int, completion: @escaping (Result<BCLSetWorkFrequencyResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.setSixAxisWorkFrequency(accelerationFrequency: 100, gyroscopeFrequency: 100) { result in
    switch result {
    case .success(let response):
        print("设置六轴工作频率成功")
    case .failure(let error):
        print("设置失败: \(error)")
    }
}
```

### 获取六轴传感器工作频率

**iOS:**
```Swift
/// 获取六轴传感器工作频率
/// - BCLGetWorkFrequencyResponse: 包含获取结果的响应模型
func getSixAxisWorkFrequency(completion: @escaping (Result<BCLGetWorkFrequencyResponse, BCLError>) -> Void)
```

### 获取六轴传感器数据（单次）

**iOS:**
```Swift
/// 获取六轴传感器-加速度数据(单次)
func getSixAxisAccelerationData(completion: @escaping (Result<BCLReadAccelerationResponse, BCLError>) -> Void)

/// 获取六轴传感器-陀螺仪数据(单次)
func getSixAxisGyroscopeData(completion: @escaping (Result<BCLReadGyroscopeResponse, BCLError>) -> Void)

/// 获取六轴传感器-加速度和陀螺仪数据(单次)
func getSixAxisAccelerationAndGyroscopeData(completion: @escaping (Result<BCLReadAccelerationAndGyroscopeResponse, BCLError>) -> Void)
```

### 获取六轴传感器实时数据（持续上传）

**iOS:**
```Swift
/// 获取六轴传感器-加速度数据(开启后一直上传直至接收到停止指令)
func getSixAxisRealTimeAccelerationData(completion: @escaping (Result<BCLReadRealTimeAccelerationResponse, BCLError>) -> Void)

/// 获取六轴传感器-陀螺仪数据(开启后一直上传直至接收到停止指令)
func getSixAxisRealTimeGyroscopeData(completion: @escaping (Result<BCLReadRealTimeGyroscopeResponse, BCLError>) -> Void)

/// 获取六轴传感器-加速度和陀螺仪数据(开启后一直上传直至接收到停止指令)
func getSixAxisRealTimeAccelerationAndGyroscopeData(completion: @escaping (Result<BCLReadRealTimeAccelerationAndGyroscopeResponse, BCLError>) -> Void)
```

### 停止六轴传感器数据上传

**iOS:**
```Swift
/// 停止六轴传感器数据上传
func stopSixAxisData(completion: @escaping (Result<BCLCloseSixAxisResponse, BCLError>) -> Void)
```

### 设置六轴传感器省电模式

**iOS:**
```Swift
/// 设置六轴传感器省电模式
func setSixAxisPowerSavingMode(completion: @escaping (Result<BCLSetPowerSavingModeResponse, BCLError>) -> Void)
```

### **寿世PPG波形传输**

**android:**

```java
 /**
     * 寿世PPG波形传输（0x3D
     * @param collectionTime 采集时间，默认30(0为一直采集)
     * @param waveformConfiguration 波形配置0:不上传 1:上传
     * @param progressConfiguration 进度配置0:不上传 1:上传
     * @param waveformSetting 波形配置0：125hz，绿色,1:25hz，绿色+红外 2:佩戴检测(无波形响应)
     * @param iHeartListener
     */
 LmAPI. GET_PPG_SHOUSHI(byte collectionTime,byte waveformConfiguration,byte progressConfiguration,byte waveformSetting,IHeartListener iHeartListener) 

 LmAPI.STOP_PPG_SHOUSHI(IHeartListener iHeartListener);//停止寿世PPG波形传输（0x3D)
```

回调

```java
public interface IHeartListener {

    /**
     * 进度
     * @param progress
     */
    void progress(int progress);


    /**
     * 常规设备的心率监测
     * @param heart 心率
     * @param heartRota 心率变异性
     * @param yaLi 压力
     * @param temp 温度
     */
    void resultData(int heart,int heartRota,int yaLi,int temp);

    /**
     * 波形图
     * @param seq 序号
     * @param number 数据个数
     * @param waveData 波形数据
     */
    void waveformData(byte seq,byte number,String waveData);

    /**
     * 间期响应
     * @param seq 序号
     * @param number 数据个数
     * @param data RR间期
     */
    void rriData(byte seq,byte number,String data);

    /**
     * 错误
     * 0	未佩戴
     * 1	佩戴(保留)
     * 2	充电不允许采集
     * 4	繁忙，不执行
     * 5	数据采集超时
     * @param code
     */
    void error(int code);

    /**
     * 采集完成
     */
    void success();

    /**
     * 停止测量
     */
    void stop();

    /**
     * 寿世定制的ppg返回
     * @param heart 心率
     * @param bloodOxygen 血氧
     */
    void resultDataSHOUSHI(int heart,int bloodOxygen);

}
```

定制化功能，涉及到的回调返回有error，resultDataSHOUSHI，waveformData，progress，success，stop

### 开始PPG波形测量
**iOS:**
```Swift
/// PPG波形测量
/// - Parameters:
///   - collectTime: 采集时间，默认30、0为一直采集
///   - waveConfig: 波形配置 0:不上传 1:上传
///   - progressConfig: 进度配置 0:不上传 1:上传
///   - waveSetting: 波形设置 0：125hz，绿色、1：25hz，绿色+红外、2：佩戴检测（无波形响应）
///   - completion: 测量结果回调
func ppgWaveFormMeasurement(collectTime: Int, waveConfig: Int, progressConfig: Int, waveSetting: Int, completion: @escaping (Result<BCLPPGWaveFormResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.ppgWaveFormMeasurement(collectTime: 30, waveConfig: 1, progressConfig: 1, waveSetting: 0) { result in
    switch result {
    case .success(let response):
        print("PPG波形测量成功")
    case .failure(let error):
        print("PPG波形测量失败: \(error)")
    }
}
```

### 停止PPG波形测量

**iOS:**
```Swift
/// PPG波形测量停止
/// - Parameters:
///   - completion: 停止结果回调
func ppgWaveFormStop(completion: @escaping (Result<BCLPPGWaveFormStopResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.ppgWaveFormStop { result in
    switch result {
    case .success(_):
        print("停止PPG波形测量成功")
    case .failure(let error):
        print("停止PPG波形测量失败: \(error)")
    }
}
```