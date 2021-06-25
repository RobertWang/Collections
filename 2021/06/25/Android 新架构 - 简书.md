> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/6bf888dfae89)

好久没关注 Android 架构了，今天看 Android 官方文档突然发现 Android 架构和之前了解的有变动，因此发出新架构信息。  
以下为老架构：

![](http://upload-images.jianshu.io/upload_images/6471979-7cedeb63dfbeea21.jpeg) old.jpeg

### 以下为新架构信息：

Android 系统架构包含以下组件：

![](http://upload-images.jianshu.io/upload_images/6471979-41c02c7a44aef509.png) Android 系统架构概览

**图 1.** Android 系统架构

*   **应用框架**。应用框架最常被应用开发者使用。作为硬件开发者，您应该非常了解开发者 API，因为很多此类 API 都可以直接映射到底层 HAL 接口，并可提供与实现驱动程序相关的实用信息。
*   **Binder IPC**。Binder 进程间通信 (IPC) 机制允许应用框架跨越进程边界并调用 Android 系统服务代码，这使得高级框架 API 能与 Android 系统服务进行交互。在应用框架级别，开发者无法看到此类通信的过程，但一切似乎都在 “按部就班地运行”。
*   **系统服务**。系统服务是专注于特定功能的模块化组件，例如窗口管理器、搜索服务或通知管理器。应用框架 API 所提供的功能可与系统服务通信，以访问底层硬件。Android 包含两组服务：“系统”（诸如窗口管理器和通知管理器之类的服务）和 “媒体”（与播放和录制媒体相关的服务）。
*   **硬件抽象层 (HAL)**。HAL 可定义一个标准接口以供硬件供应商实现，这可让 Android 忽略较低级别的驱动程序实现。借助 HAL，您可以顺利实现相关功能，而不会影响或更改更高级别的系统。HAL 实现会被封装成模块，并会由 Android 系统适时地加载。如需了解详情，请参阅[硬件抽象层 (HAL)](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture%2Fhal)。
*   **Linux 内核**。开发设备驱动程序与开发典型的 Linux 设备驱动程序类似。Android 使用的 Linux 内核版本包含一些特殊的补充功能，例如低内存终止守护进程（一个内存管理系统，可更主动地保留内存）、唤醒锁定（一种 [`PowerManager`](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Freference%2Fandroid%2Fos%2FPowerManager.html) 系统服务）、Binder IPC 驱动程序，以及对移动嵌入式平台来说非常重要的其他功能。这些补充功能主要用于增强系统功能，不会影响驱动程序开发。您可以使用任意版本的内核，只要它支持所需功能（如 Binder 驱动程序）即可。不过，我们建议您使用 Android 内核的最新版本。如需了解详情，请参阅[构建内核](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fsetup%2Fbuilding-kernels)一文。

HAL 接口定义语言 (HIDL)
-----------------

Android 8.0 重新设计了 Android 操作系统框架（在一个名为 “Treble” 的项目中），以便让制造商能够以更低的成本更轻松、更快速地将设备更新到新版 Android 系统。在这种新架构中，HAL 接口定义语言（HIDL，发音为“hide-l”）指定了 HAL 和其用户之间的接口，让用户无需重新构建 HAL，就能替换 Android 框架。

**注意**：如需详细了解 Treble 计划，请参阅开发者博文 [Treble 现已推出：Android 的模块化基础](https://links.jianshu.com/go?to=https%3A%2F%2Fandroid-developers.googleblog.com%2F2017%2F05%2Fhere-comes-treble-modular-base-for.html)和[借助 Treble 项目更快速地采用新系统](https://links.jianshu.com/go?to=https%3A%2F%2Fandroid-developers.googleblog.com%2F2018%2F05%2Ffaster-adoption-with-project-treble.html)。

利用新的供应商接口，HIDL 将供应商实现（由芯片制造商编写的设备专属底层软件）与 Android 操作系统框架分离开来。供应商或 SOC 制造商构建一次 HAL，并将其放置在设备的 `/vendor` 分区中；框架可以在自己的分区中通过[无线下载 (OTA) 更新](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Ftech%2Fota)进行替换，而无需重新编译 HAL。

旧版 Android 架构与当前基于 HIDL 的架构的区别在于对供应商接口的使用：

*   Android 7.x 及更低版本中没有正式的供应商接口，因此设备制造商必须更新大量 Android 代码才能将设备更新到新版 Android 系统：
    
    ![](http://upload-images.jianshu.io/upload_images/6471979-ae4165903f8bc11b.png) 图 2

**图 2.** 旧版 Android 更新环境

*   Android 8.0 及更高版本提供了一个稳定的新供应商接口，因此设备制造商可以访问 Android 代码中特定于硬件的部分，这样一来，设备制造商只需更新 Android 操作系统框架，即可跳过芯片制造商直接提供新的 Android 版本：
    
    ![](http://upload-images.jianshu.io/upload_images/6471979-0de6d5df65f52de5.png) 图 3

**图 3.** 当前 Android 更新环境

所有搭载 Android 8.0 及更高版本的新设备都可以利用这种新架构。为了确保供应商实现的向前兼容性，供应商接口会由[供应商测试套 (VTS)](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Ftech%2Fvts) 进行验证，该套件类似于[兼容性测试套件 (CTS)](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fcompatibility%2Fcts)。您可以使用 VTS 在旧版 Android 架构和当前 Android 架构中自动执行 HAL 和操作系统内核测试。

架构资源
----

要详细了解 Android 架构，请参阅以下部分：

*   [HAL 类型](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture%2Fhal-types)：介绍了绑定式 HAL、直通式 HAL、Same-Process (SP) HAL 和旧版 HAL。
*   [HIDL（一般信息）](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture%2Fhidl)：包含与 HAL 和其用户之间的接口有关的一般信息。
*   [HIDL (C++)](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture%2Fhidl-cpp)：包含关于为 HIDL 接口创建 C++ 实现的详情。
*   [HIDL (Java)](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture%2Fhidl-java)：包含关于 HIDL 接口的 Java 前端的详情。
*   [ConfigStore HAL](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture%2Fconfigstore)：介绍了可供访问 Android 框架只读配置项的 API。
*   [设备树叠加层](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture%2Fdto)： 详细说明了如何在 Android 中使用设备树叠加层 (DTO)。
*   [供应商原生开发套件 (VNDK)](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture%2Fvndk)：介绍了一组可供实现供应商 HAL 的供应商专用库。
*   [供应商接口对象 (VINTF)](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture%2Fvintf)：介绍了可收集设备的相关信息并通过可查询 API 提供这些信息的对象。
*   [SELinux for Android 8.0](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fsecurity%2Fselinux%2Fimages%2FSELinux_Treble.pdf)：详细介绍了 SELinux 变更和自定义。

引用文章
----

[Android 架构](https://links.jianshu.com/go?to=https%3A%2F%2Fsource.android.google.cn%2Fdevices%2Farchitecture)