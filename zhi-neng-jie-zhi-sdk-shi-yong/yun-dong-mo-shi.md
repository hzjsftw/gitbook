---
description: 一些戒指支持运动模式，可以开启运动模式，然后去运动，戒指会实时上传，如果戒指与app断连，下次连接时候，也可以通过断点续传的方式，将数据上传到app
---

# 运动模式

### android

```java
    /**
     * 开启运动模式
     */
    public static void START_SPORT(ISportListenerLite listenerLite) {
        iSportListenerLite = listenerLite;
        SEND_CMD(convertToBytes(0x38, new byte[]{0x1,2}));
    }

    /**
     * 获取历史运动模式，断点续传
     */
    public static void HIS_SPORT(ISportListenerLite listenerLite) {
        iSportListenerLite = listenerLite;
        SEND_CMD(convertToBytes(0x38, new byte[]{0x2}));
    }

    /**
     * 结束运动模式
     */
    public static void STOP_SPORT(ISportListenerLite listenerLite) {
        iSportListenerLite = listenerLite;
        SEND_CMD(convertToBytes(0x38, new byte[]{0x3}));
    }

```

返回结果是：

```java

public interface ISportListenerLite {

    /**
     * 运动模式
     * @param sportType 运动类型
     * @param step 步数
     * @param heart 心率
     * @param calorie 卡路里
     * @param time 时间
     */
    void onSport(int sportType,int step , int heart,int calorie,long time);
}
```
