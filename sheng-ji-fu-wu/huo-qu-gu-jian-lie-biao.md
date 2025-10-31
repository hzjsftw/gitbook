---
description: 这个列表，除非有用户想回退固件，或者戒指升级到指定固件，才会用到，其他用户，直接使用最新固件即可
icon: chart-simple
---

# 获取固件列表

### android提供的接口

```java
//String category, 对应固件类别，比如Z3R，Z2W之类，不传不会返回固件信息
LogicalApi.firmwareList( category,new IWebFirmwareListResult() {
                    @Override
                    public void firmwareListResult(List<Firmware> firmwareList) {
                        
                    }

                    @Override
                    public void serviceError(String errorMsg) {

                    }
                });
```

对应的实体类

```java
public class Firmware {
    /**
     *固件名称
     */
    private String fileName;
    /**
     * 固件相对路径
     */
    private String filePath;
    /**
     * 固件下载地址
     */
    private String fileUrl;
}
```

### IOS提供的接口
