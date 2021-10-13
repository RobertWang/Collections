> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weiwangchao_/article/details/71404600?locationNum=5&fps=1)

1、ffmpeg 推送视频文件，音视频的编码格式只能为 H264、AAC。

ffmpeg -re -i "E:\ 片源 \ 复仇者联盟 720p.mov" -vcodec copy -acodec copy -f flv rtmp://192.168.11.75/live/test1   
ffmpeg -re -i "E:\ 片源 \ 复仇者联盟 720p.mov" -vcodec copy -acodec copy -f flv rtmpt://192.168.11.75:8080/live/test1

![](http://www.cuplayer.com/player/uploads/allimg/170324/1G11VK3-0.jpg)

2、网络摄像机 rtsp 流转推 rtmp 直播 (不过有丢包情况，还请大家多给指点)

ffmpeg -i rtsp://ip address/original -crf 30 -preset ultrafast -acodec aac -strict experimental -ar 44100 -ac 2 -b:a 96k -vcodec libx264 -r 25 -b:v 500k -s 640*480 -f flv rtmp://ip address/live/stram

ffmpeg 参数说明：

```
 
基本选项:
-formats	输出所有可用格式
-f
 fmt	指定格式(音频或视频格式)
-i filename	指定输入文件名，在linux下当然也能指定:0.0(屏幕录制)或摄像头
-y	覆盖已有文件
-t duration	记录时长为t
-fs limit_size	设置文件大小上限
-ss time_off	从指定的时间(s)开始，
 [-]hh:mm:ss[.xxx]的格式也支持
-itsoffset time_off	设置时间偏移(s)，该选项影响所有后面的输入文件。该偏移被加到输入文件的时戳，定义一个正偏移意味着相应的流被延迟了 offset秒。 [-]hh:mm:ss[.xxx]的格式也支持
-title string	标题
-timestamp time	时间戳
-author
 string	作者
-copyright string	版权信息
-comment string	评论
-album string	album名
-v verbose	与log相关的
-target type	设置目标文件类型("vcd", "svcd",
 "dvd", "dv", "dv50", "pal-vcd", "ntsc-svcd", ...)
-dframes number	设置要记录的帧数
视频选项:
-b	指定比特率(bits/s)，似乎ffmpeg是自动VBR的，指定了就大概是平均比特率
-bitexact	使用标准比特率
-vb	指定视频比特率(bits/s)
-vframes
 number	设置转换多少桢(frame)的视频
-r rate	帧速率(fps) （可以改，确认非标准桢率会导致音画不同步，所以只能设定为15或者29.97）
-s size	指定分辨率 (320x240)
-aspect aspect	设置视频长宽比(4:3, 16:9 or 1.3333, 1.7777)
-croptop
 size	设置顶部切除尺寸(in pixels)
-cropbottom size	设置底部切除尺寸(in pixels)
-cropleft size	设置左切除尺寸 (in pixels)
-cropright size	设置右切除尺寸 (in pixels)
-padtop size	设置顶部补齐尺寸(in
 pixels)
-padbottom size	底补齐(in pixels)
-padleft size	左补齐(in pixels)
-padright size	右补齐(in pixels)
-padcolor color	补齐带颜色(000000-FFFFFF)
-vn	取消视频
-vcodec
 codec	强制使用codec编解码方式('copy' to copy stream)
-sameq	使用同样视频质量作为源（VBR）
-pass n	选择处理遍数（1或者2）。两遍编码非常有用。第一遍生成统计信息，第二遍生成精确的请求的码率
-passlogfile file	选择两遍的纪录文件名为file
-newvideo	在现在的视频流后面加入新的视频流
 
高级视频选项
-pix_fmt
 format	set pixel format, 'list' as argument shows all the pixel formats supported
-intra	仅适用帧内编码
-qscale q	以<数值>质量为基础的VBR，取值0.01-255，约小质量越好
-loop_input	设置输入流的循环数(目前只对图像有效)
-loop_output	设置输出视频的循环数，比如输出gif时设为0表示无限循环
-g
 int	设置图像组大小
-cutoff int	设置截止频率
-qmin int	设定最小质量，与-qmax（设定最大质量）共用，比如-qmin 10 -qmax 31
-qmax int	设定最大质量
-qdiff int	量化标度间最大偏差 (VBR)
-bf
 int	使用frames B 帧，支持mpeg1,mpeg2,mpeg4
音频选项:
-ab	设置比特率(单位：bit/s，也许老版是kb/s)前面-ac设为立体声时要以一半比特率来设置，比如192kbps的就设成96，转换 默认比特率都较小，要听到较高品质声音的话建议设到160kbps（80）以上。
-aframes number	设置转换多少桢(frame)的音频
-aq
 quality	设置音频质量 (指定编码)
-ar rate	设置音频采样率 (单位：Hz)，PSP只认24000
-ac channels	设置声道数，1就是单声道，2就是立体声，转换单声道的TVrip可以用1（节省一半容量），高品质的DVDrip就可以用2
-an	取消音频
-acodec codec	指定音频编码('copy'
 to copy stream)
-vol volume	设置录制音量大小(默认为256) <百分比> ，某些DVDrip的AC3轨音量极小，转换时可以用这个提高音量，比如200就是原来的2倍
-newaudio	在现在的音频流后面加入新的音频流
字幕选项:
-sn	取消字幕
-scodec
 codec	设置字幕编码('copy' to copy stream)
-newsubtitle	在当前字幕后新增
-slang code	设置字幕所用的ISO 639编码(3个字母)
Audio/Video 抓取选项:
-vc channel	设置视频捕获通道(只对DV1394)
-tvstd
 standard	设



```

```
转换为flv: 
    ffmpeg -i test.mp3 -ab 56 -ar 22050 -b 500 -r 15 -s 320x240 test.flv 
    ffmpeg -i test.wmv -ab 56 -ar 22050 -b 500 -r 15 -s 320x240 test.flv 

 转换文件格式的同时抓缩微图： 
    ffmpeg -i "test.avi" -y -f image2 -ss 8 -t 0.001 -s 350x240 'test.jpg' 

  对已有flv抓图： 
    ffmpeg -i "test.flv" -y -f image2 -ss 8 -t 0.001 -s 350x240 'test.jpg' 

  转换为3gp:
    ffmpeg -y -i test.mpeg -bitexact -vcodec h263 -b 128 -r 15 -s 176x144 -acodec aac -ac 2 -ar 22500 -ab 24 -f 3gp test.3gp 
    ffmpeg -y -i test.mpeg -ac 1 -acodec amr_nb -ar 8000 -s 176x144 -b 128 -r 15 test.3gp 


```

[相关链接：FFMPEG 提供的命令行，FFMPEG 好强大](http://www.cuplayer.com/player/PlayerCode/FFmpeg/2016/0804/2468.html)