---
description: 常规指令是属于戒指的一些常规操作，用户可以选择性使用
icon: map
---

# 常规指令

**注：无特殊标记的情况下，本SDK中返回的值皆为小端模式**

提供一些转换方法：

```java
 static final char hexDigits[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};

    /**
     * byteArr转hexString
     * <p>例如：</p>
     * bytes2HexString(new byte[] { 0, (byte) 0xa8 }) returns 00A8
     *
     * @param bytes byte数组
     * @return 16进制大写字符串
     */
    public static String bytes2HexString(byte[] bytes) {
        char[] res = new char[bytes.length << 1];
        for (int i = 0, j = 0; i < bytes.length; i++) {
            res[j++] = hexDigits[bytes[i] >>> 4 & 0x0f];
            res[j++] = hexDigits[bytes[i] & 0x0f];
        }
        return new String(res);
    }

    /**
     * hexString转byteArr
     * <p>例如：</p>
     * hexString2Bytes("00A8") returns { 0, (byte) 0xA8 }
     *
     * @param hexString 十六进制字符串
     * @return 字节数组
     */
    public static byte[] hexString2Bytes(String hexString) {
        int len = hexString.length();
        if (len % 2 != 0) {
            throw new IllegalArgumentException("长度不是偶数");
        }
        char[] hexBytes = hexString.toUpperCase().toCharArray();
        byte[] res = new byte[len >>> 1];
        for (int i = 0; i < len; i += 2) {
            res[i >> 1] = (byte) (hex2Dec(hexBytes[i]) << 4 | hex2Dec(hexBytes[i + 1]));
        }
        return res;
    }

    /**
     * hexChar转int
     *
     * @param hexChar hex单个字节
     * @return 0..15
     */
    private static int hex2Dec(char hexChar) {
        if (hexChar >= '0' && hexChar <= '9') {
            return hexChar - '0';
        } else if (hexChar >= 'A' && hexChar <= 'F') {
            return hexChar - 'A' + 10;
        } else {
            throw new IllegalArgumentException();
        }
    }
      /**
     * 把一个整形改为8位的byte数组
     *
     * @param value
     * @return
     * @throws Exception
     */
    public static byte[] longTo8Bytes(long value) {
        byte[] result = new byte[8];
        result[7] = (byte) ((value >>> 56) & 0xFF);
        result[6] = (byte) ((value >>> 48) & 0xFF);
        result[5] = (byte) ((value >>> 40) & 0xFF);
        result[4] = (byte) ((value >>> 32) & 0xFF);
        result[3] = (byte) ((value >>> 24) & 0xFF);
        result[2] = (byte) ((value >>> 16) & 0xFF);
        result[1] = (byte) ((value >>> 8) & 0xFF);
        result[0] = (byte) (value & 0xFF);
        return result;
    }

    /**
     * 把一个整形改为4位的byte数组
     *
     * @param value
     * @return
     * @throws Exception
     */
    public static byte[] longTo4Bytes(long value) {
        byte[] byteArray = new byte[4];  // 创建一个4字节数组（4字节表示一个时间戳）

        // 逐字节将时间戳的低位到高位赋值给byte数组
        for (int i = 0; i < 4; i++) {
            byteArray[i] = (byte) (value & 0xFF);
            value >>= 8;  // 右移8位，获取下一个字节
        }

        return byteArray;
    }
        /**
     * 将 long 转换为小端字节数组
     * @param value
     * @return
     */
    public static byte[] longToBytesLittleEndian(long value) {
        byte[] bytes = new byte[8];  // long 是 8 字节
        for (int i = 0; i < 8; i++) {
            bytes[i] = (byte) (value >>> (i * 8));  // 获取每个字节
        }
        return bytes;
    }
    
     /**
     * 把一个整形改为4位的byte数组
     *
     * @param value
     * @return
     * @throws Exception
     */
    public static byte[] integerTo4Bytes(int value) {
        byte[] result = new byte[4];
        result[0] = (byte) ((value >>> 24) & 0xFF);
        result[1] = (byte) ((value >>> 16) & 0xFF);
        result[2] = (byte) ((value >>> 8) & 0xFF);
        result[3] = (byte) (value & 0xFF);
        return result;
    }

    /**
     * int转换为byte[]
     * @param number
     * @return
     */
    public static byte[] intToByteArray(int number) {
        ByteBuffer buffer = ByteBuffer.allocate(4);  // 分配一个长度为4的字节缓冲区
        buffer.putInt(number);  // 将整数放入缓冲区
        return buffer.array();  // 返回byte数组
    }
    
      /**
     * byte[]转换为long
     * @param buffer
     * @return
     */
    public static long BytesToLong(byte[] buffer) {
        long  values = 0;
        for (int i = buffer.length-1; i >= 0; i--) {
            values <<= 8; values|= (buffer[i] & 0xff);
        }
        return values;
    }

    public static long fourBytesToLongLittleEndian(byte[] buffer) {

        return (buffer[0] & 0xFFL) |
                ((buffer[1] & 0xFFL) << 8) |
                ((buffer[2] & 0xFFL) << 16) |
                ((buffer[3] & 0xFFL) << 24);
    }

    public static int BytesToInt(byte[] buffer) {
        int  values = 0;
        for (int i = buffer.length-1; i >=0 ; i--) {
            values <<= 8; values|= (buffer[i] & 0xff);
        }
        return values;
    }
```

### 同步时间

接口功能：调用此接口会获取手机当前时间同步给戒指。

接口声明：

**android:**

<pre class="language-java"><code class="lang-java"><strong>//默认是当前时区
</strong><strong>LmAPI.SYNC_TIME();
</strong>//同步时间可以设置时区， 东区为正，西区为负，比如东八区8，西八区为-8
LmAPI.SYNC_TIME_ZONE();
</code></pre>

注意事项：同步时间和读取时间共用一个返回值。 参数说明：无\
返回值

```java
void syncTime(byte datum,byte[] time)
```

| 参数名称  | 类型      | 示例值  | 说明              |
| ----- | ------- | ---- | --------------- |
| datum | byte    | 0或1  | 0代表同步成功 1代表读取时间 |
| time  | byte\[] | null | 同步时间不会返回byte\[] |

简化版本

```java
public static void SYNC_TIME(ISyncTimeListenerLite listenerLite)

public interface ISyncTimeListenerLite {

    void syncTime(boolean updateTime,long timeStamp);
}
```

**iOS:**
```Swift
/// 同步时间
/// - Parameters:
///   - date: 时间戳（默认当前时间）
///   - timeZone: 时区（默认东八区）
///   - completion: 同步时间回调
func syncTime(date: Date = Date(),
                timeZone: BCLRingTimeZone = .East8,
                completion: @escaping (Result<BCLSyncTimeResponse, BCLError>) -> Void)

/// 获取当前系统时区对应的RingTimeZone
/// - Returns: 当前系统时区对应的RingTimeZone
BCLRingTimeZone.getCurrentSystemTimeZone()
```
#### 调用示例
```Swift
// 同步时间
BCLRingManager.shared.syncTime(date: Date(), timeZone: BCLRingTimeZone.getCurrentSystemTimeZone()) { (res: Result<BCLSyncTimeResponse, BCLError>) in
    switch result {
    case .success(let response):
        print("同步时间成功: \(response)")
    case .failure(let error):
        print("同步时间失败: \(error)")
    }
}
```

### 读取时间

接口功能：调用此接口会获取戒指当前时间。一般情况下用不到。

**android：**

接口声明：\
注意事项：同步时间和读取时间共用一个返回值。 参数说明：无

```java
LmAPI.READ_TIME();
```

返回值

```java
void syncTime(byte datum,byte[] time)
```

| 参数名称  | 类型      | 示例值                                                  | 说明                           |
| ----- | ------- | ---------------------------------------------------- | ---------------------------- |
| datum | byte    | 0或1                                                  | 0代表同步成功 1代表读取时间              |
| time  | byte\[] | \[48, -23, -1, 83, -111, 1, 0, 0, 8] = 1723691166000 | 读取时间成功，需转化为时间戳(小端模式，最后一位为时区) |



简化版本

```java
 public static void READ_TIME(ISyncTimeListenerLite listenerLite) 

public interface ISyncTimeListenerLite {
/**
     * 同步时间
     * @param updateTime true是同步时间，false是读取时间
     * @param timeStamp 读取操作获取的戒指的时间戳
     */
    void syncTime(boolean updateTime,long timeStamp);
}
```

byte\[]转long的转换方法:

```java
public static long bytesToLong(byte[] buffer) {
    long  values = 0;
    for (int i = buffer.length-1; i >= 0; i--) {
        values <<= 8; values|= (buffer[i] & 0xff);
    }
    return values;
}
```

**iOS:**
```Swift
/// 读取时间
/// - Parameter completion: 读取时间回调
/// - Result: 读取结果
/// - BCLReadTimeResponse: 包含时间和时区信息的响应模型
/// - BCLError: 错误信息
func readTime(completion: @escaping (Result<BCLReadTimeResponse, BCLError>) -> Void)
```
#### 调用示例
```Swift
/// 读取时间
BCLRingManager.shared.readTime { res in
    switch res {
    case let .success(response):
        BDLogger.info("timeStamp: \(response.timestamp)")
        BDLogger.info("timeZone: \(response.ringTimeZone)")
        BDLogger.info("utcDate: \(response.utcDate)")
        BDLogger.info("localDate: \(response.localDate)")
    case let .failure(error):
        BDLogger.error("读取时间失败: \(error)")
    }
}
```
### 版本信息

接口功能：版本信息 ，获取戒指的版本信息。

**android:**

接口声明：

```java
LmAPI.GET_VERSION((byte) 0x00);  //0x00获取软件版本，0x01获取硬件版本
```

参数说明：type：0x00获取软件版本 ，0x01获取硬件版本\
返回值

```java
void VERSION(byte type, String version)
```

| 参数名称    | 类型     | 示例值     | 说明                |
| ------- | ------ | ------- | ----------------- |
| type    | byte   | 0或1     | 0代表软件版本号 1代表硬件版本号 |
| version | String | 1.0.0.1 | 版本号               |

简化版本

```java
public static void GET_VERSION(boolean softVersion,IVersionListenerLite listenerLite)

public interface IVersionListenerLite {
/**
     * 返回版本号
     * @param softwareVersion 软件版本号
     * @param hardwareVersion 固件版本号
     */
    void versionResult( String softwareVersion,String hardwareVersion);
}

```

**iOS:**
```Swift
/// 读取固件版本
/// - Parameter completion: 读取固件版本回调
/// - Result: 读取结果
/// - BCLReadFirmwareResponse: 包含固件版本的响应模型
/// - BCLError: 错误信息
func readFirmware(completion: @escaping (Result<BCLReadFirmwareResponse, BCLError>) -> Void) 

/// 读取硬件版本
/// - Parameter completion: 读取硬件版本回调
/// - Result: 读取结果
/// - BCLReadHardwareResponse: 包含硬件版本的响应模型
/// - BCLError: 错误信息
func readHardware(completion: @escaping (Result<BCLReadHardwareResponse, BCLError>) -> Void) 
```
#### 调用示例
```Swift
/// 读取硬件版本
BCLRingManager.shared.readHardware { res in
    switch res {
    case let .success(response):
        BDLogger.info("硬件版本: \(response.hardwareVersion)")
    case let .failure(error):
        BDLogger.error("读取硬件版本失败: \(error)")
    }
}

/// 读取固件版本
BCLRingManager.shared.readFirmware { res in
    switch res {
    case let .success(response):
        BDLogger.info("固件版本: \(response.firmwareVersion)")
    case let .failure(error):
        BDLogger.error("读取固件版本失败: \(error)")
    }
}
```
### 电池电量

接口功能：获取电池电量、 电池状态。

**android:**

接口声明：

```java
LmAPI.GET_BATTERY((byte) 0x00);  //0x00获取电量，0x01获取充电状态
```

参数说明：type：0x00获取电量 ，0x01获取充电状态\
返回值

```java
void battery(byte status, byte datum)
```

| 参数名称   | 类型   | 示例值   | 说明              |
| ------ | ---- | ----- | --------------- |
| status | byte | 0或1   | 0代表电池电量 1代表充电状态 |
| datum  | byte | 0-100 | 电量              |
| datum  | byte | 1     | 0未充电 1充电中 2充满   |

简化版本

```java
//type 电池类型，0读取电量(充电中和充电完成，电量无效) 1充电状态，电量无效
static void GET_BATTERY(int type, IBatteryListenerLite listenerLite)

public interface IBatteryListenerLite {
    /**
     * 电量
     * @param type 获取电量还是获取充电状态 0是电量，1是充电状态
     * @param electricity 电量百分比
     */
    void battery(int type, int electricity);

    /**
     * 主动推送电量信息
     * @param type 获取电量还是获取充电状态 0是电量，1是充电状态
     * @param electricity 电量百分比
     */
    void battery_push(int type, int electricity);
}
```

**iOS:**
```Swift
/// 蓝牙设备主动推送电量信息 
/// - Parameter completion: 主动推送电量信息回调
/// - Result: 主动推送电量信息结果 0~100电量值范围、101充电中、102充满
/// - 说明：该推送一般是在戒指充电状态发生变化时会收到推送信息，例如放入充电仓时会收到、由充电仓取下时。一般戒指会推送3-5次这样，避免单次推送无法收到情况。
BCLRingManager.shared.batteryNotifyBlock: ((Int) -> Void)

/// 电量推送观察者（供外部订阅）
public var batteryNotifyObservable: Observable<Int>

/// 主动获取电量
/// - Parameter completion: 主动获取电量回调
/// - Result: 获取结果 0~100电量值范围，101充电中，102充满
/// - BCLReadBatteryResponse: 包含电量的响应模型
/// - BCLError: 错误信息
func readBattery(completion: @escaping (Result<BCLReadBatteryResponse, BCLError>) -> Void)

```

#### 调用示例
```Swift
/// 监听电量推送Block
BCLRingManager.shared.batteryNotifyBlock = { batteryLevel in
    BDLogger.info("电量推送Block: \(batteryLevel)")
}

/// 主动获取电量
BCLRingManager.shared.readBattery { res in
    switch res {
    case let .success(response):
        BDLogger.info("电量: \(response.batteryLevel)")
    case let .failure(error):
        BDLogger.error("读取电量失败: \(error)")
    }
}
```

### 读取步数

接口功能：获取当天累计步数。

这个指令针对一般戒指使用，对于累积计步方式的戒指不适用

**android:**

接口声明：

```java
LmAPI.STEP_COUNTING（）
```

参数说明：无\
返回值

```java
void stepCount(byte[] bytes)
```

| 参数名称  | 类型      | 示例值  | 说明                       |
| ----- | ------- | ---- | ------------------------ |
| bytes | byte\[] | 3303 | 步数819(小端模式，由0333转10进制得到) |

简化版本

```java
public static void STEP_COUNTING(IStepListenerLite listenerLite)

public interface IStepListenerLite {
    /**
     * 计步
     *
     * @param steps 步数
     */
    void stepCount(int steps);

    /**
     * 清除步数
     * @param
     */
    void clearStepCount();
}
```

**iOS:**

```Swift
/// 读取实时步数
/// - Parameter completion: 读取实时步数回调
/// - Result: 读取结果
/// - BCLStepCountResponse: 包含实时步数的响应模型
/// - BCLError: 错误信息
func readStepCount(completion: @escaping (Result<BCLStepCountResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
/// 读取实时步数
BCLRingManager.shared.readStepCount { result in
    switch result {
    case let .success(response):
        BDLogger.info("实时步数: \(response.stepCount)")
    case let .failure(error):
        BDLogger.error("读取实时步数失败: \(error)")
    }
}
```


### 清除步数

接口功能：清除步数。

**android:**

接口声明：

```java
LmAPI.CLEAR_COUNTING（）
```

参数说明：无\
返回值：

```java
void clearStepCount(byte data)
```

| 参数名称 | 类型   | 示例值 | 说明          |
| ---- | ---- | --- | ----------- |
| byte | data | 1   | 返回1代表清除步数成功 |

简化版本

```java
 public static void CLEAR_COUNTING(IStepListenerLite listenerLite)
 public interface IStepListenerLite {
    /**
     * 计步
     *
     * @param steps 步数
     */
    void stepCount(int steps);

    /**
     * 清除步数
     * @param
     */
    void clearStepCount();
}
```

**iOS:**

```Swift
/// 清除实时步数
/// - Parameter completion: 清除实时步数回调
/// - Result: 清除结果
/// - BCLClearStepCountResponse: 包含清除结果的响应模型
/// - BCLError: 错误信息
func clearStepCount(completion: @escaping (Result<BCLClearStepCountResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
/// 清除实时步数
BCLRingManager.shared.clearStepCount { result in
    switch result {
    case .success:
        BDLogger.info("清除步数成功")
    case let .failure(error):
        BDLogger.error("清除步数失败: \(error)")
    }
}
```

### 步数推送（定制：Z5I）
接口功能：戒指主动推送步数。

**iOS:**
```Swift
/// 步数推送观察者（供外部订阅）
public var stepNotifyObservable: Observable<Int>

/// 步数推送变化回调
public var stepNotifyBlock: ((Int) -> Void)?
```

#### 调用示例
```Swift
BCLRingManager.shared.stepNotifyBlock = { steps in
    print("步数推送: \(steps)")
}
```

### 恢复出厂设置

接口功能：恢复出厂设置

**android:**

接口声明：

```java
LmAPI.RESET()
```

参数说明：无\
返回值：无 ，有回调reset方法即认为成功&#x20;

```java
void reset(byte[] data);
```

简化版本

```java
public static void RESET(ISystemControlListenerLite listenerLite)

public interface ISystemControlListenerLite {
    /**
     * 恢复出厂设置
     */
    void reset();

    /**
     * 设置采集周期
     */
    void setCollection(boolean success);

    /**
     * 获取采集周期
     */
    void getCollection(int data);

    /**
     * 获取序列号
     * @param serial
     */
    void getSerialNum(String serial);

    /**
     * 设置序列号

     */
    void setSerialNum(boolean success);

    /**
     * 设置蓝牙名称
     */
    void setBlueToolName(boolean success);

    /**
     * 读取蓝牙名称
     * @param len 蓝牙名称长度
     * @param name 蓝牙名称
     */
    void readBlueToolName(int len,String name);
}
```

**iOS:**
```Swift
/// 恢复出厂设置
/// - Parameter completion: 恢复出厂设置回调
/// - BCLRestoreFactorySettingsResponse: 包含恢复出厂设置结果的响应模型
func restoreFactorySettings(completion: @escaping (Result<BCLRestoreFactorySettingsResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.restoreFactorySettings { result in
    switch result {
    case .success(let response):
        print("恢复出厂设置成功")
    case .failure(let error):
        print("恢复出厂设置失败: \(error)")
    }
}
```

### 采集周期设置

接口功能：采集周期设置。设置后下次有效

**android:**

接口声明：

```java
LmAPI.SET_COLLECTION(collection)//采集周期，单位秒
```

参数说明：colection：采集间隔，单位秒\
返回值：

```java
void setCollection(byte result)
```

| 参数名称   | 类型   | 示例值 | 说明                      |
| ------ | ---- | --- | ----------------------- |
| result | byte | 0，1 | 设置采集周期失败 1代表0代表设置采集周期成功 |

简化版本

```java
  /**
  *collection采集周期，秒
  **/
  public static void SET_COLLECTION(int collection,ISystemControlListenerLite listenerLite)
  public interface ISystemControlListenerLite {
    /**
     * 恢复出厂设置
     */
    void reset();

    /**
     * 设置采集周期
     */
    void setCollection(boolean success);

    /**
     * 获取采集周期
     */
    void getCollection(int data);

    /**
     * 获取序列号
     * @param serial
     */
    void getSerialNum(String serial);

    /**
     * 设置序列号

     */
    void setSerialNum(boolean success);

    /**
     * 设置蓝牙名称
     */
    void setBlueToolName(boolean success);

    /**
     * 读取蓝牙名称
     * @param len 蓝牙名称长度
     * @param name 蓝牙名称
     */
    void readBlueToolName(int len,String name);
}
```

**iOS:**
```Swift
/// 设置采集周期
/// - Parameter period: 采集周期（单位：秒）最小值为60
/// - Parameter completion: 设置采集周期回调
/// - BCLSetCollectPeriodResponse: 包含设置结果的响应模型
func setCollectPeriod(period: Int, completion: @escaping (Result<BCLSetCollectPeriodResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.setCollectPeriod(period: 300) { result in
    switch result {
    case .success(let response):
        print("设置采集周期成功")
    case .failure(let error):
        print("设置采集周期失败: \(error)")
    }
}
```

### 采集周期读取

**android:**

接口功能：采集周期读取\
接口声明：

```java
LmAPI.GET_COLLECTION()
```

参数说明：无\
返回值：

```java
void getCollection(byte[] bytes)
```

| 参数名称  | 类型      | 示例值      | 说明                  |
| ----- | ------- | -------- | ------------------- |
| bytes | byte\[] | b0040000 | 采集时间间隔 ，单位秒 如：1200s |

简化版本

```java
   public static void GET_COLLECTION(ISystemControlListenerLite listenerLite)
   public interface ISystemControlListenerLite {
    /**
     * 恢复出厂设置
     */
    void reset();

    /**
     * 设置采集周期
     */
    void setCollection(boolean success);

    /**
     * 获取采集周期
     */
    void getCollection(int data);

    /**
     * 获取序列号
     * @param serial
     */
    void getSerialNum(String serial);

    /**
     * 设置序列号

     */
    void setSerialNum(boolean success);

    /**
     * 设置蓝牙名称
     */
    void setBlueToolName(boolean success);

    /**
     * 读取蓝牙名称
     * @param len 蓝牙名称长度
     * @param name 蓝牙名称
     */
    void readBlueToolName(int len,String name);
}
```

**iOS:**
```Swift
/// 获取采集周期
/// - Parameter completion: 获取采集周期回调
/// - BCLGetCollectPeriodResponse: 包含采集周期的响应模型
func getCollectPeriod(completion: @escaping (Result<BCLGetCollectPeriodResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.getCollectPeriod { result in
    switch result {
    case .success(let response):
        print("采集周期: \(response.period)秒")
    case .failure(let error):
        print("获取采集周期失败: \(error)")
    }
}
```

### 一键自检

可以通过指令，对设备状态进行检测

**android:**

```java
LmAPI.ONE_KEY_TEST(new IOneTestListenerLite() {
    @Override
    public void oneTestErrorCode(int errData) {
       
    }
});

```

返回的错误码对应表是

\[0:1]故障码，无故障为0，有故障对应位为1

bit 1：PPG通讯错误

bit 2：Gsensor通讯错误

bit 3：PMIC通讯错误

bit 4：BatAdc错误

bit 5：TempAdc错误

bit 6：PPG led故障

bit 7：Gsensor故障

bit 8：puf加密芯片通信故障

bit 9：touch通信故障

**iOS:**
```Swift
/// 一键自检
/// - Parameter completion: 一键自检回调
/// - BCLFixtureTestingOneKeySelfInspectionResponse: 包含一键自检结果的响应模型
func oneKeySelfInspection(completion: @escaping (Result<BCLFixtureTestingOneKeySelfInspectionResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.oneKeySelfInspection { result in
    switch result {
    case .success(let response):
        print("自检结果: \(response)")
    case .failure(let error):
        print("自检失败: \(error)")
    }
}
```