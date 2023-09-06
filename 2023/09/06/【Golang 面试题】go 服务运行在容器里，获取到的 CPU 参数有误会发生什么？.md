> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/-BItOf80Oem-0QwCC7ZGRQ)

1.  G 多了会发生什么？新创建的 G 首先放入本地队列，如果本地队列满了，放全局队列。
    
2.  M 多了会发生什么？M 空闲的话，就会空闲，会被回收或者睡眠。
    
3.  P 多了会发生什么？P 实际与 CPU 核数是有关联的，如果本身机器 64 核，容器 cgroup 对资源进行隔离，最终容器使用的核数可能，比如 4 核。结果就导致 P 变多了，就导致运行调度器的物理线程也会变多，从而增加上下文切换的开销，也就是 sys cpu 变高。
    

**具体影响来自网友真实的环境：**  

```
我们踩过这个坑，现象是耗时猛增本来不到1ms的，这种情况下到了几十ms。

```

**1.1 Uber 测试**

uber 对容器下获取 CPU 核数进行压测，数据如下：

<table width="1002"><thead><tr><th>GOMAXPROCS</th><th>RPS</th><th>P50 (ms)</th><th>P99.9 (ms)</th></tr></thead><tbody><tr><td>1</td><td>28,893.18</td><td>1.46</td><td>19.70</td></tr><tr><td class="">2 (equal to quota)</td><td>44,715.07</td><td>0.84</td><td>26.38</td></tr><tr><td>3</td><td>44,212.93</td><td>0.66</td><td>30.07</td></tr><tr><td>4</td><td>41,071.15</td><td>0.57</td><td>42.94</td></tr><tr><td>8</td><td>33,111.69</td><td>0.43</td><td>64.32</td></tr><tr><td>Default (24)</td><td>22,191.40</td><td>0.45</td><td>76.19</td></tr></tbody></table>

当 GOMAXPROCS 增加到 CPU 配额以上时，我们看到 P50 略有下降，但显著增加到 P99。我们还看到，处理的 RPS(Request per second，每秒请求数) 总量也有所下降。

**二、如何获取容器下真实的 CPU 核数？**  

目前 Go 官方并无好的方式来规避在容器里获取不到真正可使用的核心数这一问题，而 Uber 提出了一种 Workaround 方法，利用 uber-go/automaxprocs 这一个包，可以在运行时根据 cgroup 为容器分配的 CPU 资源限制数来修改 GOMAXPROCS。

**2.1** **Uber 库**
==================

```
go get -u go.uber.org/automaxprocs

```

**2.2** **使用案例**

```
package main
// Importing automaxprocs automatically adjusts GOMAXPROCS.
import _ "go.uber.org/automaxprocs"
// To render a whole-file example, we need a package-level declaration.
var _ = ""
func main() {}

```

**2.3 automaxprocs 解决问题**

线上容器里的服务通常都对 CPU 资源做了限制，例如默认的 4C。但是在容器里通过 lscpu 仍然能看到宿主机的所有 CPU 核心：

automaxprocs 自动识别 cgroup 为容器分配的 cpu 限制，来纠正 gomaxprcos，保证 go 服务获取到真实的 CPU 核数。

**2.4 原理**

包级别的 init() 函数（代码位置在 automaxpROCs/automaxpROCs.go）实现了导入这个包即可产生作用：

![](https://mmbiz.qpic.cn/sz_mmbiz/caBJ5fWjGksTibmv3NRMHhSA4jwticnAPmNQF7OzSDic7amuVrRbwnLCM221zKwugk1Und6cukW9uYKJgYMu1h5hA/640?wx_fmt=other)

核心函数就是 maxpROCs.Set()；这个函数会从当前的 cgroups 里获取设置的 CPU quota，然后转换为合适的 GOMAXPROCS。

![](https://mmbiz.qpic.cn/sz_mmbiz/caBJ5fWjGksTibmv3NRMHhSA4jwticnAPmLbqK6iaAlhPoVwBd6P3On17Zbn7wapqcQxnwwE0VuIcMA3NMPqrF1CA/640?wx_fmt=other)

接下来获取进程的 cgroup 信息和 mountinfo 信息，然后通过得到进程对应的 cpu 对应 subsystem 的 CGroup path。得到 cGroup 的目录路径，我们就得到如下信息：

1.  cfs.cpu_period_us 文件记录了调度周期，单位是 us；默认值一般是 100'000，即 100 ms
    
2.  cfs.cpu_quota_us 记录了每个调度周期进程允许使用 cpu 的量，单位也是 us。值为 -1 表示无限制；对于 4C 的容器，这个值一般是 400'000 
    

```
Aside：这两个值限制的是进程使用 cpu 的时间。
上述设置下表示：每 100ms 的调度周期内，该进程可以使用 400ms 的 
cpu 时间，所以看起来的效果是可以使用4个CPU核心
更详细的内容请参考 Linux kernel 的文档：
CFS Bandwidth Control

```

quota 和 period 的比值就是 docker 为容器设置的 CPU 核数配置。这个值也是 automaxpROCs 为 runtime.GOMAXPROCS() 设置的值。

大家感兴趣的可以去看看源码。 

******| 希望能帮助你学习到新的知识点，欢迎提出宝贵的意见！！******