> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzM0MjcyMQ==&mid=2650120431&idx=1&sn=b624b666ac9500976335cd88d75d17fa&chksm=beda7b8189adf2976c8920bd9180abaff29f1e8fa493841747dc72704d4607ecf0b7611f7777&mpshare=1&scene=1&srcid=062370JgGOsa5AgVbpTePUfu&sharer_sharetime=1624415596584&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

文 | 局长  

出品 | OSC 开源社区（ID：oschina2013）

V 语言开发团队发布了首个使用 V 编写的操作系统 —— Vinix，并表示此操作系统现在可以运行 mlibc 和 bash。  

下载 nightly 版本进行体验 >>> ISO 镜像地址：https://github.com/vlang/vinix/releases/download/nightly/vinix-nightly.iso

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKicPh4BQkKI7KlvYXlmQ3iauOOzHXKHA3xP7TibiarrH3FXSt9c5OOC35BAhIEsVCnljyKr9qiaiahicxmYQ/640?wx_fmt=png)

Vinix 的源代码已遵循 GPLv2 开源许可协议托管在 GitHub，其 repo 显示它采用 V 编写，致力于成为一款现代、快速且有用的操作系统。

Vinix 暂定的目标如下：

*   保证代码尽可能简单易懂
    
*   尽量使用 V 编写
    
*   制作一个可在真实硬件上运行的可用操作系统，而不仅仅是运行在模拟器上
    
*   面向现代 64 位架构和 CPU 特性
    
*   与 Linux 保持良好的源代码级兼容性，以便移植程序
    

在谈及为何创建 Vinix 时，开发团队给出的理由是：

*   探索 V 在裸金属中进行编程的能力
    
*   针对裸金属编程的不常见需求，通过提供反馈来改进编译器
    
*   为了好玩
    

根据 Vinix 的 Readme，目前必须要安装 Docker 并让其正常运行才能构建 Vinix，也就是说暂不支持直接把 Vinix 安装到电脑上。具体步骤和注意事项：https://github.com/vlang/vinix

**延伸**

V 是一个集合了 Go 的简单和 Rust 的安全特性的静态语言，作者表示 V 与 Go 非常相似，如果你了解 Go，那么就已经了解 80% 的 V。  

V 在 Go 的基础上进行改进之处：https://vlang.io/compare#go。

V 主要特性

*   简单（作者声称可以在不到一小时内学习 V）
    
*   快速编译（编译器只有 400kb，而且无第三方依赖）
    
*   易于开发：V 在不到一秒钟的时间内完成编译
    
*   安全：没有 null、没有全局变量、没有未定义的值、边界检测、默认使用 Immutable 结构体
    
*   支持 C/C++ 转换
    
*   方便使用的交叉编译
    
*   提供跨平台 UI 库
    
*   内置图形库
    
*   内置 ORM
    
*   内置 Web 框架
    
*   ……
    

END

  

话说，你更喜欢 V 语言、Go 语言还是 Rust 语言呢？你觉得 V 语言是否有潜力呢？