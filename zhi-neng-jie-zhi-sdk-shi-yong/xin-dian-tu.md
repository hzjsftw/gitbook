---
description: >-
  心电图功能只在心电戒指上支持，可以通过BLEUtils.isSupportElectrocardiogram()判断是否支持，因为绘图比较麻烦，涉及到东西较多，sdk里提供了样例
icon: waveform
---

# 心电图

### android样例

```java
LogicalApi.startECGActivity(TestActivity2.this);
```

查看心电图样例，可以直接使用，如需定制化，可以参考项目中的(SourceCode/心电图相关)

### 对应的指令

**android：**

```java
 LmAPI.STAR_ELEC()//开启心电测量
 LmAPI.STOP_ELECTROCARDIOGRAM();//结束心电测量
```

简化版本

```java
 public static void STAR_ELEC(IECGListenerLite mIecgListener)
 public static void STOP_ELECTROCARDIOGRAM()

 public interface IECGListenerLite {
     void result(int HRValue,int[] ecgValues);
     void error(int code);
 }

```

