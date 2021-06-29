> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMDM0MzQ4Mg==&mid=2451065208&idx=1&sn=a9119b1904682c3a68c8e3a250e13016&chksm=8cbd2c2dbbcaa53bc75618f9036b45fbd9c08377d151d6ad868c42a40e383b42048b161e83a3&scene=21#wechat_redirect)

【导语】：无需 “越狱” 即可在 Mac、iPhone 和 iPad 上运行 Windows、Linux 等操作系统。

### 简介

UTM 是适用于 iOS 和 macOS 的全功能系统模拟器和虚拟机主机，基于 QEMU，可以在 Mac、iPhone 和 iPad 上运行 Windows、Linux 等。  
  

![](https://mmbiz.qpic.cn/mmbiz_png/DSU8cv1j3ibSDwSKFMSshpeTVDG881gLcRic45ytKFL3rEWFA8MdRxCRXqpBSQ62O6O4607d44OjfvXXibaEhwKrA/640?wx_fmt=png)

UTM 具有以下特点：

*   使用 QEMU 的完整系统仿真
    
*   支持 30 多种处理器，包括 x86_64、ARM64 和 RISC-V
    
*   使用 SPICE 和 QXL 的 VGA 图形模式
    
*   文本终端模式
    
*   支持 USB 设备
    
*   支持 JIT 的加速
    
*   使用最新 API 从头开始为 macOS 11 和 iOS 11+ 设计的前端
    
*   直接从设备创建、管理和运行虚拟机
    

UTM/QEMU 需要动态代码生成 (JIT) 以获得最大性能，iOS 设备上的 JIT 需要越狱；而 UTM SE 使用线程解释器，其性能优于传统解释器，但仍比 JIT 慢。这种技术与 iSH 为动态执行而执行的方式类似。因此，UTM SE 不需要越狱或任何 JIT 解决方法，并且可以作为常规应用程序加载。

为了优化大小和构建时间，UTM SE 仅支持 ARM、PPC、RISC-V 和 x86 架构。

项目地址是：

https://github.com/utmapp/UTM

### Mac 上的 UTM

*   安装。直接下载 UTM 安装包进行安装即可，下载地址是：
    
    https://github.com/utmapp/UTM/releases/latest/download/UTM.dmg
    
*   UTM 采用 Apple 的 Hypervisor 虚拟化框架，以接近本机的速度在 Apple Silicon 上运行 ARM64 操作系统。在 Intel Mac 上，可以虚拟化 x86/x64 操作系统。对于开发人员和爱好者，还有许多其他仿真处理器，包括：ARM32、MIPS、PPC 和 RISC-V。  
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/DSU8cv1j3ibSDwSKFMSshpeTVDG881gLc6hwNf7OhGfibO0NdyUPJ95vWiaUuXlkLM91sDdECBsADD8WAmVEk8icMg/640?wx_fmt=png)
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/DSU8cv1j3ibSDwSKFMSshpeTVDG881gLcib7ppaULfpkiaUNR5mas4ZHVib9bOTcHbNzoawhN96LzkIBibdCOraBYkQ/640?wx_fmt=png)
    
*   与其他免费虚拟化软件不同，UTM 是为 macOS 创建的，仅适用于 Apple 平台。UTM 的外观和感觉就像一个 Mac 应用程序，具有您所有隐私和安全功能。  
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/DSU8cv1j3ibSDwSKFMSshpeTVDG881gLcHWUicaZj0oYUhqa2Ak2eZndFEB5Vd4P7GAxA80L9seIb9Cztwn4eWFQ/640?wx_fmt=png)
    
*   在 UTM 的底层是 QEMU，这是一个有着数十年历史的免费开源仿真软件，被广泛使用和积极维护。尽管 QEMU 功能强大，但过多的命令行选项和标志对用户来说不太友好。UTM 提供了 QEMU 的灵活性，而无需相对应的陡峭学习曲线。  
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/DSU8cv1j3ibSDwSKFMSshpeTVDG881gLcniaYZSBiaBcicGmEe40HvFAykARmocoeO5MVqBjiacQuIghJfibkM1U7SVw/640?wx_fmt=png)
    

### iOS 上的 UTM

*   iOS 11、12、13：UTM 不需要越狱即可使用
    
*   iOS 14.2、14.3：如果有 Apple A12 或更新的芯片，UTM 不需要越狱；否则需要越狱
    
*   iOS 14.0、14.1、14.4 或更高版本，需要越狱
    
*   安装。先安装 AltStore，添加源 https://alt.getutm.app，通过 AltStore 下 载 UTM
    

- EOF -