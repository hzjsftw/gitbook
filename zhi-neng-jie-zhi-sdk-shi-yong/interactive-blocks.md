---
description: >-
  戒指会每隔一定周期，测量用户的体征数据，保存在戒指里，这是非常重要的数据，对于app显示用户数据，计算用户睡眠情况，都非常必要。获取历史数据，有两种方式，第一是获取所有历史数据，第二是只获取未上传数据
icon: rectangle-vertical-history
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

# 历史数据相关

### 数据保存在哪？

数据保存在哪里，决定了获取历史数据的方式。

1.如果保存在app里，可以第一次打开App的时候，同步一下所有历史数据，后续只获取未同步历史数据。

这种方式的风险就是，如果戒指在其他手机，app或者上位机上同步过数据，时间段是A-B，App再同步未上传数据的时候，只会从B以后上传，这样A-B期间的数据就丢失了。

2.如果保存到服务器上，可以在同步数据之前，获取一下该用户最后一条历史记录，然后根据这个时间，同步这个时间之后的数据，这样服务器端的历史数据就会是完整的

历史数据默认是保存在本地数据库的

### 如何保存到服务器

如果想保存到用户自己的服务器，需要将数据上传，最好再有一个查询用户最后一条记录的时间的接口，调用sdk api的时候可以传进去，获取该时间段后的数据，保证数据链完整

我们也提供了上传历史数据到勇芯服务器的接口，和拉取最后一条数据的接口，上传到我方服务器目的是计算睡眠和Timeline，这部分只做一个简介，具体内容请查看《升级服务》部分。

**android:**

```java
   //如需使用更精准的睡眠算法，获取戒指历史数据时，请调用该指令(LmAPI.READ_HISTORY不支持上传服务器操作)，这个支持一代协议,(byte) 0x00是未上传历史，(byte) 0x01是所有历史，正常情况下传0x00就可以了
   LmAPI.READ_HISTORY_UPDATE_TO_SERVER((byte) 0x00,  mac, new IHistoryListener() {
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
                }, new IWebHistoryResult() {
                    @Override
                    public void updateHistoryFinish() {
                        postView("\n历史数据上传服务器完成");
                    }
                });
```

```java
LogicalApi:
 public static void loadUserLatestHistory( IWebLastTimeResult iWebLastTimeResult)

建议在createToken接口调用成功后，调用这个接口，尽量在蓝牙连接之前，获取到时间戳
LmAPI或LmAPILite:
 public static void READ_HISTORY_UPDATE_TO_SERVER(byte type, long timeMillis)
将timeMillis设置为loadUserLatestHistory获取的时间戳(time字段)
```

### 保存到数据库

从戒指获取历史记录的时候，会同步保存到本地数据库，现在提供一些操作数据库的方法：

**android:**

```java
//查询历史数据
List<HistoryDataBean> queryHistoryData(long dayBeginTime,long dayEndTime,String mac) 
//查询历史数据按照时间进行正序
List<HistoryDataBean> queryHistoryDataOrderByTimeAsc(long dayBeginTime,long dayEndTime,String mac)
//查询历史数据按照步数进行倒叙
List<HistoryDataBean> queryHistoryDataOrderByStepCountDesc(long dayBeginTime,long dayEndTime,String mac)
//清除所有历史数据
void deleteHistoryData()；
```

参数说明：dayBeginTime ：开始时间戳，单位：秒\
dayEndTime ：结束时间戳，单位：秒\
mac ：设备的MAC地址

返回值：

```java
/**
 * 历史数据
 */
@Entity
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
    }
```

### 公版App怎么做的

1.App启动，首先清除本地数据库里的内容，从服务器拉取48小时内保存到本地，根据本地数据库里的数据，显示首页内容，同时获取睡眠数据

2.其他页面涉及到历史数据的地方，都从云端拉取

3.获取最后一条记录，解析出时间戳time

4.戒指连接成功，根据time，调用sdk的获取记录的api，LmAPI.READ\_HISTORY\_UPDATE\_TO\_SERVER

5.上传成功以后，重新获取睡眠记录

### 读取戒指历史数据

**android:**

```java
LmAPI.READ_HISTORY(byte type, long timeMillis,IHistoryListener iHistoryListener)
```

参数说明：

type: 1,获取全部历史记录；0，获取未上传的历史记录。读取过为上传历史记录，下次读取的时候，就会从上次读取时间以后算起，如果想要将之前的数据也拿到，可以在 progress自己记录，本地数据库也保存了数据，也可以通过DataApi.instance.queryHistoryData查询到。

timeMillis：秒级时间戳，0是默认所有未上传数据，传值以后，会上报该时间以后的数据，即使timeMillis在上次上传数据的时间之前 返回值：

```java
LmAPI.READ_HISTORY(type, new IHistoryListener() {
    @Override
    public void error(int code) {
        handler.removeMessages(0x99);
        dismissProgressDialog();
        switch (code) {
            case 0:
                ToastUtils.show("正在测量中,请稍后重试");
                break;
            case 1:
                ToastUtils.show("正在上传历史记录,请稍后重试");
                break;
            case 2:
                ToastUtils.show("正在删除历史记录,请稍后重试");
                break;
            default:
                break;
        }
    }
    @Override
    public void success() {
        //同步完成
    }
    @Override
    public void progress(double progress, com.lm.sdk.mode.HistoryDataBean dataBean) {
           //处理历史数据
    }
});
```

简化版本（LmApiLite）

```java
   public static void READ_HISTORY(int type, long timeMillis,IHistoryListenerLite listenerLite)

   public interface IHistoryListenerLite {
    void error(int code);
    void success();
    void progress(double progress, HistoryDataBean historyDataBean);
    void clearHistory();
}
```

### 清空戒指历史数据

```java
LmAPI.CLEAN_HISTORY（）
```

简化版本（LmApiLite）

```java
public static void CLEAN_HISTORY()
```

### 二代协议特殊处理

**android：**

二代协议会自动上传历史数据，需要在页面上设置监听历史数据上传，否则会有null异常，在Activity里进行监听，通过LmAPI或者LmAPILite调用，样例：

```java
private void READ_HISTORY_AUTO() {
    uploadServerHistoryData.clear();
    LmAPILite.READ_HISTORY_AUTO(new IHistoryListenerLite() {
        @Override
        public void error(int code) {
            readHistoryError();
           
        }

        @Override
        public void success() {
            readHistorySuccess(true);
           

        }

        @Override
        public void progress(double progress, HistoryDataBean historyDataBean) {
            uploadServerHistoryData.add(historyDataBean);
            readHistoryProgress();
        }

        @Override
        public void clearHistory() {

        }

        @Override
        public void noNewDataAvailable() {
           
        }
    });
}
```

除非有特殊情况，比如需要手动获取历史数据，使用二代协议，一般不需要再重复使用《读取戒指历史数据》
