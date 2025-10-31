---
description: 在一些特殊场合，比如ota之前，想要查看一下当前戒指的蓝牙信号，可以调用该部分内容，信号一般与设备有关，与距离有关，与环境有关
icon: wifi-fair
---

# 获取蓝牙信号

### 获取蓝牙信号

**android：**

RSSI（**接收信号强度指示**）是信号强度的意思，一般用于ota升级前对戒指的信号检测，建议 > -70

```java
    BLEService.readRomoteRssi();
    Log.i(TAG, "rssi = "+ BLEService.RSSI);
```

会有一点延时，在Activity里注册个回调获取

```java
BLEService.setCallback(new BluetoothConnectCallback() {
            @Override
            public void onConnectReceived(String data) {
             
            }

            @Override
            public void onGetRssi(int rssi) {
                App.getInstance().getDeviceBean().setRssi(rssi);
            }
        });
```

需要注意rssi变化略微延迟，数字越大，信号越强，如 -52 > -60  
这个信号量，会影响到主动测量的质量，可以在页面上，进行提示，如果信号量小于-80，则提示"蓝牙信号弱，请将设备靠近手机"，可以闪烁图标进行提示
