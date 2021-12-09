> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Lkyj42NtvqEj63DoCY5btQ)

困惑很多人的并发问题
----------

在网络开发中，我发现有很多同学对一个基础问题始终是没有彻底搞明白。那就是一台服务器最大究竟能支持多少个网络连接？我想我有必要单独发一篇文章来好好说一下这个问题。

很多同学看到这个问题的第一反应是 65535。原因是：“听说端口号最多有 65535 个，那长连接就最多保持 65535 个了”。是这样的吗？还有的人说：“应该受 TCP 连接里四元组的空间大小限制，算起来是 200 多万亿个！”

如果你对这个问题也是理解的不够彻底，那么今天讲个故事讲给你听！

一次关于服务器端并发的聊天
-------------

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxzv6h8iaQXTt7zGndbmn4sht6iasE6Y5LdW64ZAgdkxibTtah5qyGoMKqQ/640?wx_fmt=png)

> "TCP 连接四元组是源 IP 地址、源端口、目的 IP 地址和目的端口。任意一个元素发生了改变，那么就代表的是一条完全不同的连接了。拿我的 Nginx 举例，它的端口是固定使用 80。另外我的 IP 也是固定的，这样目的 IP 地址、目的端口都是固定的。剩下源 IP 地址、源端口是可变的。所以理论上我的 Nginx 上最多可以建立 2 的 32 次方（ip 数）×2 的 16 次方（port 数）个连接。这是两百多万亿的一个大数字！！"

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxzjVPBaZFsRWxsP4PT92osvJcEicb4giceBWX3pO37OnI6qCDiaeHZ63Pg/640?wx_fmt=png)

> "进程每打开一个文件（linux 下一切皆文件，包括 socket），都会消耗一定的内存资源。如果有不怀好心的人启动一个进程来无限的创建和打开新的文件，会让服务器崩溃。所以 linux 系统出于安全角度的考虑，在多个位置都限制了可打开的文件描述符的数量，包括系统级、用户级、进程级。这三个限制的含义和修改方式如下："

*   系统级：当前系统可打开的最大数量，通过 fs.file-max 参数可修改
    
*   用户级：指定用户可打开的最大数量，修改 / etc/security/limits.conf
    
*   进程级：单个进程可打开的最大数量，通过 fs.nr_open 参数可修改
    

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxbO8LmdRkuWzWnGMGR2uG5luAtmDxb11XRjq17ntiasSHbwf5NEZf7hA/640?wx_fmt=png)

> "我的接收缓存区大小是可以配置的，通过 sysctl 命令就可以查看。"

```
$ sysctl -a | grep rmem
net.ipv4.tcp_rmem = 4096 87380 8388608
net.core.rmem_default = 212992
net.core.rmem_max = 8388608


```

> "其中在 tcp_rmem" 中的第一个值是为你们的 TCP 连接所需分配的最少字节数。该值默认是 4K，最大的话 8MB 之多。也就是说你们有数据发送的时候我需要至少为对应的 socket 再分配 4K 内存，甚至可能更大。"

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxS9JZ4Rf3jw5Cic6k7yqkJuEn87oJahc7TO5wGkUBkh6P8YRTUFTINAA/640?wx_fmt=png)

> "TCP 分配发送缓存区的大小受参数 net.ipv4.tcp_wmem 配置影响。"

```
$ sysctl -a | grep wmem
net.ipv4.tcp_wmem = 4096 65536 8388608
net.core.wmem_default = 212992
net.core.wmem_max = 8388608


```

> "在 net.ipv4.tcp_wmem" 中的第一个值是发送缓存区的最小值，默认也是 4K。当然了如果数据很大的话，该缓存区实际分配的也会比默认值大。"

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxKBBZ0b8hQGzMib9I0Sw4MlFfgVqt5QSMZDgBTwFa9aeVXN1coEpGArQ/640?wx_fmt=png)

服务端百万连接达成记
----------

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxibQjHJpozOYHaYGiaD3txd4MmGXcDThW96z7oMbH9PlVaaC417GblyKA/640?wx_fmt=png)

> “准备啥呢，还记得前面说过 Linux 对最大文件对象数量有限制，所以要想完成这个实验，得在用户级、系统级、进程级等位置把这个上限加大。我们实验目的是 100W，这里都设置成 110W，这个很重要！因为得保证做实验的时候其它基础命令例如 ps，vi 等是可用的。“

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxVHMr5VAUIcFpS1P5bcUnAe9lGkkn8kd6hGAYl5w7h3x7EqV2qt4WDQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxprMrmcJjPQc142jpJNg0piauyFgqn0ic4lwuK5Na8S7lk3NLENhEOxJA/640?wx_fmt=png)

活动连接数量确实达到了 100W：  

```
$ ss -n | grep ESTAB | wc -l  
1000024


```

当前机器内存总共是 3.9GB，其中内核 Slab 占用了 3.2GB 之多。MemFree 和 Buffers 加起来也只剩下 100 多 MB 了：

```
$ cat /proc/meminfo
MemTotal:        3922956 kB
MemFree:           96652 kB
MemAvailable:       6448 kB
Buffers:           44396 kB
......
Slab:          3241244KB kB


```

通过 slabtop 命令可以查看到 densty、flip、sock_inode_cache、TCP 四个内核对象都分别有 100W 个：

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxuw4Y9XO0IHuzkHBwnCFTSkJRvKYwlwibv1kvMibfPcEBaiaVDekMtXWYA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/BBjAFF4hcwoo5ibkEGbrfCkOXicTNTjPGxPkdoiaTrf5Sdzq4ibuIIB8mN5YqjlSPR1JLTmcIdss3RJqs7XnvNNmSA/640?wx_fmt=png)

结语
--

互联网后端的业务特点之一就是高并发. 但是一台服务器最大究竟能支持多少个 TCP 连接，这个问题似乎却又在困惑着很多同学。希望今天过后，你能够将这个问题踩在脚下摩擦！

学习是一件痛苦的事情，尤其咱们号里很多读者朋友都是工作满一天了再来看我的技术号的文章的。我一直都在琢磨到底怎么样组织技术内容形式，能让大家理解起来更能省一点脑细胞呢。这篇服务器的最大并发数的文章是早就想发的，但是写了两三个版本都不满意。今天终于想出了一种让大家更容易理解的方式，算过了自己这关了。

如果您喜欢我的文章、并觉得它有用，期望您能不吝把它转发到你的朋友圈，技术群。或者哪怕是点个赞，点个再看都可以。触达更多的技术同学并收获大家的反馈将极大地提升彦飞的创作动力！

改天再讲客户端，敬请期待！！