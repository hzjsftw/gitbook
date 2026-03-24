---
description: 这部分指令，为厂家定制指令，仅供定制厂家使用
icon: industry
---

# 厂家定制指令

* **实时PPG血压测量**
* **6轴协议**
* **寿世PPG波形传输**

### **实时PPG血压测量**

接口功能：实时测量血压值和500hz的原始波形

**android：**\
接口声明：

```java
LmAPILite.GET_REAL_TIME_BP（byte time,byte isWave,byte isProgress,IRealTimePPGBpListener iRealTimePPGBpListener）
```

注意事项：戒指固件必须支持，否则无法使用。 参数说明：\
time：采集时间,默认30s\
isWave:是否上传波形。0：不上传，1：上传\
isProgress：是否上传进度。0：不上传，1：上传

```java
  LmAPILite：
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

```swift
/// 血压测量
/// - Parameters:
///   - collectTime: 采集时间(单位：秒)
///   - waveformConfig: 波形配置(0:不上传 1:上传)
///   - progressConfig: 进度配置(0:不上传 1:上传)
/// - BCLBloodPressureResponse: 包含测量结果的响应模型
func startBloodPressure(collectTime: Int, waveformConfig: Int, progressConfig: Int, completion: @escaping (Result<BCLBloodPressureResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
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

**android：**

注意事项：戒指固件必须支持，否则无法使用。 参数说明：无

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

```swift
/// 停止血压测量
/// - Parameter completion: 停止血压测量回调
/// - BCLStopBloodPressureResponse: 包含停止血压测量结果的响应模型
func stopBloodPressure(completion: @escaping (Result<BCLStopBloodPressureResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
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
 LmAPILite.TURN_OFF_6_AXIS_SENSORS( I6axisListener listenerLite) //关闭6轴传感器数据上报
 LmAPILite.READ_6_AXIS_SENSORS(I6axisListener listenerLite);//读6轴传感器加速度数据（单次）
 LmAPILite.READ_6_AXIS_ACCELERATION(I6axisListener listenerLite);//请求：读6轴传感器实时加速度数据（开启后一直上传直至接收到停止指令）
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

```swift
/// 设置六轴传感器工作频率 (暂不支持分开设置，需保证加速度、陀螺仪频率一致)
/// - Parameters:
///   - accelerationFrequency: 加速度频率 (25hz，50hz，100hz，150hz，200hz)
///   - gyroscopeFrequency: 陀螺仪频率 (25hz，50hz，100hz，150hz，200hz)
/// - BCLSetWorkFrequencyResponse: 包含设置结果的响应模型 status: 1成功，0失败
func setSixAxisWorkFrequency(accelerationFrequency: Int, gyroscopeFrequency: Int, completion: @escaping (Result<BCLSetWorkFrequencyResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
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

```swift
/// 获取六轴传感器工作频率
/// - BCLGetWorkFrequencyResponse: 包含获取结果的响应模型
func getSixAxisWorkFrequency(completion: @escaping (Result<BCLGetWorkFrequencyResponse, BCLError>) -> Void)
```

### 获取六轴传感器数据（单次）

**iOS:**

```swift
/// 获取六轴传感器-加速度数据(单次)
func getSixAxisAccelerationData(completion: @escaping (Result<BCLReadAccelerationResponse, BCLError>) -> Void)

/// 获取六轴传感器-陀螺仪数据(单次)
func getSixAxisGyroscopeData(completion: @escaping (Result<BCLReadGyroscopeResponse, BCLError>) -> Void)

/// 获取六轴传感器-加速度和陀螺仪数据(单次)
func getSixAxisAccelerationAndGyroscopeData(completion: @escaping (Result<BCLReadAccelerationAndGyroscopeResponse, BCLError>) -> Void)
```

### 获取六轴传感器实时数据（持续上传）

**iOS:**

```swift
/// 获取六轴传感器-加速度数据(开启后一直上传直至接收到停止指令)
func getSixAxisRealTimeAccelerationData(completion: @escaping (Result<BCLReadRealTimeAccelerationResponse, BCLError>) -> Void)

/// 获取六轴传感器-陀螺仪数据(开启后一直上传直至接收到停止指令)
func getSixAxisRealTimeGyroscopeData(completion: @escaping (Result<BCLReadRealTimeGyroscopeResponse, BCLError>) -> Void)

/// 获取六轴传感器-加速度和陀螺仪数据(开启后一直上传直至接收到停止指令)
func getSixAxisRealTimeAccelerationAndGyroscopeData(completion: @escaping (Result<BCLReadRealTimeAccelerationAndGyroscopeResponse, BCLError>) -> Void)
```

### 停止六轴传感器数据上传

**iOS:**

```swift
/// 停止六轴传感器数据上传
func stopSixAxisData(completion: @escaping (Result<BCLCloseSixAxisResponse, BCLError>) -> Void)
```

### 设置六轴传感器省电模式

**iOS:**

```swift
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

```swift
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

```swift
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

```swift
/// PPG波形测量停止
/// - Parameters:
///   - completion: 停止结果回调
func ppgWaveFormStop(completion: @escaping (Result<BCLPPGWaveFormStopResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
BCLRingManager.shared.ppgWaveFormStop { result in
    switch result {
    case .success(_):
        print("停止PPG波形测量成功")
    case .failure(let error):
        print("停止PPG波形测量失败: \(error)")
    }
}
```

实时PPG测量



```java
/**
 * 实时PPG测量（0x3C）
 */
/**
 *  实时PPG测量（0x3C）
 * @param time 采集时间，默认30
 * @param frequency 采集频率，默认25(保留)
 * @param ledGreen led_green电流，默认20
 * @param ledIr led_ir电流，默认20
 * @param ledRed led_red电流，默认20 （电流每档为392uA，档位上限50）
 * @param progress 进度响应0:不上传 1:上传
 * @param wave 波形响应0:不上传 1:上传
 * @param iRealTimePPGListener1 回调
 */
public static void START_REAL_TIME_PPG(int time, int frequency,int ledGreen, int ledIr, int ledRed, int progress, int wave, IRealTimePPGListener iRealTimePPGListener1) 
    /**
     * 停止实时PPG测量（0x3C）
     */
    public static void STOP_REAL_TIME_PPG() 
```

调用示例：

```java
postView("\n开启实时ppg");
//postView("\n开始读取未上传数据");

LmAPILite.START_REAL_TIME_PPG(30, 100, 20, 20, 20, 1, 1, new IRealTimePPGListener() {
    @Override
    public void time(long time, int zone) {
        postView("\ntime:"+time+",zone:"+zone);
    }

    @Override
    public void waveformData(int seq, int number, List<String[]> waveData) {
        postView("\nwaveformData seq:"+seq+",number:"+number);

        for (String[] array : waveData) {
            postView("\nwaveData:"+Arrays.toString(array));
        }
    }

    @Override
    public void progress(int progress) {
        postView("\nprogress :"+progress);
    }

    @Override
    public void RRIData(int number, byte[] rriData) {
        postView("\nRRIData number:"+number+",rriData length:"+rriData.length);
    }

    @Override
    public void result(int result0, int heartRate, int bloodOxygen, int temperature) {
        postView("\nresult result0:"+result0+",heartRate:"+heartRate+",bloodOxygen:"+bloodOxygen+",temperature:"+temperature);
    }
});
```

停止测量

```java
postView("\n停止实时ppg");
//postView("\n开始读取未上传数据");

LmAPILite.STOP_REAL_TIME_PPG();
```

### 设置定时启动运动采集

```java
    /**
     * 设置定时启动运动采集
     * @param model 0周期采集 1连续采集
     * @param startTime 开始时间
     * @param endTime 结束时间
     * @param listenerLite
     */
    public static void SET_SCHEDULED_STARTUP(int model,long startTime,long endTime,ICollectionListener listenerLite) {
```

<br>

### 获取定时启动运动采集

```java
public static void GET_SCHEDULED_STARTUP(ICollectionListener listenerLite)
```

回调：<br>

```java
/**
 * 设置定时启动运动采集3805
 * @param success
 */
public void setScheduledStartup(boolean success);

/**
 * 获取定时启动运动采集3806
 * @param model 模式 0周期采集 1连续采集
 * @param startTime 开始时间
 * @param endTime 结束时间
 */
public void getScheduledStartup(int model,long startTime,long endTime);
```
