> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/riPkWglkEgeG6EHhF695Gw)

> fmpeg 是一套跨平台的，用于音视频录制、转换、流化等操作的完善的解决方案，它是业界最负盛名的开源音视频框架

> `ffmpeg`是一套跨平台的，用于音视频录制、转换、流化等操作的完善的解决方案，它是业界最负盛名的开源音视频框架之一。许多软件都是基于 ffmpeg 开发的，本文也将 fmpeg 常用的玩法加以总结。

安装
==

这里，我们用 Windows 平台作为安装示例。首先到官方进行下载程序 _https://www.ffmpeg.org/download.html#build-windows_

![](https://mmbiz.qpic.cn/mmbiz_png/Xb3L3wnAiatgAcJ7BSXEV7XghZicj8cvZhE9mvNuCjUuoa8Raib724x1mBnA88cjuxYhbaDNRI9nAV82O4vUuRKIA/640?wx_fmt=png)

下载完成后，解压目录如下所示

![](https://mmbiz.qpic.cn/mmbiz_png/Xb3L3wnAiatgAcJ7BSXEV7XghZicj8cvZhnj20F18s3AfsI91Mks9M30xiaeX2kUd5Wc2357ibrSTWm9e5B7TTJs5g/640?wx_fmt=png)

​核心的可执行文件在`bin`目录下面。分别有三个可执行文件，其作用分别为：

*   `ffmpeg`：音视频转码、转换器
    
*   `ffplay`：简单的音视频播放器
    
*   `ffprobe`：简单的多媒体码流分析器 为了方便，我们也可以将上面三个文件添加到我们的系统环境变量中去。当然直接在目录中运行也是可以的。
    

![](https://mmbiz.qpic.cn/mmbiz_png/Xb3L3wnAiatgAcJ7BSXEV7XghZicj8cvZhDpDaQuCbFpxIjCSKIt0KxhBWHUtByt3lFky9fXEkDK1axCq6AeAXicA/640?wx_fmt=png)**验证**

我们在文件根目录运行下面命令

```
ffmpeg.exe -h


```

![](https://mmbiz.qpic.cn/mmbiz_png/Xb3L3wnAiatgAcJ7BSXEV7XghZicj8cvZhRxL17V3bRtN0ayfCaemOJUepfjxibmoMvJpTEU2KFWJ3KG3RNcZ878Q/640?wx_fmt=png)

这样，我们便安装完成了。当然如果不嫌麻烦，可以在 Linux 环境中通过源码编译安装😘

日常操作 最为适用
=========

#### 🎃下载 m3u8

现在很多视频网站都是通过 m3u8 的方式进行在线播放。我们只需通过 f12 复制视频的 m3u8 地址。便可以直接对在线视频进行下载。很实用！

![](https://mmbiz.qpic.cn/mmbiz_png/Xb3L3wnAiatgAcJ7BSXEV7XghZicj8cvZhEdia1RDZgcJkpgJJAKCl2xChFFDObJAYicTSvyf6fBuzoD8ibjeRJHmuA/640?wx_fmt=png)

```
ffmpeg.exe -i "https://vip.lz-cdn5.com/20220620/26152_476d2df1/1200k/hls/mixed.m3u8" 二狗子.mp4


```

`-i` 后面跟 m3u8 地址就行了。下载过程会自动将 ts 文件合并为 MP4 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/Xb3L3wnAiatgAcJ7BSXEV7XghZicj8cvZhqdcsibnr7D2HEBAuzCU1d9Y1q73Via16YbHO1IkH5gOSu3y4sicqkibzVQ/640?wx_fmt=png)下载完后后效果

#### 🎉提取视频中的音频

```
ffmpeg.exe -i aa.mp4 -vn -c:a copy output.aac


```

`-vn` 表示去掉视频，`-c:a copy`表示不改变音频编码，直接拷贝。

#### 🎨指定时间截图

```
ffmpeg.exe -ss 0:8:34 -i 二狗子.mp4 -vframes 1 -q:v 2 output.jpg


```

-vframes 1 指定只截取一帧，-q:v 2 表示输出的图片质量，一般是 1 到 5 之间（1 为质量最高）

![](https://mmbiz.qpic.cn/mmbiz_jpg/Xb3L3wnAiatgAcJ7BSXEV7XghZicj8cvZhoAhIHPJsLKkB6ic6jzuv8J6jm5GS9EyCFyrDZTL7SzZ8ernCj3iaW01A/640?wx_fmt=jpeg)

#### 🐼截取某时间段的视频

```
ffmpeg -ss 0:0:19 -i .\test.mp4 -to 0:13:11 -c copy test-t.mp4


```

前面的时间是开始时间，后面的时间是结束时间`-c copy`表示不对视频进行任何转码或修改，只截取视频

#### 🚣🏻为音频添加封面

```
ffmpeg -loop 1 -i cover.jpg -i input.mp3 -c:v libx264 -c:a aac -b:a 192k -shortest output.mp4


```

上面命令中，有两个输入文件，一个是封面图片`cover.jpg`，另一个是音频文件`input.mp3`。`-loop 1`参数表示图片无限循环，`-shortest`参数表示音频文件结束，输出视频就结束。

#### 🌽MP4 转 M3U8

```
ffmpeg -i input.mp4 -c:v libx264 -c:a aac -strict -2 -f hls -hls_list_size 2 -hls_time 15 output.m3u8


```

将`input.mp4` 视频文件每 `15`秒生成一个 `ts` 文件，最后生成一个 m3u8 文件，m3u8 文件是 ts 的索引文件