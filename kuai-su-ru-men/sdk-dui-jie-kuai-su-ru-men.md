---
icon: meteor
---

# sdk对接快速入门

## 前言

### 对接前准备

1. 安装上位机，上位机可以连接戒指，获取戒指中的历史数据（如果对接睡眠服务，历史数据可以分析睡眠问题），可以主动测量，方便排查戒指是否有问题。对于自己按照协议进行开发的客户，可以对比上位机返回的指令，进行调试
2. 去应用市场或者 AppStore 下载 ChipletRing，这是我们公版的 app，客户可以注册账户，绑定戒指，体验戒指的功能，并且可以和自己的 app 进行对比测试。
3. 去应用市场下载 nRF Connect，这个 app 是很好的蓝牙连接工具，可以发送指令、查看广播，固件有问题的时候，可以辅助排查。
4. 对于开发中出现的问题，建议使用上位机先测试一下，如果上位机没问题，大概率是 sdk 的问题；如果有问题，大概率是固件问题

### sdk 常用对接流程

{% stepper %}
{% step %}
### 获取 token 和最后一条历史数据时间戳

首先，需要在蓝牙连接之前创建 Token，然后再去获取服务器上的最后一条记录的时间戳。这个的作用是，可以拉取戒指里该时间戳以后的历史数据，防止出现在其他地方同步了未上传数据，戒指默认这部分数据已经上传，在 app 上再次同步的时候，不会再上传这部分数据的情况。有时间戳后，会同步该时间戳以后的数据，这样服务器上的数据就可以保证完整性。
{% endstep %}

{% step %}
### 控制指令发送间隔

戒指是单线程的，所以发送指令的时候，需要间隔开，一般间隔 200ms。需要注意的是，上传历史记录时比较耗时，如果这个时候调用主动测量的指令，会指令冲突，建议在上传完历史数据以后，才能开始主动测量。
{% endstep %}

{% step %}
### 连接设备并同步数据

蓝牙连接成功以后，首先发送复合指令的连接操作（绑定操作会清空历史数据，建议只在用户首次绑定戒指，或者换绑戒指时使用），会同步时间、会自动获取历史记录，建议传入第 1 步中获取的时间戳。
{% endstep %}

{% step %}
### 处理睡眠数据

如果非 gomore 戒指，可以在历史数据上传完成以后，获取睡眠数据。

如果是 gomore 戒指，在历史数据上传完成以后，再进行设置 gomore 用户信息、获取 gomore 睡眠；如果传入我们服务器，需要调用上传 gomore 睡眠的接口，然后可以获取 gomore 睡眠。
{% endstep %}

{% step %}
### 调用睡眠接口

可以在首页或需要的地方，调用睡眠接口，只要数据正确上传到服务器，就获取任意天的数据。
{% endstep %}
{% endstepper %}

### 快速上架的建议

1. 因为 sdk 文档内容太多，难免有产生误解的地方，研发完成 app 以后，建议将 sdk 对接流程以及调用的方法整理成一个文档，我们这边评审一下，防止出现不必要的 bug
2. 开发完 app 或者小程序以后，请同步给我们，我们这边会进行同步测试，及时发现 sdk 和固件的问题，减轻客户的压力

## android 常用对接流程

### 上传数据到服务器

{% stepper %}
{% step %}
### 获取 token

方法：createToken\
参数：key、id、回调

```
LogicalApi:
  public static void createToken(String key,String userName, ICreateToken iCreateToken)

LogicalApi.createToken("","", new ICreateToken() {
    @Override
    public void getTokenSuccess() {

    }

    @Override
    public void error(String msg) {

    }
});
```
{% endstep %}

{% step %}
### 获取最后一条历史数据的时间戳

方法：loadUserLatestHistory\
参数：结果回调\
结果：时间戳

```
LogicalApi:
 public static void loadUserLatestHistory( IWebLastTimeResult iWebLastTimeResult)
```
{% endstep %}

{% step %}
### 连接设备

调用蓝牙连接方法连接设备
{% endstep %}

{% step %}
### 设置回调

```
/**
 * LmAPI，二代协议会自动上传历史数据到服务器，需要在页面上监听该接口，一般在onCreate()回调里
 */
public static void READ_HISTORY_AUTO_UPDATE_TO_SERVER(  String mMac, IHistoryListenerLite listenerLite, IWebHistoryResult mWebHistoryResult) ;
   
IHistoryListenerLite() {
  @Override
  public void error(int code) {
      if (code == 3) {
          postView("\n出现了BIX的问题");
      }
      setMessage(TestActivity.this, "\n出现了BIX的问题");
  }

  @Override
  public void success() {
      postView("\n读取记录完成");

  }

  @Override
  public void progress(double progress, HistoryDataBean historyDataBean) {
      if (historyDataBean != null) {
          postView("\n读取记录进度:" + progress + "%");
          postView("\n记录内容:" + historyDataBean.toString());
      }

  }
}

new IWebHistoryResult() {
  @Override
  public void updateHistoryFinish() {
      postView("\n历史数据上传服务器完成");
  }
}
```
{% endstep %}

{% step %}
### 使用复合指令连接配置设备

```
/**
 * 连接
 * 秒级时间戳
 */
LmAPI.APP_CONNECT(long timeMillis);
IBindConnectRefreshListenerLite {
    @Override
    public void appConnect(SystemControlBean systemControlBean) {
        postView("\nappConnect："+systemControlBean.toString());
    }
}

/**
 * 刷新
 *
 */
LmAPI.APP_REFRESH(long timeMillis);//秒级时间戳
IBindConnectRefreshListenerLite {
    @Override
    public void appRefresh(SystemControlBean systemControlBean) {
        postView("\nappRefresh："+systemControlBean.toString());
    }
}
```

{% hint style="info" %}
说明：IHistoryListener#progress 回调方法返回的每一个 HistoryDataBean 对象就是一个记录点，数据结构是一致的，但只有获取到数据的字段有值，没有数据的字段为 0；可以在这里保存数据并上传到自己的服务器。
{% endhint %}
{% endstep %}

{% step %}
### 上传历史数据完成后处理睡眠数据

6. 上述历史数据传完后，再上传 gomore 算法的睡眠数据
{% endstep %}

{% step %}
### 设置用户信息，为 gomore 算法做准备

```
LmAPILite 调用SET_GOMORE_USER();

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
{% endstep %}

{% step %}
### 获取 gomore 睡眠数据

```
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

{% hint style="info" %}
说明：此处的数据即原始 gomore 算法的睡眠数据，可以保存到自己的服务器
{% endhint %}
{% endstep %}

{% step %}
### 上传 gomore 睡眠记录

```
//上传Gomore睡眠记录
LogicalApi.insertGomoreSleepList(List<GoMoreSleep> gomoreSleepList, IWebCommonListener iWebCommonListener)
```
{% endstep %}
{% endstepper %}

### 运动数据

#### 获取方法

1. timeline 是每天某一段时间内的运动情况

```
/**
 * startTime和endTime是秒级时间戳
 */
LogicalApi.getTimeLineWithHistory(startTime, endTime, new IWebTimeLineResult() {
    @Override
    public void timelineResult(List<MovementSegment> movementSegments) {
        
    }

    @Override
    public void serviceError(String errorMsg) {

    }
});

//startTime和endTime是秒级时间戳，比如传递当天的0点到24点
// 获取当前日期(不含时间)
LocalDate today = LocalDate.now();
// 获取系统默认时区
ZoneId zoneId = ZoneId.systemDefault();
// 当天0点(00:00:00)
LocalDateTime startOfDay = today.atStartOfDay();
// 当天24点(实际上是次日的00:00:00)
LocalDateTime endOfDay = today.plusDays(1).atStartOfDay();
// 转换为秒级时间戳
long startTimestamp = startOfDay.atZone(zoneId).toEpochSecond();
long endTimestamp = endOfDay.atZone(zoneId).toEpochSecond();
```

### 睡眠数据

#### 获取方法

1. 睡眠数据

```
/**
 * 获取睡眠，兼容gomore和传统睡眠
 * date：格式为 "yyyy-MM-dd"的时间字符串
 * 结果：
 */
LogicalApi.getSleepFromServiceWithGomore(String date, IWebSleepResult webApiResult)

/**
 * 批量获取睡眠总览，兼容Gomore睡眠和旧版本睡眠
 * 如果想要一个时间段内的睡眠描述，可以使用以下接口，这个接口不会返回每天的睡眠详情，否则会返回很慢
 * date：格式为 "yyyy-MM-dd"的时间字符串
 */
LogicalApi.getSleepDataWithGoMoreBatch(List<String> dates, IWebSleepResult webApiResult)

public interface IWebSleepResult {
    /**
     * getSleepFromServiceWithGomore 请求对应的回调
     */
    void sleepDataSuccess(Sleep2thBean sleep2thBean);
    /**
     * 错误回调
     */
    void error(String message);

    /**
     * getSleepDataWithGoMoreBatch 请求回调
     */
    void sleepDataBatchSuccess(List<SleepBatchBean> sleepBeanList);
}
```

### 情绪压力

#### 获取方式

1. 连续压力测量值\
   通过 READ\_HISTORY\_AUTO\_UPDATE\_TO\_SERVER 方法获取历史数据后，自行统计。
2. 单次压力测量值

```
  /**
    * 测试心率和心率变异性
    * @param waveForm 是否配置波形 0不上传 1上传
    * @param acqTime 采集时间，单位 秒，一般为 30 秒
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

### 心率数据

#### 获取方式

1. 连续心率测量值、连续心率变异性（HRV）\
   通过 READ\_HISTORY\_AUTO\_UPDATE\_TO\_SERVER 方法获取历史数据后，自行统计。
2. 单次测心率

```
  /**
    * 测试心率和心率变异性
    * @param waveForm 是否配置波形 0不上传 1上传
    * @param acqTime 采集时间，单位 秒，一般为 30 秒
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

### 血氧数据

#### 获取方式

1. 连续血氧测量值\
   通过 READ\_HISTORY\_AUTO\_UPDATE\_TO\_SERVER 方法获取历史数据后，自行统计。
2. 单次测血氧

```
 /**
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

 public interface IBloodOxygenListenerLite {
 
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
```

### 体温数据

#### 获取方式

1. 连续体温测量值\
   通过 READ\_HISTORY\_AUTO\_UPDATE\_TO\_SERVER 方法获取历史数据后，自行统计。
2. 单次体温测量值

```
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

### 固件升级

1. 连接成功后调用 createToken 方法创建 token，连接服务器的操作都需要先获取 token\
   测试 key：76d07e37bfe341b1a25c76c0e25f457a\
   测试 userName：1204491582@qq.com
2. 调用 otaUpdateWithCheckVersion 方法执行 ota 升级，参数是当前 ota 版本等，升级完成需要重新连接。\
   ps：另外 checkCurrentVersionNeedUpdate 方法可用，查询当前版本是否需要升级；\
   isOtaUpgrade 方法可用，检查是否正在 ota 升级

### 数据结构

```
/**
 * 历史数据
 */
public class HistoryDataBean{

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

    // 步频 - 五分钟内最大值，单位步/min
    private Integer stepRateMax;

    // 步频 - 五分钟内最小值，单位步/min
    private Integer stepRateMin;

    //呼吸率：7-25
    private Integer breathingRate;

    // 呼吸率置信度：0-100
    private Integer breathingRateConfidence;

    // 运动状态：0-久坐, 1-走路, 2-跑步
    private Integer activityStatus;

    // 总卡路里，全天累加，零点清零，单位千卡
    private Integer totalCalories;

    // 基础代谢卡路里，全天累加，零点清零，单位千卡
    private Integer basalMetabolicCalories;

    // 运动卡路里，全天累加，零点清零，单位千卡
    private Integer exerciseCalories;
}

/**
 * 睡眠数据
 */
public class Sleep2thBean {
    List<HistoryDataBean> sleepDataBeanList;//绘图用
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

/**
 * 睡眠总览
 */
public class SleepBatchBean {
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
    private int wakeTimes;        //清醒次数
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

## iOS 常用对接流程

### 上传数据到服务器

{% stepper %}
{% step %}
### 获取 token

```
/// 创建Token
/// - Parameters:
///   - apiKey: API密钥
///   - userIdentifier: 用户标识
/// - Parameter completion: 创建Token回调
func createToken(apiKey: String, userIdentifier: String, completion: @escaping (Result<String, BCLError>) -> Void)

```
{% endstep %}

{% step %}
### 获取最后一条历史数据的时间戳

```
/// 获取用户最新历史数据
/// - Parameter completion: 获取用户最新历史数据回调，数据为空时返回nil
public func loadUserLatestHistory(completion: @escaping (Result<BCLRingSDK.BCLRingDBModel?, BCLRingSDK.BCLError>) -> Void)
```
{% endstep %}

{% step %}
### 连接设备

```

// 调用蓝牙扫描后，获取到了蓝牙设备列表之后，可以通过下面该方法，传入通过扫描接口回调的 BCLDeviceInfoModel 对象，来连接设备。

 /// 连接设备
 /// - Parameters:
 ///   - device: 蓝牙设备
 ///   - isAutoReconnect: 是否自动重连(默认：false)
 ///   - autoReconnectTimeLimit: 自动重连时间限制(单位：秒,默认：300)
 ///   - autoReconnectMaxAttempts: 自动重连最大次数(默认：3)
 ///   - connectResultBlock: 连接结果回调
 ///   - Result: 连接结果
 ///   - BCLDeviceInfoModel : 蓝牙设备
 ///   - BCLError : 错误信息
 public func startConnect(device: BCLRingSDK.BCLDeviceInfoModel, isAutoReconnect: Bool = false, autoReconnectTimeLimit: TimeInterval = 300, autoReconnectMaxAttempts: Int = 3, connectResultBlock: @escaping (Result<BCLRingSDK.BCLDeviceInfoModel, BCLRingSDK.BCLError>) -> Void)


 // 如果说已经绑定过的戒指，已经知晓设备的 mac 地址了，也可以直接通过 mac 地址来连接设备，其他参数和上面的方法一样。
 // mac 连接方式，可以保证跨设备连接（比如说用户换了手机，或者说用户有多台设备），只要知道 mac 地址了，就可以连接上设备了。

 /// 连接设备
 /// - Parameters:
 ///   - macAddress: 设备的mac地址
 ///   - isAutoReconnect: 是否自动重连(默认：false)
 ///   - autoReconnectTimeLimit: 自动重连时间限制(单位：秒,默认：300)
 ///   - autoReconnectMaxAttempts: 自动重连最大次数(默认：3)
 ///   - connectResultBlock: 连接结果回调
 ///   - Result: 连接结果
 ///   - BCLDeviceInfoModel : 蓝牙设备
 ///   - BCLError : 错误信息
 public func startConnect(macAddress: String, isAutoReconnect: Bool = false, autoReconnectTimeLimit: TimeInterval = 300, autoReconnectMaxAttempts: Int = 3, connectResultBlock: @escaping (Result<BCLRingSDK.BCLDeviceInfoModel, BCLRingSDK.BCLError>) -> Void)
```
{% endstep %}

{% step %}
### 使用复合指令连接配置设备

```

// 当用户首次与该设备连接时，调用绑定接口。（如果解绑之后，再次绑定的话，调用该接口即可）该接口戒指内部会清空历史数据，保证戒指的数据不会与当前绑定的人的数据混在一起。

/// APP事件-绑定戒指
/// - Parameters:
///  - date: 时间
///   - timeZone: 时区
/// - Parameter completion: 绑定戒指回调
public func appEventBindRing(date: Date, timeZone: BCLRingSDK.BCLRingTimeZone, completion: @escaping (Result<BCLRingSDK.BCLBindRingResponse, BCLRingSDK.BCLError>) -> Void)

/// APP事件-连接戒指
/// - Parameters:
///   - date: 时间戳
///   - timeZone: 时区
///   - callbacks: 回调集合
/// - Parameter completion: 连接戒指回调
public func appEventConnectRing(date: Date, timeZone: BCLRingSDK.BCLRingTimeZone, filterTime: Date? = nil, callbacks: BCLRingSDK.BCLDataSyncCallbacks, completion: @escaping (Result<BCLRingSDK.BCLConnectRingResponse, BCLRingSDK.BCLError>) -> Void)

/// APP事件-刷新戒指
/// - Parameters:
///   - date: 时间戳
///   - timeZone: 时区
///   - callbacks: 回调集合
/// - Parameter completion: 刷新戒指回调
public func appEventRefreshRing(date: Date, timeZone: BCLRingSDK.BCLRingTimeZone, filterTime: Date? = nil, callbacks: BCLRingSDK.BCLDataSyncCallbacks, completion: @escaping (Result<BCLRingSDK.BCLRefreshRingResponse, BCLRingSDK.BCLError>) -> Void)
```
{% endstep %}

{% step %}
### 上传历史数据

从戒指中读取数据到手机中；如果已经使用了上面的复合指令的话，就不需要再调用下面两个方法了。

```
/// 读取全部历史数据
/// - Parameters:
///   - timestamp: 获取指定时间戳之后的数据（默认为0,全部）
///   - callbacks: 回调集合
///   - completion: 命令发送结果回调
func readAllHistoryData(timestamp: TimeInterval = 0, callbacks: BCLDataSyncCallbacks, completion: @escaping (Result<Void, BCLError>) -> Void)
```

```
/// 读取未上传数据
/// - Parameters:
///   - timestamp: 获取指定时间戳之后的数据（默认为0,仅获取未上传的数据）
///   - callbacks: 回调集合
///   - completion: 命令发送结果回调
func readUnUploadData(timestamp: TimeInterval = 0, callbacks: BCLDataSyncCallbacks, completion: @escaping (Result<Void, BCLError>) -> Void)
```
{% endstep %}

{% step %}
### 上传历史数据到服务器

通过上面的复合指令接口拿到数据后，可以调用下面的接口来上传数据到服务器（将数据上传到云端服务器）

```
/// 上传历史数据
/// - Parameters:
///   - historyData: BCLRingDBModel数据数组
///   - mac: 设备MAC地址
///   - completion: 上传完成回调
public func uploadHistory(historyData: [BCLRingSDK.BCLRingDBModel], mac: String, completion: @escaping (Result<Void, BCLRingSDK.BCLError>) -> Void)
```

{% hint style="info" %}
说明：BCLDataSyncCallbacks#onProgress 回调方法返回的 model 对象就是一个记录点，数据结构是一致的，但只有获取到数据的字段有值，没有数据的字段为 0；可以在这里保存数据并上传到自己的服务器。
{% endhint %}
{% endstep %}

{% step %}
### 上传 gomore 算法的睡眠数据

上述历史数据传完后，再上传 gomore 算法的睡眠数据
{% endstep %}

{% step %}
### 检查 Gomore 设备授权状态

```
 
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
 public func checkAndAuthorizeGoMore(companyKey: String, progress: @escaping (BCLRingSDK.BCLGoMoreAuthStep) -> Void, completion: @escaping (Result<BCLRingSDK.BCLGoMoreAuthResult, BCLRingSDK.BCLError>) -> Void)
```
{% endstep %}

{% step %}
### 设置用户信息，为 gomore 算法做准备

```
/// GoMore设置个人信息
/// - Parameters:
///   - age: 年龄（10-99）
///   - gender: 性别（0女性，1男性）
////   - height: 身高（100-220 cm）
///   - weight: 体重（10-150 kg）
///   - maxHeartRate: 最大心率值（nil或<0表示无效）
///   - normalHeartRate: 常态心率值（nil或<0表示无效）
///   - maxOxygenUptake: 最大摄氧量（nil或<0表示无效）
///   - completion: 设置个人信息回调
/// - Result: 设置结果
/// - BCLGoMoreSetPersonalInformationResponse: 包含设置结果的响应模型
/// - BCLError: 错误信息
func goMoreSetPersonalInformation(age: UInt8, gender: UInt8, height: UInt8, weight: UInt8, maxHeartRate: Int16?, normalHeartRate: Int8?, maxOxygenUptake: Int8?, completion: @escaping (Result<BCLGoMoreSetPersonalInformationResponse, BCLError>) -> Void)
```
{% endstep %}

{% step %}
### 获取 gomore 睡眠数据

```
/// 读取GoMore睡眠数据（读取戒指中的数据集）
/// - Parameter completion: 读取GoMore睡眠数据回调
/// - Result: 读取结果
/// - BCLGoMoreSleepDataResponse: GoMore睡眠数据响应
///   - sleepModels: [BCLGoMoreSleepModel] - 睡眠数据模型列表
/// - BCLError: 错误信息
func readGoMoreSleepData(completion: @escaping (Result<BCLGoMoreSleepDataResponse, BCLError>) -> Void)
```

{% hint style="info" %}
说明：此处的数据即原始 gomore 算法的睡眠数据，可以保存到自己的服务器
{% endhint %}
{% endstep %}

{% step %}
### 上传 gomore 睡眠记录

```
/// 上传GoMore睡眠数据到服务器
/// - Parameters:
///  - sleepModels: 睡眠数据模型列表
///  - completion: 上传结果回调
///  - Result: 上传结果
///  - Void: 无返回值
///  - BCLError: 错误信息
func uploadGoMoreSleepData(sleepModels: [BCLGoMoreSleepModel], completion: @escaping (Result<Void, BCLError>) -> Void)
```
{% endstep %}
{% endstepper %}

### 运动数据

#### 获取方法

1. 实时步数、活动热量

通过 readUnUploadData 或者 readAllHistoryData 方法获取历史数据后，自行统计。

2. 锻炼时长、活动时长

```
/// 获取时间线数据
/// - Parameters:
///   - startTime: 起始时间戳（通常为秒级时间戳；以服务端约定为准）
///   - endTime: 结束时间戳（通常为秒级时间戳；以服务端约定为准）
///   - completion: 完成回调
public func getTimeline(startTime: NSNumber, endTime: NSNumber, completion: @escaping (NSDictionary) -> Void) {
  ringManager.getTimeline(startTime: Int64(startTime.intValue), endTime: Int64(endTime.intValue)) { [weak self] res in
      guard let self else { return }
      switch res {
      case let .success(items):
          let list = items.compactMap { self.encode($0) as? [String: Any] }
          let payload = self.result(data: ["list": list])
          self.callbackQueue.async { completion(payload) }
      case let .failure(error):
          let payload = self.result(error: error)
          self.callbackQueue.async { completion(payload) }
      }
  }
}
```

### 睡眠数据

#### 获取方法

1. 睡眠数据

```
/// 获取Gomore睡眠详情数据
/// - Parameters:
///   - date: 查询日期（本地自然日时间，按调用端本机时区理解）
///   - userId: 用户ID（可选，默认为nil）
///   - completion: 完成回调，返回睡眠详情数据或错误
/// - Note: 返回的 BCLRingSleepModel 包含睡眠统计信息和睡眠数据列表
func getGoMoreSleepData(date: Date, userId: String? = nil, completion: @escaping (Result<BCLRingSleepModel, BCLError>) -> Void)
```

```
/// 批量获取Gomore睡眠数据
/// - Parameters:
///   - dates: 日期数组，格式为 "yyyy-MM-dd"，如 ["2025-12-28", "2025-12-29"]
///   - userId: 用户ID（可选，默认为nil）
///   - completion: 完成回调，返回睡眠数据摘要数组或错误
/// - Note: 返回的 BCLRingSleepDayModel 包含每日睡眠统计摘要
func getGoMoreSleepDataBatch(dates: [String], userId: String? = nil, completion: @escaping (Result<[BCLRingSleepDayModel], BCLError>) -> Void)
```

### 情绪压力

#### 获取方式

1. 连续压力测量值\
   通过 readUnUploadData 或者 readAllHistoryData 方法获取历史数据后，自行统计。
2. 单次压力测量值

```
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

返回结果中包含 **心率、心率变异性、精神压力、体温**

```
/// 停止心率测量
/// - Parameter completion: 停止心率测量回调
/// - BCLStopHeartRateResponse: 包含停止心率测量结果的响应模型
func stopHeartRate(completion: @escaping (Result<BCLStopHeartRateResponse, BCLError>) -> Void)
```

### 心率数据

#### 获取方式

1. 连续心率测量值、连续心率变异性（HRV）\
   通过 readUnUploadData 或者 readAllHistoryData 方法获取历史数据后，自行统计。
2. 单次测心率

```
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

返回结果中包含 **心率、心率变异性、精神压力、体温**

```
/// 停止心率测量
/// - Parameter completion: 停止心率测量回调
/// - BCLStopHeartRateResponse: 包含停止心率测量结果的响应模型
func stopHeartRate(completion: @escaping (Result<BCLStopHeartRateResponse, BCLError>) -> Void)
```

### 血氧数据

#### 获取方式

1. 连续血氧测量值\
   通过 readUnUploadData 或者 readAllHistoryData 方法获取历史数据后，自行统计。
2. 单次测血氧

```
/// 血氧测量
/// - Parameters:
///   - collectTime: 采集时间(单位：秒) 默认30s
///   - collectFrequency: 采集频率(单位：次/秒) 默认25hz
///   - waveformConfig: 波形配置(0:不上传 1:上传)
///   - progressConfig: 进度配置(0:不上传 1:上传)
/// - BCLBloodOxygenResponse: 包含测量结果的响应模型
func startBloodOxygen(collectTime: Int, collectFrequency: Int, waveformConfig: Int, progressConfig: Int, completion: @escaping (Result<BCLBloodOxygenResponse, BCLError>) -> Void)
```

返回结果中包含 **血氧、心率、体温**

```
/// 停止血氧测量
/// - Parameter completion: 停止血氧测量回调
/// - BCLStopBloodOxygenResponse: 包含停止血氧测量结果的响应模型
func stopBloodOxygen(completion: @escaping (Result<BCLStopBloodOxygenResponse, BCLError>) -> Void)
```

### 体温数据

#### 获取方式

1. 连续体温测量值\
   通过 readUnUploadData 或者 readAllHistoryData 方法获取历史数据后，自行统计。
2. 单次体温测量值

startHeartRate、startBloodOxygen 这两个方法中都包含体温

**此处建议用主动测量指令获取，上面两个方法虽然也包含体温数据，但如果用户没有测心率或者血氧的话，就无法获取体温数据了。**

```
/// 读取温度
/// - Parameter completion: 读取温度回调
/// - Result: 读取结果
/// - BCLTemperatureResponse: 包含温度信息的响应模型
/// - BCLError: 错误信息
public func readTemperature(completion: @escaping (Result<BCLRingSDK.BCLTemperatureResponse, BCLRingSDK.BCLError>) -> Void)
```

### 生命体征

#### 获取方式

### 固件升级

1. 步骤

1、首先，通过SDK的接口调用，检查当前固价版本是否为最新的。

2、如果不是最新版本，接口会返回最新的部件版本号、文件名和下载地址等信息。

3、然后，使用SDK提供的下载接口，从云端下载最新的固件文件

4、可以通过SDK提供的接口来检查固件升级的类型。（Nordic、Apollo、Phy）

5、根据对应的固件升级类型，然后执行相应的提供的固件升级接口，进行固件升级。

方法说明

```
/// 固件版本更新检查
/// - Parameters:
///   - version: 当前固件版本号
/// - Parameter completion: 固件版本更新检查回调
func checkFirmwareUpdate(version: String, completion: @escaping (Result<BCLFirmwareVersionInfo, BCLError>) -> Void)
```

```
/// 固件下载
/// - Parameters:
///   - url: 固件下载地址
///   - fileName: 固件文件名
///   - destinationPath: 固件文件保存路径
///   - progress: 固件下载进度回调
/// - Parameter completion: 固件下载回调
func downloadFirmware(url: String, fileName: String, destinationPath: String, progress: @escaping (Double) -> Void, completion: @escaping (Result<String, BCLError>) -> Void)
```

```
/// 获取固件升级类型
/// - Parameters:
///   - firmwareVersion: 固件版本号()
///   - completion: 获取结果回调
func getOTAType(firmwareVersion: String?, completion: @escaping (BCLOTAType) -> Void)
```

```
/// Apollo 固件升级接口
/// - Parameters:
///   - filePath: 固件文件路径
///   - progressHandler: 进度回调，返回 0-100 的进度值
///   - completion: 完成回调，返回成功或失败
func apolloUpgradeFirmware(filePath: String, progressHandler: ((Float) -> Void)? = nil, completion: @escaping (Result<Void, BCLError>) -> Void)
```

```
/// Nordic 固件升级接口
/// - Parameters:
///   - filePath: 固件文件路径
///   - progressHandler: 进度回调，返回 0-100 的进度值
///   - completion: 完成回调，返回成功或失败
func nrfUpgradeFirmware(filePath: String, fileName: String, progressHandler: ((Int) -> Void)? = nil, completion: @escaping (Result<BCLNrfUpgradeState.Stage, BCLError>) -> Void)
```

```
/// Phy 固件升级接口
/// - Parameters:
///   - filePath: 固件文件路径
///   - progressHandler: 进度回调，返回 0-100 的进度值
///   - completion: 完成回调，返回成功或失败
func phyUpgradeFirmware(filePath: String, progressHandler: @escaping (Double) -> Void, completion: @escaping (Result<BCLPhyUpgradeState, BCLError>) -> Void)
```

```
/// PHY Boot模式固件升级接口
/// 用于处理PHY固件升级过程中断导致设备卡在boot模式的情况
/// - Parameters:
///   - filePath: 固件文件路径
///   - device: Boot模式设备信息
///   - peripheral: Boot模式外设对象
///   - progressHandler: 升级进度回调，返回 0-100 的进度值
///   - completion: 升级状态回调
func phyBootModeUpgrade(filePath: String, device: BCLDeviceInfoModel, peripheral: CBPeripheral, progressHandler: @escaping (Double) -> Void, completion: @escaping (Result<BCLPhyBootModeUpgradeState, BCLError>) -> Void)
```

固件升级部分 iOS 建议看文档（[https://yongxin.gitbook.io/yongxin-docs/documentation/sheng-ji-fu-wu/ota-sheng-ji-fu-wu#iosota-sheng-ji-shuo-ming](https://yongxin.gitbook.io/yongxin-docs/documentation/sheng-ji-fu-wu/ota-sheng-ji-fu-wu#iosota-sheng-ji-shuo-ming)），写的很详细。
