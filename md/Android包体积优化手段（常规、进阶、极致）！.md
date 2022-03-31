> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/cLxYF5T5x9DuyUrgtbejwg)

> 包大小的重要性已经不需要多说，包大小直接影响用户的下载，留存，甚至部分厂商预装强制要求必须小于一定的值。

前言
--

> 包大小的重要性已经不需要多说，包大小直接影响用户的下载，留存，甚至部分厂商预装强制要求必须小于一定的值。但是随着业务的迭代开发，应用会越来越大，安装包会不停的膨胀，因此包大小缩减是一个长期持续的治理过程。

*   提升下载转化率，安装包越小，转化率越高。
    
*   降低渠道推广成本。
    
*   降低安装时间，文件拷贝、Library解压、编译ODEX、签名校验这些，包体积越大越耗时。
    
*   降低运行时内存等等。
    

环境
--

*   Android Studio Arctic Fox | 2020.3.1 Patch 2
    
*   AGP 7.0
    
*   项目地址：wanandroid_jetpack
    

优化前
---

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATYhiaaWqUQGNdB3l1eZE3tHibD2YtNicZmKQ52ahDWRVgglFbSJYVp6PWg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

*   `4.7MB`，4.2MB是google play下载的大小，会有压缩。
    

除了AS自带的`Analyzer`之外，还有ApkChecker、ClassyShark等工具。

APK的组成
------

<table data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)"><thead data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)"><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white; box-sizing: border-box !important; overflow-wrap: break-word !important;"><th data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="outline: 0px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; text-align: left; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">文件</th><th data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="outline: 0px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; text-align: left; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">描述</th></tr></thead><tbody data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)"><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white; box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">lib</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">so文件，不同的cpu架构</td></tr><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">res</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;" class="">编译后的资源文件，drawable、layout等</td></tr><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white; box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">assets</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">应用程序的资源、字体、音频文件等</td></tr><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">classes(n).dex</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">dx编译后的java文件</td></tr><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white; box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">META-INF</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">签名信息相关</td></tr><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">resources.arsc</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">二进制资源文件</td></tr><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white; box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">kotlin</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">编译后的kotlin文件</td></tr><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">AndroidManifest.xml</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;" class="">清单文件</td></tr></tbody></table>

APK构建流程
-------

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATiahoefhWFcmOq1LP7PeHQNGvu9551W9dshmJxU2fI6JXdFFTWgCvEGQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)这是官方新版的打包流程，虽然省略了一些步骤，但是大致的流程还是比较清晰的。

再次简化一下：

```
资源文件、Java文件 > dex文件 > APK


```

优化思路
----

APK本质是一个压缩文件，是打包后的产物，那可以作为切入点的阶段就是打包前、以及打包中。

*   打包前，即减少打包的文件，比如无用的资源、代码；
    
*   打包中，对打包中的产物进行压缩，比如资源文件、So文件；
    

关键词：减少、压缩。

常规操作
----

### 1.Lint检测无用资源文件

```
Analyze > Run Inspection by Name > Unused resources


```

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATITT09ufe6GXBhhvfXtmicdECTYygOekZbfW01ZjQDtD056yX6XduUKQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATg4icozDbvukzGAAp11jQGyQloJkejYVGElNkkv9PFgzkSpKZVqOdqXg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

检测结果：

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATMS6OVnw7n0MnGxu7PGibYlCKtV0FH5naIewqdXjCAJVPVKWyibwicTESw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

确定无用删除即可。

**注意：** 因为lint是本地`静态扫描`，所以动态引用的资源文件并不会识别出来，也会出现在检测列表里。

### 2.Lint检测代码

```
Analyze > Inspect code 


```

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATYCzTFSic9a29OPyr3FGhetN7ysnGV80YGCcXgjgT5W4YWna8o5zbOMA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

检测结果：

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATRp7c9HVpI1scNUyfQAJiaxq7Sq3iaSNBDKYIFcuu0C6RPX9pj3RmicCaw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

因为这个项目是用kotlin写的，所以直接看kotlin目录下的检测结果。

**注意：** 因为lint是本地`静态扫描`，所以反射、动态引用的class并不会识别出来，也会出现在检测列表里。

### 3.图片压缩

推荐使用tinypng在线压缩。

### 4.TinyPngPlugin

手动压缩毕竟不高效，可以使用TinyPngPlugin一键压缩。

plugins搜索`TinyPng`安装即可。(新版AS安装完plugin已经不需要重启了)

压缩结果：

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATiarKbeJvHzG4hI1KjHWJqN8ViaopaZToRMgV0GzNxZkMtAnStenZKibbA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

9张图片，可以看到效果还是非常可观的。如果图片多，效果更加明显。

经过上面的操作，包体积减小`4%`，这还只是一个4.7MB的APK而已。

### 5.WebP

那这9张图还能继续优化吗？可以，WebP格式的体积更小，而已AS也提供了一键转换支持。

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATOGhBJibrImpT7THRwWgicYWsibWzPvjs3ZSib7KDibaat7TicbZNvOUoaLXQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

以ic_avatar.png为例：

<table data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)"><thead data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)"><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white; box-sizing: border-box !important; overflow-wrap: break-word !important;"><th data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="outline: 0px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; text-align: left; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">ic_avatar.png</th><th data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="outline: 0px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; text-align: left; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">优化后</th></tr></thead><tbody data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)"><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white; box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">原始大小</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">113.09KB</td></tr><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">TingPng压缩</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(248, 248, 248)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">36.85KB</td></tr><tr data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; max-width: 100%; border-width: 1px 0px 0px; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white; box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">WebP</td><td data-darkmode-color-16487232940789="rgb(163, 163, 163)" data-darkmode-original-color-16487232940789="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16487232940789="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16487232940789="#fff|rgb(255,255,255)" data-style="outline: 0px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 85px; overflow-wrap: break-word !important; box-sizing: border-box !important;">8.66KB</td></tr></tbody></table>

可以看到，转WebP之后，较原始大小减少了近`93%`，恐怖如斯~

### 6.开启混淆

minifyEnabled true，默认启用`R8`代码缩减功能。

```
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }


```

慎用R8，因为：

> R8 会忽略试图修改默认优化行为的所有 ProGuard 规则，例如 -optimizations 和 - optimizationpasses。

可以开启混淆，而不使用R8。

```
android.enableR8=false
android.enableR8.libraries=false


```

混淆参考：Android混淆从入门到精通

### 7.缩减资源

shrinkResources true

假如有一些资源文件不确定还用不用，也不敢删，或者不确定需求是否会变更，所以先留着，那这种情况怎么办呢? 可以使用shrinkResources来缩减资源。

```
    buildTypes {
        debug {
            minifyEnabled false
        }
        release {
            shrinkResources true
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }


```

要配合混淆minifyEnabled一起使用才行，原理也很简单，代码移除之后，引用的资源也就变成无用资源了，才可以进一步缩减。

### 8.so文件缩减

比如集成了一个三方的直播或者浏览器，可能会提供很多so文件，起初可能是一股脑的copy进项目，但并不一定都用的到。

比如各种cpu架构的so：

```
app/build/intermediates/cmake/universal/release/obj/
├── armeabi-v7a/
│   ├── libgameengine.so
│   ├── libothercode.so
│   └── libvideocodec.so
├── arm64-v8a/
│   ├── libgameengine.so
│   ├── libothercode.so
│   └── libvideocodec.so
├── x86/
│   ├── libgameengine.so
│   ├── libothercode.so
│   └── libvideocodec.so
└── x86_64/
    ├── libgameengine.so
    ├── libothercode.so
    └── libvideocodec.so


```

目前市面上的手机cpu都是`arm`架构的，所以保留arm的一种即可（定制的除外），`armeabi-v7a`或`armeabi`都可，其他直接删除。

```
android {
    defaultConfig {
        ndk {
            abiFilters 'armeabi-v7a'
        }
    }
}


```

如果开发需要模拟器调试，就加上`x86`的架构，正式包记得去掉，或者在`local.properties`中用变量控制一下。

这块如果之前没有优化过，而又有很多so文件的话，或许可以减少30%以上，恐怖如斯！

### 9.移除未使用的备用资源

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATJ1t5JLaiaM4jBFTR0FEiaicIpK0f1nAdvbZe6DtQ8csJicS202I1Thfialw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

很多出海的应用会做国际化，但也适配不了这么多的语言。除了自己app的之外，还有一些官方的、三方的，可以统一配置支持的语言。

```
    defaultConfig {        resConfigs("en","zh","zh-rCN")    }


```

资源文件同理

```
    defaultConfig {        resConfigs("xxhdpi","xxxhdpi")    }


```

### 10.小结

针对上面的操作做个小结，看看目前效果如何。

![图片](https://mmbiz.qpic.cn/mmbiz/FoiciaVBBCfia7Otf0klL6VibpMNGupxQbATUk1icm7WuomFKhS9pTRexyrI3l1SPuA8l5CvpFAWTMydmQ48Zujib4Uw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

`2MB`，包体积减少`57%`，恐怖如斯！

如果是大型项目，收益非常可观。

> author：yechaoa

进阶操作
----

上面只是一些常规操作，下面看一些进阶操作。

### 1.resources.arsc资源混淆

资源混淆就是将原本冗长的资源路径变短，例如将res/drawable/wechat变为r/d/a。开源工具AndResGuard。

### 2.移除无用的三方库

引入之后未使用的，或者是功能下架之后未移除的。

### 3.功能重复的三方库整合

比如glide和picasso，都是图片库，保留其一即可。

### 4.ReDex

dex文件是打包中的产物，redex是facebook开源的分包优化方案。可以参考：ReDex。

### 5.so动态加载

前面已经做了so文件缩减，但是可能so文件占比还是比较大，可以考虑除了首次启动外的so文件做动态下发。也就是插件化的思想，按需加载，但是收益很大的同时，风险也很大，有很多case需要考虑到，比如下载时机、网络环境、线程进程，加载失败是否有降级策略等等。

可以参考facebook开源的SoLoader。

### 6.插件化

按需加载，收益越大风险越大，风险同上。

极致操作
----

那如果我想做到极致，还有哪些骚操作呢，ok，继续。

### 1.原生改用H5或小程序等方案

有些功能可能原生做就显得太重，比如各种促销活动，需要加载各种大图，原生既重又不够动态化，这个时候H5是一种很好的替代方案。但是如果你原本就不支持H5或者小程序的话，接入这种能力可能反而会加大包体积，做好对比。

### 2.砍功能

有些功能可能想的很美好，但上线之后收益并不大，是否需要重新思考价值点，最好找到数据依托，再跟产品打架。

### 3.修改三方库的源码，不需要的代码剔除

比如引入了一个功能很齐全的三方库utils，但实际只用到几个，对源码进行抽取也能减少包体积，同时还能减少网络下载的编译时间。

弊端就是升级成本较大。

### 4.图片网络化

即把图片上传到服务器，通过动态下载的方式减少包体积，弊端就是首次加载的时候依赖网络环境，对加载速度、流量需要做一个平衡。图片可以预加载，但是流量消耗是无法避免了，如果比较在意流量指标，需要权衡了。

### 5.DebugItem

DebugItem 里面主要包含两种信息：

*   调试的信息。函数的参数变量和所有的局部变量。
    
*   排查问题的信息。所有的指令集行号和源文件行号的对应关系。
    

去除debug信息与行号信息，如果不是极致，不推荐。可以参考支付宝的这篇 支付宝 App 构建优化解析：Android 包大小极致压缩。

### 6.R Field内联

内联R Field可以解决R Field过多导致MultiDex 65536的问题，而这一步骤对代码瘦身能够起到明显的效果。

美团代码片段：

```
ctBehaviors.each { CtBehavior ctBehavior ->
    if (!ctBehavior.isEmpty()) {
        try {
            ctBehavior.instrument(new ExprEditor() {
                @Override
                public void edit(FieldAccess f) {
                    try {
                        def fieldClassName = JavassistUtils.getClassNameFromCtClass(f.getCtClass())
                        if (shouldInlineRField(className, fieldClassName) && f.isReader()) {
                            def temp = fieldClassName.substring(fieldClassName.indexOf(ANDROID_RESOURCE_R_FLAG) + ANDROID_RESOURCE_R_FLAG.length())
                            def fieldName = f.fieldName
                            def key = "${temp}.${fieldName}"

                            if (resourceSymbols.containsKey(key)) {
                                Object obj = resourceSymbols.get(key)
                                try {
                                    if (obj instanceof Integer) {
                                        int value = ((Integer) obj).intValue()
                                        f.replace("\$_=${value};")
                                    } else if (obj instanceof Integer[]) {
                                        def obj2 = ((Integer[]) obj)
                                        StringBuilder stringBuilder = new StringBuilder()
                                        for (int index = 0; index < obj2.length; ++index) {
                                            stringBuilder.append(obj2[index].intValue())
                                            if (index != obj2.length - 1) {
                                                stringBuilder.append(",")
                                            }
                                        }
                                        f.replace("\$_ = new int[]{${stringBuilder.toString()}};")
                                    } else {
                                        throw new GradleException("Unknown ResourceSymbols Type!")
                                    }
                                } catch (NotFoundException e) {
                                    throw new GradleException(e.message)
                                } catch (CannotCompileException e) {
                                    throw new GradleException(e.message)
                                }
                            } else {
                                throw new GradleException("******** InlineRFieldTask unprocessed ${className}, ${fieldClassName}, ${f.fieldName}, ${key}")
                            }
                        }
                    } catch (NotFoundException e) {
                    }
                }
            })
        } catch (CannotCompileException e) {
        }
    }
}

```

同时可以参考字节开源的shrink-r-plugin，还有滴滴开源的booster。

### 7.图片着色器

针对同图不同色的处理，可以使用`tint`，比如原本是一个黑色的返回icon，现在另一个页面要用白色了，就不需要两张图了，而是使用tint来修改为白色即可。

```
  <ImageView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:src="@drawable/ic_back_black"
                android:tint="@android:color/white" />


```

### 8.减少ENUM的使用

每减少一个`ENUM`可以减少大约1.0到1.4 KB的大小。

包体积监控
-----

包体积监控应该作为发布流程的一个环节，最好是做到平台化、流程化，否则很难持续，没几个版本包体积又涨上来了。

大致思想：当前版本与上一个版本的包大小做对比，超过200KB需要审批。临时审批需要给出后续优化方案等等。

参考文档
----

*   Improve your code with lint checks  
    https://developer.android.com/studio/write/lint.html  
    
*   Shrink, obfuscate, and optimize your app  
    https://developer.android.google.cn/studio/build/shrink-code#shrink-resources  
    
*   Android App包瘦身优化实践  
    https://tech.meituan.com/2017/04/07/android-shrink-overall-solution.html  
    
*   ReDex  
    https://github.com/facebook/redex  
    
*   SoLoader  
    https://github.com/facebook/SoLoader  
    
*   支付宝 App 构建优化解析：Android 包大小极致压缩  
    https://mp.weixin.qq.com/s/_gnT2kjqpfMFs0kqAg4Qig  
    
*   AndResGuard  
    https://github.com/shwenzhang/AndResGuard/blob/master/README.zh-cn.md  
    
*   深入探索 Android 包体积优化  
    https://juejin.cn/post/6872920643797680136  
    
*   Android开发高手课包体积优化  
    https://time.geekbang.org/column/article/8120