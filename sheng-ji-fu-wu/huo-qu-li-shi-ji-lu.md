---
description: 鉴于有些用户需要查询服务器上，已经保存的用户历史数据，特提供一些方法
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
