> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CO-k2nv-PK0Ij-V5lTbUEQ)

> 雷池一款足够简单、足够好用、足够强的免费 WAF。

这个时代，HTTP 协议基本统治了整个互联网，搞技术的兄弟们谁还没个网站。但是你们知道吗，网络上的攻击和扫描流量非常非常非常多，即使是无人问津的小网站，每天也会被遭到大量黑客的攻击。

今天推荐给大家的就是这样一款网站防护工具，一款广受好评的社区 `WAF` 项目：**雷池**。

雷池是什么
-----

雷池一款足够简单、足够好用、足够强的免费 `WAF`。基于业界领先的语义引擎检测技术，作为反向代理接入，保护你的网站不受黑客攻击。

![](https://mmbiz.qpic.cn/mmbiz_png/5tqrztXFpukfJYloXnkricHH38EtQSEF0HWB1PtGh8NjLGD1sf8ALQoRia9jPPl9ZEic7iaGHoD7JOqKaCZBiaq5eiag/640?wx_fmt=png)

核心检测能力由智能语义分析算法驱动，专为社区而生，不让黑客越雷池半步。

![](https://mmbiz.qpic.cn/mmbiz_png/5tqrztXFpukfJYloXnkricHH38EtQSEF0vhhddWCmQ6D9C7xZAN2A6sxSNsjzuqfKoRNM7j8lDDI2BJwFib5KrbA/640?wx_fmt=png)

工作原理
----

下图是一个简单的网站流量拓扑，外部用户发出请求，经过网络最终传递到网站服务器。

![](https://mmbiz.qpic.cn/mmbiz_png/5tqrztXFpukfJYloXnkricHH38EtQSEF0tbicUCSwiamAlh8MpAc7q3X5CcfPlQQwIDK3znRic4Vnia9qoiccc9pH5Sw/640?wx_fmt=png)

此时，若外部用户中存在恶意用户，那么由恶意用户发出的攻击请求也会经过网络最终传递到网站服务器。

雷池以反向代理方式接入，优先于网站服务器接收流量，对流量中的攻击行为进行检测和清洗，将清洗过后的流量转发给网站服务器。

![](https://mmbiz.qpic.cn/mmbiz_png/5tqrztXFpukfJYloXnkricHH38EtQSEF0kupcEttLreSGOel6s1uZxdOg6iaJagocj9LLM4DEh0hZAXFQn7XPh7w/640?wx_fmt=png)

通过以上行为，最终确保外部攻击流量无法触达网站服务器。

使用方式
----

项目由若干个 Docker 容器组成，clone 仓库后调用 `setup.sh` 即可开始安装，参考如下：

```
git clone git@github.com:chaitin/safeline.git
cd safeline
bash ./setup.sh


```

安装完成后访问本地 `https://127.0.0.1:9443/` 即可开始使用。

**配置需求**

*   操作系统：Linux
    
*   指令架构：x86_64
    
*   软件依赖：Docker 20.10.6 版本以上
    
*   软件依赖：Docker Compose 2.0.0 版本以上
    
*   最小化环境：1 核 CPU / 1 GB 内存 / 5 GB 磁盘
    

项目特性
----

**便捷性**

采用容器化部署，一条命令即可完成安装，0 成本上手

安全配置开箱即用，无需人工维护，可实现安全躺平式管理

**安全性**

首创业内领先的智能语义分析算法，精准检测、低误报、难绕过

语义分析算法无规则，面对未知特征的 0day 攻击不再手足无措

**高性能**

无规则引擎，线性安全检测算法，平均请求检测延迟在 1 毫秒级别

并发能力强，单核轻松检测 2000+ TPS，只要硬件足够强，可支撑的流量规模无上限

**高可用**

流量处理引擎基于 Nginx 开发，性能与稳定性均可得到保障

内置完善的健康检查机制，服务可用性高达 99.99%

> Github 仓库：`https://github.com/chaitin/safeline`