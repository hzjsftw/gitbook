---
description: 有些情况下，如果app处理不当，会导致待机很短
---

# 待机时间短

一般有以下情况，会导致待机时间很短

1.app一直连接着戒指，并且实时调用指令，包括退到后台的时候

2.HID戒指，即包含手势操作的戒指，这种戒指需要绑定到系统蓝牙里，会导致功耗过高，可以只使用手势的时候，才去开启配对，在适当时候，从系统蓝牙解绑戒指，android有代码可以操作，iOS需要页面进行提醒，让用户主动去解绑

给出android参考样例

```java
public class TestActivity3 extends BaseActivity implements IResponseListener,  View.OnClickListener {

    TextView tv_result2;
    private BluetoothDevice mBluetoothDevice;

    public final static String TAG = "TestActivity3";
    private BluetoothAdapter mBluetoothAdapter;
    private boolean autoConnect=true;//是否需要直接连接，在手动解绑断连的时候，有时候会触发取消配对的操作
    private Handler handler = new Handler(Looper.getMainLooper()) {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 0x99:

                    Logger.show("TestActivity3", "===执行指令超时===");

                    break;
                default:
                    break;
            }
        }
    };

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test3);
        tv_result2 = findViewById(R.id.tv_result2);
        findViewById(R.id.bt_jinxingpeidui).setOnClickListener(this);
        findViewById(R.id.bt_quxiaopeidui).setOnClickListener(this);
        findViewById(R.id.bt_lanyazhilian).setOnClickListener(this);
        findViewById(R.id.bt_duankailanya).setOnClickListener(this);
        mBluetoothDevice =   App.getInstance().getDeviceBean().getDevice();
        BLEUtils.isHIDDevice=false;//设置为不是配对戒指，防止sdk自动配对，进行手动进行配对操作
    }




    @Override
    public void onClick(View v) {

            if(v.getId()== R.id.bt_jinxingpeidui) {

                postView("进行配对\n");
                boolean isbonded = mBluetoothDevice.createBond();
                autoConnect = true;
                if (!isbonded) {//兼容判断，如果监测不能配对，启用老流程
                    Logger.show(TAG, "HID绑定不匹配，直接连接");
                    BLEUtils.connectLockByBLE(TestActivity3.this, mBluetoothDevice);
                } else {
                    Logger.show(TAG, "HID绑定操作");
                    // 用BroadcastReceiver来取得搜索结果
                    IntentFilter intentFilter = new IntentFilter();
                    intentFilter.addAction(BluetoothDevice.ACTION_FOUND);//搜索发现设备
                    intentFilter.addAction(BluetoothDevice.ACTION_BOND_STATE_CHANGED);//状态改变
                    intentFilter.addAction(BluetoothAdapter.ACTION_SCAN_MODE_CHANGED);//行动扫描模式改变了
                    intentFilter.addAction(BluetoothAdapter.ACTION_STATE_CHANGED);//动作状态发生了变化
                    intentFilter.addAction(BluetoothAdapter.ACTION_REQUEST_ENABLE);
                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
                        getApplicationContext().registerReceiver(searchDevices, intentFilter, RECEIVER_EXPORTED);
                    } else {
                        getApplicationContext().registerReceiver(searchDevices, intentFilter);
                    }

                }
            }
            if(v.getId()==R.id.bt_quxiaopeidui) {

                postView("取消配对\n");
                BLEUtils.removeBond(mBluetoothDevice);
                App.needAutoConnect = false;
            }

            if(v.getId()== R.id.bt_lanyazhilian) {
                postView("蓝牙直连\n");
                BLEUtils.isHIDDevice = false;
                BLEUtils.connectLockByBLE(TestActivity3.this, mBluetoothDevice);
            }
            if(v.getId()== R.id.bt_duankailanya) {
                postView("断开蓝牙\n");
                BLEUtils.removeBond(mBluetoothDevice);
                BLEUtils.disconnectBLE(TestActivity3.this);
                App.needAutoConnect = false;
                autoConnect = false;
            }
           
           

    }

    /**
     * 蓝牙接收广播
     */
    private BroadcastReceiver searchDevices = new BroadcastReceiver() {
        //接收
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            //状态改变时
            BLEUtils.setConnecting(false);

            if (BluetoothDevice.ACTION_BOND_STATE_CHANGED.equals(action)) {
                switch (mBluetoothDevice.getBondState()) {
                    case BluetoothDevice.BOND_BONDING://正在配对
                        Log.d("BLEService", "正在配对......===============");
                        postView("正在配对......===============\n");
                        break;
                    case BluetoothDevice.BOND_BONDED://配对结束
                        if (initializeBluetooth()) {
                            Log.d("BLEService", "连接成功    =======");
                            postView("连接成功    =======\n");
                            // sendStatusChange(CONNECT_STATE_SUCCESS);
                            BLEUtils.setMac(mBluetoothDevice.getAddress());
                        } else {
                            Log.d("BLEService", "连接失败    =======");
                            postView("连接失败    =======\n");
                            //   removeBond(mBluetoothDevice);
                        }

                        break;
                    case BluetoothDevice.BOND_NONE://取消配对/未配对
                        Log.d("BLEService", "取消配对    =======");
                        postView("取消配对    =======\n");
                        if(autoConnect){
                            postView("用户点击取消配对后直连    =======\n");
                            BLEUtils.connectLockByBLE(TestActivity3.this, mBluetoothDevice);
                        }

                    default:
                        Log.d("BLEService", "default    =======");
                        postView("default    =======\n");
                        // removeBond(mBluetoothDevice);
                        break;
                }
            }
        }
    };
    // 初始化蓝牙连接
    public boolean initializeBluetooth() {
        mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
        if (mBluetoothAdapter == null) {
            return false; // 设备不支持蓝牙
        }
//        Set<BluetoothDevice> bondedDevices = mBluetoothAdapter.getBondedDevices();

        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                BLEUtils.connectLockByBLE(TestActivity3.this, mBluetoothDevice);
            }
        }, 2000);


        return true;
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        App.needAutoConnect=true;
    }
    /**
     * @param value 打印的log
     */
    public void postView(String value) {
        tv_result2.setMovementMethod(ScrollingMovementMethod.getInstance());
        tv_result2.setScrollbarFadingEnabled(false);//滚动条一直显示
        tv_result2.append(value);
        int scrollAmount = tv_result2.getLayout().getLineTop(tv_result2.getLineCount()) - tv_result2.getHeight();
        if (scrollAmount > 0)
            tv_result2.scrollTo(0, scrollAmount);
        else
            tv_result2.scrollTo(0, 0);

    }

    @Override
    public void lmBleConnecting(int code) {
        postView("lmBleConnecting    =======\n");
        BLEUtils.setConnecting(true);
    }

    @Override
    public void lmBleConnectionSucceeded(int code) {
        postView("lmBleConnectionSucceeded    =======\n");

        BLEUtils.setConnecting(false);

            Logger.show(TAG,"code="+code);
            if (code == 7) {

                BLEUtils.setGetToken(true);

        }

    }

    @Override
    public void lmBleConnectionFailed(int code) {
        postView("lmBleConnectionFailed    =======\n");
        BLEUtils.setGetToken(false);
        BLEUtils.setConnecting(false);
    }
}

```

3.一般是在app退到后台以后，定时去断连戒指，在app到前台的时候，调用一下复合指令的刷新指令，会自动上传戒指的历史数据到app。可以参考

{% content-ref url="../zhi-neng-jie-zhi-sdk-shi-yong/images-and-media.md" %}
[images-and-media.md](../zhi-neng-jie-zhi-sdk-shi-yong/images-and-media.md)
{% endcontent-ref %}
