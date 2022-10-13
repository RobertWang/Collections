> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/G4K3nIB1SsYDsOxMv7CyFg)

大厂技术 坚持周更 精选好文
==============

本文为来自 **教育 - 智能学习 - 前端团队** 成员的文章，已授权 ELab 发布。

> **智能学习前端团队** 自创立以来，团队专注于打破大众对教育的刻板印象，突破固有的教学思维，攻破各类教学屏障。旨在为每一位学生制定最合适的学习方案，予以因材施教，使优质教育随 " **触** " 可达。

桌面应用开发
======

在处于移动互联网的当下，虽然桌面应用的重要性已经不能同往日而语，但在我们平常的日常工作和生活中，还是扮演着非常重要的角色和地位。在我们的日常工作中，离不开 Lark、VSCode 等桌面应用。

相比较于移动端而言，桌面端应用的生态多种多样，因此也诞生了各种各样的桌面应用开发技术栈。本次分享将会对相关常用的一些桌面应用开发框架进行介绍和分析，同时对当下比较流行（GitHub 50k star）的跨平台桌面应用开发框架 Tauri 进行介绍。

原生技术栈
-----

原生技术栈是指通过操作系统相关 API 或者操作系统厂家（如 Apple/Microsoft）提供的 SDK / 工具来开发桌面应用的方式。使用原生技术开发的应用，通常能够在性能、体积以及系统的交互等方面做到非常不错的效果。

*   优点
    

*   构建产物体积小
    
*   性能好
    
*   系统 API 调用方便
    
*   兼容性好
    
*   和系统应用的交互融合度高，如要实现如下的一些系统原生 UI 组件非常方便
    

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9Pzyrj1bEQGBL5LUFhMvRX4kpQ7lB7sKo1iamR4OjO7yJPghibHckBYibBImtQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrHiaRdAKHYAicicesuG2epskqtsa2R8jkq54p1goZmGX2I8kWmpbpbYFtw/640?wx_fmt=png)

*   缺点
    

*   无法做到跨平台，所开发的应用只能在对应的平台上运行，如果需要跨平台运行，则需要在不同的操作系统上分别开发，开发成本高
    
*   对于使用的技术栈有限制（Windows 使用 C#，macOS 使用 ObjC/Swift）
    

### Windows 平台

作为目前使用率最高的操作系统，Windows 平台的 GUI 程序开发经历了漫长的迭代和演化的过程：Win32 API 作为 Windows GUI 开发的鼻祖，通过 C 语言调用 Windows 底层的绘图函数来进行开发；在 Win32 API 之后出现了 MFC（Microsoft Fundation Class），MFC 通过 C++ 语法将原有的 Win32 API 封装成了控件类（对话框控件、按钮控件等）；在 MFC 之后，微软推出了 Windows Form（2002 年），Windows Form 依赖于. NET 的运行时，提供了组件化的开发能力；在此之后，微软推出了 WPF（Windows Presentation Fundation，2006 年），WPF 提供了基于 XML 的语言 XAML 来描述 UI；在 Windows8 的时代，微软又推出了 UWP（Universal Windows Platform，2015 年），UWP 支持在各种平台上运行（PC/Windows Phone/Xbox），API 也支持多种语言（C++/VB/C#/JS）。

从 Windows 平台应用的开发技术迭代来看，也可以大致看出 GUI 程序的技术发展史：

1.  Win32API 时代：函数调用，指令式，Windows 系统处理
    

2.  MFC 时代：面向对象，把一些指令式调用封装成类，由来自 UI 的消息驱动程序处理数据
    

3.  Windows Form 时代：组件化，在类的基础上封装成组件，消息被封装成事件，事件驱动
    

4.  WPF 时代：使用类 XML 语言来描述 UI，引入数据驱动 UI 的理念
    

5.  UWP 时代：跨平台、多语言
    

### macOS 平台

现有的 macOS 原生应用主要基于 Cocoa 框架开发，Cocoa 是从 1980 年代由 NeXT（macOS 的前身）开发的编程环境 NeXTSTEP 和 OPENSTEP 演变而来，是面向对象的 API。

在 2020 年的 WWDC 上，苹果推出了新一代的 UI 框架 SwiftUI，和 Flutter/React 等现代 GUI 框架类似，支持声明式的方式使用 Swift 语言作为 DSL 来编写 UI，同时也支持跨平台的特性，可以在 macOS/iOS/tvOS 等多平台运行。

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrIzFr7lUiaeAREd5wjLhoWRWjcBica1IQyLG7yUvQygP8XoRJ7KLnxQyA/640?wx_fmt=png)

### Linux 平台

Linux 其源码只包含了操作系统内核的部分，桌面并不属于 Linux 源码的一部分，因此严格意义上来说，从「使用系统 API 和操作系统厂商提供的 SDK 开发的应用为原生应用」的定义上来说，并无所谓「原生技术栈」的概念。我们日常使用的发行版提供了桌面环境如 KDE、Gnome 等，Linux 发行版的这些桌面环境也提供了相关的一些库或者 API 来帮助绘制 GUI 程序，如 gtk + 等，可以认为是「原生技术栈」。

跨平台技术栈
------

### Web 技术栈

Atwood's Law: Any application that can be written in JavaScript, will eventually be written in JavaScript.

> 一个你或许不知道的冷知识，macOS 的系统设置页面是 Webview+React 写的 [1]

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9Pzyr325Ag7ze58eu0FzVcxCB4iboiavwISnbVMJyqDkm7WJWjVlhUmjooFcQ/640?wx_fmt=png)

提到跨平台，就不得不提 Web 生态了，Web 相关的技术在跨平台中永远是最受青睐的选择，无论是开发的便捷程度，还是庞大的 JS 开发者生态等等因素，都使得 Web 技术无论是在移动端还是桌面端的跨平台应用开发上都稳坐使用率最高的技术栈。

*   优点
    

*   开发成本比原生低，可以方便做到一套代码在不同操作系统上运行
    
*   实现复杂的 UI 和动效方便，可以更快地实现一些比较炫酷的 UI 界面
    

*   缺点
    

*   调用系统原生 API 不方便通常需要使用打包其他的运行时环境或 JSBridge 的方式来进行调用
    

#### Electron

Electron（原名为 Atom Shell）是 GitHub 开发的一个开源框架，最初用来开发 Atom 编辑器。它通过使用 Node.js（作为后端）和 Chromium 的渲染引擎（作为前端）完成跨平台的桌面 GUI 应用程序的开发。

*   代表应用：VSCode（303M）、Figma（213M）、Bilibili（397M）、Discord（367M）、QQ Beta（747M）、1Password8（343M）、MS Teams（264M，根据参考文献 [2]，微软正在替换 Electron 的实现，但目前看我电脑中下载的版本解包中，依然还有 Electron.framework 的文件）
    
*   优点
    

*   开发方便，技术栈适合前端同学（UI 使用 Web 技术，系统 API 交互部分使用 NodeJS）
    

*   缺点
    

*   打包体积大，需要打包 Chromium 和 NodeJS 的运行时环境
    
*   内存消耗大：Chromium 本身比较吃内存，同时 NodeJS 是 JIT 运行的，相比较 C++ 等 AOT 的语言来说内存消耗也更大。
    

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrgJOia5aPpAjiaoVhyd6BHIZk3cnKcn5w4IyGw3LL8DaQ5PKCWZoWa4EQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrL90HYffe1f8bQEPmzbbZJNgpqkTsXEW0fCqGLzR7etdG9AKOtJ963g/640?wx_fmt=png)

*   性能需要多花点时间优化
    

实际上，并不代表 Electron 技术开发的应用性能就一定不如其他技术栈，总的来说，具体的性能表现还是取决于开发者的投入，例如微软在 VSCode 的博客中给到了一个例子，能够将 VSCode 在渲染括号颜色匹配的速度提高 10000 倍 [3]。

#### CEF（Chromium Embedded Framework）

由于需要将 Chromium 和 NodeJS 的运行时打包进去，所以 Electron 构建应用的体积都会非常大，但是 CEF 的存在解决了 Electron 的这个问题（实际上，CEF 出现的时间比 Electron 早多了）。由于 Chromium 里面有许多第三方组件（如 ffmpeg 等），在开发应用的过程中，我们通常不会使用到 Chromium 的全部能力，因此 CEF 提供了一个轻量级的嵌入式 Chromium，同时还可以根据自己的需求进行裁剪。

CEF 提供了将 Chromium 嵌入到应用中，展示 Webview 的能力，同时也提供了 C++ 的一些 API，在需要做一些浏览器无法实现的原生 API 依赖的功能时（如系统文件读写等），则需要使用 C++（或其他语言，但是 CEF 的原生接口是 C++ 的）来编写相关的能力，并提供 JSBridge 给前端代码进行调用。

*   代表应用：网易云音乐、Spotify、飞书等
    

### 自渲染技术栈

要实现跨平台的 GUI 应用，比较流行的的方式是实现自渲染的管线。上层通过提供类 Canvas 的绘制、渲染和排版能力，下层使用 OpenGL/Vulkan/Metal 等图形 API 进行绘制。在 Web 的跨平台桌面应用开发技术栈发展之前，许多应用开发框架都采用了类似的思路去实现跨平台的应用开发，如 QT（C++ 语言）、Flutter（Dart 语言，基于 Skia 渲染）和 Swing（Java 语言）等。相比于 Electron 和 CEF 的方案，由于不需要打包运行时环境（Swing 除外，需要打包 JRE）和减少了 Bridge 转换，所以体积和运行效率通常会优于 Web 技术栈。

*   优点
    

*   自绘性能通常会优于 Web 跨平台技术（具体还是取决于框架实现）
    

*   缺点
    

*   开发成本略高于 Web 技术栈
    
*   实现复杂效果的能力不如原生和 Web 技术栈，通常情况下需要写更多复杂的代码（取决于具体框架的设计，这一点 Flutter 做得比较好）
    

#### Qt（C++）

Qt（/ˈkjuːt/）是一个跨平台的 C++ 应用程序开发框架，广泛用于开发 GUI 程序，在工业、嵌入式等领域的桌面程序中有着非常深入的使用。

*   代表应用：WPS Office、剪映桌面版、AutoDesk
    
*   优点
    

*   性能好，与 Native 开发的应用性能相差无几
    
*   支持的操作系统丰富，跨平台性能好
    

*   缺点
    

*   C++ 开发成本高
    
*   GPL 协议，商业版本需要给钱，不符合咱们去肥增瘦的理念
    

#### Flutter（Dart）

Flutter 是一个由 Google 开发的跨平台应用开发框架，最初只用于移动端为 Android、iOS 开发应用。2015 年 4 月，Flutter 正式发布，其目标是希望可以在跨平台的特性上，实现 120FPS 的渲染性能。2018 年，Flutter 1.0 发布，是该框架的第一个稳定版本。2022 年 5 月，Google 在 Google I/O 2022 发布了 Flutter 3.0 版本，宣布对 Windows、macOS、Linux 桌面操作系统提供支持。

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrkyZs9erUkFMYE7CXNlwtDrfV9Hqxt1sxQ6iaMwDsTLAmZFXpmg57JjA/640?wx_fmt=png)

*   代表应用：？（Flutter 在 2022 年 5 月份发布 3.0 版本，此时桌面应用才进入正式版支持，目前还并不成熟，所以在线上使用的较少，暂时没听过有啥桌面应用是用 Flutter 写的）
    
*   优点
    

*   性能好（相比较 Web 技术栈）
    
*   Dart 语言容易学习和上手、开发成本低
    

*   缺点
    

*   桌面端才刚刚发布稳定版支持，生态和稳定性都有待考量
    

#### Swing（Java）

Swing 是一个用于开发 Java GUI 应用的框架。它采用纯 Java 实现，不再依赖于本地平台的图形界面，所以可以在所有平台上保持相同的运行效果，对跨平台支持比较出色。

*   代表应用：Jetbrains IDE
    
*   优点
    

*   跨平台性能好：write once run anywhere （write once debug everywhere)
    

*   缺点
    

*   需要打包 JRE，体积大
    

总的来说，虽然不同大类技术栈的应用具体实现原理有所不同，但是相关开发的技术栈的大致特点可以归纳如下：

*   系统 API 调用和交互：原生应用 > 自渲染应用 > Web 应用
    
*   开发便捷程度：Web 应用 >> 自渲染应用 > 原生应用
    
*   打包体积：Web 应用 > 自渲染应用 > 原生应用
    
*   性能：原生应用 > 自渲染应用 > Web 应用
    

Tauri 介绍
========

从上面的介绍可以看出，不同的技术栈的实现原理和特点各有区别，但是很难做到开发便捷程度、UI 复杂效果、打包体积和性能等多个方面的兼顾，只能是根据应用的类型和具体的业务场景去决定到底使用哪种框架。

所以有没有一种开发方式，可以在性能、体积、开发等多个角度上，取得一个比较好的平衡呢？这就来到了我们今天需要介绍的桌面应用开发框架 Tauri。

> **Build an optimized, secure, and frontend-independent application for multi-platform deployment.**

从上面 Tauri 官网的宣传语可以看出 Tauri 主打的几个卖点 [4]：

*   optimized：性能高、体积小
    
*   secure：安全性强
    
*   frontend-independent：前端独立
    
*   multiplatform：跨平台
    

那么 Tauri 是如何通过在框架层面的设计来保证这样的一些特性的呢？我们一起接着往下看⬇️

Rust
----

Tauri 框架是由 Rust 语言实现的，同时 Tauri 应用的后端也是由 Rust 来编写的。Rust 是由 Mozilla 主导开发的通用、编译型的系统编程语言。设计准则为 “安全、并发、实用”，支持函数式、并发式、过程式以及面向对象的编程风格。[5] 相比较其他语言，Rust 有如下的一些特性：

*   性能高（optimized）：Rust 的性能和 C/C++ 的性能不相上下，由于 Rust 的「所有权」机制，Rust 不需要 GC，同时也能避免如 C/C++ 之类需要手动管理内存的语言忘记释放内存导致的内存泄露的问题；
    
*   安全性强（secure）：Rust 设计了一个所有权系统，其中所有值都有一个唯一的所有者，并且值的作用域与所有者的作用域相同。值可以通过不可变引用（&T）、可变引用（&mut T）或者通过值本身（T）传递。任何时候，一个变量都可以有多个不可变引用或一个可变引用，这实际上是一个显式的读写锁。Rust 编译器在编译时强制执行这些规则，并检查所有引用是否有效。能够有效避免 C/C++ 等语言中的悬垂指针等问题；
    
*   FFI 编译友好（multiplatform）：FFI 是可以用一种编程语言写的程序能调用另一种编程语言写的代码的机制，使用 Rust 可以方便地提供接口给其他语言调用；
    

WRY[1]：Webview Render Library
-----------------------------

由于 Web 技术的表现力强、开发成本低的特点，与 Electron、CEF 等框架类似，Tauri 应用的前端实现也是使用 Web 技术栈编写的。那么 Tauri 是如何解决 Electron/CEF 等框架遇到的 Chromium 内核体积过大的问题呢？

大家可能会想，如果每一个应用都需要把浏览器内核打包进去才能实现 Web 页面的渲染，要是所有的应用都共享同一个内核就好了，这样我们在分发应用的时候，不需要打包浏览器内核，只需要打包 Web 页面的资源不就好了吗？所以 Tauri 就采用了这样的一个方案，WRY 是 Tauri 封装的 Webview 框架，它在不同操作系统的平台上，封装了系统 Webview 的实现：在 macOS 上使用 WebKit.WKWebview[2]，在 Windows 上使用 Webview2[3]，在 Linux 上使用 WebKitGTK[4]。这样在运行 Tauri 应用时，会直接使用系统 Webview 来渲染应用前端的展示。

API 接口
------

对于不会使用 Rust 的同学来说，学习 Rust 还是存在着不少的学习成本，但是别担心，在需求简单的情况下，你完全可以不写 Rust 代码。Tauri 框架提供了如下的一些 API，可以方便地在 JS 中对原生能力进行调用：

*   cli：解析应用启动时的命令行参数
    
*   clipboard：对系统剪贴板的读写
    
*   dialog：展示系统文件选择、文件保存弹窗
    
*   event：给后端发出一些事件，后端监听并处理
    
*   fs：文件系统的操作，提供文件读写等能力
    
*   globalShortcut：注册全局快捷键
    
*   http：使用 Rust 的 Http 客户端进行网络请求
    
*   notification：系统通知
    
*   os：获取操作系统的一些信息
    
*   path：文件和文件夹路径处理的一些工具
    
*   process：对当前的进程进行一些操作
    
*   shell：对系统 shell 的一些操作
    

> 需要注意的是，上述的一些 API，为了保证安全性，对于权限都有着严格的限制，都是默认关闭的状态，需要修改配置以手动启用相关的功能。

进程模型
----

和 Electron 类似，Tauri 也采用的是多进程的架构（Electron 中有主进程和渲染进程），多进程的好处是能够更好更有效地利用现代多核 CPU 的能力，同时一个组件的崩溃也不会影响到其他组件的运行，因为组件被隔离在了不同的进程中。如果应用中的某个进程崩溃了，我们只要重启该进程即可。还可以通过只给每个进程分配足够完成工作的最低限度的权限，来限制潜在漏洞的破坏范围。这种模式被称为最小权限原则。

在 Tauri 中，进程分为两类：主进程和 Webview 进程。每个 Tauri 应用程序都有一个主进程，它作为应用程序的入口点，是唯一可以完整访问操作系统的组件。主进程的主要职责是使用访问权限来创建和管理应用程序窗口、系统托盘菜单或通知。Tauri 实现了必要的跨平台抽象来简化该操作。它还通过核心进程路由所有的 IPC，通过类似于事件总线的机制，可以方便地拦截、过滤和操作 IPC 消息。主进程还应该负责管理全局状态，例如数据库连接。这使你能够轻松地在窗口间同步状态，并保护你的业务敏感数据。主进程自身并不渲染实际的用户界面，它会直接利用操作系统提供的 WebView 库来实现页面渲染，不同的窗口之间会拥有不同的 WebView 进程，WebView 进程用来负责渲染对应的 UI。

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9Pzyr60dgcReJBQnxTto6De4t6VRrPibo5sRX6Ze5kmoN4JD0wBrmqgCZPPw/640?wx_fmt=png)

IPC 模式
------

### Brownfield 模式（默认）

> Brownfield 软件开发是指在现有或遗留软件系统存在的情况下开发和部署新的软件系统。[6]

使用 Tauri 的最简单和直接的模式，因为 Brownfield 模式会尽最大可能尝试与现有的前端项目兼容。在这种模式下，无需现有的浏览器前端项目进行任何操作即可迁移。

### 隔离模式

隔离模式是一种在到达 Tauri 主进程前，拦截并修改由 Webview 进程发送的 Tauri API 信息的 IPC 模式，其完全使用 JS 编写。由隔离模式保障的 JS 代码即称为隔离应用。隔离模式的目的是为开发者提供一种保护机制，防止其应用程序被预料之外或恶意的 Webview 进程调用 Tauri 主进程。隔离模式的需求来自于前端中不可信任的内容所带来的威胁，常见于需要许多依赖的应用。隔离模式在设计之初时设想的最大威胁为开发威胁，因为前端构建工具不仅仅由许许多多嵌套很深的依赖组成，而且还有很多嵌套很深的依赖被打包到最终的网页构建产物中。

*   原理：隔离模式就是在 Webview 进程和 Tauri 主进程之间注入一个安全的应用程序，用以拦截和修改传入的 IPC 信息。它使用 `<iframe>` 的沙盒特性，与 Webview 进程一起安全地执行 JS 代码。Tauri 在加载页面时会强制执行隔离模式，使所有对 Tauri 主进程的 IPC 调用必须先通过沙盒隔离应用程序。当消息准备被传递给 Tauri 主进程时，其就会被使用浏览器的 SubtleCrypto API[5] 实现加密，并传递回主前端程序，之后，它将会被直接传递给 Tauri 主进程来解密和读取数据。
    
*   步骤：
    

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrVyb4jQps0q66nRKicSkWdIRG2sXGOOZiapuAl2BHiaRibrqe2LcBkKh95g/640?wx_fmt=png)

image.png

*   缺点
    

*   由于信息经过加密，所以隔离模式相比 Brownfield 模式而言存在额外开销。除去性能敏感的应用 (使用很少依赖来提升性能的应用) 之外，使用 AES-GCM 算法来加解密 IPC 信息会对大部分应用造成相对较小的性能影响。
    
*   Windows 平台上，Webview 限制了因为沙盒环境下加载 `<iframe>` 标签内的外部文件，Tauri 在构建时实现一些步骤将脚本注入，但是 ES Modules 可能无法正常加载。
    

安全性
---

*   动态 AOT：Tauri 应用启动时将多次进行编译。通过使用 Tauri 提供的动态预编译器，可以生成每个会话都不同的代码引用，但技术上一致的代码单元；
    
*   函数 ASLR：函数地址空间布局随机化 (Address Space Layout Randomization) 将在运行时随机调整函数名称，且可以实现 OTP 哈希功能，这样将永远不会出现相同的会话。在 Tauri 应用启动时，或可选在每次执行后，随机生成函数名称。每个函数指针均使用 UID 以防止静态攻击；
    
*   自杀式函数注入：一种高级的 ASLR 技术，Rust 在运行时载入进入 Webview 的闭包 Promise 和随机的句柄，API 接口在 Promise 处理中就被锁定，在执行完毕后被立即设置成 null；
    

> 可以有效防止恶意页面被加载

*   使用 Event Bridge：Tauri 内部的信息通信使用 Event Bridge，Event Bridge 用来保证只能传递信息和指令给一个指定的中间代理，而不是直接传递不安全的函数调用；
    
*   API Allow List：Tuari 提供的 API 都是关闭状态，若没有启用相关 API，应用构建时不会包括相关功能函数的代码。这可以减少二进制文件大小及攻击面。同时 API 还有严格的权限选项，如文件读写相关 API 可以设置只读 / 只写 / 指定目录或文件等功能；
    
*   CSP：Tauri 会对本地的 HTML 页面强制使用内容安全策略，本地脚本经过哈希处理，同时样式、外部脚本经由加密随机字符串引用，防止禁止内容被加载；
    
*   反编译难：Tauri 在构建时，会将相关前端代码直接构建在二进制可执行文件中，这意味着和 Electron 的 ASAR 文件不同，具体的代码无法被轻松地反编译；
    
*   一次性 Token 和哈希：使用 OTP 加盐哈希处理重要的信息，可以在前端和 Rust 后端之间加密信息；
    

Tauri-egui[6]
-------------

因为 Tauri 应用的前端是用了 Web 相关的技术栈，因此在运行时总有办法来使用开发者模式 / 调试工具等来进行检查元素等。在某些情况下，如密码输入等场景，我们希望能够保证前端的 UI 展示是无法被修改的，这时可以使用 Tauri-egui 这个库，这个库对 egui 进行了封装，可以使用 Rust 来编写前端的 UI。

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrnV6DlVC5JW4kU44jt8IiclP60Eyg05bZfibTgoFTRIicdricnYS9s23l9Q/640?wx_fmt=png)

优点
--

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrPe3YlZ4BQrxWkWJNicaVC2t0pibNxLedia6SIe4l69KGicjmMQq7jou8Sg/640?wx_fmt=png)

Benchmark
---------

*   内存使用
    

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrGtRztLvcPibEhIgAYPicKOoBssIQtS4GQVsumQv2SVDo8bpyy5D5KicdA/640?wx_fmt=png)

*   构建产物体积
    

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIp4w8TSMEyK6J2UFXF9PzyrJfpkz0TBcU6iaAAFuibos3fJe6e3b2IzOwl9uFEAqOT5zs5VQqUV1ANg/640?wx_fmt=png)

缺点
--

*   兼容性：由于 Tauri 使用的是系统 Webview，因此在构建时前端的代码需要做 polyfill；且 Windows 平台上，由于 Webview2 只在 Windows10/11 上有默认推送安装，要想在 Windows7/8 等较低版本的平台上运行的话，还需要另外安装 Webview2 的运行时。
    

参考资料
====

[1] 在 macOS 中打开系统 Webview 的检查元素开关, https://blog.jim-nielsen.com/2022/inspecting-web-views-in-macos/

[2] 微软开始在 Teams 应用中放弃 Electron, https://blog.devgenius.io/microsoft-is-finally-ditching-electron-9e081757f9db

[3] VSCode 优化括号颜色匹配, https://code.visualstudio.com/blogs/2021/09/29/bracket-pair-colorization

[4] Tauri 官网, https://tauri.app

[5] Rust Wikipedia, https://zh.wikipedia.org/zh-cn/Rust

[6] Brownfield, https://en.wikipedia.org/wiki/Brownfield_(software_development)

❤️ 谢谢支持
-------

以上便是本次分享的全部内容，希望对你有所帮助 ^_^

喜欢的话别忘了 分享、点赞、收藏 三连哦~。

欢迎关注公众号 **ELab 团队** 收货大厂一手好文章~

> **智能学习前端团队** 自创立以来，团队专注于打破大众对教育的刻板印象，突破固有的教学思维，攻破各类教学屏障。旨在为每一位学生制定最合适的学习方案，予以因材施教，使优质教育随 " **触** " 可达。
> 
> *   字节跳动校 / 社招内推码: DYJ95U9
>     
> *   投递链接: https://job.toutiao.com/s/YK7wdrk
>     
> 
> 可凭内推码投递 **智能学习前端团队** 相关岗位哦~

### 参考资料

[1]

WRY: _https://github.com/tauri-apps/wry_

[2]

WebKit.WKWebview: _https://developer.apple.com/documentation/webkit_

[3]

Webview2: _https://developer.microsoft.com/zh-cn/microsoft-edge/webview2/_

[4]

WebKitGTK: _https://webkitgtk.org/_

[5]

SubtleCrypto API: _https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto_

[6]

Tauri-egui: _https://github.com/tauri-apps/tauri-egui_

- END -