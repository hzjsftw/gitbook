---
description: >-
  正常戒指是0点清空计步，有一些特殊戒指，不是到0点清零，是每隔5分钟清零一次，只记录5分钟之间的步数，这个需要后台来计算当前步数，也就是从0点到当前时间的步数总和，前提是先同步一下未上传历史数据，保证数据完整。
icon: shoe-prints
---

# 特殊戒指计步方案

### Android提供的接口

在扫描的时候，可以根据广播协议，进行判断是否是这种戒指

```java
//bleDeviceInfo.getStepAccumulation()>0，说明戒指，支持每5分钟步数清零，重新累积功能
BleDeviceInfo bleDeviceInfo = LogicalApi.getBleDeviceInfoWhenBleScan(device, rssi, bytes,false);
```

如果是这种戒指，获取当前步数，调用如下方法，否则还是用之前的`LmAPI.STEP_COUNTING()`

```java
 LmAPI.GET_CURRENT_STEP_FROM_SERVER(mBluetoothDevice.getAddress(), new IWebStepResult() {
                @Override
                public void getCurrentSteps(double allStep) {
                    postView("步数：\n"+allStep);
                }

                @Override
                public void error(String message) {

                }
            });
```

如果有显示步数趋势的需求，可以调用根据开始时间和结束时间，获取当前用户所有历史的接口，app端可以根据历史记录，累加单位时间内的步数，灵活显示每天，每小时等步数的趋势图

```java
LogicalApi:
    /**
     * 查询当前用户，开始时间和结束时间内的历史数据列表
     * @param startTime 开始时间戳(秒级时间戳)
     * @param endTime 结束时间戳(秒级时间戳)
     * @param webApiResult
     */
    public static void getHistoryDatasWithTime(long startTime,long endTime, IWebGetHistoryResult webApiResult) {
```

接口返回的是原始历史数据，客户端可以根据自己需求，比如计算每天24小时，各个小时的步数，可以计算某个小时0分到59分之内的步数，累加(步数是65535，是主动测量或者周期测量的特殊标志，需要排除掉)，算作该小时的步数

### IOS提供的接口
