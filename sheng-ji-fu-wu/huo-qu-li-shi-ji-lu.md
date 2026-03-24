---
description: 鉴于有些用户需要查询服务器上，已经保存的用户历史数据，特提供一些方法
icon: rectangle-vertical-history
---

# 获取历史记录

### android

LogicalApi：

```java
 /**
     * 查询当前用户，开始时间和结束时间内的历史数据列表
     * @param startTime 开始时间戳(秒级时间戳)
     * @param endTime 结束时间戳(秒级时间戳)
     * @param webApiResult
     */
    public static void getHistoryDatasWithTime(long startTime,long endTime, IWebGetHistoryResult webApiResult)
```


### iOS 

``` swift
/// 获取云端上设备历史记录信息
/// - Parameters:
///   - mac: 设备MAC地址（可选，传空字符串或nil表示获取所有设备）
///   - startTime: 开始时间戳
///   - endTime: 结束时间戳
///   - completion: 获取设备历史记录回调，返回历史记录数组或错误
func getServerDeviceHistory(mac: String? = nil,startTime: Int64,endTime: Int64,completion: @escaping (Result<[BCLRingDBModel], BCLError>) -> Void)
```