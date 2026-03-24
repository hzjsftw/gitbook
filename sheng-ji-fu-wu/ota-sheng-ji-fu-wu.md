---
description: >-
  蓝牙OTA（Over-the-Air）是一种通过蓝牙无线通信技术实现设备固件（Firmware）远程升级的技术。该服务支持从云端拉取最新的固件，需保证与戒指处于连接状态，建议rssi
  > -70并且电量>50。sdk会根据固件版本，自动选择对应的升级方式，包括阿波罗，phy，dfu模式
icon: bluetooth
---

# OTA升级服务

### android提供的接口

android根据需要，提供了三个接口

**OtaApi.otaUpdateWithCheckVersion**

&#x20;该接口包含了检查版本号version(调用 LmAPI.GET\_VERSION((byte) 0x00)获取或者是二代协议返回的版本号)，从云端拉取最新固件，自动升级功能，ota升级完成以后，要延时3s重连一下戒指

```java
OtaApi.otaUpdateWithCheckVersion(version, TestActivity.this, App.getInstance().getDeviceBean().getDevice(), App.getInstance().getDeviceBean().getRssi(), new LmOtaProgressListener() {
                    @Override
                    public void error(String message) {
                        postView("\nota升级出错："+message);
                    }

                    @Override
                    public void onProgress(int i) {
                      //  postView("\nota升级进度:"+i);
                        Logger.show("OTA","OTA升级"+i);
                    }

                    @Override
                    public void onComplete() {
                        postView("\nota升级结束");
                      
                    }

                    @Override
                    public void isLatestVersion() {
                        postView("\n已是最新版本");
                    }
                });
```

#### OtaApi.checkCurrentVersionNeedUpdate

检查当前硬件版本是否需要更新，用于在页面上需要显示更新信息的需求

```java
   OtaApi.checkCurrentVersionNeedUpdate(version, TestActivity.this, new ICheckOtaVersion() {
                    @Override
                    public void checkVersionResult(boolean needUpdate) {
                        
                    }
                });
```

#### OtaApi.otaUpdateWithVersion

在调用OtaApi.checkCurrentVersionNeedUpdate后，需要更新固件时调用，去掉了检查是否更新的步骤

```java
 OtaApi.otaUpdateWithVersion(version, App.getInstance().getDeviceBean().getDevice(), App.getInstance().getDeviceBean().getRssi(), new LmOtaProgressListener() {
                    @Override
                    public void error(String message) {
                        
                    }

                    @Override
                    public void onProgress(int i) {

                    }

                    @Override
                    public void onComplete() {

                    }

                    @Override
                    public void isLatestVersion() {

                    }
                });
```

对应的回调

```java
public interface LmOtaProgressListener {
        /**
         * 更新出错
         * @param message
         */
        void error(String message);

        /**
         * 更新进度
         * @param i
         */
        void onProgress(int i);

        /**
         * 更新完成
         */
        void onComplete();

        /**
         * 已经是最新版本
         */
        void isLatestVersion();
}
public interface ICheckOtaVersion {
    /**
     * 是否需要更新
     * @param needUpdate 是否需要更新
     * @param version 新固件的版本号
     */
    void checkVersionResult(boolean needUpdate,String version);
}

```

#### **phy模式下升级异常的处理**

戒指是phy固件的情况下，如果升级失败，戒指名字会变成PPlusOTA，mac地址最后一位会加1，如果去搜索连接，因为和sdk里边的特征值是不同的，导致无法连接上。这种情况下，如果扫描到的设备是PPlusOTA，应该继续进行固件升级操作，而不是进行sdk连接蓝牙的操作。搜索的时候，应该PPlusOTA和sdk提供的 `LogicalApi.getBleDeviceInfoWhenBleScan(device, rssi, bytes,false);`结合，防止显示不出来PPlusOTA名字的设备

升级完成以后，刷新一下蓝牙搜索列表，然后再进行sdk的连接。

这里边需要涉及到版本号的保存问题，因为固件升级中断以后，无法正常连接，就无法获取到固件的版本号，需要想办法保存上一次升级时的固件，这个建议保存到服务器，分别保存设备名称，mac地址，软件版本号，固件版本号，这样，如果搜索到PPlusOTA，就将mac地址最后一位减1，和数据库里的匹配，获取软件版本号，这个就涉及到设备信息的保存，比较复杂一些。

简单点的是把升级失败的版本号保存在app里，这个应该在升级前保存，因为有可能用户关掉蓝牙，或者杀死APP，或者固件问题，导致没有错误回调，保存在本地的问题就是，如果切换过戒指，或者多个升级错误的戒指，或者杀掉APP，没办法根据mac对应软件版本号，下一次无法升级到正确版本。

这个版本号怎么对应到戒指，是需要根据各自公司的情况，来决定用哪种方式。总的思路就是根据mac获取到当时戒指的软件版本号，然后去服务端拉取固件，进行升级 提供一些参考代码，我这边把PPlusOTA的mac地址减1获取到正确的版本号，然后通过ota进行升级

```java
 /**
     * phy模式下的戒指升级失败，mac地址会减一，返回之前正确的mac，去后台查询对应的软件版本号，
     * 我用的这个方法otaUpdateWithVersion，是结合我这边app的需求，客户可以使用已经对接的ota升级方式
     * @return
     */
    private String pplusOtaMacMinusOne(String macAddress){
        // 将 MAC 地址的最后一位提取出来
        String[] parts = macAddress.split(":");
        String lastPart = parts[5];  // 获取最后一位 C9

        // 将十六进制字符串转换为十进制数
        int lastPartDecimal = Integer.parseInt(lastPart, 16);

        // 将其减去 1
        lastPartDecimal -= 1;

        // 如果减去 1 后变为负数，需要处理溢出情况
        if (lastPartDecimal < 0) {
            lastPartDecimal = 255;  // 例如，如果是 00，减去 1 就变成 FF
        }

        // 将十进制数转换回十六进制并格式化为两位大写字母
        String newLastPart = String.format("%02X", lastPartDecimal);

        // 将最后一位替换为新的值
        parts[5] = newLastPart;

        // 重新拼接成新的 MAC 地址
        return String.join(":", parts);
    }

 private void checkPhyOtaUpdate(){

        SearchModel.getDeviceByMac(pplusOtaMacMinusOne(deviceBean.getDevice().getAddress())).subscribe(new BaseSubscriber(SearchActivity.this,true) {
            @Override
            public void onSuccess(String data) {
                Type listType = new TypeToken<LomoRingDevice>() {
                }.getType();
                LomoRingDevice lomoRingDevice = App.getInstance().getGson().fromJson(data, listType);

                OtaApi.checkVersion(lomoRingDevice.getSoftwareVersion(), new VersionCallback() {
                    @Override
                    public void success(String newVersion) {

                        if (!StringUtils.isEmpty(newVersion)) {
                            updateDevicePopup = new XPopup.Builder(SearchActivity.this)
                                    .isDestroyOnDismiss(true) 
                                    .popupAnimation(PopupAnimation.TranslateFromTop)
                                    .asCustom(new NotificationMsgPopup(SearchActivity.this, getRsString(R.string.OTAinit))).show();

                            runOnUiThread(new Runnable() {
                                @Override
                                public void run() {
                                    phyOta(newVersion);
                                }
                            });
                        }
                        dismissProgressDialog();
                    }

                    @Override
                    public void error() {
                        dismissProgressDialog();
                    }
                }, SearchActivity.this);

            }

            @Override
            public void onFail(int code, String message) {
                super.onFail(code, message);
                dismissProgressDialog();
            }
        });




    }

    private void phyOta(String newVersion){
        mSimpleLoadingDialog = new SimpleLoadingDialog(this);
        OtaApi.otaUpdateWithVersion(newVersion, deviceBean.getDevice(), deviceBean.getRssi(),SearchActivity.this ,new LmOtaProgressListener() {
            @Override
            public void error(String message) {
                ToastUtils.show(message);
            }

            @Override
            public void onProgress(int i) {
                App.isUpdating = true;
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        mSimpleLoadingDialog.showFirst(getString(R.string.update_ing) + i + "%");
                    }
                });


            }

            @Override
            public void onComplete() {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        search(true);//重新扫描
                        mSimpleLoadingDialog.dismiss();
                        updateDevicePopup.dismiss();
                    }
                });

            }

            @Override
            public void isLatestVersion() {

            }
        });
    }
```

**意外杀死app后的恢复**

升级过程中，如果意外杀死app，再启动app的时候，需要按照广播形式进行，搜索到名称是pplusOTA的时候，直接连接，在连接过程中，设置sdk保存的ota文件地址，进行升级

<pre class="language-java"><code class="lang-java">boolean isReUpData;
<strong>public void lmBleConnecting(int type) {
</strong>    
    if (isReUpData &#x26;&#x26; type == 2) {
        BLEUtils.setUpgrade(true);

        getSupportActivity().runOnUiThread(new Runnable() {
            @Override
            public void run() {
                ((BaseActivity) getContext()).setMessage(getString(R.string.start_update));
            }
        });
        String filePath = PreferencesUtils.getString(Constants.Prefer.DOWN_VERSION_FILE);
        OtaApi.setUpdateFile(filePath);
        OtaApi.startUpdate(App.getInstance().getDeviceBean().getDevice(), App.getInstance().getDeviceBean().getRssi(), new LmOTACallback() {
            @Override
            public void onDeviceStateChange(int i) {

            }

            @Override
            public void onProgress(int i, int i1) {

                getSupportActivity().runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        if (i1 >= 100) {
                            isReUpData = false;
                        }
                        ((BaseActivity) getContext()).setMessage(getString(R.string.update_ing) + i1 + "%");
                    }
                });
            }

            @Override
            public void onComplete() {
                getSupportActivity().runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        ((BaseActivity) getContext()).dismissProgressDialog();
                    }
                });
            }
        });
    }
}
</code></pre>

```java
String filePath = PreferencesUtils.getString(Constants.Prefer.DOWN_VERSION_FILE);
```

这个代码是获取ota保存的上一次未升级完的的ota文件地址，可以设置一下，继续升级

sdk提供了获取上一次未升级成功的戒指的mac，在连接的时候，如果是这个mac，默认直接走扫描广播

```java

 //固件升级的mac地址，防止固件中断后，不能进入扫描，导致无法升级
 String lastMac = PreferencesUtils.getString(Constants.Prefer.LAST_MAC);
 UserInfo userInfo = App.getInstance().getUserInfo();
 //这块可以是自己的逻辑
 if (userInfo != null && getSupportActivity() != null) {
        if (!"1".equals(userInfo.getBindingIndicatorBit()+"") && !connecting&&!mac.equals(lastMac)) {
        directConnection(remote);
        Logger.show("不是HID的戒指", "直接连接");
         return;
    }
 }
 Logger.show("ConnectDevice", " 蓝牙 startLeScan 连接   ");
 BLEUtils.stopLeScan(getSupportActivity(), leScanCallback);
 BLEUtils.startLeScan(getSupportActivity(), leScanCallback);
```

扫描广播的代码

```java
boolean connecting = false;

@SuppressLint("MissingPermission")
private BluetoothAdapter.LeScanCallback leScanCallback = new BluetoothAdapter.LeScanCallback() {
    @Override
    public void onLeScan(BluetoothDevice device, int rssi, byte[] bytes) {
        if (device == null || StringUtils.isEmpty(device.getName())||connecting) {//有戒指正在连接，不能再进行扫描，防止连接到另外的戒指

            return;
        }
        isReUpData = false;
        if (device.getName().contains("PPlusOTA")) {
            Logger.show("ConnectDevice", "PPlusOTA升级: "+device.getName()+",mac:"+device.getAddress());
            isReUpData = true;
            BLEUtils.stopLeScan(getSupportActivity(), leScanCallback);
            BLEUtils.connectLockByBLE(getSupportActivity(), device);
            connecting=true;
            App.getInstance().setDeviceBean(new BleDeviceInfo(device, rssi));
        }
     
    }
};
```

### IOS_OTA升级说明
1、首先，通过SDK的接口调用，检查当前固价版本是否为最新的。

2、如果不是最新版本，接口会返回最新的部件版本号、文件名和下载地址等信息。

3、然后，使用SDK提供的下载接口，从云端下载最新的固件文件

4、可以通过SDK提供的接口来检查固件升级的类型。（Nordic、Apollo、Phy）

5、根据对应的固件升级类型，然后执行相应的提供的固件升级接口，进行固件升级。

**iOS:固件版本更新检查**
```Swift
/// 固件版本更新检查
/// - Parameters:
///   - version: 当前固件版本号
/// - Parameter completion: 固件版本更新检查回调
func checkFirmwareUpdate(version: String, completion: @escaping (Result<BCLFirmwareVersionInfo, BCLError>) -> Void)
```
### 调用示例
```Swift
BCLRingManager.shared.checkFirmwareUpdate(version: "5.5.1.6Z2Y") { result in
    switch result {
    case let .success(versionInfo):
        // 有新版本
        if versionInfo.hasNewVersion {
            BDLogger.info("""
            ✅ 发现新版本：
            - 版本号：\(versionInfo.version ?? "")
            - 下载地址：\(versionInfo.downloadUrl ?? "")
            - 文件名：\(versionInfo.fileName ?? "")
            """)
            // 这里可以调用下载固件接口，下载最新固件，然后进行升级
        } else {// 已是最新版本
            BDLogger.info("✅ 当前已是最新版本")
        }
        BDLogger.info("📝 消息：\(String(describing: versionInfo.version))")
    case let .failure(error):
        switch error {
        case let .network(.invalidParameters(message)):
            BDLogger.error("❌ 参数无效，请检查版本号格式: \(message)")
        case let .network(.httpError(code)):
            BDLogger.error("❌ HTTP请求失败：状态码 \(code)")
        case let .network(.serverError(code, message)):
            BDLogger.error("❌ 服务器错误：[\(code)] \(message)")
        case .network(.invalidResponse):
            BDLogger.error("❌ 响应数据无效")
        case let .network(.decodingError(error)):
            BDLogger.error("❌ 数据解析失败：\(error.localizedDescription)")
        case let .network(.networkError(message)):
            BDLogger.error("❌ 网络错误：\(message)")
        case let .network(.tokenError(message)):
            BDLogger.error("❌ Token异常：\(message)")
        default:
            BDLogger.error("❌ 其他错误：\(error)")
        }
    }
}
```

**iOS:固件文件下载**
```Swift
/// 固件下载
/// - Parameters:
///   - url: 固件下载地址
///   - fileName: 固件文件名
///   - destinationPath: 固件文件保存路径
///   - progress: 固件下载进度回调
/// - Parameter completion: 固件下载回调
func downloadFirmware(url: String, fileName: String, destinationPath: String, progress: @escaping (Double) -> Void, completion: @escaping (Result<String, BCLError>) -> Void)
```
### 调用示例
```Swift
let fileName = "2.7.4.8Z27.hex16"
let downloadUrl = "http://221.226.159.58:22222/profile/upload/2025/04/01/2.7.4.8Z27.hex16"
//  固件下载保存路径
let destinationPath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).first!
BCLRingManager.shared.downloadFirmware(url: downloadUrl, fileName: fileName, destinationPath: destinationPath, progress: { progress in
    BDLogger.info("固件下载进度：\(progress)")
}, completion: { result in
    switch result {
    case let .success(filePath):
        BDLogger.info("固件下载成功：\(filePath)")
    case let .failure(error):
        BDLogger.error("固件下载失败：\(error)")
    }
})
```

**iOS:固件升级类型获取**
```Swift
/// 获取固件升级类型
/// - Parameters:
///   - firmwareVersion: 固件版本号()
///   - completion: 获取结果回调
func getOTAType(firmwareVersion: String?, completion: @escaping (BCLOTAType) -> Void)
```
### 调用示例
```Swift
BCLRingManager.shared.getOTAType(firmwareVersion: "5.5.1.6Z2Y") { response in
    BDLogger.info("固件升级类型:\(response.rawValue)")
    switch response.rawValue {
    case 0:
        BDLogger.error("固件升级类型: 未知")
        break
    case 1:
        BDLogger.info("固件升级类型: Apollo")
        // Apollo固件升级 查看以下方法
//                    func apolloUpgradeFirmware(filePath: String, progressHandler: ((Float) -> Void)? = nil, completion: @escaping (Result<Void, BCLError>) -> Void)
        break
    case 2:
        BDLogger.info("固件升级类型: Nordic")
        // Nordic固件升级 查看以下方法
//                    func nrfUpgradeFirmware(filePath: String, fileName: String, progressHandler: ((Int) -> Void)? = nil, completion: @escaping (Result<BCLNrfUpgradeState.Stage, BCLError>) -> Void)
        break
    case 3:
        BDLogger.info("固件升级类型: Phy")
        // Phy固件升级 查看以下方法
//                    func phyUpgradeFirmware(filePath: String, progressHandler: ((Double) -> Void)? = nil, completion: @escaping (Result<BCLPhyUpgradeState, BCLError>) -> Void)
        break
    default:
        break
    }
}
```

**iOS:Apollo固件升级**
```Swift
/// Apollo 固件升级接口
/// - Parameters:
///   - filePath: 固件文件路径
///   - progressHandler: 进度回调，返回 0-100 的进度值
///   - completion: 完成回调，返回成功或失败
func apolloUpgradeFirmware(filePath: String, progressHandler: ((Float) -> Void)? = nil, completion: @escaping (Result<Void, BCLError>) -> Void)
```
### 调用示例
```Swift
let fileurl = URL(fileURLWithPath: "固件文件保存路径")
BCLRingManager.shared.apolloUpgradeFirmware(filePath: fileurl.path, progressHandler: { [weak self] progress in
    BDLogger.info("Apollo升级进度：\(progress)%")
    self?.showLoading("升级进度：\(progress)%")
},
completion: { [weak self] result in
    self?.hideLoading()
    switch result {
    case .success:
        BDLogger.info("Apollo固件升级成功")
        self?.showSuccess("Apollo固件升级成功")
    case let .failure(error):
        BDLogger.error("Apollo固件升级失败：\(error)")
        self?.showError("升级失败：\(error.localizedDescription)")
    }
}
)
```

**iOS:Nordic固件升级**
```Swift
/// Nordic 固件升级接口
/// - Parameters:
///   - filePath: 固件文件路径
///   - progressHandler: 进度回调，返回 0-100 的进度值
///   - completion: 完成回调，返回成功或失败
func nrfUpgradeFirmware(filePath: String, fileName: String, progressHandler: ((Int) -> Void)? = nil, completion: @escaping (Result<BCLNrfUpgradeState.Stage, BCLError>) -> Void)
```
### 调用示例
```Swift
let fileName = ""
let fileURL = URL(fileURLWithPath: "固件文件保存路径")
// 开始Nordic固件升级会自动断开连接后并重新连接戒指
BCLRingManager.shared.nrfUpgradeFirmware(filePath: fileURL.path, fileName: fileName) { [weak self] progress in
    BDLogger.info("Nordic升级进度：\(progress)%")
    self?.showLoading("升级进度：\(progress)%")
} completion: { [weak self] res in
    switch res {
    case let .success(state):
        if state == .rebooting {
            BDLogger.info("Nordic固件升级-设备重启中")
            self?.showLoading("设备重启中")
        } else if state == .completed {
            BDLogger.info("Nordic固件升级成功")
            self?.hideLoading()
            self?.showSuccess("Nordic固件升级成功")
        }
    case let .failure(error):
        BDLogger.error("Nordic固件升级失败：\(error)")
        self?.hideLoading()
        self?.showError("升级失败：\(error.localizedDescription)")
    }
}
```

**iOS:Phy固件升级**
```Swift
/// Phy 固件升级接口
/// - Parameters:
///   - filePath: 固件文件路径
///   - progressHandler: 进度回调，返回 0-100 的进度值
///   - completion: 完成回调，返回成功或失败
func phyUpgradeFirmware(filePath: String, progressHandler: @escaping (Double) -> Void, completion: @escaping (Result<BCLPhyUpgradeState, BCLError>) -> Void)
```
### 调用示例
```Swift
// 如果开启了自动重连，需要先关掉
BCLRingManager.shared.isAutoReconnectEnabled = false
let fileURL = URL(fileURLWithPath: "固件文件保存路径")
BCLRingManager.shared.phyUpgradeFirmware(filePath: fileURL.path) { [weak self] progress in
    let progressPercent = Int(progress * 100)
    BDLogger.info("Phy升级进度：\(progressPercent)%")
    self?.showLoading("升级进度：\(progressPercent)%")
} completion: { [weak self] res in
    self?.hideLoading()
    switch res {
    case let .success(state):
        BDLogger.info("Phy固件升级成功：\(state)")
        self?.showSuccess("Phy固件升级成功")
        // 升级成功后，需要重新连接戒指，根据需要决定是否开启自动重连功能（注意此处）
        //  BCLRingManager.shared.startConnect(macAddress: deviceMacAddress, isAutoReconnect: true)
    case let .failure(error):
        BDLogger.error("Phy固件升级失败：\(error)")
        self?.showError("升级失败：\(error.localizedDescription)")
        // 升级失败后，可以尝试使用 PHY Boot 模式固件升级接口进行恢复升级
    }
}
```

**iOS:PHY Boot模式固件升级**
```Swift
/// PHY Boot模式固件升级接口
/// 用于处理PHY固件升级过程中断导致设备卡在boot模式的情况
/// - Parameters:
///   - filePath: 固件文件路径
///   - device: Boot模式设备信息
///   - peripheral: Boot模式外设对象
///   - progressHandler: 升级进度回调，返回 0-100 的进度值
///   - completion: 升级状态回调
func phyBootModeUpgrade(filePath: String, device: BCLDeviceInfoModel, peripheral: CBPeripheral, progressHandler: @escaping (Double) -> Void, completion: @escaping (Result<BCLPhyBootModeUpgradeState, BCLError>) -> Void)
```
### 调用示例
```Swift
// 📢注意：因phy固件戒指升级中断、失败会导致戒指进入boot模式，所以需要通过蓝牙搜索找到该戒指并进行连接后再执行phyBootModeUpgrade进行固件升级
BCLRingManager.shared.stopScan()
BCLRingManager.shared.startScan { [weak self] result in
    guard let self = self else { return }
    switch result {
    case let .success(deviceList):
        for deviceInfo in deviceList {
          //  扫描到PHY Boot模式设备
          if deviceInfo.isphyBootMode {
              BDLogger.info("扫描到PHY Boot模式设备：\(deviceInfo.name ?? "")，mac：\(deviceInfo.macAddress ?? "")")
              // 停止扫描
              BCLRingManager.shared.stopScan()
              AppRingManager.shared.connectBluetoothDevice(deviceInfo: deviceInfo) { [weak self] status, _ in
                // 通过deviceInfo.isPhyBootMode判断戒指是否处于phyBoot模式
                //  处理phyBoot模式固件升级
                guard !deviceInfo.isPhyBootMode else {
                    // 📢注意：此处在连接成功之后需要延迟1s进行Phy固件的boot模式固件升级
                    DispatchQueue.main.asyncAfter(deadline: .now() + 1) { [weak self] in
                        // 如果开启了自动重连，需要先关掉
                        BCLRingManager.shared.isAutoReconnectEnabled = false
                        let fileURL = URL(fileURLWithPath: "固件文件保存路径")
                        let currentDevice = BCLRingManager.shared.currentConnectedDevice
                        BCLRingManager.shared.phyBootModeUpgrade(filePath: fileURL.path, device: currentDevice, peripheral: currentDevice.peripheral) { [weak self] progress in
                            BDLogger.info("PHY Boot Mode升级进度：\(Int(progress))%")
                            self?.showLoading("升级进度：\(Int(progress))%")
                        } completion: { [weak self] res in
                            self?.hideLoading()
                            switch res {
                            case let .success(response):
                                BDLogger.info("PHY Boot Mode升级成功：\(response)")
                                self?.showSuccess("PHY Boot Mode固件升级成功")
                                // 升级成功后，需要重新连接戒指，根据需要决定是否开启自动重连功能（注意此处）
                                //  BCLRingManager.shared.startConnect(macAddress: deviceMacAddress, isAutoReconnect: true)
                            case let .failure(error):
                                BDLogger.error("PHY Boot Mode升级失败：\(error)")
                                self?.showError("升级失败：\(error.localizedDescription)")
                            }
                        }
                    }
                  return
                }
              }
           }
        }
    case let .failure(error):
        BDLogger.error("扫描设备失败：\(error)")
    }
}
```


