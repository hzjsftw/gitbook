---
description: 睡眠服务是一项很重要的服务，也是一个非常复杂的功能，睡眠是在服务器计算，根据大数据来分析，会持续更新。部分戒指支持gomore睡眠算法，这个可以在戒指端计算
icon: face-sleeping
---

# 睡眠服务

## 传统睡眠

### 计算原理

睡眠是根据戒指采集的数据来计算，根据数据中的睡眠类型，可以根据sleepType来判断是否睡眠状态数据 1：清醒 2：浅睡 3：深睡 4：快速眼动 ，然后再根据大数据分析，得到准备的睡眠数据

### 数据完整性

为了保证服务器上数据完整性，防止服务器故障，或者操作戒指数据失误的情况，比如在其他设备上同步过数据，再次同步未上传数据，其他设备上同步过的数据会丢失，提供一个获取服务器上最后一条记录的接口，将该记录的时间戳传入到上传历史数据的接口里，上传该时间戳以后的数据到服务器。

**android：**

```java
LogicalApi:
 public static void loadUserLatestHistory( IWebLastTimeResult iWebLastTimeResult)

public interface IWebLastTimeResult {

    /**
     * 当前用户服务器上最后一条数据的时间
     */
    void userLastHistoryTime(long time);
    void error(String errorMsg);//接口或者服务器错误
}

建议在createToken接口调用成功后，调用这个接口，尽量在蓝牙连接之前，获取到时间戳
```

**iOS:**

```swift
/// 获取用户最新历史数据
/// - Parameter completion: 获取用户最新历史数据回调
func loadUserLatestHistory(completion: @escaping (Result<BCLRingDBModel, BCLError>) -> Void)
```

### 上传历史数据

因为睡眠需要历史数据，所以在获取睡眠之前，需要将戒指数据调接口上传到勇芯的服务器，然后再调用接口获取睡眠信息，如果切换用户，建议先把本地数据库里的数据删除，防止把之前用户的数据传到当前用户的信息里，导致计算出错

**android：**

数据库清空数据

```java
DataApi.instance.deleteHistoryData();
```

如果使用睡眠服务，上传数据，需要使用以下接口，这个接口的作用是，从戒指获取历史数据，保存到本地数据库，同时分批次上传到服务器上，一般情况下，只获取未上传数据即可，时间戳可以使用loadUserLatestHistory得到的time字段

```java
LmAPI:

/**
     * 读取戒指本地历史数据上传到服务器
     * @param type (byte) 0x00是未上传历史，(byte) 0x01是所有历史
     * @param timeMillis 获取时间戳以后的历史记录，秒级时间戳
     * @param mMac 对应的mac地址
     * @param iHistoryListener
     * @param mWebHistoryResult
     */
    public static void READ_HISTORY_UPDATE_TO_SERVER(byte type, long timeMillis, String mMac, IHistoryListener iHistoryListener, IWebHistoryResult mWebHistoryResult) {
   
   public interface IHistoryListener {
    /**
     * 获取戒指记录出错
     * @param code
     */
    void error(int code);
    /**
     * 获取戒指记录成功
     */
    void success();
    /**
     * 获取戒指记录进度
     * @param
     */
    void progress(double progress, HistoryDataBean historyDataBean);
    /**
     * 没有更多数据了
     */
    void noNewDataAvailable();
}

public interface IWebHistoryResult {
    /**
     * 历史数据上传服务器完成
     */
    void updateHistoryFinish();

    /**
     * 接口或者服务器错误
     * @param errorMsg
     */
    void serviceError(String errorMsg);
}

```

简化版：

```java
LmAPILite:
  /**
     * 读取戒指本地历史数据
     * @param type (byte) 0x00是未上传历史，(byte) 0x01是所有历史
     * @param timeMillis 获取时间戳以后的历史记录，秒级时间戳
     * @param mMac 对应的mac地址
     * @param listenerLite
     * @param mWebHistoryResult
     */
    public static void READ_HISTORY_UPDATE_TO_SERVER(byte type, long timeMillis,  String mMac, IHistoryListenerLite listenerLite, IWebHistoryResult mWebHistoryResult) {

public interface IHistoryListenerLite {
    /**
     * 错误
     * @param code 错误码
     */
    void error(int code);

    void success();

    /**
     * 测量进度
     * @param progress 进度
     * @param historyDataBean 结果
     */
    void progress(double progress, HistoryDataBean historyDataBean);

    void clearHistory();

    void noNewDataAvailable();
}

public interface IWebHistoryResult {
    /**
     * 历史数据上传服务器完成
     */
    void updateHistoryFinish();

    /**
     * 接口或者服务器错误
     * @param errorMsg
     */
    void serviceError(String errorMsg);
}

```

如果使用的是二代协议，二代协议是会自动上传历史数据的，需要在页面上设置监听，使用下述方法上传历史数据 **特别说明：** iOS 二代协议连接指令、刷新指令可以获取到戒指的历史数据，需要自行调用上传历史数据接口上传到服务器。

```java
  /**
     * LmAPILite或者LmAPI，二代协议会自动上传历史数据到服务器，需要在页面上监听该接口
     */
    public static void READ_HISTORY_AUTO_UPDATE_TO_SERVER(  String mMac, IHistoryListenerLite listenerLite, IWebHistoryResult mWebHistoryResult) ;
    
```

二代协议的时间戳控制，是通过来控制的。具体用法参照

{% content-ref url="../zhi-neng-jie-zhi-sdk-shi-yong/er-dai-xie-yi.md" %}
[er-dai-xie-yi.md](../zhi-neng-jie-zhi-sdk-shi-yong/er-dai-xie-yi.md)
{% endcontent-ref %}

```java
public static void APP_CONNECT(long timeMillis)
public static void APP_REFRESH(long timeMillis)
```

**iOS:**

```swift
/// 读取全部历史数据
/// - Parameters:
///   - timestamp: 获取指定时间戳之后的数据（默认为0,全部）
///   - callbacks: 回调集合
///   - completion: 命令发送结果回调
func readAllHistoryData(timestamp: TimeInterval = 0, callbacks: BCLDataSyncCallbacks, completion: @escaping (Result<Void, BCLError>) -> Void)
```

#### 调用示例

```swift
// 创建回调结构体
let callbacks = BCLDataSyncCallbacks(
    onProgress: { totalNumber, currentIndex, progress, model in
        BDLogger.info("全部历史同步进度：\(currentIndex)/\(totalNumber) (\(progress)%)")
        BDLogger.info("当前数据：\(model.localizedDescription)")
    },
    onStatusChanged: { status in
        BDLogger.info("全部历史同步状态变化：\(status)")
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
        BDLogger.info("全部历史同步完成，共获取 \(models.count) 条记录")
        BDLogger.info("\(models)")
        // 注意：如果需要使用云端睡眠算法，可在此处将当前同步完成的历史数据同步上传到云端进而获取睡眠相关数据
    },
    onError: { error in
        BDLogger.error("全部历史同步出错：\(error.localizedDescription)")
    }
)

// 调用读取方法
BCLRingManager.shared.readAllHistoryData(callbacks: callbacks) { result in
    switch result {
    case .success:
        BDLogger.info("开始全部历史数据同步")
    case let .failure(error):
        BDLogger.error("启动全部历史同步失败：\(error.localizedDescription)")
    }
}
```

### 读取未上传数据

```swift
/// 读取未上传数据
/// - Parameters:
///   - timestamp: 获取指定时间戳之后的数据（默认为0,仅获取未上传的数据）
///   - callbacks: 回调集合
///   - completion: 命令发送结果回调
func readUnUploadData(timestamp: TimeInterval = 0, callbacks: BCLDataSyncCallbacks, completion: @escaping (Result<Void, BCLError>) -> Void)
```

#### 调用示例

```swift
let callbacks = BCLDataSyncCallbacks(
    onProgress: { totalNumber, currentIndex, progress, model in
        BDLogger.info("同步进度：\(currentIndex)/\(totalNumber) (\(progress)%)")
        BDLogger.info("当前数据：\(model.localizedDescription)")
    },
    onStatusChanged: { status in
        BDLogger.info("同步状态变化：\(status)")
        switch status {
        case .syncing:
            BDLogger.info("同步中...")
        case .noData:
            BDLogger.info("无数据")
        case .completed:
            BDLogger.info("同步完成")
        case .error:
            BDLogger.error("同步出错")
        }
    },
    onCompleted: { models in
        BDLogger.info("同步完成，共获取 \(models.count) 条记录")
        BDLogger.info("\(models)")
        // 注意：如果需要使用云端睡眠算法，可在此处将当前同步完成的历史数据同步上传到云端进而获取睡眠相关数据
    },
    onError: { error in
        BDLogger.error("同步出错：\(error.localizedDescription)")
    }
)

// 调用读取方法
//  - timestamp: 获取指定时间戳之后的数据（默认为0,仅获取未上传的数据）注意：部分固件可能不支持该过滤参数。
BCLRingManager.shared.readUnUploadData(timestamp: 0, callbacks: callbacks) { result in
    switch result {
    case .success:
        BDLogger.info("开始数据同步")
    case let .failure(error):
        BDLogger.error("启动同步失败：\(error.localizedDescription)")
    }
}
```

### 删除戒指中全部历史数据

**iOS:**

```swift
/// 固件支持二代协议的话，建议调用绑定指令。就不需要调用该方法了。
/// 删除全部历史数据
/// - Parameter completion: 删除全部历史数据回调
/// - BCLDeleteAllHistoryDataResponse: 包含删除结果的响应模型
func deleteRingAllHistoryData(completion: @escaping (Result<BCLDeleteAllHistoryDataResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
BCLRingManager.shared.deleteRingAllHistoryData { result in
    switch result {
    case .success(_):
        print("删除历史数据成功")
    case .failure(let error):
        print("删除历史数据失败: \(error)")
    }
}
```

**iOS:**

```swift
/// 上传历史数据
/// - Parameters:
///   - historyData: BCLRingDBModel数据数组
///   - mac: 设备MAC地址
///   - completion: 上传完成回调
func uploadHistory(historyData: [BCLRingDBModel],
                   mac: String,
                   completion: @escaping (Result<Void, BCLError>) -> Void)
```

### 历史数据字段

```java
 
  public class HistoryDataBean{

    @Id
    private Long id;

    private String mac;

    /**
     * 总数据包数 4个字节
     */
    private long totalNumber;
    /**
     * 当前第几包 4个字节
     */
    private long indexNumber;
    /**
     * 当前记录时间 4个字节
     */
    private long time;
    /**
     * 今天累计步数 2个字节
     */
    private int stepCount;
    /**
     * 心率 1个字节
     */
    private int heartRate;
    /**
     * 血氧 1个字节
     */
    private int bloodOxygen;
    /**
     * 心率变异性 1个字节
     */
    private int heartRateVariability;
    /**
     * 精神压力指数 1个字节
     */
    private int stressIndex;
    /**
     * 温度 2个字节
     */
    private int temperature;
    /**
     * 运动激烈程度 1个字节
     */
    private int exerciseIntensity;
    /**
     * 睡眠类型 1个字节
     * 0：无效
     * 1：清醒
     * 2：浅睡
     * 3：深睡
     * 4.眼动期
     */
    private int sleepType;
    /**
     * 测量标志
     */
    private int measurementMarker;
    /**
     * 保留 1个字节
     */
    private int reserve;
    /**
     * RR间期 1个字节
     */
    private int rrCount;
    /**
     * RR数组数据 1个字节
     */
    private byte[] rrBytes;

    /**
     * RR数组数据 String类型
     */
    private String rrBytesString;

    /**
     * 呼吸率
     */
    private String respiratoryRate;

    /**
     * 时区
     */
    @Transient
    private int timeZone;

    /**
     * 计步类型，0是普通，1是累积
     */
    @Transient
    private int stepCountingType;


    /**
     * 测量类型
     */
    private Integer measureType;

    /**
     * 单位时间的计步
     */
    private Integer stepsPerUnitTime;

    /**
     * 运行累计标志
     */
    private Integer accumulationFlag;
    /**
     * 电量
     */
    private Integer batteryLevel;


    /**
     * 电池电压
     */
    private Integer batteryVoltage;
    /**
     * 不佩戴消极原因：0：光学 1：算法
     */
    private Integer notWornReason;
    /**
     * 记录校准标志：0：已校准，1：未校准
     */
    private Integer recordCalibrationFlag;

    /**
     * 环境光强度系数：0-100%
     */
    private Integer ambientLightIntensityFactor ;

    /**
     * 温度个数
     */
    private String temperatureData ;

    // 步频 - 五分钟内最大值，单位步/min-gomore戒指支持
    private Integer stepRateMax;

    // 步频 - 五分钟内最小值，单位步/min-gomore戒指支持
    private Integer stepRateMin;

    //呼吸率：7-25-gomore戒指支持
    private Integer breathingRate;

    // 呼吸率置信度：0-100-gomore戒指支持
    private Integer breathingRateConfidence;

    // 运动状态：0-久坐, 1-走路, 2-跑步-gomore戒指支持
    private Integer activityStatus;

    // 总卡路里，全天累加，零点清零，单位千卡-gomore戒指支持
    private Integer totalCalories;

    // 基础代谢卡路里，全天累加，零点清零，单位千卡-gomore戒指支持
    private Integer basalMetabolicCalories;

    // 运动卡路里，全天累加，零点清零，单位千卡-gomore戒指支持
    private Integer exerciseCalories;

    // hardAdt
    private Integer hardAdt;
    }
```

### 历史数据字段-iOS

```swift

/// 历史数据结构
@objc public class BCLRingDBModel: NSObject, Codable {
    /// 总数据包数
    public var totalNumber: Int?
    /// 当前第几包
    public var indexNumber: Int?
    /// 当前记录时间
    public var time: Int64?
    /// 今天累计步数
    public var stepCount: Int?
    /// 心率
    public var heartRate: Int?
    /// 血氧
    public var bloodOxygen: Int?
    /// 心率变异性
    public var heartRateVariability: Int?
    /// 精神压力指数
    public var stressIndex: Int?
    /// 温度
    public var temperature: Int?
    /// 运动激烈程度  0：静止
    public var exerciseIntensity: Int?
    /// 睡眠类型 0：无效、1：清醒、2：浅睡、3：深睡、4.眼动期
    public var sleepType: Int?
    /// 主动测量标志位
    public var measureMarker: Int?
    /// 预留
    public var reserve: Int?
    /// RR间期
    public var rrCount: Int?
    /// RR数组数据
    public var rrBytes: Data?
    /// 测量类型
    public var measureType: Int?
    /// 单位时间计步（5分钟清除一次）
    public var stepsPerUnitTime: Int?
    /// 运行累计标志：0-255累加循环，重启归零
    public var accumulationFlag: Int?
    /// 电量：0-100%
    public var batteryLevel: Int?
    /// 电池电压：4266=4.266V，精度0.001V（毫伏）
    public var batteryVoltage: Int?
    /// 未佩戴原因：0-光学, 1-算法
    public var notWornReason: Int?
    /// 记录校准标志：0-已校准, 1-未校准
    public var recordCalibrationFlag: Int?
    /// 环境光强度系数：0-100%
    public var ambientLightIntensityFactor: Int?
    /// 温度数据（原始数据：第1字节=温度个数N，后续N*2字节=温度值数组）
    public var temperatureData: Data?

    /// 步频 - 五分钟内最大值，单位步/min
    public var stepRateMax: Int?
    /// 步频 - 五分钟内最小值，单位步/min
    public var stepRateMin: Int?
    /// 呼吸率：7-25
    public var breathingRate: Int?
    /// 呼吸率置信度：0-100
    public var breathingRateConfidence: Int?
    /// 运动状态：0-久坐, 1-走路, 2-跑步
    public var activityStatus: Int?
    /// 总卡路里，全天累加，零点清零，单位千卡
    public var totalCalories: Int?
    /// 基础代谢卡路里，全天累加，零点清零，单位千卡
    public var basalMetabolicCalories: Int?
    /// 运动卡路里，全天累加，零点清零，单位千卡
    public var exerciseCalories: Int?
    /// 硬件ADT值（Analog-to-Digital Threshold）
    public var hardAdt: Int?
}

```

### 获取睡眠

**android:**

获取睡眠数据，需要在IWebHistoryResult的updateHistoryFinish回调里获取，保证云端数据已经保存，才能计算出睡眠数据

```java
                String dateTimeString = "2025-02-12";

                LogicalApi.getSleepDataFromService( dateTimeString, new IWebSleepResult() {
                    @Override
                    public void sleepDataSuccess(Sleep2thBean sleep2thBean) {
                        // 定义日期时间格式
                        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                        // 将时间戳转换为 Date 对象
                        Date startDate = new Date(sleep2thBean.getStartTime()*1000);
                        // 将时间戳转换为 Date 对象
                        Date endDate = new Date(sleep2thBean.getEndTime()*1000);
                        postView("\n入睡时间:" + sdf.format(startDate)+"\n清醒时间:" +sdf.format(endDate)+"\n睡眠小时:" + sleep2thBean.getHours()+"\n睡眠分钟:" + sleep2thBean.getMinutes() );

                    }

                    @Override
                    public void error(String message) {

                    }

                });
```

\
返回的数据结构：

```java
public class Sleep2thBean {
    List<HistoryDataBean> sleepDataBeanList;
    long startTime;//第一次入睡时间
    long endTime;//最后一次醒来时间
    long sleepTime;//睡眠时长，包含清醒时间
    long deepTime ;//深睡眠时长
    long lowTime ;//浅睡眠时长
    long ydTime ;//眼动时间
    long qxTime;//清醒时长
    long seconds;//睡眠时长(秒)
    int hours;//睡眠时长(小时)，不包含清醒时间
    int minutes;//睡眠时长(分钟)，不包含清醒时间
    long sjTime;//实际睡眠时长
    double xiaolv;//睡眠效率
    double shenshui;//深睡比例
    int score ;//睡眠评分
    int wakeupCount;//清醒次数(传统睡眠返回)
    int waso;//入睡后的总清醒时间（单位分钟）,gomore算法使用
    String tips;//评价(保留)
    int sleepDataType;//1是1代睡眠，2是2代睡眠,3是gomore算法
    int voMax;//最大摄氧量

    double resultTemperture;//平均体温
    int xshours;//小睡使用shortSleepList，暂时用不到这个字段
    int xsminutes;//小睡使用shortSleepList，暂时用不到这个字段
    String xsAllhours;//小睡使用shortSleepList，暂时用不到这个字段
    String xsAllminutes;//小睡使用shortSleepList，暂时用不到这个字段
    String sleepLog;//计算睡眠产生的日志
    List<HistoryDataBean> historyBeanList;
    String noSleepResult;//无睡眠的原因
    List<Sleep2thBean> shortSleepList;//短睡眠列表(目前只有gomore支持，传统算法后续支持)
    }
   
```

**iOS:**

```数据结构
/// 睡眠模型
 public class BCLRingSleepModel {
    /// 睡眠时长（小时）（不包含清醒时间）
     public var hours: Int = 0
    /// 睡眠时长（分钟）（不包含清醒时间）
     public var minutes: Int = 0
    /// 睡眠总时长（秒）（包含清醒时间）
     public var sleepTime: Int64 = 0
    /// 睡眠时长（秒）
     public var seconds: Int64 = 0
    /// 深度睡眠时间
     public var deepTime: Int64 = 0
    /// 浅度睡眠时间
     public var lowTime: Int64 = 0
    /// 眼动时间
     public var ydTime: Int64 = 0
    /// 清醒时间
     public var qxTime: Int64 = 0
    /// 入睡时间戳（第一次入睡时间）
     public var startTime: Int64 = 0
    /// 清醒时间戳 （最后一次醒来时间）
     public var endTime: Int64 = 0
    /// 实际睡眠时长
     public var sjTime: Int64 = 0
    /// 睡眠效率
     public var xiaolv: Double = 0.0
    /// 深睡比例
     public var shenshui: Double = 0.0
    /// 睡眠得分
     public var score: Int = 0
    /// 清醒次数（传统睡眠返回）
     public var wakeupCount: Int = 0
    /// 入睡后的总清醒时间（单位分钟），gomore算法使用
     public var waso: Int = 0
    /// 睡眠数据类型：1是1代睡眠，2是2代睡眠，3是gomore算法
     public var sleepDataType: Int = 0
    /// 平均体温
     public var resultTemperture: Double = 0.0
    /// 小睡时长（小时）- 小睡使用shortSleepList
     public var xshours: Int = 0
    /// 小睡时长（分钟）- 小睡使用shortSleepList
     public var xsminutes: Int = 0
    /// 小睡总时长（小时）- 小睡使用shortSleepList
     public var xsAllhours: String?
    /// 小睡总时长（分钟）- 小睡使用shortSleepList
     public var xsAllminutes: String?
    /// 最大摄氧量
     public var voMax: Int = 0
    /// 计算睡眠产生的日志
     public var sleepLog: String?
    /// 历史数据列表
     public var historyBeanList: [BCLRingDBModel]?
    /// 无睡眠的原因
     public var noSleepResult: String?
    /// 短睡眠列表（目前只有gomore支持，传统算法后续支持）
     public var shortSleepList: [BCLRingSleepModel]?
    /// 睡眠数据列表
     public var sleepDataList: [BCLRingDBModel] = []
}
```

```swift
/// 获取睡眠数据
/// - Parameters:
///   - date: 查询日期
///   - timeZone: 时区（📢：睡眠数据查询时区需要固定使用东8区）
///   - completion: 获取睡眠数据回调
func getSleepData(date: Date, timeZone: BCLRingTimeZone = .East8, completion: @escaping (Result<BCLRingSleepModel, BCLError>) -> Void)

/// 获取睡眠数据（指定时间范围）
/// - Parameters:
///   - datas: 日期数据数组["2025-05-01"，"2025-05-02"]
///   - completion: 获取睡眠数据回调
func getSleepDataByTimeRange(datas: [String], completion: @escaping (Result<[BCLRingSleepDayModel], BCLError>) -> Void)
```

如果想要一个时间段内的睡眠描述，可以使用以下接口，这个接口不会返回每天的睡眠详情，否则会返回很慢

<pre class="language-java"><code class="lang-java">//根据日期组合，批量获取睡眠记录，日期格式类似于 "2025-05-11"
LogicalApi.getSleepDataBatchFromService(List&#x3C;String> dates, IWebSleepResult webApiResult)
<strong>//对应的数据类型：
</strong>public class SleepBatchBean {

    /**
     * 日期
     */
    private String dayString;
    /**
     * 深睡时长
     */
    private long highCount;

    /**
     * 浅睡时长
     */
    private long lowCount;

    /**
     * 眼动时长
     */
    private long ksCount;
    /**
     * 清醒时长
     */
    private long qxCount;
    /**
     * 睡眠时长
     */
    private long dayCount;
    /**
     * 睡眠时长
     */
    private long time;
    }
</code></pre>

对应接口的回调

```java
public interface IWebSleepResult {
    /**
     * 获取睡眠数据
     * @param sleep2thBean
     */
    void sleepDataSuccess(Sleep2thBean sleep2thBean);

    /**
     * 获取睡眠失败
     * @param message
     */
    void error(String message);

    /**
     * 批量获取睡眠总览
     * @param sleepBeanList
     */
    void sleepDataBatchSuccess( List<SleepBean> sleepBeanList);
}
```

如果需要绘图，比如清醒时间，深睡时间等，需要将原始数据进行处理一下，处理代码是：

```java
initSleepChat(thBean.getStartTime(),thBean.getSleepDataBeanList());
//thBean是接口返回的Sleep2thBean
public void initSleepChat( long showStartTime,List<HistoryDataBean> historyDataBeanList) {
        List<SleepChartBean> list = new ArrayList<>();
        int totalCount = 0;//记录条数
        int lastType = 0;
        long endTime = 0;
        boolean wakeUp = false;
        for (int i = 0; i < historyDataBeanList.size(); i++) {
            HistoryDataBean dataBean = historyDataBeanList.get(i);
            String time = DateUtils.longToString(dataBean.getTime() * 1000, "yyyy-MM-dd HH:mm");

            totalCount++;
            if (lastType == 0) {
                if(dataBean.getSleepType()>1){
                    lastType = dataBean.getSleepType();
                    list.add(new SleepChartBean(lastType, time, time));

                }

            } else if (lastType != dataBean.getSleepType()) {
                long changeTime = historyDataBeanList.get(i -1).getTime();
                time = DateUtils.longToString(changeTime * 1000, "yyyy-MM-dd HH:mm");
                list.get(list.size() - 1).setEndTime(time);
                long longtime = dataBean.getTime()-historyDataBeanList.get(i-1).getTime();
                lastType = dataBean.getSleepType();
                if(longtime < 90 * 60) {//两条数据相差不超过90分钟
                    list.add(new SleepChartBean(lastType, time, time));
                }
            }

            if(i >0 ){//识别间隔时间
                long longtime = dataBean.getTime()-historyDataBeanList.get(i-1).getTime();

                if(longtime>90*60){//超出一个小时
                  long  setEnd=DateUtils.stringToLong(list.get(list.size() - 1).getEndTime(), "yyyy-MM-dd HH:mm");
                    String endTime1 = DateUtils.longToString(setEnd, "yyyy-MM-dd HH:mm");
                    list.get(list.size() - 1).setEndTime(endTime1);
                    list.get(list.size() - 1).setSleepType(historyDataBeanList.get(i-1).getSleepType());
                    endTime = DateUtils.stringToLong(list.get(list.size() - 1).getEndTime(), "yyyy-MM-dd HH:mm");

                    break;//终止循环
                }
            }
            if (list.size() > 0 && (totalCount == historyDataBeanList.size() || wakeUp)) { //最后一条数据
                list.get(list.size() - 1).setEndTime(time);
                list.get(list.size() - 1).setSleepType(dataBean.getSleepType());
                endTime = DateUtils.stringToLong(list.get(list.size() - 1).getEndTime(), "yyyy-MM-dd HH:mm");
                break;//终止循环
            }
        }
            //showStartTime 睡眠开始时间
           //endTime睡眠结束时间
    }
    
    
    /**
     * 将字符串转换为时间戳
     */
    public static long stringToLong(String time, String format) {
        SimpleDateFormat sdf = new SimpleDateFormat(format);
        Date date = new Date();
        try {
            date = sdf.parse(time);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date.getTime();
    }

    /**
     * 将时间戳转换为字符串
     *
     * @param time   时间戳
     * @param format 字符串格式  例如:yyyy-MM-dd HH:mm:ss
     * @return
     */
    public static String longToString(long time, String format) {
        SimpleDateFormat formatter = new SimpleDateFormat(format);
        Date curDate = new Date(time);
        return formatter.format(curDate);
    }
```

处理完以后得数据，可以根据sleepType来判断是否睡眠状态数据 1：清醒 2：浅睡 3：深睡 4：快速眼动 用户可以根据这些类别，按照时间排序，计算出清醒等的开始时间和结束时间

**iOS:**

图表绘制：先将数据进行预处理，处理代码是：

```swift
func caculateSleepChartDatas(needToRenderSleepDatas: [BCLRingDBModel]) -> [[String: Int]] {
        var startTime = 0 // 入睡开始时间，用于判断
        var startDataX = 0
        var endDataX = 0
        var preSleepDataY = -1
        var preModel: BCLRingDBModel?
        let renderSleepdatas = needToRenderSleepDatas
        // 0代表曲线上的深睡  1代表曲线上的浅睡 2代表曲线上的眼动 3代表曲线上的清醒
        let sleepDataYDict: [Int: Int] = [3: 0, 2: 1, 4: 2, 1: 3]
        var sleepChartDatas: [[String: Int]] = []

        renderSleepdatas.enumerated().forEach { index, model in
            // 防止第一个睡眠数据和第二个睡眠数据不同时，会遗漏掉第一个数据
            if index == 1
                && model.sleepType != 0
                && model.sleepType != preModel!.sleepType
                && isHourInRange(timestamp: TimeInterval(startTime), hourCondition1: 18, hourCondition2: 4) {
                let currentSleepDataY = sleepDataYDict[preModel!.sleepType ?? 0]!
                endDataX = 5 * 60
                sleepChartDatas.append(["x": startDataX, "x2": endDataX, "y": currentSleepDataY])
                startDataX = endDataX
                // endDataX += 5 * 60 //用秒数做x轴
                preSleepDataY = currentSleepDataY
            }

            if index == 0 { // 第一个肯定有效
                startTime = Int(model.time ?? 0)
                startDataX = 0
                endDataX = 0
                preSleepDataY = sleepDataYDict[model.sleepType ?? 0]!
                preModel = model
            } else if model.sleepType != 0 { // 有效
                let currentSleepDataY = sleepDataYDict[model.sleepType ?? 0]!
                if preSleepDataY != currentSleepDataY, startDataX < endDataX { // 转折点
                    sleepChartDatas.append(["x": startDataX, "x2": endDataX, "y": preSleepDataY])
                    // 起点重新计算
                    startDataX = endDataX
                }
                endDataX += Int(model.time! - (preModel!.time!)) // 用秒数做x轴
                preSleepDataY = currentSleepDataY
                preModel = model
            } else { // 睡眠点无效
                if preSleepDataY != -1, startDataX < endDataX { // 前一个睡眠点不是无效点，记录下来
                    sleepChartDatas.append(["x": startDataX, "x2": endDataX, "y": preSleepDataY])
                }
                // 起点和结束点一起前进
                endDataX += Int(model.time ?? 0 - (preModel!.time ?? 0))
                startDataX = endDataX
                preSleepDataY = -1
                preModel = model
            }
        }
        if startDataX < endDataX { // 闭环
            sleepChartDatas.append(["x": startDataX, "x2": endDataX, "y": preSleepDataY])
        }
        return sleepChartDatas
    }

    func isHourInRange(timestamp: TimeInterval, hourCondition1: Int, hourCondition2: Int) -> Bool {
        // 将秒时间戳转换为日期对象
        let date = Date(timeIntervalSince1970: timestamp)
        // 创建一个日历对象
        let calendar = Calendar.current
        // 获取日期对象的小时
        let hour = calendar.component(.hour, from: date)
        // 判断小时是否超出指定范围内,入睡时间应该在18点之后或者凌晨4点之前
        // 夜间清醒2-4之间可以有120分钟
        return hour >= hourCondition1 || hour <= hourCondition2
    }
```

## Gomore睡眠

Gomore睡眠是集成在戒指里，可以直接通过指令，获取用户睡眠信息，用户可以保存睡眠信息到自己服务器，也可以调用我们的接口，保存到我们云端。如果要保证GoMore算法可以正常使用，需要进行授权。

步骤：

1.需要调用指令，获取设备的deviceID

2.调用授权接口，传入deviceID，获取key

3.然后调用指令，将key写入到戒指里

首先通过二代协议的返回值，判断戒指是否支持Gomore睡眠

```java
   private int gomoreSleep;//固件是否支持gomore睡眠算法,1支持0不支持
```

### 授权

一般量产的时候，就已经授权过了，不需要用户自己授权，这个授权，只是作为量产授权出错，或者有遗漏的情况下，补充授权

```java
/**
     * 对gomore戒指进行key授权
     * @param mac 戒指mac
     * @param companyApiKey 分配的公司key
     * @param miGomoreListener
     */
LogicalApi.goMoreAuthorizationKey(String mac, String companyApiKey , GoMoreUtils.IGomoreListener miGomoreListener)
```

### 指令

如果要保证睡眠准确性，需要在授权以后，设置一下用户信息。

```java
LmAPI 或者 LmAPILite 调用SET_GOMORE_USER();

    /**
     *  gomore设置个人信息
     * @param age 年龄（0-99）
     * @param sex 性别（0女性，1男性）
     * @param height 身高（100-220 cm） 小于100设置为100
     * @param weight 体重（10-150 kg）
     * @param maximumHeartRate 最大心率值（138-220） 不设置就设为-1
     * @param normalHeartRate 常态心率值（40-100） 不设置就设为-1
     * @param maximalOxygenUptake 最大摄氧量（ml/kg/min） 不设置就设为-1
     * @param listenerLite
     */
    public static void SET_GOMORE_USER(int age,int sex,int height,int weight,int maximumHeartRate,int normalHeartRate,int maximalOxygenUptake,IGoMoreUserListener listenerLite) 
```

复合指令在Gomore固件里，返回戒指里保存的用户信息，可以与自己的用户信息对比，如果不一致，调用指令，进行设置

{% content-ref url="../zhi-neng-jie-zhi-sdk-shi-yong/er-dai-xie-yi.md" %}
[er-dai-xie-yi.md](../zhi-neng-jie-zhi-sdk-shi-yong/er-dai-xie-yi.md)
{% endcontent-ref %}

通过蓝牙指令获取gomore睡眠结果

```java
 List<GoMoreSleepBean> gomoreSleepList=new ArrayList<>();
 LmAPILite.GET_GOMORE_SLEEP(new IGoMoreListener() {
                @Override
                public void overviewOfSleep(GoMoreSleep goMoreSleep) {
                    //睡眠总览
                    Gson gson = new Gson();
                    String json = gson.toJson(goMoreSleep);
                    gomoreSleepList.add(gson.fromJson(json, GoMoreSleepBean.class));
                }

                @Override
                public void sleepStaging(GoMoreSleep goMoreSleep) {
                    //睡眠分期，是每一分钟的睡眠状态
                    Gson gson = new Gson();
                    String json = gson.toJson(goMoreSleep);
                    gomoreSleepList.add(gson.fromJson(json, GoMoreSleepBean.class));
                }

                @Override
                public void dataUploadFinish() {
                    //数据获取完成，用户可以提交到自己服务器，也可以上传到我们服务器
                    
                }

                @Override
                public void noSleepData() {
                    //没有数据
                   
                }
            });
```

睡眠实体类

```java
/**
 * gomore算法睡眠数据
 */
public class GoMoreSleep {
    /**
     * 0为睡眠总览响应
     * 1为睡眠分期响应
     * 2为无睡眠数据
     */
    private int resType;
    private long startTs;//开始时间戳（单位秒）
    private long endTs;//结束时间戳（单位秒）
    private int numEpochs;      // Number of Stages 有效数据长度（最大为2880）
    private int latency;            //睡眠潜伏期（单位分钟）
    private int wakeTimes;        //清醒时间（单位分钟）
    private int totalSleepTime;     //不包含清醒时间的总睡眠时间（单位分钟）
    private int waso;               //入睡后的总清醒时间（单位分钟）
    private int sleepPeriod;        //睡眠时间（单位分钟）
    private int efficiency;         //睡眠效率（除以100为百分比）
    private int wakeRatio;          //清醒与睡眠比例（除以100为百分比）
    private int remRatio;           //眼动与睡眠比例（除以100为百分比）
    private int lightRatio;         //浅睡与睡眠比例（除以100为百分比）
    private int deepRatio;          //深睡与睡眠比例（除以100为百分比）
    private int wakeNumMinutes;     //清醒时间（单位分钟）
    private int remNumMinutes;      //眼动时间（单位分钟）
    private int lightNumMinutes;    //浅睡时间（单位分钟）
    private int deepNumMinutes;     //深睡时间（单位分钟）
    private int score;              //睡眠评分，100满
    private int type;             //类型：长睡/短睡（1长睡2短睡，短睡时只有开始结束时间戳）
    private int packNum;//（总包数）
    private int packNo;//（包序号
    private int stageNum;//（当前包stages数组长度）
    private short[] stages;//睡眠分期,0：唤醒，1：眼动，2：浅睡，3：深睡
}
```

### 服务器相关接口

看公司需求，如果需要保存到自己数据库，这一步可以省略

```java
//上传Gomore睡眠记录
LogicalApi.insertGomoreSleepList(List<GoMoreSleep> gomoreSleepList, IWebCommonListener iWebCommonListener)
//获取睡眠，兼容gomore和传统睡眠，替换掉LogicalApi.getSleepDataFromService
LogicalApi.getSleepFromServiceWithGomore(String date, IWebSleepResult webApiResult)
//批量获取睡眠总览，兼容Gomore睡眠和旧版本睡眠，替换掉LogicalApi.getSleepDataBatchFromService
LogicalApi.getSleepDataWithGoMoreBatch(List<String> dates, IWebSleepResult webApiResult)
```

如果是Gomore睡眠，绘图需要改变一下，每个分期是60秒记录一次睡眠状态，需要特殊处理 根据sleepDataType进行判断，如果是3，是Gomore睡眠

```java
 /**
     * 适配gomore算法的睡眠绘图
     * @param showStartTime
     * @param endTimeData
     * @param historyDataBeanList
     */
    public void initSleepChatGomore(long showStartTime, long endTimeData, List<HistoryDataBean> historyDataBeanList) {
        List<SleepChartBean> list = new ArrayList<>();
        if (historyDataBeanList != null && !historyDataBeanList.isEmpty()) {
            int currentSleepType = historyDataBeanList.get(0).getSleepType();
            long startTime = historyDataBeanList.get(0).getTime();
            long currentEndTime = historyDataBeanList.get(0).getTime();

            for (int i = 1; i < historyDataBeanList.size(); i++) {
                HistoryDataBean currentBean = historyDataBeanList.get(i);
                HistoryDataBean previousBean = historyDataBeanList.get(i - 1);

                // 检查是否连续（时间相差60秒）且睡眠类型相同
                boolean isContinuous = (currentBean.getTime() == previousBean.getTime() + 60);
                boolean sameSleepType = (currentBean.getSleepType() == currentSleepType);

                if (isContinuous && sameSleepType) {
                    // 连续且类型相同，更新结束时间
                    currentEndTime = currentBean.getTime();
                } else {
                    // 不连续或类型不同，保存当前时间段
                    String startTimeStr = com.lm.sdk.library.utils.DateUtils.longToString(startTime * 1000, "yyyy-MM-dd HH:mm:ss");
                    String endTimeStr = com.lm.sdk.library.utils.DateUtils.longToString(currentEndTime * 1000, "yyyy-MM-dd HH:mm:ss");
                    list.add(new SleepChartBean(currentSleepType, startTimeStr, endTimeStr));

                    // 开始新的时间段
                    currentSleepType = currentBean.getSleepType();
                    startTime = currentBean.getTime();
                    currentEndTime = currentBean.getTime();
                }
            }

            // 添加最后一个时间段
            String startTimeStr = com.lm.sdk.library.utils.DateUtils.longToString(startTime * 1000, "yyyy-MM-dd HH:mm:ss");
            String endTimeStr = com.lm.sdk.library.utils.DateUtils.longToString(currentEndTime * 1000, "yyyy-MM-dd HH:mm:ss");
            list.add(new SleepChartBean(currentSleepType, startTimeStr, endTimeStr));
        }

    }

```

## Gomore-iOS相关接口

### 授权相关接口

```swift
    /// GoMore一键授权
    /// 整合授权状态查询、pKey申请、授权、保存等操作的一键式接口
    /// - Parameters:
    ///   - companyKey: 公司API密钥
    ///   - progress: 进度回调，返回当前执行步骤
    ///   - completion: 完成回调，返回授权结果或错误
    /// - Note: 该接口会自动完成以下流程：
    ///   1. 查询设备授权状态获取mcuId
    ///   2. 查询服务端pKey状态
    ///   3. 如果服务端有pKey则直接返回已授权
    ///   4. 如果服务端没有pKey则申请新pKey
    ///   5. 发送pKey到设备完成授权
    ///   6. 保存授权信息到服务端
    func checkAndAuthorizeGoMore(companyKey: String,
                                  progress: @escaping (BCLGoMoreAuthStep) -> Void,
                                  completion: @escaping (Result<BCLGoMoreAuthResult, BCLError>) -> Void)

    /// 查询GoMore授权状态
    /// - Parameter completion: 查询结果回调
    /// - Result: 查询结果
    /// - BCLGoMoreAuthStatusResponse: 包含授权状态和MCU ID的响应模型
    /// - BCLError: 错误信息
    func queryGoMoreAuthStatus(completion: @escaping (Result<BCLGoMoreAuthStatusResponse, BCLError>) -> Void)

    /// 下发GoMore授权PKey
    /// - Parameters:
    ///   - pKey: 64字节的授权密钥字符串
    ///   - completion: 下发结果回调
    /// - Result: 下发结果
    /// - BCLGoMorePKeyResponse: 包含授权状态和MCU ID的响应模型
    /// - BCLError: 错误信息
    func sendGoMorePKey(pKey: String, completion: @escaping (Result<BCLGoMorePKeyResponse, BCLError>) -> Void)

    /// 查询Gomore PKey授权状态
    /// - Parameters:
    ///   - deviceId: 设备标识符（如 "0102030405060708"）
    ///   - completion: 完成回调
    /// - Result: 查询结果
    ///   - String?: 授权密钥（pKey）表示已授权，nil表示未授权
    ///   - BCLError: 错误信息
    /// - Note: 用于查询Gomore戒指在服务端的授权信息状态
    func getGomorePKeyStatus(deviceId: String, completion: @escaping (Result<String?, BCLError>) -> Void)

    /// 请求Gomore PKey授权信息
    /// - Parameters:
    ///   - deviceId: 设备标识符
    ///   - apiKey: API密钥
    ///   - completion: 完成回调，如果有pKey则返回pKey，否则返回nil授权信息获取失败
    /// - Note: 获取Gomore认证密钥，用于后续Gomore相关操作
    func requestGomorePKey(deviceId: String, apiKey: String, completion: @escaping (Result<String?, BCLError>) -> Void)

    /// 保存Gomore PKey
    /// - Parameters:
    ///   - companyApiKey: 公司API密钥
    ///   - deviceId: 设备标识符
    ///   - mac: 设备MAC地址
    ///   - pkey: Gomore PKey
    ///   - completion: 完成回调
    /// - Note: 将Gomore设备信息保存到服务器
    func saveGomorePKey(companyApiKey: String, deviceId: String, mac: String, pkey: String, completion: @escaping (Result<Void, BCLError>) -> Void)

```

### Gomore个人信息相关接口

```swift
    /// GoMore设置个人信息
    /// - Parameters:
    ///   - age: 年龄（10-99）
    ///   - gender: 性别（0女性，1男性）
    ///   - height: 身高（100-220 cm）
    ///   - weight: 体重（10-150 kg）
    ///   - maxHeartRate: 最大心率值（nil或<0表示无效）
    ///   - normalHeartRate: 常态心率值（nil或<0表示无效）
    ///   - maxOxygenUptake: 最大摄氧量（nil或<0表示无效）
    ///   - completion: 设置个人信息回调
    /// - Result: 设置结果
    /// - BCLGoMoreSetPersonalInformationResponse: 包含设置结果的响应模型
    /// - BCLError: 错误信息
    func goMoreSetPersonalInformation(age: UInt8, gender: UInt8, height: UInt8, weight: UInt8, maxHeartRate: Int16?, normalHeartRate: Int8?, maxOxygenUptake: Int8?, completion: @escaping (Result<BCLGoMoreSetPersonalInformationResponse, BCLError>) -> Void)

    /// GoMore获取个人信息
    /// - Parameter completion: 获取个人信息回调
    /// - Result: 获取结果
    /// - BCLGoMoreReadPersonalInformationResponse: 包含个人信息的响应模型
    /// - BCLError: 错误信息
    func goMoreGetPersonalInformation(completion: @escaping (Result<BCLGoMoreReadPersonalInformationResponse, BCLError>) -> Void)

```

### 睡眠数据相关接口

```swift
    /// 读取GoMore睡眠数据（读取戒指中的数据集）
    /// - Parameter completion: 读取GoMore睡眠数据回调
    /// - Result: 读取结果
    /// - BCLGoMoreSleepDataResponse: GoMore睡眠数据响应
    ///   - sleepModels: [BCLGoMoreSleepModel] - 睡眠数据模型列表
    /// - BCLError: 错误信息
    func readGoMoreSleepData(completion: @escaping (Result<BCLGoMoreSleepDataResponse, BCLError>) -> Void)

    /// 上传GoMore睡眠数据到服务器
    /// - Parameters:
    ///  - sleepModels: 睡眠数据模型列表
    ///  - completion: 上传结果回调
    ///  - Result: 上传结果
    ///  - Void: 无返回值
    ///  - BCLError: 错误信息
    func uploadGoMoreSleepData(sleepModels: [BCLGoMoreSleepModel], completion: @escaping (Result<Void, BCLError>) -> Void)

    /// 获取Gomore睡眠详情数据
    /// - Parameters:
    ///   - date: 查询日期（本地自然日时间，按调用端本机时区理解）
    ///   - userId: 用户ID（可选，默认为nil）
    ///   - completion: 完成回调，返回睡眠详情数据或错误
    /// - Note: 返回的 BCLRingSleepModel 包含睡眠统计信息和睡眠数据列表
    func getGoMoreSleepData(date: Date, userId: String? = nil, completion: @escaping (Result<BCLRingSleepModel, BCLError>) -> Void)

    /// 批量获取Gomore睡眠数据
    /// - Parameters:
    ///   - dates: 日期数组，格式为 "yyyy-MM-dd"，如 ["2025-12-28", "2025-12-29"]
    ///   - userId: 用户ID（可选，默认为nil）
    ///   - completion: 完成回调，返回睡眠数据摘要数组或错误
    /// - Note: 返回的 BCLRingSleepDayModel 包含每日睡眠统计摘要
    func getGoMoreSleepDataBatch(dates: [String], userId: String? = nil, completion: @escaping (Result<[BCLRingSleepDayModel], BCLError>) -> Void)

```

### 注意事项:iOS端

1、Gomore戒指绑定成功之后需要检查授权状态，可以调用checkAndAuthorizeGoMore接口进行处理。

2、Gomore戒指连接成功之后需要核对用户个人信息，age、gender、height、weight、maxHeartRate、normalHeartRate、maxOxygenUptake，如果不一致，需要调用goMoreSetPersonalInformation接口进行设置。确保在合理范围内，不然会影响睡眠计算等。（复合指令中绑定、连接、刷新接口会返回戒指中的用户信息，可以自行进行对比）

3、Gomore戒指睡眠数据读取，需要先调用readGoMoreSleepData接口，读取戒指中的数据集，然后调用uploadGoMoreSleepData接口，将数据上传到服务器。

4、查询用户睡眠数据可直接调用getGoMoreSleepData接口，获取睡眠详情数据。

5、查询用户睡眠数据批量可直接调用getGoMoreSleepDataBatch接口，获取睡眠数据摘要数组。

6、接口使用示例可以参考Demo工程
