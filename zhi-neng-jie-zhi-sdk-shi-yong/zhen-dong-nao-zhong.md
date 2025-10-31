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

