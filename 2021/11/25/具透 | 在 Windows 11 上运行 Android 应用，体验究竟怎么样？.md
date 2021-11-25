> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/69446)

> 在 Windows 11 上运行 Android 应用具体需要什么条件？实际使用体验怎么样？我们第一时间进行了配置和测试，希望能够为你解答这些问题。

正式版推出半个多月后，Windows 11 最为重磅的新功能 —— Windows Subsystem for Android（以下简称 WSA）终于也于近期开启了公开测试。

**关联阅读：**[Windows 11 正式版发布，升级后一定不要错过这些新特性](https://sspai.com/post/69072)

在 Windows 11 上运行 Android 应用具体需要什么条件？实际使用体验怎么样？我们第一时间进行了配置和测试，希望本文能够为你解答这些问题。

必不可少的准备工作
---------

尽管目前 Windows 11 正式版已经上线了，但想要在 Windows 11 中直接运行、体验 Android 应用，我们还需要满足一些额外的要求。

**系统方面，本次测试向 Window 预览体验计划中的 Beta 渠道开放**，建议大家在非主力设备上进行测试和体验。如果你的设备满足升级 Windows 11 的基本 [硬件要求](https://www.microsoft.com/en-us/windows/windows-11-specifications)，升级到 Windows 11 正式版后可以前往「设置 > Windows 更新 > Windows 预览体验计划」中选择加入 Beta 渠道。加入后重启进行系统更新即可。

![](https://cdn.sspai.com/editor/u_/c5ohv7lb34t8tgjcf230.png)

**与此同时，因为需要虚拟化技术支持**，总共分为两部分：

*   BIOS 中的虚拟化开关
*   Windows 11 中的 Hyper-V 功能

BIOS 中的虚拟化开关，需要进入 BIOS 中设置，具体设置可能因主板厂商、OEM 厂商命名而有所不同，可以在搜索引擎中输入「主板 / 电脑的具体型号」「打开虚拟化」这样的关键词，通过官方文档指导打开。打开后你将能在「任务管理器 > 性能 > CPU」下找到虚拟化已打开的标志。

Hyper-V 功能则需要 Windows 系统本身是专业及以上版本。满足系统版本要求后，在 Windows 中搜索、打开「启用或关闭 Windows 功能」，找到并勾选 Hyper-V 功能，待 Windows 安装完成后重启即可。

![](https://cdn.sspai.com/editor/u_/c5ohv7tb34t8s6bde31g.png)

**注意**：如果你满足系统版本要求但 Hyper-V 选项为不可勾选添加的状态，则可能需要跟随微软 [官方指南](https://docs.microsoft.com/zh-cn/windows-server/virtualization/hyper-v/system-requirements-for-hyper-v-on-windows#:%7E:text=on%20Windows%20Server.-,General%20requirements,the%20processor%20must%20have%20SLAT.) 来做进一步的确认。另外如果本身你的电脑就是虚拟机，比如云电脑等，则需要检查电脑是否支持 [嵌套虚拟化技术](https://docs.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)，支持嵌套虚拟化的设备也可以使用 Windows Subsystem for Android™。

**上述工作完成后，接下来就是软件环境方面的准备了**。

首先，打开 Microsoft Store 并检查更新，保证 Microsoft Store 本身为最新版本。

然后在 Windows 11 系统设置中将「国家或地区」更改为「美国」。

![](https://cdn.sspai.com/editor/u_/c5ohv7tb34t8tgjcf23g.png)

最后点击 [这个链接](http://aka.ms/AmazonAppstore) 跳转到 Amazon Appstore 的商店界面，点击打开，就能跳转到 Amazon Appstore 进行登录并下载、安装 Android 应用了（需要美区 Amazon 账号）。

![](https://cdn.sspai.com/editor/u_/c5ohv8db34t8t2fr5j00.png) ![](https://cdn.sspai.com/editor/u_/c5ohv8lb34t8t0u362j0.png) ![](https://cdn.sspai.com/editor/u_/c5ohv8lb34t8t0u362jg.png) ![](https://cdn.sspai.com/editor/u_/c5ohv95b34t8t0u362k0.png)

如果你的设备「不受支持」
------------

微软从 Windows 11 开始就有这样那样的奇怪「硬件要求」，这次的 Android 子系统也不例外。与之类似的，我们也可以通过一些特殊手段绕过上述限制。

### 在不受支持的系统上安装 Hyper-V

国内大部分情况下售出的 Windows 系列笔记本都是 Windows 家庭版，而 Hyper-V 至少需要 Windows 专业版才能打开，从家庭版升级到专业版需要额外再花费的 879.99 元，如果只使用 Windows Subsystem for Android™ 的话成本也未必太高了。

参考 XDA 给出的 [方法](https://www.xda-developers.com/how-to-install-hyper-v-windows-11-home/)，我们也整理出了一个新方法帮助大家在 Windows 11 家庭版中打开 Hyper-V。

第一步：打开记事本并输入下列内容：

```
pushd "%~dp0"

dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hv.txt

for /f %%i in ('findstr /i . hv.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"

del hv.txt

Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL

pause
```

第二步：将文件保存为 `Hyper-V.cmd`，格式为`所有格式`。

第三步：右键该文件，选择`以管理员身份运行`。将会看到如下结果

![](https://cdn.sspai.com/editor/u_/c5ohv95b34t8t2fr5j0g.png)

等待所有的进度条走完，重启你的电脑即可。重启完毕后你将能在开始菜单中看到 Hyper-V。

### 在 Windows 11 Dev 通道上安装

如果这次你是因为 Windows 预览体验计划是 Dev 通道导致的不能使用 Windows Windows Subsystem for Android™，那我们可以通过侧载应用进行解决。

第一步，我们前往 https://store.rg-adguard.net/，在搜索框中输入 https://www.microsoft.com/store/productId/9P3395VX91NR，Channel 选择 Slow 再进行搜索。找到大小是 1.2GB 的名称类似 [`MicrosoftCorporationII.WindowsSubsystemForAndroid_x.x.xxxxx.x_neutral_~_8wekyb3d8bbwe.msixbundle`](http://tlu.dl.delivery.mp.microsoft.com/filestreamingservice/files/433b1665-c732-486a-99ee-e2c610cd10d4?P1=1634823773&P2=404&P3=2&P4=XojAD2GWc234C2PSEkwW4dHgtuOzrd8BxPHYCL7J5zm66g2aipISpNC83mvuceAv8BV3MQoV2pjK38ys2FIpTg==) 的文件进行下载。

![](https://cdn.sspai.com/editor/u_/c5ohv9lb34t8s6bde320.png)

第二步，按下`win+x` 选择 Windows Terminal（管理员）输入下列代码。

```
Add-AppxPackage -Path "C:\lincoln\download\wsa.msixbundle"(可以通过右键文件-复制为路径获得)
```

等待完成，稍后你也能在开始菜单中看到 Windows Windows Subsystem for Android™ 的图标了。

把「旗舰机」塞进 Windows
----------------

是 Android 怎么又能不跑个分看看速度呢，在 AMD 5950X 和 64G DDR4 3600Mhz 内存的加持下使用安兔兔简单对 Windows Windows Subsystem for Android™ 跑了一个分，最后结果如下图。

![](https://cdn.sspai.com/editor/u_/c5ohv9lb34t8t2fr5j10.png)

在 x86-64 模拟 arm64-v8a 的情况下，CPU 和内存得分均是当下旗舰手机的两倍，可以说非常恐怖了。

![](https://cdn.sspai.com/editor/u_/c5ohv9tb34t8t0u362kg.png)

另外跑分时我还注意到了 5950X 其实并没有完全跑满，有不少核心的负载很低，我一开始怀疑是 Windows Windows Subsystem for Android™ 只跑在一部分的核心上，后续使用 `adb shell&cat /proc/cpuinfo` 命令检查 CPU 的时候才发现其实所有的核心都已经传递进去了，只是安兔兔跑分确实压榨不出所有的性能而已。

![](https://cdn.sspai.com/editor/u_/c5ohva5b34t8s6bde32g.png)这个预装载界面都能感觉到时不时会掉帧一下

原神作为新晋性能测试软件，我们自然也试着安装了下，结论是可以安装，在 5950X 上中高画质还算的上流畅，只不过经常性的出现掉帧很难受，这里我很怀疑和 GPU 有关（可以看到安兔兔的 GPU 跑分为 0 分）。此外游戏体验也不算好，除了适配了 WSAD 方向键，其他均没有适配，如果打算在 Windows Windows Subsystem for Android™ 上游玩原神可能要失望了；微软官方也表示，键盘和鼠标右键的需要进行额外适配。

如何安装你想要的应用
----------

值得一提的是，现阶段 Amazon Appstore 不仅对账号区域有着异常严格的要求，商店内提供的 Android 应用数量也十分有限。

既然在系统版本满足条件的前提下 Windows 11 已经具备了运行 Android 应用的基础，我们自然也可以通过 adb 指令旁加载（sideload）的方式来安装其它 Android 应用。

**关联阅读：**

*   [Windows 操作系统下的 ADB 环境配置](https://sspai.com/post/40471)
*   [免 root 玩转 Android 设备：如何从零开始使用 adb](https://sspai.com/post/57427)

确保 Windows 系统 adb 环境配置正确后：

1.  在 Windows 中搜索并运行 Windows Subsystem for Android
2.  打开开发人员模式，此时设置下方还会提供用于无线 adb 调试的地址及端口
3.  在终端工具中使用 `adb connect` 指令和上面获取到的地址、端口，完成 adb 连接

至此，我们就可以在终端中使用 adb 指令进行应用旁加载了。当前我们从网络上获取到的 Android 应用安装包一般分为 apk 文件和多 APK 文件两种格式。

**关联阅读：**

*   [靠谱的 Android 应用上哪儿找？你可以试试这些选择](https://sspai.com/post/55197)
*   [看懂「非典型」APK 文件，解锁手动安装 Netflix 的正确姿势](https://sspai.com/post/60228)

**针对 apk 文件**，我们使用 `adb install` 指令一般就能进行安装。假设你下载好的安装包路径为 C:\clyde\store.apk，则进行安装完整指令为 `adb install **C:\clyde\store.apk**`**。**

当然，你也可以在输入 `adb install` 指令后直接拖拽 apk 文件进入终端然后回车执行。

**针对多 APK 文件**，如 APKMirror 下载的 apks、APKPure 下载的 xapk，我们需要首先更改其后缀为 .zip 进行解压，解压得到 apk 文件后，选择需要的文件使用 `adb install-multiple` 指令进行安装。

这里需要安装的 apk 文件大体分为两部分，应用主体（一般为 **base.apk**）和配置文件（一般为 **split_xxx.apk**），前者为必选，后者根据机型和环境选择。

![](https://cdn.sspai.com/editor/u_/c5ohvatb34t8s6bde330.png)

例如在上图展示的 Netflix 拆包文件中，我们一般会用到的 apk 文件除了 base.apk 还包括：

*   **split_config.arm64_v8a.apk**，针对 arm64_v8a 处理器架构的配置文件
*   **split_config.zh.apk**，针对简体中文的语言包
*   **split_config.xxxhdpi.apk**，针对高分辨率设备的图形素材
*   **split_InAppWidevine.config.arm64_v8a.apk**，针对 arm64_v8a 处理器架构的 Widevine 认证配置
*   **split_voip.config.xxhdpi.apk**，针对高分辨率设备的 VoIP 支持

因此完整的安装命令应该是：

```
adb install-multiple "base.apk" "split_config.arm64_v8a.apk" "split_config.zh.apk" "split_config.xxxhdpi.apk" "split_InAppWidevine.config.arm64_v8a.apk" "split_voip.config.xxhdpi.apk"
```

当然，因为输入比较繁琐，这里也建议大家通过重命名、拖拽等方式来简化安装流程。

超乎预期的融合体验
---------

我们再把目光聚焦回 Android 应用上。

通过 Amazon Appstore 或旁加载安装好 Android 应用后，这些应用便会以图标的方式出现在 Windows 11 的开始菜单中了。

![](https://cdn.sspai.com/editor/u_/c5ohvb5b34t8s6bde33g.png)

这里首先建议大家通过上面提到的旁加载方式安装一个应用商店，比如国内的 [OPPO 软件商店](https://www.oppomobile.com/)、针对 Google Play 的 [Aurora Store](https://auroraoss.com/) 等，这类应用商店能够极大程度地弥补 Amazon Appstore 在应用数量上的短板，同时绕开繁琐的旁加载流程。

![](https://cdn.sspai.com/editor/u_/c5ohvbdb34t8s6bde340.png)

实际使用体验方面，目前我们测试过的大部分 Android 应用，包括 Today Weather、钱迹、微信甚至原神都能正常运行，部分应用因为缺少 Google 服务框架，会有无法进行定位、或无法拉起内购的情况。

最令人印象深刻的是，在 Windows 中调整窗口大小时，适配了大屏设备的 Android 应用也能根据实际情况在移动版布局和平板布局之间灵活切换。

![](https://cdn.sspai.com/editor/u_/c5ohvbtb34t8t0u362lg.png)

**Android 应用与 Windows 11 系统之间的交互也融合得很好**。比如一些应用中的下拉刷新行为，在手机上通过触屏操作非常直观，但在 Windows 中则变成了更贴合键鼠的交互——滚动到页面顶部后鼠标滚轮继续上滑即可；与之类似的，长按行为也可以通过鼠标右键单击触发。

跨平台体验方面，输入法和剪贴板在 Windows 11 和 Android 应用之间可以无缝衔接，直接使用 Windows 11 的输入法就能完成在 Android 应用中的文本输入，复制、剪切的文本内容也能通过快捷键直接完成。

![](https://cdn.sspai.com/editor/u_/c5ohvc5b34t8tgjcf240.png)

总体而言，Andorid 应用目前在 Windows 11 上的使用和交互体验都很不错。微软也比较聪明，从相关 [开发者文档](https://docs.microsoft.com/zh-cn/windows/android/wsa/) 中可以看到微软在这一块调用了大量 Chrome OS 和 Android 的交互标准。

唯一比较麻烦的是文件传输，现阶段我们还不能像 MIUI+ 这类跨屏方案那样直接拖拽完成文件互传，在 Android 应用中发送文件时，也无法直接浏览到 Windows 11 中的存储空间——我们姑且可以将 Android 应用和 Windows 11 系统看作是两个完全独立的空间，并且不支持从计算机向「手机」中写入。

![](https://cdn.sspai.com/editor/u_/c5ohvcdb34t8tgjcf24g.png)

管理 Android 应用获取到的文件倒是简单一些，在 Windows Subsystem for Android 面板顶部找到「文件」并点击即可。

![](https://cdn.sspai.com/editor/u_/c5ohvcdb34t8t2fr5j1g.png)

最后，Android 应用的卸载也可以直接通过开始菜单来完成，卸载体验与 Windows 原生应用无异：

![](https://cdn.sspai.com/editor/u_/c5ohvctb34t8s6bde34g.png)

其它细节
----

通过安装 Inware，我们也得到了关于 Android Subsystem for Windows 的更多细节，如系统版本为 Android 11、安全更新补丁级别为 9 月 5 日，内核版本则是 5.10.x。

![](https://cdn.sspai.com/editor/u_/c5ohvd5b34t8t0u362m0.png) ![](https://cdn.sspai.com/editor/u_/c5ohvddb34t8s6bde350.png)

再比如 WSA 支持的 ABI 非常多，涵盖包括 x86_64、x86、arm64-v8a/v7a 和 armeabi 在内的常见架构。所以在上文提到的多 APK 安装环节中，我们几乎可以选择任意版本的配置文件进行安装。

另外它也是支持 url scheme，可以从属性中看到通用的链接是 `wsa://包名`，因此理论上可以跟上具体的参数传递给特定 App，实现某些高级的自动化操作。

![](https://cdn.sspai.com/editor/u_/c5ohvdlb34t8t2fr5j20.png)

此外，根据微软官方文档，这套 Android 系统有自己独立的后台的生命周期，也就是「杀后台」的方式。

![](https://cdn.sspai.com/editor/u_/c5ohvdlb34t8s6bde35g.png)

简单来说系统总共有三种状态：

*   运行
*   轻量低电耗模式
*   不再运行

如果没有任何 Android App 运行，系统将由运行状态变为轻量低电耗模式状态，在收到用户活动或者通知时会重新回到运行状态。否则 7 分钟后，继续没有 Android App 运行的情况下系统会变为不再运行的状态；直到用户重新启动 App 或是收到了新的通知，这时系统会再回到运行状态。这样理论上能很好的节约笔记本上的各项资源，不止运行得更多还能和以前一样流畅。

以上便是本次 Windows Subsystem for Android 测试中值得你关注的流程与细节，这样的体验能让你满意吗？你在 WSA 上还想见到哪些功能？欢迎在评论区留言分享。

参考来源：[Announcing Android™ apps on Windows 11 Preview for Windows Insiders in the Beta Channel](https://blogs.windows.com/windows-insider/2021/10/20/announcing-android-apps-on-windows-11-preview-for-windows-insiders-in-the-beta-channel/)

> 下载少数派 [客户端](https://sspai.com/page/client) 、关注 [少数派公众号](https://sspai.com/s/J71e) ，了解更妙的数字生活 🍃

> 想申请成为少数派作者？[冲！](https://sspai.com/apply/writing)