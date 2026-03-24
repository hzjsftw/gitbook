---
description: 支持语音的戒指，可以通过触摸戒指，或者通过App下发指令，控制戒指进行音频字节传输，传输到App上，App进行解析保存为音乐文件，或者将语音转换为文本
icon: microphone-lines
---

# 语音录制

## **android:**

### 获取主动推送音频信息

接口功能：获取主动推送音频信息(需要开启HID中的触摸语音，按住戒指上的磨砂区域，进行录音，戒指主动推送音频信息)

```java
LmAPI.GET_CONTROL_AUDIO_ADPCM();
```

回调：

```java
 public void GET_CONTROL_AUDIO_ADPCM(byte pcmType) {
        if(pcmType==0x0){//是PCM
                new Handler().postDelayed(new Runnable() {
                    @Override
                    public void run() {
                        //设置推送adpcm
                        LmAPI.CONTROL_AUDIO_ADPCM_AUDIO((byte) 0x1);
                    }
                },200);
        }
    }
```

### 设置主动推送音频信息

接口功能：设置主动推送音频信息，如果是pcm格式，建议设置成adpcm，防止丢包

接口声明：

```java
//设置推送adpcm
LmAPI.CONTROL_AUDIO_ADPCM_AUDIO((byte) 0x1);
```

### 控制音频传输

接口功能：控制音频传输

接口声明：

```java
LmAPI.CONTROL_AUDIO_ADPCM(byte data)
```

参数说明：(byte) 0x1 开启，(byte) 0x0 关闭 回调：

```java
@Override
   public void CONTROL_AUDIO(byte[] bytes) {
       //通过以上设置，默认都是adpcm格式
 byte[] adToPcm = new AdPcmTool().adpcmToPcmFromJNI(bytes);
  }
```

**注：返回的数据是byte数组，adpcm格式转为pcm格式，保存到文件中**

简化版本

```java


 //获取主动推送音频信息，通过getControlAudioAdpcmResult(boolean adpcm)返回，
 //如果不是adpcm，建议调用PUSH_AUDIO_INFORMATION设置成adpcm
 public static void GET_CONTROL_AUDIO_ADPCM(IAudioListenerLite listenerLite) 


 //设置主动推送音频信息，是否开启adpcm格式，通过pushAudioInformationResult返回，
 //success为true说明设置adpcm格式正确
 public static void PUSH_AUDIO_INFORMATION(boolean isAdPcm,IAudioListenerLite listenerLite)


 //控制adpcm格式音频传输，是否开启adpcm格式，建议设置成adpcm，即isOpen设置成true，否则有丢包的风险，
 // 数据信息通过controlAudioResult
 //方法返回，结果已经转码为pcm，不需要像复杂版本，调用new AdPcmTool().adpcmToPcmFromJNI(bytes)
 public static void CONTROL_ADPCM_TRANSFER(boolean isOpen,IAudioListenerLite listenerLite)


 public interface IAudioListenerLite {
    /**
     *控制音频传输
     * @param bytes 音频数据，普通音频戒指，解码过的数据
     */
    void controlAudioResult(byte[] bytes);

    /**
     *控制音频传输
     * @param bytes 音频数据，原始数据，定制戒指转码需要使用
     */
    void controlAudioRawDataResult(byte[] bytes);

    /**
     * 获取主动推送音频信息
     * @param adpcm 是否adpcm格式
     */
    void getControlAudioAdpcmResult(boolean adpcm);

    /**
     * 获取主动推送音频信息
     * @param success 是否成功
     */
    void pushAudioInformationResult(boolean success);


    /**
     * 戒指主动推送音频完成(touch拿开)
        讯飞定制
     */
    void TOUCH_AUDIO_FINISH_XUN_FEI();



    /**
     * 开始/停止录音
     */
    void recordingResult(boolean result);
}
```

录音戒指灯光含义：

* 录音的时候绿灯亮
* 充电的时候呼吸灯
* 蓝牙连接亮蓝灯2s
* 断开连接闪烁3次蓝灯

### 音频转码

固件目前分为三种音频，一种是普通的单麦克风8k音频，一种是高清单麦克风8k音频，一种是双麦克风16k音频，分别对应着三种解码方式

1.普通的单麦克风8k音频：

CONTROL\_AUDIO(byte\[] bytes) 返回，LmAPI需要自己解码，LmApiLite已经封装了解码方式，返回的是解码后的数据，不需要手动解码

```java
@Override
   public void CONTROL_AUDIO(byte[] bytes) {
       //通过以上设置，默认都是adpcm格式
 byte[] adToPcm = new AdPcmTool().adpcmToPcmFromJNI(bytes);
  }
```

2.高清单麦克风8k音频

这个需要原始数据，因为涉及到资源释放，sdk不好把握时机，需要用户自己调用方法解码，原始数据返回：

如果用的是LmAPI，CONTROL\_AUDIO(byte\[] bytes)就是原始数据，如果是LmApiLite，

```java
 void controlAudioRawDataResult(byte[] bytes);
```

返回的是原始数据

解码逻辑：

先初始化工具

```java
private AdPcmTool adPcmTool = new AdPcmTool();
```

单声道解码：

```java
 byte[] bytesMono = adPcmTool.decodeADPCMMonoChannel(bytes, bytes.length);
```

解码完，进行资源释放

```java
adPcmTool.resetAllDecoders();
```

3.双麦克风16k音频

双声道解码：

```java
byte[] bytesDual = adPcmTool.decodeADPCMDualChannel(bytes, bytes.length);
```

解码完，进行资源释放

```java
adPcmTool.resetAllDecoders();
```

## ios:

### 控制PCM格式音频传输

**iOS:**

```swift
/// 音频传输 - 控制PCM格式音频传输
/// - Parameters:
///   - isOpen: 是否打开
/// - Parameter completion: 控制PCM格式音频传输回调
func controlPCMFormatAudio(isOpen: Bool, completion: @escaping (Result<BCLControlPCMFormatResponse, BCLError>) -> Void)
```

### 控制ADPCM格式音频传输

**iOS:**

```swift
/// 音频传输 - 控制ADPCM格式音频传输
/// - Parameters:
///   - isOpen: 是否打开
/// - Parameter completion: 控制ADPCM格式音频传输回调
func controlADPCMFormatAudio(isOpen: Bool, completion: @escaping (Result<BCLControlADPCMFormatResponse, BCLError>) -> Void)
```

### 主动推送音频信息

**iOS:**

```swift
/// 音频传输 - 设置主动推送音频信息
/// - Parameters:
///   - audioType: 音频类型 .pcm .adpcm
/// - Parameter completion: 设置主动推送音频信息回调
func setActivePushAudioInfo(audioType: BCLAudioType, completion: @escaping (Result<BCLSetActivePushAudioInfoResponse, BCLError>) -> Void)

/// 音频传输 - 获取主动推送音频信息
/// - Parameter completion: 获取主动推送音频信息回调
func getActivePushAudioInfo(completion: @escaping (Result<BCLGetActivePushAudioInfoResponse, BCLError>) -> Void)
```

### 音频格式转换

**iOS:**

```swift
/// 音频格式转换 - 将ADPCM格式音频数据转换为PCM格式
/// - Parameter adpcmData: ADPCM格式的音频数据
/// - Returns: PCM格式的音频数据，转换失败返回nil
func convertAdpcmToPcm(adpcmData: Data) -> Data?
```

### 开始录音（固件定制：Z5J）

**iOS:**

```swift
/// 音频传输 - 戒指开始录音
/// - Parameters:
///   - isOpen: 是否打开录音 true:打开 false:关闭
///   - totalDuration: 总录音时长，单位秒
///   - sliceDuration: 切片保存时长，单位秒
///   - completion: 戒指开始录音回调
/// - BCLRingStartRecordingResponse: 包含状态的响应模型(0:失败 1:成功)
func ringStartRecording(isOpen: Bool, totalDuration: UInt32, sliceDuration: UInt32, completion: @escaping (Result<BCLRingStartRecordingResponse, BCLError>) -> Void)
```

### 双麦克风16k音频解码

**iOS:**

```swift
/// 初始化立体声ADPCM处理器
///
/// 在开始立体声音频解码之前调用此方法初始化处理器。
/// 处理器会维护左右声道的独立编解码状态。
///
/// - Returns: 初始化是否成功
func initStereoAdpcmProcessor() -> Bool


/// 立体声ADPCM解码 - 将立体声ADPCM格式音频数据转换为PCM格式
///
/// 将交错格式的立体声ADPCM数据解码为交错格式的立体声PCM数据。
/// 如果处理器未初始化，会自动进行初始化。
///
/// - Parameters:
///   - adpcmData: 立体声ADPCM格式的音频数据（交错格式：左0,右0,左1,右1...）
///   - sampleCount: 每个声道的样本数
/// - Returns: 解码后的立体声PCM数据（交错格式：L0,R0,L1,R1...），转换失败返回nil
///
/// - Note: 输出PCM数据大小为 sampleCount * 2(声道) * 2(字节) = sampleCount * 4 字节
func decodeStereoAdpcm(adpcmData: Data, sampleCount: Int) -> Data?

/// 单声道ADPCM解码（8K高清音频）
///
/// 将单声道ADPCM数据解码为单声道16位PCM数据。
/// 如果处理器未初始化，会自动进行初始化。
///
/// - Parameter adpcmData: 单声道ADPCM格式数据（每字节包含2个4位样本）
/// - Returns: PCM格式音频数据（16位有符号整数），失败返回nil
///
/// - Note: 输出PCM数据大小为 adpcmData.count * 2(样本) * 2(字节) = adpcmData.count * 4 字节
func decodeMonoAdpcm(adpcmData: Data) -> Data?


/// 立体声ADPCM传输完成
///
/// 当音频传输完成时调用此方法，会重置处理器状态以准备下一次传输。
/// 对应Android端的 `dualChannelTransferFinish` 方法。
func stereoAdpcmTransferFinish()

```
