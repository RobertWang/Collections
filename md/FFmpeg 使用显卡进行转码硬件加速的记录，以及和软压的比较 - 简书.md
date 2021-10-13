> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/1d645b9d26d5)

1. 环境
-----

操作系统：Windows 10  
FFmpeg 版本：20171204  
显卡：GTX 965M

2. 过程
-----

最近是有比较多的压制需求，使用 libx265 软压的速度实在是慢的受不了，所以还是希望能用显卡硬压起码速度快一点。之前有人跟我提过硬压质量似乎不及软压，但是决定还是试一试。在 ffmpeg 官网找到硬压的[相关信息](https://link.jianshu.com?t=https://trac.ffmpeg.org/wiki/HWAccelIntro)。  
由于我用的是 windows，所以驱动基本没有特别配置。而且 windows 版的 ffmpeg 也是参数配置好的，所以这方面没有考虑太多。linux 平台可能需要配置一下参数啥的。  
压制分为两步，先是对视频解码再编码。ffmpeg 在两步都提供了硬件加速方案。  
在官网给出的例子是基于 h264 的，h265 的硬件参数啥的可以用：

`ffmpeg -codecs | sls cuvid` _(备注：sls 是 powershell 的命令，类似于 linux 下的 grep 命令)_

可以看到这条：

`DEV.L. hevc H.265 / HEVC (High Efficiency Video Coding) (decoders: hevc hevc_qsv hevc_cuvid ) (encoders: libx265 nvenc_hevc hevc_nvenc hevc_qsv )`

解码器提供了`hevc`, `hevc_qsv`, `hevc_cuvid`; 编码器提供了`libx265`, `nvenc_hevc`, `hevc_nvenc`, `hevc_qsv`，但是这个`nvenc_hevc`其实已经作废了，你用它的话他会提示你自动给你转到`hevc_nvenc`。  
解码器的这三个用法我是不太懂有啥区别，也没去做太多研究，因为在实践中使用硬解的话是没办法同时硬压字幕的，会报错，况且硬解对于整体压制速度并没有太大提升，所以就抛弃硬解了。  
编码器的部分，`libx265`就是软压，`hevc_qsv`似乎是英特尔的集显硬压，具体看[这里](https://link.jianshu.com?t=https://www.intel.com/content/dam/www/public/us/en/documents/white-papers/cloud-computing-quicksync-video-ffmpeg-white-paper.pdf)。那么留给 n 卡的只有`hevc_nvenc`可以用了。  
使用这条命令来查看该方法的参数：

`ffmpeg -h encoder=hevc_nvenc`

可以得到可用参数，我们这里探究的是 - cq 参数，给出的描述是：

`-cq <float> E..V.... Set target quality level (0 to 51, 0 means automatic) for constant quality mode in VBR rate control (from 0 to 51) (default 0)`

我感兴趣的原因是它和 libx265，也就是软压的 - crf 参数很类似。所以接下来都是在其他参数不考虑的情况下对不同 cq 的对比。

### 3. 不同 cq 值的对比

我用的是谍影重重 5 的预告片压制测试，原视频数据如下：

```
Format                      : MPEG-4
Format profile              : QuickTime
Codec ID                    : qt   2005.03 (qt  )
File size                   : 35.6 MiB
Duration                    : 30 s 30 ms
Overall bit rate            : 9 938 kb/s
Encoded date                : UTC 2016-02-08 06:40:30
Tagged date                 : UTC 2016-02-08 06:40:30
Writing library             : Apple QuickTime 7.7.3


```

在使用命令  
`ffmpeg -i original.mov -c:v hevc_nvenc -cq X cqx.mp4`  
进行测试后。结果如下：  
**Libx265 （软压）**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 5.53 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 1 544 kb/s
Writing application         : Lavf58.2.103


```

**-cq 0(默认)**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 8.21 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 2 290 kb/s
Writing application         : Lavf58.2.103


```

**-cq 1**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 8.21 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 2 290 kb/s
Writing application         : Lavf58.2.103


```

**-cq 10**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 8.21 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 2 290 kb/s
Writing application         : Lavf58.2.103


```

**-cq 20**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 8.21 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 2 290 kb/s
Writing application         : Lavf58.2.103


```

**-cq 30**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 8.20 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 2 286 kb/s
Writing application         : Lavf58.2.103


```

**-cq 35**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 5.52 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 1 539 kb/s
Writing application         : Lavf58.2.103


```

**-cq 38**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 4.06 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 1 132 kb/s
Writing application         : Lavf58.2.103


```

**-cq 41**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 3.06 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 855 kb/s
Writing application         : Lavf58.2.103


```

**-cq 51**

```
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/mp41)
File size                   : 1.41 MiB
Duration                    : 30 s 70 ms
Overall bit rate            : 392 kb/s
Writing application         : Lavf58.2.103


```

对比视频在[这里](https://link.jianshu.com?t=https://www.bilibili.com/video/av17105682)。

4. 结论
-----

可以看到 cq 在 1 到 30 的变化并不大，在 41 以上画面基本上是没办法看了。在和 libx265 的默认软压对比后，-cq 值落在 35 到 40 之间是比较好的选择。  
在后续的实际应用中，我在压制画面动作较少的视频，如交响乐视频的情况下，-cq 37 是一个对于我来说比较好的选择。