> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [learn.microsoft.com](https://learn.microsoft.com/zh-cn/windows-hardware/design/device-experiences/wireless-projection-understanding)

Windows 10 提供无缝无线投影体验。 在构建无线投影解决方案时，了解该功能包括的内容非常重要。

[](#the-windows-10-wireless-projection-user-interface)

Windows 10 无线投影用户界面
-------------------

首先，Windows 提供允许用户连接到无线接收器的本机连接体验。 可通过多种方式连接到无线接收器：

*   **通过操作中心。** 在操作中心 (图 1) ，单击 **“连接快速操作**”。
    
*   **使用热键。** 选择 **Windows** **徽标键 +****K** (图 2) 。
    
*   使用设备选取器 UI。 支持强制转换的 Windows 应用包含设备选取器 UI，例如 Windows Movie & TV 应用 (图 3) 或 Edge 浏览器中的 **“强制转换为设备**” 功能。
    

<table aria-label="表 1"><thead><tr><th>&nbsp;</th><th>&nbsp;</th><th>&nbsp;</th></tr></thead><tbody><tr><td><img class="" src="https://learn.microsoft.com/zh-cn/windows-hardware/design/device-experiences/images/wireless-projection-image002.png"></td><td><img class="" src="https://learn.microsoft.com/zh-cn/windows-hardware/design/device-experiences/images/wireless-projection-image003.png"></td><td><img class="" src="https://learn.microsoft.com/zh-cn/windows-hardware/design/device-experiences/images/wireless-projection-image004.png"></td></tr><tr><td>图 1：连接快速操作</td><td>图 2：连接 UI</td><td>图 3：设备选取器 UI</td></tr></tbody></table>

Windows 10 支持和管理用于创建无线投影流的两种不同方法。 这两种方法都由 Windows 在后台处理，并利用上面显示的完全相同的 UI。

[](#wireless-projection-using-miracast)

使用 Miracast 的无线投影
-----------------

自第一个 Windows 10 版本以来，对 Miracast 的支持一直存在，自那时以来，Windows 对 Miracast 作为投影体验的投资只会增加。

基于 Miracast 的无线投影有多项优点：

*   一种简单的连接体验，允许用户查找并连接到 Miracast 接收器。
    
*   实现 Miracast 标准，以确保与数亿个 Miracast 设备的互操作性。
    
*   经过微调的本机 RTSP 堆栈适用于 Miracast，无需在 Windows 10 操作系统之外使用其他软件。
    
*   支持 UIBC (用户输入返回通道) ，它允许 Miracast 接收器 (触摸、触笔、鼠标、键盘和游戏板) 输入来控制 Miracast 发送方，前提是 --- 且仅当 --- 用户明确允许这样做。
    
*   与行业领先的 Microsoft Miracast 接收器 (Microsoft 无线显示适配器) 以及领先的第三方 Miracast 接收器的高质量互操作性。
    
*   如果 HDCP 密钥) 存在，则支持 “受保护内容” (投影。
    
*   支持在连接到 Miracast 接收器时使用基于 PIN 的配对。
    
*   持久配置文件，记住你是否过去已连接到特定的 Miracast 接收器。 能够记住用于重新连接到 Miracast 接收器的配置文件，减少了在后续连接上进行连接所需的时间。
    
*   支持可启用其他功能的 Miracast 扩展，从而显著改善 Miracast 体验。
    

[](#wireless-projection-over-an-existing-wi-fi-network)

通过现有 Wi-Fi 网络进行无线投影
-------------------

观察到，在 90% 的情况下，当用户启动无线投影流时，他们使用的设备已连接到现有的 Wi-Fi 网络，无论是家庭还是企业。 为此，Microsoft 扩展了通过本地网络而不是通过 Windows 10 创意者更新中的直接无线链接发送 Miracast 流的功能。

通过现有 Wi-Fi 网络进行无线投影有多项优点：

*   此解决方案利用现有连接，可显著缩短内容的投影时间。
    
*   电脑无需管理两个同时出现的连接（一个 Wi-Fi 连接和一个 Wi-Fi 直接连接，都是连接到到接收器），每个连接只能获得最大带宽的一半。
    
*   使用现有连接可简化无线设备需要执行的工作，这既提高了可靠性，又提供了非常稳定的流。
    
*   用户不必更改连接到接收方的方式，因为他们使用的 UX (如图 2-4) 所示。
    
*   仅当通过以太网或安全 Wi-Fi 网络信任连接时，Windows 才会选择通过现有连接投影。
    
*   无需对电脑上的无线驱动程序或硬件进行更改。
    
*   这也适用于未针对 Miracast over Wi-Fi Direct 进行优化的旧式无线硬件。
    
*   Windows 会自动检测接收方何时支持此功能，并在适用时通过现有网络路径发送视频流。