> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247506503&idx=3&sn=d7541673f71cc00e50ded3a9185a6777&chksm=e8797f3ddf0ef62bc72daaf3515f6028a9deccb6c747684dc933fe116ff3c7f7b7800b0e964d&mpshare=1&scene=1&srcid=06013d0s4lGwJt2TCtCJAXHa&sharer_sharetime=1622554180117&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

‍‍

> PS：本周关键词『思考』、『行动』、『坚持』、『自律』。

本文介绍下 `Android`音视频开发中的`AudioRecord`的使用，案例将会在前面`MediaCodec`录制`MP4`的基础上进行，使用`AudioRecord`录制音频数据并将其合成到`MP4`中，`Android`音视频同系列文章如下：

*   [如何正确编译 ijkplayer](http://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484158&idx=1&sn=2e0b6de7d4e93e2e02717a9bd046fe96&chksm=fd298aceca5e03d8252a57e796dca3e02e94a73451da696c09be2f3f0ba4094dc4d0bc7af891&scene=21#wechat_redirect)  
    
*   [音视频开发基础知识](http://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484757&idx=1&sn=95f860dd70ad41dd99896802de646846&chksm=fd298d65ca5e0473c1594b0af2ee04415c614c6b50b70e440fb0e993dabf77b8f5a25f9f7846&scene=21#wechat_redirect)
    
*   [](http://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484785&idx=1&sn=b03dd295c79e2e99d558752a7fef8a57&chksm=fd298d41ca5e0457e5d520e2b97fc079dbdc8e9ab4ce1ae16cb8673250c3aa04ba7e75d10baf&scene=21#wechat_redirect)[音频帧、视频帧及其同步](http://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484785&idx=1&sn=b03dd295c79e2e99d558752a7fef8a57&chksm=fd298d41ca5e0457e5d520e2b97fc079dbdc8e9ab4ce1ae16cb8673250c3aa04ba7e75d10baf&scene=21#wechat_redirect)
    
*   [](http://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484828&idx=1&sn=eed76722cda3a25ca589808be7b9e315&chksm=fd298dacca5e04bae5b59c6f41a6928efe38a0eb7ef6cc592f3109688fc8c93486f51d46684f&scene=21#wechat_redirect)[Camera2、MediaCodec 录制 mp4](http://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484828&idx=1&sn=eed76722cda3a25ca589808be7b9e315&chksm=fd298dacca5e04bae5b59c6f41a6928efe38a0eb7ef6cc592f3109688fc8c93486f51d46684f&scene=21#wechat_redirect)
    
*   [MediaCodec 详解](http://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484865&idx=1&sn=174b8ca702466e83e72c7115d91b06ea&chksm=fd298df1ca5e04e7b2df9dc9f21e5cfe3e910204c905d8605f648ce6f6404432a83ae52a23a3&scene=21#wechat_redirect)
    
*   [](http://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484970&idx=1&sn=66c51f329fe521b37b4c5329199b9818&chksm=fd298e1aca5e070cf97622c1280b9ed0fec32bf99062c530111d3b10904795a3e55807f75d51&scene=21#wechat_redirect)[音频基础知识](http://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484970&idx=1&sn=66c51f329fe521b37b4c5329199b9818&chksm=fd298e1aca5e070cf97622c1280b9ed0fec32bf99062c530111d3b10904795a3e55807f75d51&scene=21#wechat_redirect)
    

本文的主要内容如下：

1.  AudioRecord 介绍
    
2.  AudioRecord 生命周期
    
3.  AudioRecord 音频数据读取
    
4.  直接缓冲区和字节序
    
5.  AudioRecord 使用
    

#### AudioRecord 介绍

创建`AudioRecord`的参数及说明如下：

*   audioSource：表示音频源，音频源定义在`MediaRecorder.AudioSource`中，如常见的音频源主麦克风`MediaRecorder.AudioSource.MIC`等。
    
*   sampleRateInHz：表示以赫兹为单位的采样率，其含义是每个通道每秒的采样数，常见采样率中只有 44100Hz 的采样率可以保证在所有设备上正常使用，可以通过`getSampleRate`获取实际采样率，这个采样率不是音频内容播放的采样率，比如可以在采样率为 48000Hz 的设备上播放采样率为 8000Hz 的声音，对应平台会自动处理采样率转换，因此不会以 6 倍的速度播放。
    
*   channelConfig：表示声道数，声道定义在`AudioFormat`中，常见的声道中只有单声道`AudioFormat.CHANNEL_IN_MONO`能保证在所有设备上正常使用，其他的比如`AudioFormat.CHANNEL_IN_STEREO`表示双声道，也就是立体声。
    
*   audioFormat：表示`AudioRecord`返回的音频数据的格式，对于线性 `PCM`来说，反应每个样本大小（8、16、32 位）及表现形式（整型、浮点型），音频格式定义在`AudioFormat`中，常见的音频数据格式中只有`AudioFormat.ENCODING_PCM_16BIT`可以保证在所有的设备上正常使用，像`AudioFormat.ENCODING_PCM_8BIT`不能保证在所有设备上正常使用。
    
*   bufferSizeInBytes：表示写入音频数据的缓冲区的大小，该值不能小于`getMinBufferSize`的大小，即不能小于`AudioRecord`所需的最小缓冲区的大小，否则将导致`AudioRecord`初始化失败，该缓冲区大小并不能保证在负载情况下顺利录制，必要时可选择更大值。
    

#### AudioRecord 生命周期

`AudioRecord`的生命周期状态包括 `STATE_UNINITIALIZED`、`STATE_INITIALIZED`、`RECORDSTATE_RECORDING`和`RECORDSTATE_STOPPED`，分别对应未初始化、已初始化、录制中、停止录制，如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/5kgRlkZhZcWag4b48xb8go2GDl9cVbeMFZ75h9R7Dea6arTv2xHbSXBz0nLRXzTG52iaN3sCs3ibaCeGtz9sBibsA/640?wx_fmt=png)

简单说明一下：

1.  未创建之前或者`release`之后`AudioRecord`都进入`STATE_UNINITIALIZED`状态。
    
2.  创建`AudioRecord`时进入`STATE_INITIALIZED`状态。
    
3.  调用`startRecording`进入`RECORDSTATE_RECORDING`状态。
    
4.  调用`stop`进入`RECORDSTATE_STOPPED`状态。
    

那么如何获取`AudioRecord`的状态呢，可以通过`getState`和`getRecordingState`获取其状态，为保证正确使用可在使用`AudioRecord`对象操作之前进行其状态的判断。

#### AudioRecord 音频数据读取

`AudioRecord` 提供的三种读取音频数据的方式，如下：

```
// 1. 读取音频数据，音频格式为AudioFormat＃ENCODING_PCM_8BIT
int read(@NonNull byte[] audioData, int offsetInBytes, int sizeInBytes)
// 2. 读取音频数据，音频格式为AudioFormat＃ENCODING_PCM_16BIT
int read(@NonNull short[] audioData, int offsetInShorts, int sizeInShorts)
// 3. 读取音频数据，见后面章节
int read(@NonNull ByteBuffer audioBuffer, int sizeInBytes)
```

读取音频数据的返回值大于等于 0，读取音频数据常见异常如下：

1.  ERROR_INVALID_OPERATION：表示`AudioRecord` 未初始化。
    
2.  ERROR_BAD_VALUE：表示参数无效。
    
3.  ERROR_DEAD_OBJECT：表示已经传输了一些音频数据的情况下不返回错误码，将在下次 `read`返回处返回错误码。
    

上面三个 `read` 函数都是从硬件音频设备读取音频数据，前两个主要的区别就是音频格式不同，分别是 8 位、16  位，对应的量化等级则是 2^8 和 2^16 量化等级。

第三个`read`函数在读取音频数据时，会将其记录在直接缓冲区 (`DirectBuffer`) 中，如果此缓冲区不是 `DirectBuffer` 则一直返回 0，也就是使用第三个`read`函数时传入的参数`audioBuffer`必须是一个 `DirectBuffer`，否则不能正确读取到音频数据，此时，该`Buffer`的`position`将保持不变，缓冲区中的数据的音频格式则取决于`AudioRecord`中指定的格式，且字节存放的方式为本机字节序。

#### 直接缓冲区和字节序

上面提到了两个概念直接缓冲区和字节序，这里简单说明一下：

##### 直接缓冲区

`DirectBuffer` 是 NIO 里面的东西，这里简单看下普通缓冲区和直接缓冲区的一些区别。

*   普通缓冲区
    

```
ByteBuffer buf = ByteBuffer.allocate(1024);
public static ByteBuffer allocate(int capacity) {
    if (capacity < 0)
        throw new IllegalArgumentException();
    return new HeapByteBuffer(capacity, capacity);
}
```

可知普通缓冲区从堆上分配一个字节缓冲区，该缓冲区受 JVM 的管理，意味着在合适的时候是可以被 GC 回收的，GC 回收伴随着内存的整理，某种程度上对性能是有影响的。

*   直接缓冲区
    

```
ByteBuffer buf = ByteBuffer.allocateDirect(1024);
public static ByteBuffer allocateDirect(int capacity) {
    // Android-changed: Android's DirectByteBuffers carry a MemoryRef.
    // return new DirectByteBuffer(capacity);
    DirectByteBuffer.MemoryRef memoryRef = new DirectByteBuffer.MemoryRef(capacity);
    return new DirectByteBuffer(capacity, memoryRef);
}
```

上面是 Android 中的`DirectBuffer` 的实现，可见是从内存中分配的，这种方式获得的缓冲区的获取成本是释放成本都是巨大的，但是可以驻留在垃圾回收堆的外部，一般分配给大型、寿命长的缓冲区，最后分配此缓冲区能够带来显著的性能提升才进行分配，是否是`DirectBuffer` 可以通过 `isDirect`来确定。

##### 字节序

字节序指的是字节在内存中的存放方式，字节序主要分为两类：BIG-ENDIAN 和 LITTLE-ENDIAN，通俗的称之为网络字节序和本机字节序，具体如下：

*   本机字节序，即 LITTLE-ENDIAN(小字节序、低字节序)，即低位字节排放在内存的低地址端，高位字节排放在内存的高地址端，与之对应的还有网络字节序。
    
*   网络字节序，一般指的是 TCP/IP 协议中使用的字节序，因为  TCP/IP  各层协议将字节序定义为 BIG-ENDIAN，所以网络字节序一般指的是 BIG-ENDIAN。
    

#### AudioRecord 的使用

记得在前面的文章 [Camera2、MediaCodec 录制 mp4](https://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484828&idx=1&sn=eed76722cda3a25ca589808be7b9e315&scene=21#wechat_redirect) 中只是录制了视频，侧重于`MediaCodec`的使用，这里将在视频录制的基础上使用`AudioRecord`添加音频的录制，并将其合成到`MP4`文件中，其关键步骤如下：

1.  开启一个线程使用`AudioRecord`读取硬件的音频数据，开线程可以避免卡顿，文末案例中也有代码示例，见 `AudioEncode2`，参考如下：
    

```
/**
 * 音频读取Runnable
 */
class RecordRunnable : Runnable{
    override fun run() {
        val byteArray = ByteArray(bufferSize)
        // 录制状态 -1表示默认状态，1表述录制状态，0表示停止录制
        while (recording == 1){
            val result = mAudioRecord.read(byteArray, 0, bufferSize)
            if (result > 0){
                val resultArray = ByteArray(result)
                System.arraycopy(byteArray, 0, resultArray, 0, result)
                quene.offer(resultArray)
            }
        }
        // 自定义流结束的数据
        if (recording == 0){
            val stopArray = byteArrayOf((-100).toByte())
            quene.offer(stopArray)
        }
    }
}
```

这里提一下，如果只是使用`AudioRecord`录制音频数据，当读取到音频数据可将音频数据写入文件即可。

2.  读取到音频数据要想合成到`MP4`中需要先进行音频数据的编码，音频数据编码器配置如下：
    

```
// 音频数据编码器配置
private fun initAudioCodec() {
    L.i(TAG, "init Codec start")
    try {
        val mediaFormat =
            MediaFormat.createAudioFormat(
                MediaFormat.MIMETYPE_AUDIO_AAC,
                RecordConfig.SAMPLE_RATE,
                2
            )
        mAudioCodec = MediaCodec.createEncoderByType(MediaFormat.MIMETYPE_AUDIO_AAC)
        mediaFormat.setInteger(MediaFormat.KEY_BIT_RATE, 96000)
        mediaFormat.setInteger(
            MediaFormat.KEY_AAC_PROFILE,
            MediaCodecInfo.CodecProfileLevel.AACObjectLC
        )
        mediaFormat.setInteger(MediaFormat.KEY_MAX_INPUT_SIZE, 8192)
        mAudioCodec.setCallback(this)
        mAudioCodec.configure(mediaFormat, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE)
    } catch (e: Exception) {
        L.i(TAG, "init error:${e.message}")
    }
    L.i(TAG, "init Codec end")
}
```

关于编码也就是`MediaCodec`的使用可以参考前面下面两篇文章：

*   [MediaCodec 详解](https://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484865&idx=1&sn=174b8ca702466e83e72c7115d91b06ea&scene=21#wechat_redirect)
    
*   [Camera2、MediaCodec 录制 mp4](https://mp.weixin.qq.com/s?__biz=MzU3NTA3MDU1OQ==&mid=2247484828&idx=1&sn=eed76722cda3a25ca589808be7b9e315&scene=21#wechat_redirect)
    

这里使用`MediaCodec`的异步处理模式进行音频数据的编码，这里将不贴代码了，注意一点就是填充和释放`Buffer`的时候一定要判断条件，如果`InputBuffer`一直不释放则会导致无可用的`InputBuffer`使用导致音频编码失败，还有就是流结束的处理。

3.  文件的合成使用`MediaMuxer`，`MediaMuxer`在启动之前必须确保添加好视轨和音轨
    

```
override fun onOutputFormatChanged(codec: MediaCodec, format: MediaFormat) {
    L.i(TAG, "onOutputFormatChanged format:${format}")
    // 添加音轨
    addAudioTrack(format)
    // 如果音轨和视轨都添加的情况下才启动MediaMuxer
    if (RecordConfig.videoTrackIndex != -1) {
        mAudioMuxer.start()
        RecordConfig.isMuxerStart = true
        L.i(TAG, "onOutputFormatChanged isMuxerStart:${RecordConfig.isMuxerStart}")
    }
}
// 添加音轨
private fun addAudioTrack(format: MediaFormat) {
    L.i(TAG, "addAudioTrack format:${format}")
    RecordConfig.audioTrackIndex = mAudioMuxer.addTrack(format)
    RecordConfig.isAddAudioTrack = true
}
// ...
```

`AudioRecord`的使用基本如上。