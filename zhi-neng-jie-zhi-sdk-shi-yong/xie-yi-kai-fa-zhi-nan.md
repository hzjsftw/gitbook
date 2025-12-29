---
description: 由于有的用户需要使用蓝牙协议进行开发，特给出解析蓝牙协议的案例
---

# 协议开发指南

## 以同步和获取戒指时间指令为例

<figure><img src="../.gitbook/assets/ScreenShot_2025-12-29_105557_529.png" alt=""><figcaption></figcaption></figure>

解析样例：\
发送指令组合：0x00，随机值，0x10是类别，实际数值，java代码组合如下

```java
  public static byte[] convertToBytes(int cmd, byte[] value) {
        byte[] data = new byte[3 + value.length];
        data[0] = 0x00;
        data[1] = (byte) new Random().nextInt(254);
        data[2] = (byte) cmd;
        System.arraycopy(value, 0, data, 3, value.length);
        return data;
    }
```

发送样例:

```java
   /**
     * 发送指令
     */
    public static void SEND_CMD(byte[] data) {
       
        BLEService.sendCmd(data);
    }

```

发送数据指令组合样例：

```java
 /**
     * 把一个整形改为8位的byte数组
     *
     * @param value
     * @return
     * @throws Exception
     */
    public static byte[] longTo8Bytes(long value) {
        byte[] result = new byte[8];
        result[7] = (byte) ((value >>> 56) & 0xFF);
        result[6] = (byte) ((value >>> 48) & 0xFF);
        result[5] = (byte) ((value >>> 40) & 0xFF);
        result[4] = (byte) ((value >>> 32) & 0xFF);
        result[3] = (byte) ((value >>> 24) & 0xFF);
        result[2] = (byte) ((value >>> 16) & 0xFF);
        result[1] = (byte) ((value >>> 8) & 0xFF);
        result[0] = (byte) (value & 0xFF);
        return result;
    }
```

同步时间：

```java
 public static void SYNC_TIME_ZONE(byte zone) {
        long currentTimeMillis = TimeUtils.getTimeWithZone();
        byte[] bytes = ConvertUtils.longTo8Bytes(currentTimeMillis);
        byte[] data = new byte[10];
        data[0] = 0x00;
        System.arraycopy(bytes, 0, data, 1, bytes.length);
        data[9] = zone;
        SEND_CMD(convertToBytes(0x10, data));
    }
```

h

除非特殊规定，发送的指令都是小端模式，

<figure><img src="../.gitbook/assets/ScreenShot_2025-12-29_110031_129.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/ScreenShot_2025-12-29_110048_160.png" alt=""><figcaption></figcaption></figure>
