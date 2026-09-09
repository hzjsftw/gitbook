---
description: sdk里保存的历史数据，是原始数据，不利于根据日志分析问题，sdk提供了转换的方法，可以生成txt文件。
icon: rectangle-history-circle-user
---

# 读取sdk里的历史记录

### **android：**



```java
LmAPILite类：
/**
 * 将历史数据通过转换，解析成服务端分析问题的格式，保存到本地LocalHistoryLog.txt，只限于客户
 开发过程中调试问题，或者客户正式发布app后，有让用户提交问题反馈的功能，
 将日志传到服务端，进行分析。（在保存时，会将txt清空，建议传入需要排查的时间段内的所有历史数据）
 * @param historyDataBeanList
 */
public static void writeHistoryLogToTxt( List<HistoryDataBean> historyDataBeanList){
    StringBuilder historyLog = new StringBuilder();
    if(historyDataBeanList==null){
        return;
    }
    for (HistoryDataBean lomoRingHistory : historyDataBeanList) {

        byte[] rrBytesList = lomoRingHistory.getRrBytes();
        if(rrBytesList==null){
            continue;
        }
        String rrbytesStr="";
        StringBuilder rrBytesStr = new StringBuilder();
        if (rrBytesList.length != 0) {
            rrBytesStr.append("[");
            for (float aFloat : rrBytesList) {
                rrBytesStr.append((int) aFloat);
                rrBytesStr.append(":");
            }
            rrBytesStr.deleteCharAt(rrBytesStr.length() - 1);
            rrBytesStr.append("]");
            rrbytesStr=rrBytesStr.toString();

        }
        long res1 = 0;
        long res2 = 0;
        if (lomoRingHistory.getReserve() != 0) {
            res1 = lomoRingHistory.getReserve() & 0xFF;
            res2 = (lomoRingHistory.getReserve() >> 8) & 0xFF;
        }

        String message = String.format("[%-4d:%4d]%s step:%5d hr:%3d spo2:%3d hrv:%3d sprit:%d temp:%4d sport:%d sleep:%d pi:%5d res:%5d res1:%5d res2:%5d rr_num:%2d array:%s",
                lomoRingHistory.getTotalNumber(), lomoRingHistory.getIndexNumber(), DateUtils.longToString(lomoRingHistory.getTime()*1000,"yyyy.MM.dd_HH.mm.ss"), lomoRingHistory.getStepCount(),
                lomoRingHistory.getHeartRate(), lomoRingHistory.getBloodOxygen(), lomoRingHistory.getHeartRateVariability(),
                lomoRingHistory.getStressIndex(), lomoRingHistory.getTemperature(), lomoRingHistory.getExerciseIntensity(),
                lomoRingHistory.getSleepType(),  lomoRingHistory.getMeasurementMarker(), lomoRingHistory.getReserve(), res1, res2, lomoRingHistory.getRrCount(), rrbytesStr
        );
        historyLog.append(message).append("\n");
    }
    ImageSaverUtil.saveImageToInternalStorage(application, historyLog.toString(), "LM", "LocalHistoryLog.txt", false);

}
```
