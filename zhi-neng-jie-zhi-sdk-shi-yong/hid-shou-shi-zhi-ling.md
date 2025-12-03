---
description: HID戒指是手势戒指，可以通过手势，对手机和电脑进行控制，包括刷视频，拍照，听音乐，PPT，按压戒指上传实时音频，打响指拍照等
icon: hand
---

# HID手势指令

### 设置戒指的HID模式

**android：**

先给出样例：

```java
                byte[] hidBytes = new byte[3];
                hidBytes[0] = 0x04;             //触摸功能，样例是上传实时音频(其他的参考参数说明，设置0x01,0x02,(byte) 0xFF等)
                hidBytes[1] = (byte) 0xFF;      //手势功能，样例是关闭(其他的参考参数说明，设置0x01,0x02,(byte) 0xFF等)
                hidBytes[2] = 0x00;             //系统类型 0：安卓  1：IOS  2：鸿蒙
                LmAPI.SET_HID(hidBytes,TestActivity2.this);
```

参数说明：

| 参数名称     | 类型   | 示例值        | 说明                                                                              |
| -------- | ---- | ---------- | ------------------------------------------------------------------------------- |
| byte\[0] | byte | 4          | <p>触摸hid 模式0：刷视频模式<br>1：拍照模式<br>2：音乐模式<br>3: ppt模式<br>4：上传实时音频<br>0xFF:关闭</p>   |
| byte\[1] | byte | (byte)0xFF | <p>手势hid 模式0：刷视频模式<br>1：拍照模式<br>2：音乐模式<br>3：ppt模式<br>4：打响指(拍照)模式<br>0xFF:关闭</p> |
| byte\[2] | byte | 0          | <p>系统类型：<br>0：安卓<br>1：IOS<br>2：鸿蒙</p>                                           |

返回值：

```java
   @Override
    public void SET_HID(byte result) {
        if(result == (byte)0x00){
            postView("\n设置HID失败");
        }else if(result == (byte)0x01){
            postView("\n设置HID成功");
        }
    }
```

| 参数名称   | 类型   | 示例值 | 说明              |
| ------ | ---- | --- | --------------- |
| result | byte | 0,1 | 0代表设置失败 1代表设置成功 |

简化版本

```java
  /**
     * 设置HID
     * @param touchMode  触摸hid 模式
     * 0：刷视频模式
     * 1：拍照模式
     * 2：音乐模式
     * 3: ppt模式
     * 4：上传实时音频
     * 255:关闭
     * @param gestureMode 手势hid 模式
     * 0：刷视频模式
     * 1：拍照模式
     * 2：音乐模式
     * 3：ppt模式
     * 4：打响指(拍照)模式
     * 255:关闭
     * @param context
     * @param listenerLite
     */
    public static void SET_HID(int touchMode,int gestureMode, Context context,IHIDListenerLite listenerLite)
```

**iOS:**
```Swift
/// 设置HID模式
/// - Parameters:
///   - touchMode: 触摸模式 0:刷视屏模式、1:拍照模式、2:音乐模式、3:PPT模式、4:上传实时音频模式 255:关闭
///   - gestureMode: 手势模式 0:刷视频模式、1:拍照模式、2:音乐模式、3:PPT模式、4:打响指（拍照）模式 255:关闭
///   - systemType: 系统类型 0:Android 1:iOS 2:鸿蒙 3:Windows
///   - deviceModelName: 设备型号名称
///   - screenHeightPixel: 屏幕高度像素
///   - screenWidthPixel: 屏幕宽度像素
/// - Parameter completion: 设置HID模式回调
func setHIDMode(touchMode: Int, gestureMode: Int, systemType: Int, deviceModelName: String, screenHeightPixel: Int, screenWidthPixel: Int, completion: @escaping (Result<BCLSetHIDModeResponse, BCLError>) -> Void)
```

#### 调用示例
```Swift
BCLRingManager.shared.setHIDMode(
    touchMode: 0,
    gestureMode: 1,
    systemType: 1,
    deviceModelName: BCLRingManager.shared.getMobileDeviceModelName(),
    screenHeightPixel: BCLRingManager.shared.getMobileDeviceScreenHeightPixel(),
    screenWidthPixel: BCLRingManager.shared.getMobileDeviceScreenWidthPixel()
) { result in
    switch result {
    case .success(_):
        print("设置HID模式成功")
    case .failure(let error):
        print("设置HID模式失败: \(error)")
    }
}
```

**iOS:**
```Swift
/// 获取当前手机设备型号名称
/// - Returns: 当前手机设备型号名称
func getMobileDeviceModelName() -> String

/// 获取当前手机设备屏幕宽度像素
/// - Returns: 当前手机设备屏幕宽度像素
func getMobileDeviceScreenWidthPixel() -> Int

/// 获取当前手机设备屏幕高度像素
/// - Returns: 当前手机设备屏幕高度像素
func getMobileDeviceScreenHeightPixel() -> Int
```

### 获取HID功能码

获取连接戒指支持的HID功能，只有支持的功能，才能通过设置戒指的HID模式的方法进行设置

**android:**

```java
LmAPI.GET_HID_CODE((byte)0x00);
```

参数说明：

| 参数名称 | 类型   | 示例值 | 说明                                         |
| ---- | ---- | --- | ------------------------------------------ |
| byte | byte | 0   | <p>系统类型：<br>0：安卓<br>1：IOS<br>2：windows</p> |
| 回调：  |      |     |                                            |

```java
    @Override
    public void GET_HID_CODE(byte[] bytes) {
        Logger.show("getHidCode", "支持与否：" + bytes[0] + " 触摸功能：" + bytes[1] + " 空中手势：" + bytes[9] + "\n");

        Logger.show("byteToBitString", byteToBitString(bytes[1]));
        char[] touchModes = byteToBitString(bytes[1]).toCharArray();
        char[] gestureModes = byteToBitString(bytes[9]).toCharArray();

        if (bytes[0] == 0) {
            postView("\n不支持HID功能");
        } else {
            postView("\n支持HID功能");
        }
        if ("00000000".equals(byteToBitString(bytes[1]))) {//不支持触摸功能
            postView("\n不支持触摸功能");
        } else {
            postView("\n支持触摸功能");
        }

        if (touchModes[touchModes.length - 1] == '1') {//拍照
            postView("\n支持触摸拍照功能");
        } else {
            postView("\n不支持触摸拍照功能");
        }

        if (touchModes[touchModes.length - 2] == '1') {//短视频
            postView("\n支持触摸短视频功能");
        } else {
            postView("\n不支持触摸短视频功能");
        }

        if (touchModes[touchModes.length - 3] == '1') {//音乐
            postView("\n支持触摸音乐功能");
        } else {
            postView("\n不支持触摸音乐功能");
        }

        if (touchModes[touchModes.length - 5] == '1') {//音频
            postView("\n支持触摸音频功能");
        } else {
            postView("\n不支持触摸音频功能");
        }

        if ("00000000".equals(byteToBitString(bytes[9]))) {//不支持空中手势
            postView("\n不支持空中手势功能");
        } else {
            postView("\n支持空中手势功能");
        }

        if (gestureModes[gestureModes.length - 1] == '1') {//拍照
            postView("\n支持手势拍照功能");
        } else {
            postView("\n不支持手势拍照功能");
        }

        if (gestureModes[gestureModes.length - 2] == '1') {//短视频
            postView("\n支持手势短视频功能");
        } else {
            postView("\n不支持手势短视频功能");
        }

        if (gestureModes[gestureModes.length - 3] == '1') {//音乐
            postView("\n支持手势音乐功能");
        } else {
            postView("\n不支持手势音乐功能");
        }

        if (gestureModes[gestureModes.length - 5] == '1') {//打响指（拍照）
            postView("\n支持打响指（拍照）功能");
        } else {
            postView("\n不支持打响指（拍照）功能");
        }
    }
```

简化版本

```java
  public static void GET_HID_CODE(int system,IHIDListenerLite listenerLite){
  public interface IHIDListenerLite {
 
     /**
      * 设置HID模式  0代表失败，1代表成功
      */
     void setHIDResut(boolean success);
 
     /**
      * 获取HID模式
      * @param touchMode  手势
      * @param gestureMode   触控
      * @param system  系统
      */
     void getHIDInfo(int touchMode,int gestureMode,int system);
 
     /**
      * 获取HID功能码
      * @param HIDSupport HID功能支持
      * @param touchSupport 触摸功能
      *  @param gestureSupport 手势功能
      */
     void getHidCode(boolean HIDSupport, TouchSupport touchSupport, GestureSupport gestureSupport);
 
 }
```

**iOS:**
```Swift
/// 获取HID功能码
/// - Parameter completion: 获取HID功能码回调
func getHIDFunctionCode(completion: @escaping (Result<BCLGetHIDFunctionCodeResponse, BCLError>) -> Void)
```

### 获取HID

获取当前戒指的HID模式，触摸各功能和手势各功能的开关状态

**android:**

```java
LmAPI.GET_HID();
```

参数说明：无\
返回值：

```java
    @Override
    public void GET_HID(byte touch, byte gesture, byte system) {
        postView("\n当前触摸hid模式：" + touch + "\n当前手势hid模式：" + gesture + "\n当前系统：" + system);
    }
```

| 参数名称    | 类型   | 示例值 | 说明                                                                              |
| ------- | ---- | --- | ------------------------------------------------------------------------------- |
| touch   | byte | 4   | <p>触摸hid 模式0：刷视频模式<br>1：拍照模式<br>2：音乐模式<br>3：ppt模式<br>4：上传实时音频<br>0xFF:关闭</p>    |
| gesture | byte | -1  | <p>手势hid 模式0：刷视频模式<br>1：拍照模式<br>2：音乐模式<br>3：ppt模式<br>4：打响指(拍照)模式<br>0xFF:关闭</p> |
| system  | byte | 0   | <p>系统类型 0：安卓<br>1：IOS<br>2：WINDOWS</p>                                          |

**注：-1和0xFF含义一样，代表关闭**

简化版本

```java
 public static void GET_HID(IHIDListenerLite listenerLite)
```

**iOS:**
```Swift
/// 获取当前HID模式
/// - Parameter completion: 获取当前HID模式回调
func getCurrentHIDMode(completion: @escaping (Result<BCLGetCurrentHIDModeResponse, BCLError>) -> Void)
```

### 手势功能设置（定制功能：Z4I）

**iOS:**
```Swift
/// 设置手势功能
/// - Parameters:
///   - swipeUpGesture: 上滑手势  1：音乐暂停/开始、2：音乐下一首、3：音乐上一首、4: 音量+、5：音量-、6：拍照、255:关闭
///   - swipeDownGesture: 下滑手势
///   - snapGesture: 打响指手势
///   - pinchGesture: 捏一捏手势
/// - Parameter completion: 设置手势功能回调
func setGestureFunction(swipeUpGesture: Int, swipeDownGesture: Int, snapGesture: Int, pinchGesture: Int, completion: @escaping (Result<BCLSetGestureFunctionResponse, BCLError>) -> Void)

/// 读取手势功能
/// - Parameter completion: 读取手势功能回调
func readGestureFunction(completion: @escaping (Result<BCLReadGestureFunctionResponse, BCLError>) -> Void)
```
