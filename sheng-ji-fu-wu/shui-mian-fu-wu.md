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
     * 读取戒指本地历史数据
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

如果使用的是二代协议，二代协议是会自动上传历史数据的，需要在页面上设置监听，使用下述方法上传历史数据

```java
  /**
     * 二代协议会自动上传历史数据到服务器，需要在页面上监听该接口
     */
    public static void READ_HISTORY_AUTO_UPDATE_TO_SERVER(  String mMac, IHistoryListenerLite listenerLite, IWebHistoryResult mWebHistoryResult) {
        iHistoryListener = listenerLite;
        iWebHistoryResult = mWebHistoryResult;
        mac=mMac;
        uploadHistoryData.clear();

    }
```

二代协议的时间戳控制，是通过来控制的。具体用法参照

{% content-ref url="../zhi-neng-jie-zhi-sdk-shi-yong/er-dai-xie-yi.md" %}
[er-dai-xie-yi.md](../zhi-neng-jie-zhi-sdk-shi-yong/er-dai-xie-yi.md)
{% endcontent-ref %}

```java
public static void APP_CONNECT(long timeMillis)
public static void APP_REFRESH(long timeMillis)
```

### 数据库字段

```java
 /
   public class HistoryDataBean{

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
    private int timeZone;

    /**
     * 计步类型，0是普通，1是累积
     */
    private int stepCountingType;
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
