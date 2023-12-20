> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/669269507)

ffmpeg 简介与图片格式介绍
----------------

windows 安装 ffmpeg，从如下网站下载 release 版本 [https://www.gyan.dev/ffmpeg/builds/](https://link.zhihu.com/?target=https%3A//www.gyan.dev/ffmpeg/builds/)

ffmpeg 6.1 版本仍然不支持 heic 的图片格式，未来可能会支持，具体见该 issue： [https://trac.ffmpeg.org/ticket/6521](https://link.zhihu.com/?target=https%3A//trac.ffmpeg.org/ticket/6521)

图片格式压缩率：jpeg < webp < heif < avif

图片压缩率比较高的还有个 jxl 格式，但是太新了而且各个平台支持都不到位，只能战未来了。

avif 格式的图片由于 android、windows、macos、各大浏览器都支持，未来覆盖率必然超过 heif，heif 由于专利授权原因，在浏览器上覆盖很差，chrome 浏览器打不开 heif 格式图片。不过 avif 格式由于比较新，目前在老版本的系统、浏览器中支持率还不够。

avif 格式其实跟 heif 格式类似，都是使用视频编码器技术应用到图片领域，heif 的编码器用的跟 h.265\hevc 是一样的，avif 则是使用 av1 编码器，av1 编码器压缩率比 hevc 更高，目前国内 B 站算是比较早使用 av1 编码视频以及 avif 编码图片的平台，其技术团队有相关公开分享它们在编码器领域的研究。

网页端的图片格式转换工具，推荐使用 [https://squoosh.app/](https://link.zhihu.com/?target=https%3A//squoosh.app/) ，图片格式支持非常全面。

查看图片格式详情
--------

```
ffprobe example.webp

```

图片格式转换
------

### jpeg 转 avif

```
ffmpeg -i source.jpg -c:v libaom-av1 -still-picture 1 dest.avif

```

### 转换 jpeg 为 avif debug 级别 日志

```
ffmpeg -v debug -i source.jpg -c:v libaom-av1 -still-picture 1 dest.avif

```

### 转换 jpeg 为 webp

```
ffmpeg -i source.jpg -c:v libwebp dest.webp

```

### 转换 jpeg 为 jxl

```
ffmpeg -i source.jpg -c:v libjxl dest.jxl

```

### 转换 avif 为 webp

```
ffmpeg -i source.avif -c:v libwebp dest.webp

```

```
ffmpeg -i source.avif  dest.jpg

```

```
ffmpeg -i source.webp -c:v libwebp dest.webp

```