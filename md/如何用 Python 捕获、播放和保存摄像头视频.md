> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652594197&idx=1&sn=c97be88286c6a6a5f6cdfaaf97009f60&chksm=8465485fb312c149d5ee6819e32f745476ae52cc516e0cd9fa5653ffe52f5b3f5f0a615601b6&mpshare=1&scene=1&srcid=0615ETz8UkMT2hiRqb5Sl4Wp&sharer_sharetime=1655283030002&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

今天分享一下 Python 操作视频最基本的操作，包括读取和播放视频和保存视频。  

### 读取视频

#### 从相机中读取视频

对于有摄像头的设备，例如带摄像头的笔记本电脑，我们可以直接调起电脑的摄像头，读取摄像头的视频流。

`cv.imshow` 方法用来显示视频的帧。我们播放视频的原理就是逐帧播放。

在最后，不要忘记通过 `cap.release()` 释放俘虏。

运行这段代码，你就可以看到一个弹窗实时地播放你电脑摄像头中的图像了。

#### 从文件中播放视频

与从相机捕获相同，只是用视频文件名更改摄像机索引。

另外，在显示视频时，可以通过 `cv.waitKey()` 来控制视频播放的速度。如果设置太小，则视频将非常快，相当于倍速播放；而如果太大，则视频将变得很慢，相当于延迟播放。正常情况下 25 毫秒就可以了。

```
import cv2 as cvcap = cv.VideoCapture(0)if not cap.isOpened():    print("Cannot open camera")    exit()while True:    # 逐帧捕获    ret, frame = cap.read()    # 如果正确读取帧，ret为True    if not ret:        break    # 显示结果帧    cv.imshow('frame', frame)    if cv.waitKey(1) == ord('q'):        break# 完成所有操作后，释放捕获器cap.release()cv.destroyAllWindows()
```

运行这段代码，你就可以看到一个弹窗播放你选择的视频文件了。

### 保存视频

从相机读取视频，我们可以将视频保存到本地。我们捕捉一个视频，一帧一帧地处理，如果我们想要保存这个视频，非常简单，只需使用 `cv.VideoWriter()`。

*   参数 1：输出文件名，例如: output.mp4。
    
*   参数 2：FourCC 代码，FourCC 是用于指定视频编解码器的 4 字节代码。
    
*   参数 3：帧率的数量。
    
*   参数 4：帧大小。
    
*   参数 5：颜色标志。如果为 True，正常颜色输出，否则就是灰色图像输出。
    

> cv2.VideoWriter_fourcc(‘P’,‘I’,‘M’,‘1’) = MPEG-1 codec cv2.VideoWriter_fourcc(‘M’,‘J’,‘P’,‘G’) = motion-jpeg codec --> mp4v cv2.VideoWriter_fourcc(‘M’, ‘P’, ‘4’, ‘2’) = MPEG-4.2 codec cv2.VideoWriter_fourcc(‘D’, ‘I’, ‘V’, ‘3’) = MPEG-4.3 codec cv2.VideoWriter_fourcc(‘D’, ‘I’, ‘V’, ‘X’) = MPEG-4 codec --> avi cv2.VideoWriter_fourcc(‘U’, ‘2’, ‘6’, ‘3’) = H263 codec cv2.VideoWriter_fourcc(‘I’, ‘2’, ‘6’, ‘3’) = H263I codec cv2.VideoWriter_fourcc(‘F’, ‘L’, ‘V’, ‘1’) = FLV1 codec

```
import cv2 as cvcap = cv.VideoCapture('video.mp4')while cap.isOpened():    ret, frame = cap.read()    # 如果正确读取帧，ret为True    if not ret:        break    cv.imshow('frame', frame)    if cv.waitKey(1) == ord('q'):        breakcap.release()cv.destroyAllWindows()
```

上面几段代码中，如果想要退出视频操作，敲击键盘的 `q` 就可以。

### 总结

以上就是今天要介绍的内容了，使用 python-opencv 来操作视频还是比较简单的。

- EOF -