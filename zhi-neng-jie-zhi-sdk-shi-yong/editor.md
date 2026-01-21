---
description: 本部分内容为使用sdk之前的准备工作
icon: square-code
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

# SDK移植

### sdk下载

[sdk地址](https://github.com/BravechipSpace/ChipletRing-APPSDK/releases)

sdk在releases里

### 环境要求

**安卓：**

* SDK库格式：aar
* 开发语言：JAVA
* Android系统版本：6.0以上
* 蓝牙版本：5.0以上

**IOS：**

* IOS系统版本：13.0以上
* 语言版本：Swift 5.0以上
* IDE版本：xcode 16以上
* 蓝牙版本：5.0以上

**注意事项：因Xcode不同版本中使用到的Swift编译器版本问题，可能会导致集成后编译错误，可以联系我们提供对应的SDK库**

### 库文件添加

```pod
# SDK用到的库
pod 'Foil', '~> 5.1.2'
pod 'NordicDFU'
pod 'RxSwift', '~> 6.9.0'
pod 'RxCocoa', '~> 6.9.0'
pod 'SwiftDate'
pod 'SwiftyBeaver', '1.9.5'
pod 'Zip'
```

**安卓：**

1.将./Android/library/aar文件，放在libs目录下。 2.目前只提供离线sdk，所以需要用户手动添加一些依赖，后期会做优化，版本号可以使用自己工程里的

```java
implementation 'androidx.appcompat:appcompat:1.3.1'
implementation 'org.greenrobot:greendao:3.3.0'
implementation 'androidx.localbroadcastmanager:localbroadcastmanager:1.1.0'
implementation 'org.ligboy.retrofit2:converter-fastjson-android:2.1.0'
implementation 'com.squareup.retrofit2:adapter-rxjava:2.3.0'
implementation 'org.jetbrains:annotations:15.0'
implementation  'com.google.code.gson:gson:2.11.0'
implementation 'com.zhy:okhttputils:2.6.2'
```

**IOS：**

1、手动集成：将./IOS/library/iOS\_New\_Framework/BCLRingSDK.framework文件夹拖入到Xcode的项目中，选择“Copy items if needed”选项。

2、Cocoapods集成：待支持

3、Swift Package Manager集成：待支持

### 权限设置

**安卓：**

配置所需权限（在程序中也需要动态检测权限），在Manifest.xml中加入以下代码:

```xml
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

**IOS：**

```xml
# Info.plist 增加蓝牙权限
Privacy - Bluetooth Peripheral Usage Description
Privacy - Bluetooth Always Usage Description
```

### SDK使用

**安卓：**

1.在Application的onCreate方法中进行初始化

<mark style="color:red;">建议新客户直接使用LmAPILite简化版本</mark>

```java
LmAPI.init(this);
LmAPI.setDebug(true);
//如使用简化版本，需要初始化LmAPILite
LmAPILite.init(this);
LmAPILite.setDebug(true);
//如果app需要提供给海外用户，可以设置这个为true，sdk会切换到新加坡服务器
AppConfig.setOverseas(true);
```

2.在BaseActivity类中启用监听，该监听用于监听蓝牙连接状态和戒指的应答&#x20;

```java
如果是一般模式，可以实现IResponseListener，是简化模式，就实现IResponseListenerLite
BaseActivity implements IResponseListener

一般模式：
在onResume里调用注册，先移除一下，防止弹出配对框，导致onResume重复调用，多次注册
LmAPI.removeWLSCmdListener(this);
LmAPI.addWLSCmdListener(this, this);
在onStop里清除监听
LmAPI.removeWLSCmdListener(this)
简化模式换成LmAPILite即可，后续调用指令都使用LmAPILite开头
```

**IOS：**

1、初始化相关配置

```swift
// 配置网络地址（国外地址：.overseas、国内地址：.domestic（默认））
// 如果是提供给国外用户使用，需要设置为国外地址
BCLRingManager.shared.networkRegion = .overseas

// 配置指令超时时间，默认5s，如果超过5s没有响应，则认为超时,部分固件超时时间可能更长，可以设置为8s,也可以不设置，使用默认值
BCLRingManager.shared.commandTimeout = 8

// 应用启动后可以检查蓝牙权限
BCLRingManager.shared.checkBluetoothPermission { auth in
    switch auth {
    case .allowedAlways:
        print("蓝牙权限：已授权")
    case .denied:
        print("蓝牙权限：未授权")
    case .notDetermined:
        print("蓝牙权限：未确定")
    case .restricted:
        print("蓝牙权限：受限")
    default:
        break
    }
}

```
