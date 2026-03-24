---
description: 公版app重连逻辑：如果app在前台，蓝牙断开，延时重连。如果app退到后台，定时断连蓝牙，防止功耗高，app到前台时，进行重连，同步数据
icon: plug
---

# Android公版app的重连

## app连接蓝牙代码

以下是app连接和重连代码，是兼容了多种戒指（普通戒指直接连接，HID戒指是需要弹出绑定框，绑定到系统蓝牙），适配了多种情况，如果开发者戒指类型比较少，情况比较少，可以从中获取需要的代码。

### 适配情况

1.app在后台，或者戒指已连接，或者戒指正在连接，不需要重连

2.在app进行Ota结束以后，需要重连，直接走广播，获取新的配置

3.如果系统蓝牙已经有绑定并且是连接状态的戒指，直接连接，不需要扫描，如果是已保存未连接，则先解绑，再去直连

4.公版app的服务器保存了用户信息，保存了用户使用的戒指的类型，如果不是HID戒指，直接连接

5.如果是HID的戒指，并且系统蓝牙没有绑定，就需要扫描进行连接，因为直接连接，有可能连接不上

### 扫描过程中的逻辑

1.如果是phy固件ota升级过程中的状态，也就是蓝牙名称是PPlusOTA，进行直接连接。其他版本的固件可以忽略这个

2.如果不是ota升级进行的连接，扫描过程中，如果是HID模式，继续走扫描，防止出现用户在系统蓝牙内解绑设备，直连导致无法配对，如果不是，直接连接

3.如果扫描到的mac和用户信息里的mac一致，说明是这个用户的戒指，根据广播信息，判断是否是勇芯的戒指，如果是，直接连接，停止扫描，获取戒指信息，保存到本地和服务器

```java

boolean connecting = false;//是否正在连接
 boolean isReUpData;//是否Ota升级出错后的重连，针对phy固件
  private void executeBluetoothConnection() {
        connecting = false;
        handler.removeMessages(0);
        handler.sendEmptyMessageDelayed(0, timeOut);
        refreshLayout.setRefreshing(true);
        Logger.show("ConnectDevice", "mac :" + mac);

        if (StringUtils.isEmpty(mac)) {
            mac = PreferencesUtils.getString("address");
        }

        BluetoothDevice remote = BluetoothAdapter.getDefaultAdapter().getRemoteDevice(mac);

        if (BLEService.isGetToken()) {
            Logger.show("ConnectDevice", "蓝牙已连接");
            String status = getRsString(R.string.connecting);

            if (((MainActivity) getSupportActivity()) != null) {
                ((MainActivity) getSupportActivity()).aboutFragmentSetConnectStatus(status);
            }
            refreshLayout.setRefreshing(false);
        } else if (remote != null && (mac).equalsIgnoreCase(remote.getAddress())) {
            App.getInstance().setDeviceBean(new BleDeviceInfo(remote, -50));
            Set<BluetoothDevice> bondedDevices = BluetoothAdapter.getDefaultAdapter().getBondedDevices();

            if (App.getInstance().otaUpdate) {
                Logger.show("ConnectDevice", "ota升级过了，直接走广播，获取新的配置 ");
                BLEUtils.stopLeScan(getSupportActivity(), leScanCallback);
                BLEUtils.startLeScan(getSupportActivity(), leScanCallback);
            } else {
                //如果是HID戒指，先判断戒指在系统蓝牙内是否已经连接，如果连接，直接直连，如果是已保存未连接状态，先解绑，再重新配对绑定
                if (bondedDevices.contains(remote)) {
                    Logger.show("ConnectDevice", "系统蓝牙已经有绑定的戒指，直接连接 ");
                    BluetoothAdapter adapter = BluetoothAdapter.getDefaultAdapter();
                    Set<BluetoothDevice> pairedDevices = adapter.getBondedDevices();
                    BluetoothStatusChecker checker = new BluetoothStatusChecker(getSupportActivity());
                    for (BluetoothDevice device : pairedDevices) {
                        if (device.getAddress().equals(mac)) {
                            // 在后台线程中调用
                            new Thread(() -> {

                                // 状态检查
                                checker.printDeviceStatus(device);

                                boolean isConnected = checker.isDeviceConnected(device);

                                getSupportActivity().runOnUiThread(() -> {
                                    // 更新UI
                                    if (isConnected) {
                                        // 设备已连接
                                        Logger.show(TAG,"设备已连接");
                                    } else {
                                        // 设备已配对但未连接
                                        Logger.show(TAG,"设备已配对但未连接");
                                        try {
                                            SystemUtils.removeBond(App.getInstance().getDeviceBean().getDevice());
                                        } catch (Exception e) {
                                            e.printStackTrace();
                                        }
                                    }
                                    directConnection(remote);
                                });
                            }).start();

                        }
                    }

                } else {
                    String lastMac = PreferencesUtils.getString(Constants.Prefer.LAST_MAC);
                    UserInfo userInfo = App.getInstance().getUserInfo();
                    if (userInfo != null && getSupportActivity() != null) {
                        if (!"1".equals(userInfo.getBindingIndicatorBit() + "") && !connecting && !mac.equals(lastMac)) {
                            directConnection(remote);
                            Logger.show("不是HID的戒指", "直接连接");
                            return;
                        }
                    }
                    Logger.show("ConnectDevice", "蓝牙 startLeScan 连接");
                    BLEUtils.stopLeScan(getSupportActivity(), leScanCallback);
                    BLEUtils.startLeScan(getSupportActivity(), leScanCallback);
                }
            }
        } else {
            Logger.show("ConnectDevice", "蓝牙1 startLeScan 连接");
            BLEUtils.stopLeScan(getSupportActivity(), leScanCallback);
            BLEUtils.startLeScan(getSupportActivity(), leScanCallback);
        }
    }
    
    @SuppressLint("MissingPermission")
    private BluetoothAdapter.LeScanCallback leScanCallback = new BluetoothAdapter.LeScanCallback() {
        @Override
        public void onLeScan(BluetoothDevice device, int rssi, byte[] bytes) {
            if (device == null ||connecting) {//有戒指正在连接，不能再进行扫描，防止连接到另外的戒指

                return;
            }
            isReUpData = false;
            if (!StringUtils.isEmpty(device.getName())&&device.getName().contains("PPlusOTA")) {
                Logger.show("ConnectDevice", "PPlusOTA升级: "+device.getName()+",mac:"+device.getAddress());
                isReUpData = true;
                BLEUtils.stopLeScan(getSupportActivity(), leScanCallback);
                BLEUtils.connectLockByBLE(getSupportActivity(), device);
                connecting=true;
                App.getInstance().setDeviceBean(new BleDeviceInfo(device, rssi));
            }
           // Logger.show("ConnectDevice", "onLeScan");
            if (!App.getInstance().otaUpdate) {//如果不是ota升级进行的连接
                //扫描过程中，如果已经获取到用户信息，从云端拉取用户设备信息，如果是HID模式，继续走扫描，防止出现用户在系统蓝牙内解绑设备，直连导致无法配对，如果不是，直接连接
                UserInfo userInfo = App.getInstance().getUserInfo();
                if (userInfo != null && getSupportActivity() != null) {
                    if (!"1".equals(userInfo.getBindingIndicatorBit()+"") && !connecting) {
                        BluetoothDevice remote = BluetoothAdapter.getDefaultAdapter().getRemoteDevice(mac);
                        directConnection(remote);
                        Logger.show("connecting", "connecting");
                        return;
                    }
                }
            }
            if ((mac).equalsIgnoreCase(device.getAddress()) &&!BLEService.isGetToken()) {

                if (dataEntityList.contains(device)) {
                    return;
                }

                try {
                    if (getSupportActivity() == null) {
                        Log.i("getSupportActivity","null");
                        return;
                    }
                    //是否符合条件，符合条件，会返回戒指设备信息
                    BleDeviceInfo bleDeviceInfo = LogicalApi.getBleDeviceInfoWhenBleScan(device, rssi, bytes,false);
                    if (bleDeviceInfo == null) {
                        Log.i("bleDeviceInfo","null");
                        return;
                    }

                    App.getInstance().setDeviceBean(bleDeviceInfo);
                    connectDevice(bleDeviceInfo,device);
                    connecting = true;
                    dataEntityList.add(device);
                    Logger.show("ConnectDevice", "device:"+device.getAddress());
                    BLEUtils.stopLeScan(getSupportActivity(), leScanCallback);
                    BLEUtils.connectLockByBLE(getSupportActivity(), device);
                    App.getInstance().otaUpdate = false;
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    };


    private void connectDevice(BleDeviceInfo bleDeviceInfo,BluetoothDevice bluetoothDevice) {

        PreferencesUtils.setIsHidDevice(bleDeviceInfo.getBindingIndicatorBit()+"");
        BLEUtils.isHIDDevice=bleDeviceInfo.getBindingIndicatorBit()==1;
        PreferencesUtils.setThDirectives(bleDeviceInfo.getCommunicationProtocolVersion());
        //上传设备信息到服务器
        ((MainActivity) getSupportActivity()).getPresenter().uploadDeviceInfoWhenBind(bluetoothDevice.getAddress(),bluetoothDevice.getName(),bleDeviceInfo.getChargingIndicator(),bleDeviceInfo.getBindingIndicatorBit(),bleDeviceInfo.getCommunicationProtocolVersion());
    }
```

## app需要重连的情况

### 1.app在前台

戒指发生了断连，如指令超时，戒指放进充电仓，戒指拿远了，都可能发生断连，这时就需要在app上开启定时重连(mac地址不为空，并且戒指断连，戒指不是在连接中，用户信息存在，不是Ota以后的断连)

```java
  Handler handler = new Handler(new Handler.Callback() {
        @Override
        public boolean handleMessage(@NonNull Message msg) {
 
            if (msg.what == 101) {
                if (ActManager.getAppManager().currentActivity() != null ) {
                    String mac = PreferencesUtils.getString("address");
                    if (!TextUtils.isEmpty(mac) && !BLEUtils.isGetToken()&& !BLEUtils.isConnecting() && App.getInstance().getUserInfo() != null&&!App.getInstance().otaUpdate) {
                        Logger.show("TAG", "Handler  延迟重连  resetConnect 1111 ");
                        resetConnect();
                    }
                }
            }
            return false;
        }
    });
    
     /**
     * 执行重新连接
     */
    public void resetConnect() {
        if (!BLEUtils.isConnected()) {
            String address = PreferencesUtils.getString("address");
            if (!StringUtils.isEmpty(address)) {
                Logger.show("MainActivity", " 执行重新连接");
                connect(address);
            }
        }
    }
 
 @Override
    public void lmBleConnectionFailed(int i) {
        BLEUtils.setGetToken(false);
             Log.e("ConnectDevice", " 蓝牙 connectionFailed");
            handler.removeMessages(101);
        if(!App.isBackground){
            handler.sendEmptyMessageDelayed(101, 3000);
         }

        }
        
```

### 2.定时断连

如果已经设置了前台服务，会伴随App活跃情况，一直连接，可以设置定时断连，防止功耗过高，一般在Application里设置

{% content-ref url="android-qian-tai-fu-wu.md" %}
[android-qian-tai-fu-wu.md](android-qian-tai-fu-wu.md)
{% endcontent-ref %}

```java
 private Activity currerntActivity;

 public boolean disconnectByTimer = false;//蓝牙是否通过计时器断开连接的


private CountDownTimer countDownTimer= // 创建一个倒计时器，设定总时间为5分钟，每1秒回调一次
        new CountDownTimer(60000*5, 1000) {
            @Override
            public void onTick(long millisUntilFinished) {

            }

            @Override
            public void onFinish() {
                // 倒计时完成后的操作
                System.out.println("计时结束!");
                if(currerntActivity!=null){
                    if(BLEUtils.isConnected()){
                        disconnectByTimer=true;
                    }
                    BLEUtils.disconnectBLE(currerntActivity);
                }

            }
        };



 private void initBackgroundCallBack() {
        registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle bundle) {

            }

            @Override
            public void onActivityStarted(Activity activity) {
                Logger.show("App", " onActivityStarted");
                currerntActivity = activity;
                countActivity++;
                Logger.show("App", " isBackground" + isBackground);

                if (countActivity >= 1 && isBackground) {
                    isBackground = false;
                    if (appLifecycleListener != null) {
                        Logger.show("App", "  appLifecycleListener.foreground()");
                        try {
                            appLifecycleListener.foreground();
                            Logger.show("App", " foreground() called successfully");
                        } catch (Exception e) {
                            Logger.show("App", " Error calling foreground(): " + e.getMessage());
                            e.printStackTrace();
                        }
                    } else {
                        Logger.show("App", " appLifecycleListener is null");
                    }

                    handler.removeMessages(0x999);
                    try {
                        if (countDownTimer != null) {
                            countDownTimer.cancel();
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }

            }

            @Override
            public void onActivityResumed(Activity activity) {

            }

            @Override
            public void onActivityPaused(Activity activity) {

            }

            @Override
            public void onActivityStopped(Activity activity) {
                Logger.show("App", " onActivityStopped,isBackground:"+isBackground);
                countActivity--;
                if (countActivity <= 0 && !isBackground) {
                    isBackground = true;

                    if(appLifecycleListener!=null){
                        Logger.show("App", " appLifecycleListener.background()");
                        appLifecycleListener.background();
                    }
                    //说明应用进入了后台
                    if(BLEUtils.isGetToken()){
                       String isHid= PreferencesUtils.getISHidDevice();
                       if("1".equals(isHid)){//支持HID功能的戒指，退到后台后，10分钟后断开连接
                           countDownTimer=new CountDownTimer(60000*10,1000) {
                               @Override
                               public void onTick(long millisUntilFinished) {

                               }

                               @Override
                               public void onFinish() {
                                   if(currerntActivity!=null){
                                       BLEUtils.disconnectBLE(currerntActivity);
                                   }
                               }
                           };
                           try{
                               countDownTimer.start();
                           }catch (Exception e){
                               e.printStackTrace();
                           }
                       }else{
                           try{
                               countDownTimer.start();
                           }catch (Exception e){
                               e.printStackTrace();
                           }
                       }

                    }

                }

            }

            @Override
            public void onActivitySaveInstanceState(Activity activity, Bundle bundle) {

            }

            @Override
            public void onActivityDestroyed(Activity activity) {
                try{
                    countDownTimer.cancel();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        });
    }
```

### 3.app从后台到前台

如果已经断连，进行重连，这个可以做在设备页面或者主页面，取决于在哪里调用戒指指令

```java
App.getInstance().setAppLifecycleListener(new AppLifecycleListener() {
    @Override
    public void background() {
        Logger.show("background", "background");
        isBackgroundToForground = true;
    }

    @Override
    public void foreground() {
        Logger.show("foreground", "foreground");
        if(isBackgroundToForground){
            Logger.show("MainActivity", "APP后台到前台");

            if (BLEUtils.isGetToken()&&endConnectionCommand) {
                Logger.show("MainActivity", "刷新指令");
               //可以在此刷新未上传历史数据

            } else {
                Logger.show("MainActivity", "准备重连");
                if (colorFragment != null && !BLEUtils.isConnecting()) {
                    Logger.show("MainActivity", "去重连");
                    BLEUtils.setGetToken(false);
                    BLEUtils.disconnectBLE(MainActivity.this);
                    resetConnect();
                }
            }

        }


    }
});
```
