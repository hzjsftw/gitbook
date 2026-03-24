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
 LmAPILite.STAR_ELEC()//开启心电测量
 LmAPILite.STOP_ELECTROCARDIOGRAM();//结束心电测量
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

**iOS:**

```swift
/// 心电图-采集人体心电
/// - Parameter completion: 采集人体心电回调
func startTakeECG(completion: @escaping (Result<BCLTakeECGResponse, BCLError>) -> Void)
```

#### 调用示例

```swift
BCLRingManager.shared.startTakeECG { res in
    switch res {
    case let .success(response):
        // 数据范围统计
        if !response.ecgValues.isEmpty {
            let minVal = response.ecgValues.min() ?? 0
            let maxVal = response.ecgValues.max() ?? 0
            let avgVal = response.ecgValues.reduce(0, +) / response.ecgValues.count
            BDLogger.info("[原始数据统计] 最小值: \(minVal), 最大值: \(maxVal), 平均值: \(avgVal)")
        }
    case let .failure(error):
        BDLogger.error("开始测量失败: \(error)")
    }
}
```

```swift
/// 心电图-采集模拟信号
/// - Parameter completion: 采集模拟信号回调
func startTakeECGSimulator(completion: @escaping (Result<BCLTakeECGSimulatorResponse, BCLError>) -> Void)
```

```swift
/// 心电图-停止采集
/// - Parameter completion: 停止采集回调
func stopECG(completion: @escaping (Result<BCLStopECGResponse, BCLError>) -> Void)
```
