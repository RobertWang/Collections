> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_44256803/article/details/100152659)

Windows 下载 ffmpeg：[https://ffmpeg.zeranoe.com/builds/](https://ffmpeg.zeranoe.com/builds/)  
添加环境变量（省略操作步骤）：在 PATH 中加入 ffmpeg 二进制目录路径（例如：D:\Program Files (x86)\ffmpeg\bin）  
CMD 中输入命令：

```
ffmpeg -protocol_whitelist "file,http,https,rtp,udp,tcp,tls" -i <path to m3u8 file> <path to output file>

```

例如：m3u8 文件放在桌面，要输出的 output.mp4 也放桌面则命令应该为

```
ffmpeg -protocol_whitelist "file,http,https,rtp,udp,tcp,tls" -i C:\Users\username\Desktop\m3u8 C:\Users\username\Desktop\output.mp4

```

当然如果直接在桌面打开 CMD 窗口就可以直接用文件名而不需要绝对路径了：

```
ffmpeg -protocol_whitelist "file,http,https,rtp,udp,tcp,tls" -i m3u8 output.mp4

```

注意：若 m3u8 中存在`#EXT-X-DISCONTINUITY`语句则表示视频不连续，例如某些视频网站需要在播放途中插入广告，这种应用场景下则会出现该语句。ffmpeg 是不支持该语句的，不过 bilibili 版本的 ffmpeg 好像支持？（[https://github.com/Bilibili/FFmpeg](https://github.com/Bilibili/FFmpeg)）

临时解决方案是片段不多时手动分成多个连续 m3u8 下载后再合并视频  
[https://blog.csdn.net/doublefi123/article/details/47276739](https://blog.csdn.net/doublefi123/article/details/47276739)

2020 年 12 月 8 日  
若有 Invalid data found when processing input 报错，加上选项：-allowed_extensions ALL