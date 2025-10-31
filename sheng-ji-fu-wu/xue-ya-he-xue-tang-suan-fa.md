---
description: 这个功能需要特定戒指才支持，原理是基于戒指传输的波形值，经过python算法，给出具体的血压值或者血糖值
icon: hand-holding-droplet
---

# 血压和血糖算法

### android提供的接口

使用LmApi或者LmApiLite，调用`BLOOD_PRESSURE_APP`测量波形值

```java
/**
     *血压测量（0x3E）
     */
    /**
     * @param collectionTime            采集时间，默认30
     * @param waveformConfiguration     波形配置0:不上传 1:上传
     * @param progressConfiguration     进度配置0:不上传 1:上传
     * @param iBloodPressureAPPListener
     */
    public static void BLOOD_PRESSURE_APP(byte collectionTime, byte waveformConfiguration, byte progressConfiguration, IBloodPressureAPPListener iBloodPressureAPPListener) {
```

然后调用服务器接口，获取血压或者血糖

LogicalApi

```java
 /**
     * 根据血压波形值获取血压值或者血糖值
     * @param mac 连接蓝牙的mac
     * @param waveFormValue 波形值
     * @param testType 测量类型，输入"血压"或者"血糖"
     * @param webApiResult
     */
getBloodPressureOrSugar(String mac, String waveFormValue, String testType,IWebBloodPressureAndSugarResult webApiResult)
```

调用样例：

```java

         LmAPI.BLOOD_PRESSURE_APP((byte) 60, (byte) 1, (byte) 1, new IBloodPressureAPPListener() {
                    @Override
                    public void progress(int progress) {
                        postView("\n血压或血糖测量:"+progress );
                    }

                    @Override
                    public void resultData(int heartRate, int systolicPressure, int diastolicPressure) {
                       // postView("\nheartRate"+heartRate+",systolicPressure:" +systolicPressure+",diastolicPressure:"+diastolicPressure);
                    }

                    @Override
                    public void waveformData(int seq, int number, String waveData) {
                        postView("\nwaveData:"+waveData);
                        waveString.append(waveData);
                    }

                    @Override
                    public void error(int code) {
                        postView("\n血压或血糖测量:"+code);
                    }

                    @Override
                    public void success() {
                        postView("\n血压或血糖测量:success");
                        //根据测量要求，testType 测量类型，输入"血压"或者"血糖"
                        LogicalApi.getBloodPressureOrSugar(App.getInstance().getDeviceBean().getDevice().getAddress(), waveString.toString(),"血压", new IWebBloodPressureResult() {
                              @Override
                              public void bloodPressureResult(String diastolic, String systolic) {
                                  //血压结果
                              }
          
                              @Override
                              public void bloodSugarResult(String sugar) {
                                  //血糖结果
                              }
          
                              @Override
                              public void serviceError(String errorMsg) {
                                //接口出错
                              }
                          });
                    }

                    @Override
                    public void stop() {
                        postView("\n血压测量:stop");
                    }
                });
```

### IOS提供的接口
