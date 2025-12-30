---
description: 根据开始时间和结束时间，获取时间线，包括跑步和行走，前提是需要上传历史数据到服务器
icon: watch-smart
---

# 时间线

### android提供的接口

```java
LogicalApi.getTimeLineWithHistory(startTime, endTime, new IWebTimeLineResult() {
                    @Override
                    public void timelineResult(List<MovementSegment> movementSegments) {
                        
                    }

                    @Override
                    public void serviceError(String errorMsg) {

                    }
                });
                
        //startTime和endTime是秒级时间戳，比如传递当天的0点到24点
       // 获取当前日期(不含时间)
        LocalDate today = LocalDate.now();
        // 获取系统默认时区
        ZoneId zoneId = ZoneId.systemDefault();
        // 当天0点(00:00:00)
        LocalDateTime startOfDay = today.atStartOfDay();
        // 当天24点(实际上是次日的00:00:00)
        LocalDateTime endOfDay = today.plusDays(1).atStartOfDay();
        // 转换为秒级时间戳
        long startTimestamp = startOfDay.atZone(zoneId).toEpochSecond();
        long endTimestamp = endOfDay.atZone(zoneId).toEpochSecond();
```

对应的实体类：

```java
public class MovementSegment {
    private  long startTime;    //运动段起始时间
    private  long endTime;      //运动段结束时间
    private  String type;       //"walk" or "run" or "sit"

    public long getStartTime() {
        return startTime;
    }

    public void setStartTime(long startTime) {
        this.startTime = startTime;
    }

    public long getEndTime() {
        return endTime;
    }

    public void setEndTime(long endTime) {
        this.endTime = endTime;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }
}
```

### IOS提供的接口
