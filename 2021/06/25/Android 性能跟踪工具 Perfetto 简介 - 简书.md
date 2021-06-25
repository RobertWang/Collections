> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/34ee9353cd4c)

Perfetto 是 Android 10 中引入的全新平台级跟踪工具。这是适用于 Android、Linux 和 Chrome 的更加通用和复杂的开源跟踪项目。与 Systrace 不同，它提供数据源超集，可让您以 protobuf 编码的二进制流形式记录任意长度的跟踪记录。您可以在 [Perfetto 界面](https://links.jianshu.com/go?to=https%3A%2F%2Fui.perfetto.dev%2F%23%21%2F)（`此链接必须用Chrome打开`）中打开这些跟踪记录。  

![](http://upload-images.jianshu.io/upload_images/6471979-680461e93d755d9d.jpg) 图 1

上图为 Perfetto 跟踪记录视图示例，其中显示了与某个应用之间大约 20 秒的交互情况

#### 生成文件

进入 android sdk 目录下路径：  
AndroidSdk/platform-tools/systrace

运行以下命令生成文件

> python systrace.py [options] [category1] [category2] ... [categoryN]

生成的文件可用 Perfetto 打开显示的就为 `图1`信息。

### 参考文档

[系统跟踪概览](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Ftopic%2Fperformance%2Ftracing)