---
description: 现在介绍sdk涉及到蓝牙连接，断连部分的方法
icon: bluetooth-b
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# 蓝牙连接与断连

## android部分

BLEUtils是使用蓝牙搜索、连接、 断开的公共类 ，统一由IResponseListener(简化版本是IResponseListenerLite)接口反馈。

### 搜索设备和广播解析

接口功能：开启蓝牙发现，发现周围的蓝牙设备并获取其广播的数据，解析广播数据判断是否符合智能戒指的广播格式，如果符合则从广播数据中获取配置信息。 接口声明：

```java
BLEUtils.startLeScan(Context context, BluetoothAdapter.LeScanCallback leScanCallback);
```

参数说明：context：上下文 leScanCallback：蓝牙搜索的回调\
返回值

```java
void onLeScan(BluetoothDevice device, int rssi, byte[] bytes)
```

该接口的返回值说明如下：

```java
private BluetoothAdapter.LeScanCallback leScanCallback = new BluetoothAdapter.LeScanCallback() {
    @Override
    public void onLeScan(BluetoothDevice device, int rssi, byte[] bytes) {
        //处理搜索到的设备
    }
};
```

注意事项：在开发者调试时候，发现不到设备的情况下，可用公版APP进行绑定对比测试，一般情况下，戒指正常的话，手机靠近戒指，RSSI（**接收信号强度指示**）的值大于-60。

sdk封装根据蓝牙扫描广播，获取是否符合条件的戒指，并返回该戒指的设备信息的方法LogicalApi.getBleDeviceInfoWhenBleScan，设备信息包括：

是否HID戒指(hidDevice:1是0非，兼容老版本戒指)

是否支持二代协议(communicationProtocolVersion:1不支持2支持)

是否支持绑定(bindingIndicatorBit,0不支持绑定、配对(仅软连接) 1绑定和配对 2仅支持配对)

充电指示位(chargingIndicator,1代表未充电 2代表充电中)

计步类型（stepAccumulation，0代表普通计步，即0点清零，最后一条数据为当前步数，1代表累积计步，统计的是每5分钟的步数，当前步数需要累加，步数是65535为主动测量，需排除掉）

```java
BLEUtils.startLeScan(this, leScanCallback);
 private BluetoothAdapter.LeScanCallback leScanCallback = new BluetoothAdapter.LeScanCallback() {
        @Override
        public void onLeScan(BluetoothDevice device, int rssi, byte[] bytes) {
            if (device == null || TextUtils.isEmpty(device.getName())) {
                return;
            }
            //是否符合条件，符合条件，会返回戒指设备信息，第四个参数是指定特定版本戒指，快康公司传递true，其他公司都是false
            BleDeviceInfo bleDeviceInfo = LogicalApi.getBleDeviceInfoWhenBleScan(device, rssi, bytes,false);
           
        }
    };
```

设置BLEUtils.isHIDDevice=deviceBean.getBindingIndicatorBit()==1;如果是HID模式的戒指，可以走强连接模式连接蓝牙，保证稳定性

重要: HID的戒指连接，需要将AndroidManifest.xml里的activity添加一个属性，因为HID戒指连接，会修改手机配置，如果不加，会导致重启或者连接多次的问题：

```java
 android:configChanges="fontScale|keyboard|keyboardHidden|locale|orientation|screenLayout|uiMode|screenSize|navigation"
 
```

**iOS:**
```Swift
/// 扫描蓝牙设备
/// - Parameter scanResultBlock: 扫描结果回调
/// - Result: 扫描结果
///   - [BCLDeviceInfoModel]: 蓝牙设备列表
///   - BCLError: 错误信息
func startScan(scanResultBlock: @escaping (Result<[BCLDeviceInfoModel], BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.startScan { result in
    switch result {
    case .success(let devices):
        print("扫描到设备列表: \(devices)")
        for device in devices {
            print("设备名称: \(device.name ?? "未知"), MAC: \(device.mac ?? "未知")")
        }
    case .failure(let error):
        print("扫描失败: \(error)")
    }
}
```

### 停止搜索

接口功能：蓝牙连接以后，可以关闭蓝牙搜索功能。\
接口声明：

```java
BLEUtils.stopLeScan(Context context, BluetoothAdapter.LeScanCallback leScanCallback);
```

参数说明：context：上下文 leScanCallback：蓝牙搜索的回调\
返回值：无

**iOS:**
```Swift
/// 停止扫描
func stopScan()
```

#### 调用示例
```Swift
BCLRingManager.shared.stopScan()
```

### 连接设备

接口功能：发起连接蓝牙设备。 接口声明：

```java
BLEUtils.connectLockByBLE(Context context, BluetoothDevice bluetoothDevice);
```

参数说明：context：上下文\
bluetoothDevice ：蓝牙设备\
返回值：

```java
@Override
public void lmBleConnecting(int code) {
    //正在连接
}
@Override
public void lmBleConnectionSucceeded(int code) {
    //连接成功
}
@Override
public void lmBleConnectionFailed(int code) {
    //连接失败
}
//断连后或者设备故障，发送指令超时，cmd是超时的指令
 @Override
    public void timeOut(String cmd) {

    }
```

为了保证断连后重连，需要在以上回调里，设置一些属性

```java
@Override
public void lmBleConnecting(int code) {
    //正在连接
  BLEUtils.setConnecting(true);//连接中，防止重复连接
}
@Override
public void lmBleConnectionSucceeded(int code) {
    //连接成功
   BLEUtils.setConnecting(false);
   BLEUtils.setGetToken(true);//连接成功
}
@Override
public void lmBleConnectionFailed(int code) {
    //连接失败
  BLEUtils.setGetToken(false);//连接失败
  BLEUtils.setConnecting(false);
}
```

BLEUtils.setConnecting//蓝牙是否在连接中，防止重复连接&#x20;

BLEUtils.setGetToken//是否已连接到蓝牙，这个按自己项目需求调用，公版app是在连接过后，指令走完，才算连接成功。

BLEUtils.isGetToken()，根据这个是否等于true，可以获取戒指是否已经连接，如果连接，可以进行发送指令的操作了


**iOS:**

```Swift
// 注意：该方法建议在扫描蓝牙设备后调用。

/// 连接设备
/// - Parameters:
///   - device: 蓝牙设备模型
///   - isAutoReconnect: 是否自动重连(默认：false)
///   - autoReconnectTimeLimit: 自动重连时间限制(单位：秒,默认：300)
///   - autoReconnectMaxAttempts: 自动重连最大次数(默认：3)
///   - connectResultBlock: 连接结果回调
func startConnect(device: BCLDeviceInfoModel,
                  isAutoReconnect: Bool = false,
                  autoReconnectTimeLimit: TimeInterval = 300,
                  autoReconnectMaxAttempts: Int = 3,
                  connectResultBlock: @escaping (Result<BCLDeviceInfoModel, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.startConnect(device: deviceModel, isAutoReconnect: true) { result in
    switch result {
    case .success(let device):
        print("连接成功: \(device.name ?? "未知")")
    case .failure(let error):
        print("连接失败: \(error)")
    }
}
```

```Swift
// 该方法建议在用户已绑定设备后，例如App启动时调用，或者是由后台进入前台等场景调用。Mac地址作为蓝牙设备唯一标识符，可以直接连接到指定设备。即使用户切换到另一个手机，只要知道设备的Mac地址，且蓝牙设备在广播中的时候，也可以直接进行连接。

/// 连接设备
/// - Parameters:
///   - macAddress: 设备的mac地址
///   - isAutoReconnect: 是否自动重连(默认：false)
///   - autoReconnectTimeLimit: 自动重连时间限制(单位：秒,默认：300)
///   - autoReconnectMaxAttempts: 自动重连最大次数(默认：3)
///   - connectResultBlock: 连接结果回调
func startConnect(macAddress: String,
                  isAutoReconnect: Bool = false,
                  autoReconnectTimeLimit: TimeInterval = 300,
                  autoReconnectMaxAttempts: Int = 3,
                  connectResultBlock: @escaping (Result<BCLDeviceInfoModel, BCLError>) -> Void)
```

#### 调用示例
```Swift
// 建议打开自动重连，App在使用过程中如果蓝牙意外断开，可以自动重连
BCLRingManager.shared.startConnect(macAddress: "XX:XX:XX:XX:XX:XX", isAutoReconnect: true) { result in
    switch result {
    case .success(let device):
        print("连接成功: \(device.name ?? "未知")")
    case .failure(let error):
        print("连接失败: \(error)")
    }
}
```

```Swift
// 设置自动重连(如需手动处理的时候可以通过该属性控制是否自动重连，默认false)
BCLRingManager.shared.isAutoReconnectEnabled
```

### 断开蓝牙

接口功能：断开设备。\
接口声明：

```java
BLEUtils.disconnectBLE(Context context);
```

参数说明：context：上下文\
返回值：无

**iOS:**
```Swift
/// 断开连接
/// - Note：手动断开蓝牙连接，如果设置了自动重连等，将不会进行自动重连，也会涉及到一些SDK中缓存的设备信息清理操作（平时无需调用，如果涉及到用户退出登录、等情况可以按需调用）
func disconnect()

/// 断开蓝牙设备连接(固件定制需求使用)
/// - Parameter peripheral: 蓝牙设备
/// - Note: 该方法适用于HID模式下的戒指，需要断开当前设备连接，进而重新连接蓝牙设备触发系统配对弹窗功能
func disconnect(peripheral: CBPeripheral?)
```

#### 调用示例
```Swift
// 断开当前连接
BCLRingManager.shared.disconnect()

// 断开指定设备
BCLRingManager.shared.disconnect(peripheral: peripheral)
```

### 蓝牙重连

接口功能：在戒指连接断开时，重连设备。接口声明：

```java
        BluetoothDevice remote  = BluetoothAdapter.getDefaultAdapter().getRemoteDevice(mac);
                if(remote != null){
                    BLEUtils.connectLockByBLE(this,remote);
                }
```

参数说明：mac：戒指mac地址\
返回值：无

### 解除绑定

换绑戒指时，建议将之前戒指解除绑定，调用指令，清理掉历史数据，否则有可能出现A戴过的戒指，B去戴，造成B的数据里有A的数据，或者一个人戴多个戒指睡眠，最后睡眠数据重叠的情况

```java
 //断开连接
  BLEUtils.disconnectBLE(getSupportActivity());
//解除系统蓝牙绑定，防止下次搜索不到戒指
  BLEUtils.removeBond(BLEService.getmBluetoothDevice());
```
