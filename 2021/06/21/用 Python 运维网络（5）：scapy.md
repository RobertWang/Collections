> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/60853522)

本文参考了文章：

[弈心：网络工程师的 Python 之路 ---Scapy 基础篇​zhuanlan.zhihu.com![图标](https://pic1.zhimg.com/v2-81da1338241259e8962963ebd7d25b78_ipico.jpg)](https://zhuanlan.zhihu.com/p/51002301) [弈心：网络工程师的 Python 之路 ---Scapy 应用篇​zhuanlan.zhihu.com![图标](https://pic1.zhimg.com/v2-81da1338241259e8962963ebd7d25b78_ipico.jpg)](https://zhuanlan.zhihu.com/p/53264096)

**什么是 Scapy?**
--------------

**根据 scapy 官方的定义：**

> Scapy is a Python program that enables the user to send, sniff and dissect and forge network packets. This capability allows construction of tools that can probe, scan or attack networks.  
> In other words, Scapy is a powerful interactive packet manipulation program. It is able to forge or decode packets of a wide number of protocols, send them on the wire, capture them, match requests and replies, and much more. Scapy can easily handle most classical tasks like scanning, tracerouting, probing, unit tests, attacks or network discovery. It can replace hping, arpspoof, arp-sk, arping, p0f and even some parts of Nmap, tcpdump, and tshark).

大意就是：Scapy 是一个强大的，用 Python 编写的交互式数据包处理程序，它能让用户发送、嗅探、解析，以及伪造网络报文，从而用来侦测、扫描和向网络发动攻击。Scapy 可以轻松地处理扫描 (scanning)、路由跟踪(tracerouting)、探测(probing)、单元测试(unit tests)、攻击(attacks) 和发现网络 (network discorvery) 之类的传统任务。它可以代替`hping`,`arpspoof`,`arp-sk`,`arping`,`p0f` 甚至是部分的`Nmap`,`tcpdump`和`tshark` 的功能。

**Scapy 实验运行环境和拓扑：**

网络环境同前几篇：

几台交换机的管理 ip 地址为：

*   L3-1 192.168.56.10/24
*   L2-1 192.168.11.253/24
*   L2-2 192.168.12.253/24
*   L2-3 192.168.13.253/24
*   L2-4 192.168.14.253/24
*   L2-5 192.168.15.253/24

> 用户名: python  
> 密 码: 123

scapy 运行在 win7 或 ubuntu 上，安装很简单，网上有很多教程。

[https://www.cnblogs.com/qingkongwuyun/p/8508733.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/qingkongwuyun/p/8508733.html)

**测试环境：**
---------

**ubuntu 下：**

win7 下：

![](https://pic2.zhimg.com/v2-7b7d32091ff24e8832596773a5bb75f9_b.jpg)

用 ls() 函数来查看 scapy 支持的网络协议, （由于输出内容太长，只截取部分以供参考）。

可以看到网工们耳熟能详的 ARP, BOOTP, Dot1Q, DHCP, DNS, GRE, HSRP, ICMP, IP, NTP, RIP, SNMP, STP, PPPoE, TCP, TFTP, UDP 等等统统都支持。

![](https://pic1.zhimg.com/v2-58c16a9ffca2ce516b7cf5246ec5fac8_b.jpg)

除了 ls() 外，还可以用 lsc() 函数来查看 scapy 的指令集（函数）。比较常用的函数包括 arpcachepoison（用于 arp 毒化攻击，也叫 arp 欺骗攻击），arping（用于构造一个 ARP 的 who-has 包） ，send(用于发 3 层报文)，sendp（用于发 2 层报文）, sniff（用于网络嗅探，类似 Wireshark 和 tcpdump）, sr（发送 + 接收 3 层报文），srp（发送 + 接收 2 层报文）等等

![](https://pic1.zhimg.com/v2-37adf9fe937d09f01ab1ef70d4b17df8_b.jpg)

这里还可以用使用 ls() 的携带参数模式，比如 ls(IP) 来查看 IP 包的各种默认参数。

![](https://pic4.zhimg.com/v2-e097248a76145c68b020db7ca7c1eb23_b.jpg)![](https://pic1.zhimg.com/v2-0e89a62f438885f3ff3cac1d00a3560c_b.jpg)

**实验 1**
--------

**实验目的：使用 IP() 函数构造一个目的地址为 192.168.12.101（即拓扑中的 PC2）的 IP 报文，然后用 send() 函数将该 IP 报文发送给 PC2，在 PC2 上开启抓包软件以验证是否收到该报文。**

a. 首先用 IP() 函数构造一个目的地址为 192.168.12.101 的 IP 报文，将它实例化给 ip 这个变量。

```
ip = IP(dst='192.168.192.168.12.101')
```

![](https://pic3.zhimg.com/v2-c60ea565a4d9b82b3bf730e9d6592426_b.png)

b. 用 ls(ip) 查看该 IP 报文的内容，可以发现 src 已经变为 192.168.56.1（本机的 IP），dst 变为了 192.168.12.101。 一个最基本的 IP 报文就构造好了。

```
ls(ip)
```

![](https://pic1.zhimg.com/v2-faacbf07486d6d0b6b7e4c89784c40a4_b.jpg)

c. 构造好了 IP 报文 (src=192.168.56.1, dst=192.168.12.101) 后，我们就可以用 send()这个函数来把它发送给 PC2 了。

为了验证 PC2 确实接收到了我们发送的报文，在 PC2 上开启抓包功能。

![](https://pic1.zhimg.com/v2-671e7c8666db02e53e5a0900e411bf00_b.jpg)

在 scapy 上输入 send(ip,iface='VirtualBox Host-Only Network') 将该报文发出去，注意后面的 iface 参数用来指定端口，该参数可选。

![](https://pic1.zhimg.com/v2-c40eb898474434233a039f38d2f2af20_b.png)

抓包结果：

![](https://pic3.zhimg.com/v2-fef807a54841e5dfb79e6c3cdd52018a_b.jpg)

这时可以看到我们已经抓到了从 192.168.56.1 发来的 IP 报文，注意 Protocol: IPv6 Hop-by-Hop Option (0)，这是因为该包的 proto 位为 0, 不代表任何协议。

**实验 2**
--------

**实验目的：除了 send() 外，scapy 还有个 sendp() 函数，两者的区别是前者是发送三层报文，后者则是发送二层报文，实验 2 将演示如何用 sendp() 来构造二层报文。**

  
a. 用 sendp() 配合 Ether() 和 arp() 函数来构造一个 ARP 报文，命令如下

```
sendp(Ether(dst='ff:ff:ff:ff:ff:ff') / ARP(hwsrc = '01:23:45:67:89:01', psrc = '192.168.56.1', hwdst = 'ff:ff:ff:ff:ff:ff', pdst = '192.168.12.101') / 'abc', iface='VirtualBox Host-Only Network')
```

这里我们构造了一个源 MAC 地址为 01:23:45:67:89:01, 源 IP 地址为 192.168.56.1, 目标 MAC 地址为 ff:ff:ff:ff:ff:ff，目标 IP 地址为 192.168.12.101，payload 为 abc 的 ARP 报文。

b. 可以另开一个控制台，启用 sniff() 来抓包，并将抓包的内容实例化到 data 这个变量上。

```
>>> data = sniff()
```

使用 show() 命令监控本地的收发包情报

```
>>> data.show()
```

注意是 arp 包，所以不能跨网段，需要在 56 段监听才行：

![](https://pic2.zhimg.com/v2-126ea8b0ee70bc1d99982e55094fb605_b.jpg)![](https://pic3.zhimg.com/v2-aa9373cea66ede5f0ceecb2903781c1a_b.jpg)

可以看到该报文 ARP 部分的内容和 ARP 报文的结构完全一致

![](https://pic2.zhimg.com/v2-1e5d1226ba83f655187aa2660eb8c979_b.jpg)

**hardware type(HTPYE) 为 0x0001 的时候，表示 Ethernet**

**protocol type(PTPYE) 为 0x0800 的时候，表示 IPv4**

**hardware length (HLEN) 为 0x06 的时候，表示 MAC 地址长度为 6byte**

**protocol length(PLEN) 为 0x04 的时候，表示 IP 地址长度为 4byte**

**ARP 包有 request 和 response 之分，request 包的 OPER(Opcode) 位为 0x0001 （也就是这里的 who has）, response 包的 OPER 位为 0x0002。**

**最后的 payload 位 (padding) 即为我们自己定制的内容'abc'。**

**实验 3**
--------

**实验目的：从实验 1 和实验 2 的例子可以看出：send() 和 sendp() 函数只能发送报文，不能接收返回的报文。如果要想查看返回的 3 层报文，需要用到 sr() 函数，实验 3 将演示如何使用 sr() 函数。**

a. 用 sr() 向 PC2 发一个 ICMP 包，可以看到返回的结果是一个 tuple（元组），该元组里的元素是两个列表，其中一个列表叫 Results（响应），另一个叫 Unanswered（未响应）。

```
>>> sr(IP(dst = '192.168.43.1') / ICMP())
```

![](https://pic1.zhimg.com/v2-818648855c8124a4d4a2291bd56c42f8_b.png)

**这里可以看到 192.168.43.1 响应了这个 ICMP 包，所以在 Results 后面的 ICMP: 显示 1。**

b. 如果向一个不存在的 IP，比如 192.168.43.2 发 ICMP 包，那么这时会看到 scapy 在找不到该 IP 的 MAC 地址（因为目标 IP 192.168.43.2 和我们的主机 192.168.43.1 在同一个网段下，这里要触发 ARP 寻找目标 IP 对应的 MAC 地址）的时候，转用广播。当然广播也找不到目标 IP，这里可以 Ctrl+C 强行终止。

```
sr(IP(dst = '192.168.43.2') / ICMP())
```

![](https://pic4.zhimg.com/v2-740ded6c64785278dc126adb47254b97_b.png)

c. 我们可以将 sr() 函数返回的元组里的两个元素分别赋值给两个变量，第一个变量叫 ans，对应 Results（响应）这个元素，第二个变量叫 unans，对应 Unanswered（未响应）这个元素。

```
ans, unans = sr(IP(dst = '192.168.43.2') / ICMP())
```

![](https://pic3.zhimg.com/v2-4107ae13dc953bfbd3e6467effe4cb7a_b.jpg)

d. 这里还可以进一步用 show(), summary(), nsummary() 等方法来查看 ans 的内容，这里可以看到 192.168.43.114 向 192.168.43.1 发送了 echo-request 的 ICMP 包，192.168.43.1 向 192.168.43.114 回了一个 echo-reply 的 ICMP 包。

![](https://pic3.zhimg.com/v2-6f79b6bdca4e6aa1adccc1a254da3b4a_b.jpg)

e. 如果想要查看该 ICMP 包更多的信息，还可以用 ans[0]（ans 本身是个列表）来查看，因为这里我们只向 192.168.2.11 发送了一个 echo-request 包，所以用 [0] 来查看列表里的第一个元素。

```
ans[0]
```

![](https://pic1.zhimg.com/v2-040dfc3a2a82769923e9806d467692c8_b.png)

可以看到 ans[0] 本身又是一个包含了两个元素的元组，我们可以继续用 ans[0][0] 和 ans[0][1] 查看这两个元素。

```
ans[0][0]
ans[0][1]
```

![](https://pic1.zhimg.com/v2-44fa20bac749dc38da48dc78edf89b70_b.png)![](https://pic2.zhimg.com/v2-c1204f4975591b274332a2ee495f6201_b.jpg)

**实验 4**
--------

**实验目的：实验 3 讲到了 sr(), 它是用来接收返回的 3 层报文。实验 4 将使用 srp() 来接收返回的 2 层报文。**

a. 用 srp() 配合 Ether() 和 ARP() 构造一个 arp 报文，二层目的地址为 ff:ff:ff:ff:ff:ff，三层目的地址为 192.168.56.0/24, 因为我们是向整个 / 24 网络发送 arp, 耗时会很长，所以这里用 timeout = 5，表示将整个过程限制在 5 秒钟之内完成，最后的 iface 参数前面讲过就不解释了。

```
ans, unans = srp(Ether(dst = "ff:ff:ff:ff:ff:ff") / ARP(pdst = "192.168.56.0/24"), timeout = 5, iface = "VirtualBox Host-Only Network")
```

![](https://pic4.zhimg.com/v2-e709f8bf9a7f53d19818240f9d1dcb5b_b.png)![](https://pic3.zhimg.com/v2-9534290dd7b9a0c224e7c0263232e30a_b.jpg)

b. 实验环境 56 段有 5 台设备，从上图可以看到我们收到了 5 个 answers，符合我们的实验环境，下面用 ans.summary() 来具体看看到底是哪 5 个 IP 响应了我们的'who has'类型的 arp 报文。

```
ans.summary()
```

![](https://pic3.zhimg.com/v2-cf3b2e73867fbff4a6ea74f4fdd40ca2_b.jpg)

这里可以看到 192.168.56.1, 192.168.56.100, 192.168.56.101， 192.168.56.10， 192.168.56.253 响应了我们的'who has'类型的 arp 报文，并且能看到它们各自对应的 MAC 地址。

c. 用 unans.summary() 来查看那些没有回复'who has'类型 arp 报文的 IP 地址

```
unans.summary()
```

![](https://pic1.zhimg.com/v2-cedff1ec878366809ff0e3c8b3097194_b.jpg)

**实验 5**
--------

**实验目的：使用 tcp() 函数构造四层报文，理解和应用 RandShort()，RandNum() 和 Fuzz() 函数。**

a. 实验开始前，首先对外网络接口进行抓包。

![](https://pic4.zhimg.com/v2-388fbf8f156831500709f404e956ecbb_b.jpg)![](https://pic2.zhimg.com/v2-6edf0db9d3787d09e20769f75573cfb9_b.jpg)

b. 在 scapy 上使用 ip() 和 tcp() 函数来构造一个目的地 IP 为 [http://www.baidu.com](https://link.zhihu.com/?target=http%3A//www.baidu.com)，源端口为 30，目的端口为 80 的 TCP SYN 报文。

```
>>> ans, unans = sr(IP(dst = "www.baidu.com") / TCP(sport = 30, dport = 80, flags = "S"))
```

![](https://pic3.zhimg.com/v2-5bedfdab5a3999511600fbab7c7af00e_b.png)

c. TCP SYN 报文发送后，监听网卡的抓包软件上发现了发出的报文

![](https://pic2.zhimg.com/v2-2fc65c6d0256ea1f991499d575c60871_b.jpg)

d. 在 scapy 上输入 ans[0] 继续验证从主机发出的包，以及远端收到的包。

```
ans[0]
```

![](https://pic3.zhimg.com/v2-87a5f432ccd61b056cbdc3ea90106b36_b.png)

e. TCP 端口号除了手动指定外，还可以使用 RandShort(), RandNum() 和 Fuzz() 这几个函数来让 scapy 帮你自动生成一个随机的端口号，通常可以用作 sport(源端口号)。

首先来看 RandShort()，RandShort() 会在 1-65535 的范围内随机生成一个 TCP 端口号，将上面的 sport = 30 替换成 sport = RandShort() 即可使用。

```
>>> ans, unans = sr(IP(dst = "www.baidu.com") / TCP(sport = RandShort(), dport = 80, flags = "S"))
```

![](https://pic2.zhimg.com/v2-aee676f83bed63921b49528100d1afc1_b.png)

**这里可以看到 RandShort() 替我们随机生成了 13116 这个 TCP 源端口号**

f. 如果你想指定 scapy 生成端口号的范围，可以使用 RandNum()，比如你只想在 1000-1500 这个范围内生成端口号，可以使用 RandNum(1000,1500) 来指定，举例如下：

```
ans, unans = sr(IP(dst = "www.bing.com") / TCP(sport = RandNum(1000,1500), dport = 80, flags = "S"))
```

![](https://pic1.zhimg.com/v2-670551a893628739131690f2fca2acc8_b.jpg)

**这里 RandNum() 帮我们生成了 1246 这个源端口号**

**由于我们指定的范围是 1000-1500，很有可能和一些知名的端口号重复，这个时候会出现 sport 显示的不是端口号，而是具体的网络协议名字的情况。**

g. 最后来讲下 fuzz() 函数，前面的 RandShort() 和 RandNum() 都是写在 sport 后面的（当然也可以写在 dport 后面用来随机生成目的端口号），用 fuzz() 的话则可以省略 sport 这部分，fuzz() 会帮你检测到你漏写了 sport，然后帮你随机生成一个 sport 也就是源端口号。

使用 fuzz() 的命令如下：

```
ans, unans = sr(IP(dst = "www.163.com") / fuzz(TCP(dport = 80, flags = "S")))
```

![](https://pic1.zhimg.com/v2-a04975725d981243d55390319db0e650_b.jpg)

**这里看到 fuzz() 函数已经替我们随机生成了 7673 这个源端口号**

![](https://pic2.zhimg.com/v2-960ce5d1c0949fd9da6013fa4c95b235_b.jpg)

此后的内容为 Scapy 的实际应用，包括怎么使用 Scapy 进行 TCP 的 SYN 扫描、ACK 扫描、FIN 扫描、Xmas 扫描, Null 扫描，怎么使用 Scapy 执行 TCP SYN flooding 攻击，ARP 欺骗攻击，DHCP 饥饿攻击，怎么用 Scapy 探测 rogue DHCP 服务器等等。

**实验 6 -- TCP SYN 扫描**
----------------------

**实验目的：**使用 TCP SYN 扫描交换机 L2-1(192.168.11.101)的 53（DNS）， 80 （HTTP）， 23 （FTP）端口，知道如何判断端口是被关闭了 (closed) 还是被过滤了(filtered)，两者各自有什么特征。

**实验原理：**TCP 三次握手的原理和过程相信大家都知道。根据 RFC 793，当发送端的 TCP SYN 包发出后，大致会有下面四种情况发生:

1.  目的端口在接收端打开（也可以说接收端正在侦听该端口），收到 SYN 包的接收端回复 **SYN/ACK 包**给发送端，收到 SYN/ACK 包的发送端此时知道**目的端口是打开的 (open)。**
2.  目的端口在接收端被关闭，收到 SYN 包的接收端回复 **RST 包**给发送端，告知发送端**该目的端口已经被关闭了 (closed)。**
3.  如果发送端和接收端之间有防火墙或者使用 ACL 进行包过滤的路由器，那么 **SYN 包在到达接收端之前就被防火墙或路由器拦截下来，**此时防火墙或路由器会回复一个类型 3（Unreachable，不可达）的 ICMP 包（注意不再是 TCP 包）给发送端告知**该目的端口已经被过滤了 (filtered)。**
4.  如果 ICMP 在防火墙或路由器上被关闭了，这时 SYN 包会被防火墙、路由器 "静悄悄" 地丢弃，路由器和防火墙不会发送类型 3 的 ICMP 包告知发送端。此时发送端收不到任何回应 (no response)，这里我们同样可以判断**该目的端口已经被过滤了 (filtered)。**

知道实验原理后来看代码：

```
#!/usr/bin/env python
# _*_ coding: utf-8 _*_
"""
 @author: antenna
 @license: (C) Copyright 2019, Antenna.
 @contact: lilyef2000@gmail.com
 @software: 
 @file: synscan.py
 @time: 2019/3/30 3:11
 @desc:
"""
from scapy.all import *

target = 'www.cnblog.com'

ans, unans = sr(IP(dst = target) / TCP(sport = RandShort(), dport = [21, 80, 53], flags = "S"), timeout = 5)

for sent, received in ans:
        if received.haslayer(TCP) and str(received[TCP].flags) == "SA":
                print("Port " + str(sent[TCP].dport) + " of " + target + " is OPEN!")
        elif received.haslayer(TCP) and str(received[TCP].flags) == "RA":
                print("Port " + str(sent[TCP].dport) + " of " + target + " is closed!")
        elif received.haslayer(ICMP) and str(received[ICMP].type) == "3":
                print("Port " + str(sent[TCP].dport) + " of " + target + " is filtered!")

for sent in unans:
        print(str(sent[TCP].dport) + " is filtered!")
```

*   这里我们使用 sr() 函数对'www.cnblog.com'做端口 21，80，53 的 TCP SYN 扫描（注意 flags = "S"）, timeout 设为 5 秒。sr() 函数返回的是一个元组，该元组下面有两个元素，一个是 Results，一个是 Unanswered，我们用 ans 来表示 Results，也就是被响应的包，用 unans 来表示 Unanswered，表示没有被响应的包。

```
ans, unans = sr(IP(dst = target) / TCP(sport = RandShort(), dport = [21, 80, 123], flags = "S"), timeout = 5)
```

*   ans 和 unans 各自又含两个包，一个是发出去的包，一个是接收到的包，以 ans[0][0]和 ans[0][1]为例，第一个 [0] 表示抓到的第一个包，第二个 [0] 和[1]分别表示第一个包里发出的包和接收到的包。举例如下：

```
>>> target = 'www.cnblog.com'
>>> ans, unans = sr(IP(dst = target) / TCP(sport = RandShort(), dport = [21, 80, 53], flags = "S"),
 timeout = 5)
Begin emission:
Finished sending 3 packets.
.....***
Received 8 packets, got 3 answers, remaining 0 packets
>>> ans[0][0]  # （第一个包里发出的包）
<IP  frag=0 proto=tcp dst=150.129.42.39 |<TCP  sport=32179 dport=ftp flags=S |>>
>>> ans[0][1]  # （第一个包里接收到的包）
<IP  version=4 ihl=5 tos=0x0 len=72 id=6115 flags= frag=0 ttl=47 proto=icmp chksum=0xc70f src=150.12
9.42.39 dst=192.168.43.114 |<ICMP  type=dest-unreach code=host-prohibited chksum=0x1dc6 reserved=0 l
ength=0 nexthopmtu=0 |<IPerror  version=4 ihl=5 tos=0x0 len=44 id=1 flags= frag=0 ttl=46 proto=tcp c
hksum=0xe008 src=192.168.43.114 dst=150.129.42.39 |<TCPerror  sport=32179 dport=ftp seq=0 ack=0 data
ofs=6 reserved=0 flags=S window=8192 chksum=0xdd48 urgptr=0 options=[('MSS', 536)] |>>>>
>>>
```

*   正因如此，所以下面 for loop 里的 sent, received 分别代表的是 ans[0][0]、ans[0][1](抓到的第一个端口为 22 的包)，ans[1][0]、ans[1][1]（抓到的第二个端口为 80 的包）以及 ans[2][0]、ans[2][1]（抓到的第三个端口为 53 的包）里的内容

```
for sent, received in ans:
```

*   haslayer() 函数返回的是布尔值，用来判断**从接收端返回的包 (received)** 里所含协议的类型，这里用来判断该 received 包是否包含 TCP 协议，并且该包里 TCP 的 flag 位是否为 SA, SA 代表 SYN/ACK，如果这两个条件都满足，则说明该端口在接收端是打开的 (Open)，然后将该信息打印出来。

```
if received.haslayer(TCP) and str(received[TCP].flags) == "SA":
                print("Port " + str(sent[TCP].dport) + " of " + target + " is OPEN!")
```

*   同理，如果返回的包是 TCP 包，并且该 TCP 包的 flag 位为 RA（RA 表示 Reset+），则说明该端口在接收端已经被关闭 (closed)，将该信息打印出来。

```
elif received.haslayer(TCP) and str(received[TCP].flags) == "RA":
                print("Port " + str(sent[TCP].dport) + " of " + target + " is closed!")
```

*   如果返回的包是 ICMP 包，并且该 ICMP 包的类型为 3，则说明该端口被路由器或者防火墙过滤了 (filtered)，将该信息打印出来。

```
elif received.haslayer(ICMP) and str(received[ICMP].type) == "3":
                print("Port " + str(sent[TCP].dport) + " of " + target + " is filtered!")
```

*   最后，如果发送端没有收到任何回复 (no response)，我们同样可以判断该端口被路由器或者防火墙过滤了 (filtered)，将该信息打印出来。

```
for sent in unans:
        print(str(sent[TCP].dport) + " is filtered!")
```

**执行代码看效果：**

*   目标网站如何，我也不清楚，查查看：

```
antenna@pythonmanager:~$ sudo python3 synscan.py 
Begin emission:
.Finished sending 3 packets.
***
Received 4 packets, got 3 answers, remaining 0 packets
Port 21 of www.cnblog.com is OPEN!
Port 80 of www.cnblog.com is OPEN!
Port 53 of www.cnblog.com is OPEN!
```

*   执行代码后，会发现所有端口都是打开的

![](https://pic1.zhimg.com/v2-60c566a445745722ee2891abbbc9dc70_b.png)

可以尝试搭建环境，添加 acl 或防火墙来验证更多的效果。

实验 7 -- 使用 scapy 分析路由
---------------------

不说废话，直接上代码：

```
import os,sys,time,subprocess
import warnings,logging

warnings.filterwarnings("ignore", category=DeprecationWarning)
logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
from scapy.all import traceroute
domains = "113.108.238.121 180.96.12.11"  #  raw_input('Please input one or more IP/domain: ')
target =  domains.split(' ')
dport = [80]
if len(target) >= 1 and target[0]!='':
    res,unans = traceroute(target,dport=dport,retry=-2)
    res.graph(target="> test.svg", type="svg")
    time.sleep(1)
    subprocess.Popen(r"C:\ImageMagick\convert test.svg test.png", shell=True)
else:
    print("IP/domain number of errors,exit")
```

代码说明：

需要下载 ImageMagick 转换生成的 svg 格式的路由图到 png 格式。

代码执行结果：

```
Begin emission:
*********.*******.........*********Finished sending 60 packets.
*******.Begin emission:
Finished sending 28 packets.
Begin emission:
Finished sending 28 packets.

Received 43 packets, got 32 answers, remaining 26 packets
   113.108.238.121:tcp80 180.96.12.11:tcp80 
1  192.168.43.1    11    192.168.43.1    11 
4  114.247.23.185  11    114.247.23.185  11 
5  -                     221.219.202.213 11 
6  202.96.12.21    11    202.96.12.5     11 
7  -                     219.158.4.158   11 
8  219.158.9.38    11    -                  
9  202.97.17.153   11    202.97.88.253   11 
10 202.97.63.213   11    -                  
11 -                     202.102.69.14   11 
12 -                     180.96.35.46    11 
13 -                     180.96.65.162   11 
14 -                     180.96.65.157   11 
15 -                     180.96.65.10    11 
16 -                     180.96.65.9     11 
17 -                     180.96.65.10    11 
18 -                     180.96.65.9     11 
19 -                     180.96.65.10    11 
20 -                     180.96.65.161   11 
21 -                     180.96.65.10    11 
22 -                     180.96.65.157   11 
23 -                     180.96.65.162   11 
24 -                     180.96.65.141   11 
25 -                     180.96.65.162   11 
26 -                     180.96.65.9     11 
27 -                     180.96.65.10    11 
28 -                     180.96.65.157   11 
29 -                     180.96.65.10    11 
30 -                     180.96.65.161   11 

Process finished with exit code 0
```

![](https://pic1.zhimg.com/v2-216b610d469c1f79b3f803276cdcc484_b.jpg)

怎么早没发现 scapy 还有这功能，太牛了。