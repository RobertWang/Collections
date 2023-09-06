> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MtzfVfyWGO5VErsjbtQFsQ)

开发一款 App，难不难？

软件工程师 Thomas SIMON 闲暇之余用实践回答道：其实说难也不难。

当他拿起一个 Android 手机，想要将其作为电脑摄像头和麦克风使用时，他只写了 40 行代码就实现了这一功能。

当他把自己的这段经历分享到网上时，没想到，引起了多位开发者的共鸣，纷纷表示，这种感觉太熟悉了。

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuiaWuOOltcXGw7ib2k7VxjuFicC232shJptArjJS7Eic0bTxgUH0aOKicLhFjRsHU2PpyiakjUdibU4J6nXw/640?wx_fmt=png)

接下来，我们将从 Thomas SIMON 的经历中了解开发者那些本该避开的坑。

**![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAujX5kS5KQ6BaBUsy1RqR06QuwjkSP1G6wEJHaJCLTONqlcQexqRgJcIICxofIOJs6B6tWBfibb7now/640?wx_fmt=png)**

**需要一个 App，把 Android 手机变成电脑摄像头和麦克风**

之所以想要开发一款 App，是因为作为一名软件工程师，Thomas SIMON 平时保持着视频录制分享技术的习惯。

此前，他主要使用 Android 手机来录制视频。与台式机上的 Linux 系统相比，Android 系统的兼容性很好，而且它很少出错，即使出错，也是以众人可接受的方式，譬如电池电量不足、系统更新等等。

后来，Thomas SIMON 决定和一位朋友一起录制他们技术分享，并将其作为播客片段上传到网络上，期间需要对录制的文件进行剪辑、制作等等，还是电脑好操作一些。

不过，Thomas SIMON 的 Linux 系统电脑并没有摄像头，所以在他的计划中，原来是想要将 Linux 电脑作为主设备进行录制，然后使用手机上的摄像头和麦克风加入通话。

然而，在录制期间需要管理两台设备非常麻烦，况且搭载 Linux 系统的电脑设备本身就有一个不错的麦克风，所以，此时只要有一个 Linux 兼容的网络摄像头存在就可以解决难题。

在研究市场上现有的网络摄像头之后，Thomas SIMON 认为那些设备不仅价格昂贵，而且质量还不如几年前的中档手机后置摄像头。

恰巧他手头正好有一部三星 S20，手机的后置镜头是长焦镜头，非常适合拍摄人像，平行光线让人脸看起来清晰、漂亮。

在这样的背景下，Thomas SIMON 萌生了自己开发一个 Android App 的想法，只需要通过 Wi-Fi 或其他方式将手机摄像头的数据流重定向到他的电脑上，让他的 Linux 电脑相信 Android 手机是一个摄像头，或者让 Linux 的应用程序（如 Google Meet 和 Zoom）相信它是一个摄像头即可。

也许有人会说，其实市场中也早已有了这样的软件，譬如 DroidCam。DroidCam 软件分为两部分，一部分是手机上安装的软件，称为服务器；另一部分是 PC 上安装的软件，称为客户端。只需要 PC 和手机连接到同一个 Wi-Fi，就可以把手机作为电脑摄像头。

不过，Thomas SIMON 在使用后发现，DroidCam 是一款被广告限制且需要付费的应用程序，而且并非所有摄像头都能显示。同时，经过测试，它的质量令人无法接受，即使是 720p 的低分辨率也被官方推荐。

无法忍受之下，Thomas SIMON 决定自己开发一款  Android App，把 Android 手机相机变成电脑的摄像头。

**![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAujX5kS5KQ6BaBUsy1RqR06QDyP5HTUOXsPlJWd79yygiasaqXicpN7ibIfqiak5WFpaxE1mGxxpfmMjiaA/640?wx_fmt=png)**

**这有什么难的？**

通过多年的积累，Thomas SIMON 已经掌握了全栈、Linux、视频流等方面的所有专业知识。其表示，” 虽然我一直对手机应用程序敬而远之，但我的整个职业生涯都是朝着网络、服务器应用程序和桌面原生应用程序方向发展的。“

所以，对于 Thomas SIMON 而言，开发一款 Android App 难度其实并不大。而且在他的规划中，他只想要一个包含选择相机、分辨率和退出键这几个菜单栏的 App， 并不需要有太多的设计感。

于是，Thomas SIMON 开始寻找制作 Android 应用程序的最简单方法。他发现，几乎每个专家都在积极推动 Android Studio 作为 IDE 开发环境。

不难想象，他下载了一个 Android Studio，结果发现，这是一个 1.1GB 的 .tar.gz 压缩包，一旦安装了所需的工具，最终**会占用 20GB（分布在多个文件夹中）的空间**。

安装好工具之后，Thomas SIMON 开始寻找可以处理摄像头的官方 APIs。

简而言之，Thomas SIMON 试过 cameraX，这是一个 Jetpack 库，旨在帮助简化相机应用程序的开发，但它太高级了。所以 Thomas SIMON 最终使用了 camera2。

Thomas SIMON 下载了一个 camera2 的官方示例项目，其实他只想拼接一个 http 服务器并用来传输帧，在他的设想中，整个过程应该只需要几分钟就可以完成的。没想到，厄运从这里才真正开始...... 

Thomas SIMON 表示，这个官方示例项目（github.com/android/camera-samples/tree/main/Camera2Basic）的作用只有「显示摄像头、并拍摄一张照片」，但是没想到它包含了很多文件。

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuiaWuOOltcXGw7ib2k7VxjuFicq7fTtdqUK6v1rqEbo6yT2DHtv8UaR60O4GxeibC0PfSjHwibrQ52jicEg/640?wx_fmt=png)

仅以 gradle 为关键词搜索，就有一大堆文件：

```
$:/tmp/Camera2Basic$ find . -type f -name "_gradle_"./gradlew.bat./gradle.properties./gradlew./settings.gradle./utils/build.gradle./gradle/wrapper/gradle-wrapper.jar./gradle/wrapper/gradle-wrapper.properties./build.gradle./app/build.gradle

```

Thomas SIMON  表示，自己不可能花时间去慢慢理解这些东西，他只是要修改一些代码，来实现自己想要的功能。

不过，还没等他开始寻找代码片段，Android Studio 就开始跳出了一个提示：

"不支持 Java，点击此处更新 gradle thingy"。  

于是，事情似乎进入了循环：

Thomas SIMON 点击。

事情发生了变化，左侧窗格中的文件改变了结构，标签页反复打开和关闭。

" 建议更新项目，点击启动 AGP 升级助手”

Thomas SIMON 继续点击：

显示许多选项，其中一些已预选，一个按钮上写着 "运行选定步骤"。

Thomas SIMON 再次点击：

事情发生了变化，左侧窗格中的文件改变了结构，选项卡反复打开和关闭。

" 建议更新项目，单击启动 AGP 升级助手

Thomas SIMON 点击：

显示大量选项，其中一些已预选，一个按钮显示 "运行选定步骤"。

Thomas SIMON 继续点击：

事情发生了变化，左侧窗格中的文件改变了结构，标签反复打开和关闭。

Thomas SIMON 无语道，“这是一款很棒的点击冒险游戏。”

折腾了一会，Thomas SIMON 的 AGP 和 gradle 终于不再跳出更新提示。

接下来，正式进入代码部分。

写过不少 Java 和 Scala 代码的 Thomas SIMON 发现这部分的代码是用 Kotlin 写的。

“从这里开始，我喜欢我所看到的，代码很清晰。它在 xml 中定义一个视图，在代码中用标识符与之关联，非常标准且可预测。清晰的代码让我不需要学习任何东西就能提高工作效率”，Thomas SIMON 说道。

紧接着，Thomas SIMON 调用一个 Android API 来制作一个简单的 http 服务器，然后把它插入、测试，它就能工作了！

现在，Thomas SIMON 的电脑浏览器标签页上有一个 mjpeg 流。对于实时调用来说，质量和延迟都还可以。这样，他就可以在终端输入一行命令将其转换为 Linux 网络摄像头设备：

```
ffmpeg -f mjpeg -i "http://192.168.1.2:8080" -vf "format=yuv420p" -f v4l2 /dev/video0

```

整体而言，Thomas SIMON 表示，网络世界虽然有一些不必要的步骤和配置，但这个 Android 世界简直是疯了。

他下载了官方示例之后，删除了未使用的视图，添加了选择相机、分辨率和质量的下拉菜单，将所有功能移至前台服务，以便在锁定手机的情况下保持激活状态，并将其发布在 GitHub 上（https://github.com/Ruddle/RemoteCam），整个项目花了他两个下午的时间（其中大部分时间都在了解 Android 希望你怎么做）。

**![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAujX5kS5KQ6BaBUsy1RqR06Q0QtgZAx8xsoTReptvArfwbn9MvHGVfV98Qkl5PMRS6MCt2ljwLJwoQ/640?wx_fmt=png)**

**实际工作量应该如何？**

Thomas SIMON 认为，在理想的情况下，在手机端，只需一行代码就足够了，根本不需要上面那样复杂的操作。倘若我们想编写一个应用程序，只是为了准确地指定数据流，而不是依赖天才们已经开发出来的高级工具（如桌面上的 ffmpeg 和 v4l2）。

在理想的世界里，具有这种规格的 App 应具备：

*   允许配置相机、分辨率和质量 / 比特率
    
*   没有设计，只有原始功能
    
*   在本地网络上传输帧流
    

Thomas SIMON 表示，只需一个 40 行伪代码（pseudo code，又称虚拟代码，是一种高层次描述算法的方法，它可能综合使用多种编程语言的语法、保留字，甚至会用到自然语言）的文件就能实现以上功能：

```
 1CAMERA.getPermission() or quit()
 2NETWORK.getPermission() or quit()
 3
 4queue=Producer(size=1)
 5
 6server = NETWORK.createHttpServer(port = 8080)
 7server.onRequest = req ->  
 8    queue.dropConsumers() //only allows 1 client, drop old ones
 9    req.sendHeader("Content-Type","multipart/x-mixed-replace;boundary=FRAME")
10    queue.consumeUntilDrop( frame, consumer -> 
11        data = "--FRAME=\r\nContent-Type=image/jpeg\r\n".bytes+frame.bytes
12        req.sendBytes(data) or queue.drop(consumer)
13        frame.close() // Free camera memory of this frame
14    )
15    req.close()
16
17options = STORAGE.get("config") or {sensor:0, format: "jpeg", fps:30, resolution:[1280,720]}
18
19session = CAMERA.open(options)
20session.onFrame= frame -> queue.pushTry(frame) or frame.close()
21
22UI.insert(UI.text).text= "Choose a sensor:"
23dropdownSensors          = UI.insert(UI.dropdown)
24dropdownSensors.selection= session.sensor
25dropdownSensors.values   = CAMERA.sensors.map(_.name)
26dropdownSensors.onSelect = index -> 
27    STORAGE.set("config",  options + {sensor: index})
28    restart()
29
30UI.insert(UI.text).text= "Choose a resolution:"
31dropdownResolution          = UI.insert(UI.dropdown)
32dropdownResolution.selection= session.resolutions.indexOf(session.resolution)
33dropdownResolution.values   = session.resolutions.map(_.0+"x"+_.1)
34dropdownResolution.onSelect = index -> 
35    STORAGE.set("config",  options + {resolution: session.resolutions[index]})
36    restart()
37
38quitBtn         = UI.insert(UI.button)
39quitBtn.onClick = quit
40quitBtn.text    = "quit"

```

这段伪代码虽然简单，但明确指出了所需的内容。 其中，对 API 的访问如 CAMERA（摄像头）、NETWORK（网络）、UI（用户界面）和 STORAGE（存储）等都非常明显易懂。

无需导入、无需 gradle、无需指定无用的用户界面文件、无需多个事实来源。该文件非常简短，Thomas SIMON 甚至懒得将存储密钥命名为 "config"。

Thomas SIMON 解释道，“伪代码假定是托管执行，就像 Kotlin 或 Java 一样，允许非常容易地使用高阶函数。支持这种单文本文件应用程序并不需要什么新东西。我们只需选择一种语言，如 Python、Javascript 或 Kotlin，然后编写一个库来公开 API。我们甚至可以开发一个标准的 Android 应用程序，作为操作系统来执行这些单文本文件应用程序。但 Google 并不允许这样做。从技术上讲，你可以进行动态代码加载，但一旦 Google 发现你可以让人们绕过 Play 商店来加载小于 1kB 的应用程序，你就完了。此外，Google 和 Apple 已经在极力阻止 PWA（一种让网页拥有原生应用程序功能的技术）运行得太好。”

对于 Thomas SIMON 而言，他大概只用了 10 分钟就写出了这段伪代码。其中有 5 分钟是在网上查找如何在多部分 http 流中分离帧（获取信息的难度出乎意料）。

就目前而言，其实通过一个 LLM（比如 ChatGPT，甚至是本地运行的 LLM）可以在 10 秒内写完这段代码，而且至少有一半的时间不会出错。

经过 Gzip 压缩后，一个文本文件应用程序的大小为 688 字节。

```
1$> gzip -k text_app; ls -lh
21,5K text_app
3688  text_app.gz

```

相比之下，Thomas SIMON 用 Android Studio 生成的 APK 只有 14.9MB。

至于用户可能存疑的所有的 gradle 配置文件都在哪里？Thomas SIMON 表示，这些文件中 99% 的内容都是用来处理糟糕的抽象。

最后的 1% 可能只是文件顶部的一些注释，如

```
// name:RemoteCam// author:Thomas SIMON// version:1.0// icon: image/png;base64,ABC...

```

对于安全性、兼容性和更新等问题，Thomas SIMO 解释道，这些问题都已经解决了。

*   安全性：只需根据你信任的文件进行签名检查。
    
*   兼容性：只需在文件顶部注释 minAndroidVersion。
    
*   更新：替换应用程序文件即可。
    

这些问题都不能成为你在 Android 系统上制作应用程序的复杂性的借口。

这些问题都不能由应用程序商店自行解决（应用程序本身只有 40 行）。

**![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAujX5kS5KQ6BaBUsy1RqR06Q3SAo8dibFicWvibnB5u4gdHLgb2AbA6UJ8VImKnyW92hibqZefwpDbPYAQ/640?wx_fmt=png)**

**结论**

最后，Thomas SIMO 仅用了 40 行代码就开发了一个网络摄像头 App，并将代码开源在了 GitHub 上（https://github.com/Ruddle/RemoteCam）。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Pn4Sm0RsAuiaWuOOltcXGw7ib2k7VxjuFicYatX2WYLcZgIeFsgsSXQVe0DtYHaPycPvRLCv9hnfGwknYXs6Ceaog/640?wx_fmt=jpeg)

Thomas SIMO 表示，他用这款应用程序进行了两次视频通话，每次持续时间都在 1 小时 30 分钟以上，它的表现非常出色。

经过自己开发了一款 App，他也终于明白为什么 DroidCam 试图推销付费模式，主要是因为开发一款 App 必须忍受的非必要工作和挫折实在太多，代码没写多少，但其中的工具安装、项目支持文件实在过于臃肿，最终导致你想向别人收费来寻求心理平衡罢了。

来源：https://thomassimon.dev/ps/2