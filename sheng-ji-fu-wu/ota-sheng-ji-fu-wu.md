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

### IOS提供的接口
