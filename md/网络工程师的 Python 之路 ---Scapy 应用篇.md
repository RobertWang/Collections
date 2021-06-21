> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/53264096)

**版权声明：我已加入 “维权骑士”(**[http://rightknights.com](https://link.zhihu.com/?target=http%3A//rightknights.com/)**)的版权保护计划，所有知乎专栏 “网路行者” 下的文章均为我本人（知乎 ID：弈心）原创，未经允许不得转载。**

**如果你喜欢我的文章，请关注我的知乎专栏 “网路行者”**[https://zhuanlan.zhihu.com/c_126268929](https://zhuanlan.zhihu.com/c_126268929)**, 里面有更多像本文一样深度讲解计算机网络技术的优质文章。**

《[Scapy 基础篇](https://zhuanlan.zhihu.com/p/51002301)》里已经向大家介绍了 Scapy 的一些基础知识，这篇将重点讲解 Scapy 的实际应用，包括怎么使用 Scapy 进行 TCP 的 SYN 扫描、ACK 扫描、FIN 扫描、Xmas 扫描, Null 扫描，怎么使用 Scapy 执行 TCP SYN flooding 攻击，ARP 欺骗攻击，DHCP 饥饿攻击，怎么用 Scapy 探测 rogue DHCP 服务器等等。以上所有扫描、攻击方法都可以借助黑客工具完成，比如上述 TCP 各种类型的扫描靠大名鼎鼎的 Nmap 都能实现，而实现其他上述攻击的工具更是数不胜数，但是很少有人知道或者深挖过这些工具背后的原理，“知其然还要知其所以然” 是笔者写这篇《Scapy 应用篇》的初衷。

本篇将融合 Python 代码讲解，如果你对 Python 不熟，可以参考我专栏置顶的《网络工程师的 Python 之路 -- 初级篇》。同以往一样，本篇会以实验的方式一一对上述知识点进行呈现和讲解，文章一星期一更。

**Scapy 实验运行环境和拓扑：**
--------------------

**本篇的实验运行环境以及网络实验拓扑和**《[Scapy 基础篇](https://zhuanlan.zhihu.com/p/51002301)》**完全一样，这里简单回顾一下：**

操作系统：Windows 8.1 上跑 CentOS 7(VMware 虚拟机)

网络设备：GNS3 运行的思科三层交换机

网络设备版本：思科 IOS (vios_12-ADVENTERPRISEK9-M)

**Python 版本：2.7.5 （Python 为使用 Scapy 之必备，关于 Python 的安装教程请参考**[《初级篇》）](https://zhuanlan.zhihu.com/p/34932386)

局域网 IP 地址段：192.168.2.0 /24

运行 Scapy 的客户端: 192.168.2.1

Layer3Switch-1: 192.168.2.11

Layer3Switch-2: 192.168.2.12

Layer3Switch-3: 192.168.2.13

Layer3Switch-4: 192.168.2.14

Layer3Switch-5: 192.168.2.15

**所有的交换机已经预配好了 SSH，用户名: python 密码: 123**

**实验 1 -- TCP SYN 扫描**
----------------------

**实验目的：**使用 TCP SYN 扫描交换机 S1(192.168.2.11)的 22（SSH）， 80 （HTTP）， 123 （NTP）端口，知道如何判断端口是被关闭了 (closed) 还是被过滤了(filtered)，两者各自有什么特征。

**实验原理：**TCP 三次握手的原理和过程相信大家都知道。根据 RFC 793，当发送端的 TCP SYN 包发出后，大致会有下面四种情况发生:

1.  目的端口在接收端打开（也可以说接收端正在侦听该端口），收到 SYN 包的接收端回复 **SYN/ACK 包**给发送端，收到 SYN/ACK 包的发送端此时知道**目的端口是打开的 (open)。**
2.  目的端口在接收端被关闭，收到 SYN 包的接收端回复 **RST 包**给发送端，告知发送端**该目的端口已经被关闭了 (closed)。**
3.  如果发送端和接收端之间有防火墙或者使用 ACL 进行包过滤的路由器，那么 **SYN 包在到达接收端之前就被防火墙或路由器拦截下来，**此时防火墙或路由器会回复一个类型 3（Unreachable，不可达）的 ICMP 包（注意不再是 TCP 包）给发送端告知**该目的端口已经被过滤了 (filtered)。**
4.  如果 ICMP 在防火墙或路由器上被关闭了，这时 SYN 包会被防火墙、路由器 "静悄悄" 地丢弃，路由器和防火墙不会发送类型 3 的 ICMP 包告知发送端。此时发送端收不到任何回应 (no response)，这里我们同样可以判断**该目的端口已经被过滤了 (filtered)。**

知道实验原理后来看代码：

```
from scapy.all import *

target = '192.168.2.11'

ans, unans = sr(IP(dst = target) / TCP(sport = RandShort(), dport = [22, 80, 123], flags = "S"), timeout = 5)

for sent, received in ans:
        if received.haslayer(TCP) and str(received[TCP].flags) == "SA":
                print "Port " + str(sent[TCP].dport) + " of " + target + " is OPEN!"
        elif received.haslayer(TCP) and str(received[TCP].flags) == "RA":
                print "Port " + str(sent[TCP].dport) + " of " + target + " is closed!"
        elif received.haslayer(ICMP) and str(received[ICMP].type) == "3":
                print "Port " + str(sent[TCP].dport) + " of " + target + " is filtered!"

for sent in unans:
        print str(sent[TCP].dport) + " is filtered!"
```

*   代码里涉及到的 Scapy 的 sr()，IP(), TCP()，RandShort() 等函数在《Scapy 基础篇》里已经讲过了，这里不再赘述。
*   这里我们使用 sr() 函数对 192.168.2.11(即 S1) 做端口 22，80，123 的 TCP SYN 扫描（注意 flags = "S"）, timeout 设为 5 秒。sr() 函数返回的是一个元组，该元组下面有两个元素，一个是 Results，一个是 Unanswered，我们用 ans 来表示 Results，也就是被响应的包，用 unans 来表示 Unanswered，表示没有被响应的包。

```
ans, unans = sr(IP(dst = target) / TCP(sport = RandShort(), dport = [22, 80, 123], flags = "S"), timeout = 5)
```

*   ans 和 unans 各自又含两个包，一个是发出去的包，一个是接收到的包，以 ans[0][0]和 ans[0][1]为例，第一个 [0] 表示抓到的第一个包，第二个 [0] 和[1]分别表示第一个包里发出的包和接收到的包。举例如下：

```
>>> ans[0][0] （第一个包里发出的包）
<IP frag=0 proto=tcp dst=192.168.2.11 |<TCP dport=sunrpc flags=A |>>
>>> ans[0][1] （第一个包里接收到的包）
<IP version=4 ihl=5 tos=0x0 len=40 id=10338 flags= frag=0 ttl=255 proto=tcp chksum=0xe11 src=192.168.2.11 dst=192.168.2.1 options=[] |<TCP sport=sunrpc dport=ftp_data seq=0 ack=0 dataofs=5 reserved=0 flags=R window=0 chksum=0x2a01 urgptr=0 |<Padding load='\x00\x00\x00\x00\x00\x00' |>>>
```

*   正因如此，所以下面 for loop 里的 sent, received 分别代表的是 ans[0][0]、ans[0][1](抓到的第一个端口为 22 的包)，ans[1][0]、ans[1][1]（抓到的第二个端口为 80 的包）以及 ans[2][0]、ans[2][1]（抓到的第三个端口为 123 的包）里的内容

```
for sent, received in ans:
```

*   haslayer() 函数返回的是布尔值，用来判断**从接收端返回的包 (received)** 里所含协议的类型，这里用来判断该 received 包是否包含 TCP 协议，并且该包里 TCP 的 flag 位是否为 SA, SA 代表 SYN/ACK，如果这两个条件都满足，则说明该端口在接收端是打开的 (Open)，然后将该信息打印出来。

```
if received.haslayer(TCP) and str(received[TCP].flags) == "SA":
                print "Port " + str(sent[TCP].dport) + " of " + target + " is OPEN!"
```

*   同理，如果返回的包是 TCP 包，并且该 TCP 包的 flag 位为 RA（RA 表示 Reset+），则说明该端口在接收端已经被关闭 (closed)，将该信息打印出来。

```
elif received.haslayer(TCP) and str(received[TCP].flags) == "RA":
                print "Port " + str(sent[TCP].dport) + " of " + target + " is closed!"
```

*   如果返回的包是 ICMP 包，并且该 ICMP 包的类型为 3，则说明该端口被路由器或者防火墙过滤了 (filtered)，将该信息打印出来。

```
elif received.haslayer(ICMP) and str(received[ICMP].type) == "3":
                print "Port " + str(sent[TCP].dport) + " of " + target + " is filtered!"
```

*   最后，如果发送端没有收到任何回复 (no response)，我们同样可以判断该端口被路由器或者防火墙过滤了 (filtered)，将该信息打印出来。

```
for sent in unans:
        print str(sent[TCP].dport) + " is filtered!"
```

**执行代码看效果：**

*   执行代码前，我们的 S1 上面只开启了 SSH, 也就是端口 22，另外两个端口 80 和 123 是被关闭的，S1 上面没有开启 ACL 做包过滤，S1 和我们的 Scapy 主机之间也没有防火墙。

*   执行代码后，你会看到端口 22 是打开的，端口 80 和 123 是关闭的 (我们收到了 S1 发来的 TCP RST 包）
*   这时在 S1 上开启 http server，打开 80 端口

![](https://pic1.zhimg.com/v2-1fc76d2f3df220cb04fcf714a81367bc_b.jpg)

*   再次运行我们的 scapy 代码，可以发现现在端口 80 也打开了。

![](https://pic4.zhimg.com/v2-fe2ed5ae8235fc6665b230fea867e0eb_b.jpg)

*   在 S1 上配置 ACL，deny 掉 80 端口。

![](https://pic1.zhimg.com/v2-90242864f00bc0dcdc1d304b4502e574_b.jpg)

*   再次执行 scapy 代码，可以看到此时 “80 端口已经被过滤，路径上存在防火墙或者带有 ACL 的路由器！” 的信息。

![](https://pic4.zhimg.com/v2-8544692bc872a0a4b745dc8f18772e27_b.jpg)