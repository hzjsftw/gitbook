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

除非特殊规定，发送的指令都是小端模式，

<figure><img src="../.gitbook/assets/ScreenShot_2025-12-29_110031_129.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/ScreenShot_2025-12-29_110048_160.png" alt=""><figcaption></figcaption></figure>
