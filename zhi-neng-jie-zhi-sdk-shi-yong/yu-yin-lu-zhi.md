---
description: >-
  支持语音的戒指，可以通过触摸戒指，或者通过App下发指令，控制戒指进行音频字节传输，传输到App上，App进行解析保存为音乐文件，或者将语音转换为文本，LmAPI已废弃，建议都用LmAPILite里的方法
icon: microphone-lines
---

# 语音录制

## **android:**

### 获取主动推送音频信息

接口功能：获取主动推送音频信息(需要开启HID中的触摸语音，按住戒指上的磨砂区域，进行录音，戒指主动推送音频信息)

```java
  LmAPILite.GET_CONTROL_AUDIO_ADPCM(listenerLite);
```

监听定义：

```java
 public interface IAudioListenerLite {
    /**
     * 音频数据传输，用户可以保存到本地pcm文件里
     * @param bytes 音频数据，普通音频戒指，解码过的数据
     * @param audioType 0传统单声道音频，1新算法单声道，2新算法双声道
     */
    void controlAudioResult(byte[] bytes,int audioType);

    /**
     *音频数据传输
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

### app控制音频传输

```java
LmAPILite.CONTROL_AUDIO_ADPCM((byte) 0x1, listenerLite);//开启录音

LmAPILite.CONTROL_AUDIO_ADPCM((byte) 0x0, listenerLite);//停止录音
 
```

录音戒指灯光含义：

* 录音的时候绿灯亮
* 充电的时候呼吸灯
* 蓝牙连接亮蓝灯2s
* 断开连接闪烁3次蓝灯

如果是触摸或者双击戒指，戒指主动推送的音频数据，可以在传输途中进行定时，如果1s没有收到数据，默认传输结束

### 音频保存

可以在本地新建一个pcm文件，将byte\[]数据写入

```java
 public static void convertByteToPcm(byte[] bytes, String filePath) throws IOException {
        File file = new File(filePath);
        try (BufferedSink sink = Okio.buffer(Okio.sink(file,true))) {
            sink.write(bytes);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```

### 音频播放

```java


import android.media.AudioAttributes;
import android.media.AudioFormat;
import android.media.AudioManager;
import android.media.AudioTrack;
import android.util.Log;

import com.lm.library.utils.DateUtils;

import java.io.File;
import java.io.FileInputStream;
import java.util.Date;

public class PcmAudioUtils {

    private AudioTrack audioTrack;

    private String filePath;
    private File file;
    private  int bufferSizeInBytes;
    private Thread audioTrackThread;
    private int currentPosition;
    private double lastTime;
    private double elapsedSeconds;//已播放时间
    private double audioDurationInSeconds;//总时长
    int sampleRateInHz = 8000; // 采样率
    int channelConfig = AudioFormat.CHANNEL_OUT_MONO; // 声道配置
    int audioFormat = AudioFormat.ENCODING_PCM_16BIT; // 音频格式
    boolean pauseTag=false;
    private IAudioProgress iAudioProgress;
    private String  playState="none";
    private double totalBuffSize=0;
    private long fileSize;

    private boolean dualMicrophone=false;//是否双麦克风
    public PcmAudioUtils(String filePath) {
        file = new File(filePath);
        fileSize=  file.length();
        init();
    }
    public PcmAudioUtils(String filePath,boolean dualMicrophone) {
        file = new File(filePath);
        fileSize=  file.length();
        this.dualMicrophone=dualMicrophone;
        init();
    }


    /**
     * 初始化
     */
    public void init() {

        if (!file.exists()) return;
        if(dualMicrophone){
            sampleRateInHz=16000;//双声道，16k
            channelConfig = AudioFormat.CHANNEL_OUT_STEREO ; // 立体声
            audioFormat = AudioFormat.ENCODING_PCM_16BIT;
        }
        int streamType = AudioManager.STREAM_MUSIC; // 音频流类型

         bufferSizeInBytes = AudioTrack.getMinBufferSize(sampleRateInHz, channelConfig, audioFormat); // 缓冲区大小

        /**
         * 设置音频信息属性
         * 1.设置支持多媒体属性，比如audio，video
         * 2.设置音频格式，比如 music
         */
        AudioAttributes attributes = new AudioAttributes.Builder()
                .setUsage(AudioAttributes.USAGE_MEDIA)
                .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
                .build();
        /**
         * 设置音频格式
         * 1. 设置采样率
         * 2. 设置采样位数
         * 3. 设置声道
         */
        AudioFormat format = new AudioFormat.Builder()
                .setSampleRate(sampleRateInHz)
                .setEncoding(audioFormat)
                .setChannelMask(channelConfig)
                .build();
        audioTrack = new AudioTrack(attributes,format,bufferSizeInBytes,AudioTrack.MODE_STREAM, AudioManager.AUDIO_SESSION_ID_GENERATE);

    }

    /**
     * 停止播放录音,并释放资源
     */
    public void stopPlay() {
        init();
        playState="stopPlay";
        if (audioTrack != null) {
            audioTrack.release();
        }
    }

    /**
     * 暂停
     */
    public void pausePlay() {
        init();
        if (audioTrack != null) {
            audioTrack.pause();
        }

        pauseTag=true;
        playState="pausePlay";
    }


    public void play(){
        init();
        long currentTimeMillis = System.currentTimeMillis();;
        pauseTag=false;

        if(audioTrack==null){
            return;
        }
        audioTrack.play();
        playState="play";

        audioTrackThread=   new Thread(() -> {
            FileInputStream fileInputStream = null;
            lastTime=(double)currentTimeMillis;
            try {
                fileInputStream = new FileInputStream(file);
                byte[] buffer = new byte[bufferSizeInBytes];
                Log.i("PcmAudioUtils", "playAudio: "+bufferSizeInBytes);
                if (currentPosition > 0) {
                    fileInputStream.skip(currentPosition);
                }
                int read = 0;
                while (read != -1&&!pauseTag) {
                    read = fileInputStream.read(buffer);
                    totalBuffSize+=bufferSizeInBytes;
                    //将缓冲区buffer写入audioTrack进行播放
                    int write = audioTrack.write(buffer, 0, buffer.length);
                    currentPosition+=write;
                    elapsedSeconds+=( (double) System.currentTimeMillis() -lastTime);
                    if(totalBuffSize>fileSize){
                        totalBuffSize=fileSize;
                    }

                    long fileTime= (long) (totalBuffSize/8000/2);
                    if(dualMicrophone){
                        fileTime= (long) (totalBuffSize/16000/4);
                    }
                    if(fileTime>=0&&totalBuffSize>0){
                        fileTime+=1;//多1秒吧，要不然比实际的时间短一点
                    }
                    Date date = new Date(fileTime * 1000);
                    long timestamp = date.getTime();

                  // Log.d("AudioTrack", "已播放时间：" + DateUtils.longToString(timestamp,"mm:ss"));
                    if(iAudioProgress!=null){
                        iAudioProgress.playProgress(DateUtils.longToString(timestamp,"mm:ss"));
                    }

                }
                if(!pauseTag){
                    audioTrack.stop();
                    audioTrack.release();
                    if(iAudioProgress!=null){
                        iAudioProgress.setStop();
                    }
                    currentPosition=0;
                }

            } catch (Throwable e) {

            }

        });
        audioTrackThread.start();
    }


    public void setiAudioProgress(IAudioProgress iAudioProgress) {
        this.iAudioProgress = iAudioProgress;
    }

    public interface IAudioProgress{
        void playProgress(String progress);
        void setStop();
    }

    public void stop() {
        pauseTag=true;
        init();

        playState="none";
        audioTrack.stop();
        audioTrack.release();

    }

    public String getPlayState() {
        return playState;
    }
}
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
