---
description: 戒指会周期测量体征数据，用户也可以通过主动测量指令，进行数据测量
icon: ruler-horizontal
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

# 主动测量指令

这些指令，是用户可以通过指令，控制戒指进行体征数据的测量，包括：

* **测量心率**
* &#x20;**测量温度**
* **测量血氧**
* **测量血压（特定固件）**
* **测量血糖（特定固件）**

蓝牙信号会影响到测量的质量，在测量的时候，可以周期获取蓝牙信号，如果小于-80，可以进行闪烁图标进行提示“蓝牙信号弱，请将设备靠近手机”，防止信号太差，导致测量结果错误的情况

### **测量心率**

实时测量心率值，此测量支持输出心率，心率变异性，温度结果，同时支持输出RR间期数组。

**android:**

```java
/**
 * 测试心率和心率变异性
 *
 * @param waveForm       是否配置波形 0不上传 1上传
 * @param iHeartListener 心率监测监听
 */
public static void GET_HEART_ROTA(byte waveForm, byte acqTime, IHeartListener iHeartListener) {
    LmAPI.iHeartListener = iHeartListener;
    SEND_CMD(convertToBytes(0x31, new byte[]{0x00, acqTime, 25, waveForm, 0x01, 0x01}));
}

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

简化版本

```java
     /**
     * 测试心率和心率变异性
     * @param waveForm 是否配置波形 0不上传 1上传
     */
    public static void GET_HEART_ROTA(int waveForm,int acqTime,IHeartListenerLite listenerLite)

   public interface IHeartListenerLite {

    /**
     * 进度
     * @param progress
     */
    void progress(int progress);

    /**
     * 测量结果
     * @param heart 心率
     * @param heartRota 心率变异性
     * @param yaLi  压力
     * @param temp 温度
     */
    void resultData(int heart,int heartRota,int yaLi,int temp);

    /**
     *
     * @param serialNumber 数据序号
     * @param numberOfData 数据数量
     * @param waveData 波形图
     */
    void waveformData(int serialNumber,int numberOfData,String waveData);

    /**
     *
     * @param serialNumber 数据序号
     * @param numberOfData 数据数量
     * @param data
     */
    void rriData(int serialNumber,int numberOfData,String data);

    /**
     * 错误
     * @param code 错误码
     * @param message
     */
    void error(int code,String message);

    void success();

    void stopHeart();
}
```

**iOS:**
```Swift
/// 心率测量
/// - Parameters:
///   - collectTime: 采集时间(单位：秒) 默认为30s
///   - collectFrequency: 采集频率(单位：次/秒) 默认为25hz
///   - waveformConfig: 波形配置(0:不上传 1:上传)
///   - progressConfig: 进度配置(0:不上传 1:上传) 建议上传
///   - intervalConfig: 间期配置(0:不上传 1:上传)
///   - callbacks: 回调集合
///   - completion: 测量结果回调
func startHeartRate(collectTime: Int, collectFrequency: Int, waveformConfig: Int, progressConfig: Int, intervalConfig: Int, callbacks: BCLHeartRateCallbacks, completion: @escaping (Result<Void, BCLError>) -> Void)
```
#### 调用示例

```Swift
let callBacks = BCLHeartRateCallbacks(
    onProgress: { progress in
        BDLogger.info("测量进度: \(progress)%")
        self.showLoading("测量中... \(progress)%", userInteractionEnabled: false)
    },
    onStatusChanged: { status in
        switch status {
        case .completed:
            BDLogger.info("测量完成")
            self.hideLoading()
            self.showSuccess("心率: \(heartRateValue)次/分\n心率变异性: \(hrvValue)\n精神压力指数: \(stressValue)\n温度: \(String(format: "%.2f°C", Double(temperatureValue) * 0.01))")
            // 清理回调
        case .measuring:
            BDLogger.info("测量中...")
        case .busy:
            BDLogger.error("设备正忙，无法开始测量")
            self.hideLoading()
            self.showError("设备正忙，无法开始测量")
        case .notWearing:
            BDLogger.error("设备未佩戴，请先佩戴设备")
            self.hideLoading()
            self.showError("设备未佩戴，请先佩戴设备")
        case .dataCollectionTimeout:
            BDLogger.error("数据采集超时")
            self.hideLoading()
            self.showError("数据采集超时")
        default:
            break
        }
    },
    onMeasureValue: { heartRate, heartRateVariability, stressIndex, temperature in
        BDLogger.info("心率: \(heartRate ?? 0)次/分")
        heartRateValue = heartRate ?? 0
        BDLogger.info("心率变异性: \(heartRateVariability ?? 0)")
        hrvValue = heartRateVariability ?? 0
        BDLogger.info("精神压力指数: \(stressIndex ?? 0)")
        stressValue = stressIndex ?? 0
        if let temp = temperature {
            BDLogger.info("温度：\(String(format: "%.2f°C", Double(temp) * 0.01))")
            temperatureValue = temp
        }
    },
    onWaveform: { seq, num, datas in
        // 处理波形数据
        BDLogger.info("波形数据: 序号\(seq), 数量\(num)")
        BDLogger.info("波形数据: \(datas)")
    },
    onRRInterval: { seq, num, datas in
        // 处理间期数据
        BDLogger.info("间期数据: 序号\(seq), 数量\(num)")
        BDLogger.info("间期数据: \(datas)")
    },
    onError: { error in
        BDLogger.info("错误: \(error)")
        self.hideLoading()
        switch error {
        case .heartRate(.notWearing):
            self.showError("设备未佩戴，无法测量")
        case .heartRate(.busy):
            self.showError("设备正忙，无法测量")
        default:
            self.showError("测量过程中发生错误: \(error)")
        }
    }
)

// 开始测量
BCLRingManager.shared.startHeartRate(collectTime: 30,
                                     collectFrequency: 25,
                                     waveformConfig: 1,
                                     progressConfig: 1,
                                     intervalConfig: 1,
                                     callbacks: callBacks) { result in
    switch result {
    case .success:
        break
    case let .failure(error):
        BDLogger.error("启动心率测量失败: \(error)")
        if case .heartRate(.measurementStopped) = error { // 停止测量
            // 用户主动停止测量，无需提示错误
        }
    }
}
```

### 停止心率测量

**iOS:**
```Swift
/// 停止心率测量
/// - Parameter completion: 停止心率测量回调
/// - BCLStopHeartRateResponse: 包含停止心率测量结果的响应模型
func stopHeartRate(completion: @escaping (Result<BCLStopHeartRateResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.stopHeartRate { result in
    switch result {
    case .success:
        BDLogger.info("已停止心率测量")
    case let .failure(error):
        BDLogger.error("停止心率测量失败: \(error)")
    }
}
```

主动测量以后，如果立刻从戒指读取未上传数据，有可能读取不到最新结果，因为戒指保存会有延迟，用户可以延时几秒获取，或者把测量数据保存到本地数据库，本地数据库已做去重操作，不用担心后续未上传数据上传以后，有重复数据的情况 。以下是样例：

```java
  HistoryDataBean entity = new HistoryDataBean();
                entity.setMac(BLEUtils.mac);
                entity.setTime(System.currentTimeMillis() / 1000);
                entity.setHeartRate(heart);
                entity.setHeartRateVariability(heartRota);
                entity.setStressIndex(yaLi);
                entity.setTemperature(temp);
                entity.setSleepType(0);
                DataApi.instance.insertBatch(entity);

```

### **测量温度**

接口功能： 测量温度。\
接口声明：

<pre class="language-java"><code class="lang-java"> LmAPI.READ_TEMP (ITempListener iTempListener)

<strong>public interface ITempListener {
</strong>
    void resultData(int temp);
    void testing(int num);

    void error(int code);
}

</code></pre>

参数说明：ITempListener: 此接口是测量温度的监听\
返回值：

| 参数名称       | 类型  | 示范值     | 说明                                       |
| ---------- | --- | ------- | ---------------------------------------- |
| resultData | int | 3612    | 温度的结果，代表36.12℃                           |
| testing    | int | 100，200 | 测量中                                      |
| error      | int | 2，3，4，5 | <p>2：未佩戴<br>3：繁忙<br>4：充电中<br>5：温度值无效</p> |

简化版本

```java
   public static void READ_TEMP(ITempListenerLite listenerLite)

   public interface ITempListenerLite {

    /**
     * 测量体温完成后的结果
     * @param temp
     */
    void resultData(int temp);

    /**
     *测量中的数据
     * @param temp
     */
    void testing(int temp);

    /**
     * 错误
     * @param code
     */
    void error(int code);

}
```

简化版本

```java
   public static void READ_TEMP(ITempListenerLite listenerLite)

   public interface ITempListenerLite {

    /**
     * 测量体温完成后的结果
     * @param temp
     */
    void resultData(int temp);

    /**
     *测量中的数据
     * @param temp
     */
    void testing(int temp);

    /**
     * 错误
     * @param code
     */
    void error(int code);

}
```

**iOS:**
```Swift
/// 读取温度
/// - Parameter completion: 读取温度回调
/// - BCLTemperatureResponse: 包含温度信息的响应模型
func readTemperature(completion: @escaping (Result<BCLTemperatureResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.readTemperature { result in
    switch result {
    case let .success(response):
        if let error = response.status.error {
            switch error {
            case let .temperature(tempError):
                switch tempError {
                case .measuring:
                    BDLogger.info("测量中，请等待...")
                    BDLogger.info("温度值：\(response.temperature ?? 0)")
                    if let temperature = response.temperature, temperature <= 1000 {
                          BDLogger.info("当前测量进度：\(String(format: "%.2f", Double(temperature) * 0.1))%")
                      } else {
                          BDLogger.info("测量中，请等待...\n温度值：\(response.temperature ?? 0)")
                      }
                case .charging:
                    BDLogger.error("设备正在充电，无法测量")
                case .notWearing:
                    BDLogger.error("检测未佩戴，测量失败")
                case .invalid:
                    BDLogger.error("无效数据")
                case .busy:
                    BDLogger.error("设备繁忙")
                @unknown default:
                    BDLogger.error("未知温度错误")
                }
            default:
                BDLogger.error("读取温度失败: \(error)")
            }
        } else if let temperature = response.temperature {
            BDLogger.info("测量完成，温度：\(String(format: "%.2f", Double(temperature) * 0.01))℃")
        } else {
            self.hideLoading()
            BDLogger.error("无效的温度数据")
        }
    case let .failure(error):
        // 处理连接错误等其他错误
        BDLogger.error("读取温度失败: \(error)")
    }
}
```

**测量血氧**

接口功能：测量血氧。\
接口声明：

**android:**

<pre class="language-java"><code class="lang-java">    /**
     * 测试心率和血氧、温度
     * 默认采集30s
     * waveForm 波形配置0:不上传 1:上传
     */
<strong>    LmAPI.GET_HEART_Q2(byte waveForm,IQ2Listener iQ2Listener)
</strong>
    /**
     * 测试心率和血氧、温度
     * 可以自己配置采集时间
     * time： [0]:采集时间，默认30(0为一直采集，保留)
     * waveForm 波形配置0:不上传 1:上传
     */
     LmAPI.GET_HEART_Q2_WITH_TIME(byte waveForm,IQ2Listener iQ2Listener)

</code></pre>

```java
public interface IQ2Listener {

    void progress(int progress);


    void resultData(int heart,int q2,int temp);
    void waveformData(byte seq,byte number,String waveDate);

    void error(int code);

    void success();

}
```

简化版本

<pre class="language-java"><code class="lang-java">   /**
     * 测试心率和血氧、温度
     * 默认采集30s
     * waveForm 波形配置0:不上传 1:上传
     */
   public static void GET_HEART_Q2(byte waveForm,IBloodOxygenListenerLite listenerLite)
   
  
    /**
     * 测试心率和血氧、温度
     * 可以自己配置采集时间
     * time： [0]:采集时间，默认30(0为一直采集，保留)
     * waveForm 波形配置0:不上传 1:上传
     */
   public static void GET_HEART_Q2_WITH_TIME(byte waveForm, byte time, IBloodOxygenListenerLite listenerLite)
 
<strong>   public interface IBloodOxygenListenerLite {
</strong>   
       /**
        * 测量进度
        * @param progress
        */
       void progress(int progress);
   
       /**
        * 测量结果
        * @param heartRate 心率
        * @param bloodOxygen 血氧
        * @param temperature 温度
        */
       void resultData(int heartRate,int bloodOxygen,int temperature);
       //seq 序号，number数量，波形图
   
       /**
        *
        * @param serialNumber 数据序号
        * @param numberOfData 数据数量
        * @param waveformData 波形图
        */
       void waveformData(int serialNumber,int numberOfData,String waveformData);
   
       /**
        * 错误
        * @param code 错误码
        * @param message 错误消息
        */
       void error(int code,String message);
   
       void success();
   
       void stopQ2();
   
   }
</code></pre>

### 开始血氧测量

**iOS:**
```Swift
/// 血氧测量
/// - Parameters:
///   - collectTime: 采集时间(单位：秒) 默认30s
///   - collectFrequency: 采集频率(单位：次/秒) 默认25hz
///   - waveformConfig: 波形配置(0:不上传 1:上传)
///   - progressConfig: 进度配置(0:不上传 1:上传)
/// - BCLBloodOxygenResponse: 包含测量结果的响应模型
func startBloodOxygen(collectTime: Int, collectFrequency: Int, waveformConfig: Int, progressConfig: Int, completion: @escaping (Result<BCLBloodOxygenResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
// 设置回调
BCLBloodOxygenResponse.setCallbacks(BCLBloodOxygenCallbacks(
    onProgress: { progress in
        BDLogger.info("测量进度: \(progress)%")
    },
    onStatusChanged: { status in
        switch status {
        case .completed:
            BDLogger.info("测量完成")
            BDLogger.info("血氧: \(bloodOxygenValue)%\n心率: \(heartRateValue)次/分\n温度: \(String(format: "%.2f°C", Double(temperatureValue) * 0.01))")
            // 清理回调
            BCLBloodOxygenResponse.cleanupCurrentMeasurement()
        case .measuring:
            BDLogger.info("测量中...")
        case .busy:
            BDLogger.error("设备正忙，无法开始测量")
            // 清理回调
            BCLBloodOxygenResponse.cleanupCurrentMeasurement()
        case .chargingNotAllowed:
            BDLogger.error("设备正在充电，无法测量")
            // 清理回调
            BCLBloodOxygenResponse.cleanupCurrentMeasurement()
        case .notWearing:
            BDLogger.error("设备未佩戴，请先佩戴设备")
            // 清理回调
            BCLBloodOxygenResponse.cleanupCurrentMeasurement()
        case .dataCollectionTimeout:
            BDLogger.error("数据采集超时")
            // 清理回调
            BCLBloodOxygenResponse.cleanupCurrentMeasurement()
        default:
            break
        }
    },
    onMeasureValue: { bloodOxygen, heartRate, temperature in
        BDLogger.info("血氧: \(bloodOxygen ?? 0)%")
        bloodOxygenValue = bloodOxygen ?? 0
        BDLogger.info("心率: \(heartRate ?? 0)次/分")
        heartRateValue = heartRate ?? 0
        // 温度 (需要先解包，然后转换)
        if let temp = temperature {
            BDLogger.info("温度：\(String(format: "%.2f°C", Double(temp) * 0.01))")
            temperatureValue = temp
        }

    },
    onPerfusionRate: { rate in
        BDLogger.info("灌注率: \(rate)")
    },
    onBloodPressure: { diastolic, systolic in
        BDLogger.info("血压: \(systolic)/\(diastolic)mmHg")
    },
    onWaveform: { seq, num, datas in
        // 处理波形数据
        BDLogger.info("波形数据: 序号\(seq), 数量\(num)")
        BDLogger.info("波形数据: \(datas)")
    },
    onError: { error in
        BDLogger.error("错误: \(error)")
        switch error {
        case .bloodOxygen(.notWearing):
            BDLogger.error("设备未佩戴，无法测量")
        case .bloodOxygen(.chargingNotAllowed):
            BDLogger.error("设备正在充电，无法测量")
        case .bloodOxygen(.busy):
            BDLogger.error("设备正忙，无法测量")
        default:
            BDLogger.error("测量过程中发生错误: \(error)")
        }
    }
))

// 开始测量
// 注意：此处的参数需要根据实际需求进行调整
BCLRingManager.shared.startBloodOxygen(collectTime: 30,
                                       collectFrequency: 25,
                                       waveformConfig: 1,
                                       progressConfig: 1) { result in
    switch result {
    case .success:
        break
    case let .failure(error):
        BDLogger.error("启动血氧测量失败: \(error)")
        if case .bloodOxygen(.measurementStopped) = error { // 停止测量
            // 用户主动停止测量，无需提示错误
        }
        // 发生错误时清理回调
        BCLBloodOxygenResponse.cleanupCurrentMeasurement()
    }
}
```

### 停止血氧测量

**iOS:**
```Swift
/// 停止血氧测量
/// - Parameter completion: 停止血氧测量回调
/// - BCLStopBloodOxygenResponse: 包含停止血氧测量结果的响应模型
func stopBloodOxygen(completion: @escaping (Result<BCLStopBloodOxygenResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.stopBloodOxygen { result in
    switch result {
    case .success:
        BDLogger.info("已停止血氧测量")
    case let .failure(error):
        BDLogger.error("停止血氧测量失败: \(error)")
    }
}
```


### **测量血压和血糖（特定固件）**

基于戒指传输的波形值，经过python算法，给出具体的血压值或者血糖值，调用样例，前提已经申请了token，具体申请步骤，参照《升级服务》

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}

```java

         LmAPI.BLOOD_PRESSURE_APP((byte) 60, (byte) 1, (byte) 1, new IBloodPressureAPPListener() {
                    @Override
                    public void progress(int progress) {
                        postView("\n血压或血糖测量:"+progress );
                    }

                    @Override
                    public void resultData(int heartRate, int systolicPressure, int diastolicPressure) {
                       // postView("\nheartRate"+heartRate+",systolicPressure:" +systolicPressure+",diastolicPressure:"+diastolicPressure);
                    }

                    @Override
                    public void waveformData(int seq, int number, String waveData) {
                        postView("\nwaveData:"+waveData);
                        waveString.append(waveData);
                    }

                    @Override
                    public void error(int code) {
                        postView("\n血压或血糖测量:"+code);
                    }

                    @Override
                    public void success() {
                        postView("\n血压或血糖测量:success");
                        //根据测量要求，testType 测量类型，输入"血压"或者"血糖"
                        LogicalApi.getBloodPressureOrSugar(App.getInstance().getDeviceBean().getDevice().getAddress(), waveString.toString(),"血压", new IWebBloodPressureResult() {
                              @Override
                              public void bloodPressureResult(String diastolic, String systolic) {
                                  //血压结果
                              }
          
                              @Override
                              public void bloodSugarResult(String sugar) {
                                  //血糖结果
                              }
          
                              @Override
                              public void serviceError(String errorMsg) {
                                //接口出错
                              }
                          });
                    }

                    @Override
                    public void stop() {
                        postView("\n血压测量:stop");
                    }
                });
```

参数说明：

```java
 /**
     *血压或血糖测量
     * @param collectionTime 采集时间，默认30，血糖血压需要60s的波形
     * @param waveformConfiguration 波形配置0:不上传 1:上传
     * @param progressConfiguration 进度配置0:不上传 1:上传
     * @param iBloodPressureAPPListener
     */
BLOOD_PRESSURE_APP(byte collectionTime,byte waveformConfiguration,byte progressConfiguration,IBloodPressureAPPListener iBloodPressureAPPListener)
```

服务接口说明：

<pre class="language-java"><code class="lang-java"> /**
     * 根据血压波形值获取血压值或者血糖值
     * @param mac 连接蓝牙的mac
     * @param waveFormValue 波形值
     * @param testType 测量类型，输入"血压"或者"血糖"
     * @param webApiResult
     */
<strong>LogicalApi.getBloodPressureOrSugar(String mac, String waveFormValue, String testType,IWebBloodPressureAndSugarResult webApiResult)
</strong></code></pre>
