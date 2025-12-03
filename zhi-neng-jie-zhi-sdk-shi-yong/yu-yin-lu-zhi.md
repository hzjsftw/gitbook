---
description: 支持语音的戒指，可以通过触摸戒指，或者通过App下发指令，控制戒指进行音频字节传输，传输到App上，App进行解析保存为音乐文件，或者将语音转换为文本
icon: microphone-lines
---

# 语音录制

**android:**

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

接口功能：设置主动推送音频信息，如果是pcm格式，建议设置成adpcm，防止丢包&#x20;

接口声明：

```java
//设置推送adpcm
LmAPI.CONTROL_AUDIO_ADPCM_AUDIO((byte) 0x1);
```

### 控制音频传输&#x20;

接口功能：控制音频传输&#x20;

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
      * @param bytes
      */
     void controlAudioResult(byte[] bytes);
 
     /**
      *获取主动推送音频信息
      */
     void getControlAudioAdpcmResult(boolean adpcm);
 
     /**
      *获取主动推送音频信息
      */
     void pushAudioInformationResult(boolean success);
 }
```

录音戒指灯光含义：

* 录音的时候绿灯亮
* 充电的时候呼吸灯
* 蓝牙连接亮蓝灯2s
* 断开连接闪烁3次蓝灯

### 控制PCM格式音频传输

**iOS:**
```Swift
/// 音频传输 - 控制PCM格式音频传输
/// - Parameters:
///   - isOpen: 是否打开
/// - Parameter completion: 控制PCM格式音频传输回调
func controlPCMFormatAudio(isOpen: Bool, completion: @escaping (Result<BCLControlPCMFormatResponse, BCLError>) -> Void)
```

### 控制ADPCM格式音频传输

**iOS:**
```Swift
/// 音频传输 - 控制ADPCM格式音频传输
/// - Parameters:
///   - isOpen: 是否打开
/// - Parameter completion: 控制ADPCM格式音频传输回调
func controlADPCMFormatAudio(isOpen: Bool, completion: @escaping (Result<BCLControlADPCMFormatResponse, BCLError>) -> Void)
```

### 主动推送音频信息

**iOS:**
```Swift
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
```Swift
/// 音频格式转换 - 将ADPCM格式音频数据转换为PCM格式
/// - Parameter adpcmData: ADPCM格式的音频数据
/// - Returns: PCM格式的音频数据，转换失败返回nil
func convertAdpcmToPcm(adpcmData: Data) -> Data?
```

### 开始录音（固件定制：Z5J）
**iOS:**
```Swift
/// 音频传输 - 戒指开始录音
/// - Parameters:
///   - isOpen: 是否打开录音 true:打开 false:关闭
///   - totalDuration: 总录音时长，单位秒
///   - sliceDuration: 切片保存时长，单位秒
///   - completion: 戒指开始录音回调
/// - BCLRingStartRecordingResponse: 包含状态的响应模型(0:失败 1:成功)
func ringStartRecording(isOpen: Bool, totalDuration: UInt32, sliceDuration: UInt32, completion: @escaping (Result<BCLRingStartRecordingResponse, BCLError>) -> Void)
```