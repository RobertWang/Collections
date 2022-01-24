> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_19604635/article/details/105517805)

### 文章目录

*   *   *   [1. 公司 IT 之现状](#1_IT_1)
        *   [2. 虚拟化特征](#2__7)
        *   [3. 虚拟化厂商、技术](#3__18)
        *   [4. Hyper-v3.0 新功能](#4_Hyperv30_26)
        *   [5. Hyper-v 3.0 部署需求](#5_Hyperv_30__30)
        *   [6. Hyper-v 3.0 网络](#6_Hyperv_30__37)
        *   *   [6.1. 网络类型解释](#61__44)
        *   [7. 问题](#7__60)
        *   [8. Hyper-v 备份](#8_Hyperv_76)
        *   [9. 最佳管理平台](#9__81)

### 1. 公司 IT 之现状

*   硬件服务器过多，投入过大
    *   服务器硬件使用率低
        *   服务器管理和维护工组繁杂

### 2. [虚拟化](https://so.csdn.net/so/search?q=%E8%99%9A%E6%8B%9F%E5%8C%96&spm=1001.2101.3001.7020)特征

*   打包
    *   将整个系统. 包括硬件配置、Windows 以及程序打包成文档
        *   整合
    *   一台物理服务器上同时跑多台虚机
        *   备份
    *   既然是文件当然容易备份和恢复
        *   迁移
    *   可以在其他服务器上不加修改的运行

### 3. 虚拟化厂商、技术

<table><thead><tr><th align="center"></th><th align="center">vmware</th><th align="center">citrix</th><th align="center">Microsoft</th></tr></thead><tbody><tr><td align="center">服务器虚拟化</td><td align="center">VMware ESX</td><td align="center">XenServer</td><td align="center">Hyper-v</td></tr><tr><td align="center">应用程序虚拟化</td><td align="center">VMware ThinApp</td><td align="center">XenApp</td><td align="center">App-v/RD App</td></tr><tr><td align="center">虚拟桌面架构</td><td align="center">Vmware View</td><td align="center">XenDesktop</td><td align="center">RDS VDI</td></tr></tbody></table>

### 4. Hyper-v3.0 新功能

![](https://imgconvert.csdnimg.cn/aHR0cDovL3lsenowNy1pbWFnZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAyMC8wNC8xNC9mM2E1MWI5M2M1ZjM0LmpwZw?x-oss-process=image/format,png)

### 5. Hyper-v 3.0 部署需求

*   服务器 CPU 支持虚拟化 (Intel 支持 VT, AMD 支持 AMD-V) 并且在 BIOS 中启用虚拟化支持
*   在 BIOS 中企业 DEP(数据执行保护)
*   安装 Windows Server 2012 , Windows 8 X64 系统
*   安装 Hyper-v 组件

### 6. Hyper-v 3.0 网络

*   有两种类型: 网络适配器和旧版的网络适配器
    *   网络适配器: 支持 10Gbps, 需要安装驱动才能使用
    *   旧版网络适配器: 100Mb 网卡, PXE 网卡
*   三种网络类型: 外部网络, 内部网络和专用网络

#### 6.1. 网络类型解释

![](https://imgconvert.csdnimg.cn/aHR0cDovL3lsenowNy1pbWFnZS5vc3MtY24tYmVpamluZy5hbGl5dW5jcy5jb20vMjAyMC8wNC8xNC84MjJiNjQwYmY1ZWRiLmpwZw?x-oss-process=image/format,png)

现在公司中有两个服务器，连到公网 HostA、HostB  
Host A 假设只有一个网卡，现在可以上网了。  
不能只有宿主机能上网，还需要虚机来上网，  
这种情况下就需要 为宿主机创建一个外部网络。  
物理机 hostA 会断开 10S 网络左右，Host A 创建一个网卡  
网卡这里称为 EX，原先的网卡变成了交换机，这样都可以上网  
从网络角度来讲 虚拟机和物理机 地位是一样的 都是交换机上的两个接口。

内部网络：创建一个内部网络就意味着创建一个内部交换机。跟这些公网都没有任何关系。创建完之后会在宿主机的本地连接里面创建一个网卡，与外部网络隔离 但是可以访问宿主机的资源。

专用：完全独立，不能访问 int 宿主机。

### 7. 问题

同一个交换机连接了两台 Host 主机，IP 地址分别为 192.168.0.10/24  
和 172.16.0.20/24 每台 Host 上都有一个 VM , 网卡都是外部类型,  
VM1 的 IP 地址为 10.0.0.10/8, VM2 为 10.0.0.20/8  
**问: VM1 和 VM2 能相互访问不?**

答案：可以

1. 同一台主机的两个内部类型网卡能不能互通？2. 会不会创建两个内部的虚拟交换机？

1. 不能。是独立的。  
2. 是的。

### 8. Hyper-v 备份

System Center Date Protection Manager  
Symantec Backup EXEC

### 9. 最佳管理平台

System Center Virtual Machine Manager

*   统一的管理平台, 更加方便的虚拟机和存储的迁移
*   对所有的宿主机和虚拟机性能全面的监控
*   智能的对虚拟机进行迁移
*   P2V, V2V
*   Portal 功能，更加方面的为公司提供虚拟化服务