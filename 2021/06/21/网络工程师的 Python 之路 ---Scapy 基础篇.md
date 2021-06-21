> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/51002301)

**版权声明：我已加入 “维权骑士”(**[http://rightknights.com](https://link.zhihu.com/?target=http%3A//rightknights.com/)**)的版权保护计划，所有知乎专栏 “网路行者” 下的文章均为我本人（知乎 ID：弈心）原创，未经允许不得转载。**

**如果你喜欢我的文章，请关注我的知乎专栏 “网路行者”**[https://zhuanlan.zhihu.com/c_126268929](https://zhuanlan.zhihu.com/c_126268929)**, 里面有更多像本文一样深度讲解计算机网络技术的优质文章。**

**19 Dec 更新实验 4，实验 5**

笔者在《[网络工程师的 Python 之路 --- 初级篇](https://zhuanlan.zhihu.com/p/34932386)》中曾提到要写一篇关于 Scapy 的教程，当时还有热衷用 Python 玩爬虫的朋友把 Scapy 和 Scrapy 搞混了，这是两个风马牛不相及的东西，虽然它俩名字确实很像。相比爬虫工具 Scrapy，Scapy 更适合网络工程师学习和使用。

《Scapy 篇》我将由浅入深讲解 scapy 的基础应用以及如何用 scapy 编写黑客工具，包括如何使用 scapy 执行 SYN flooding 攻击、ARP 欺骗、DHCP 饥饿攻击、DHCP rogue server 攻击等等的脚本编写，本文将分为《scapy 基础篇》和《scapy 应用篇》，《应用篇》的连接如下：

[弈心：网络工程师的 Python 之路 ---Scapy 应用篇​zhuanlan.zhihu.com![图标](https://pic1.zhimg.com/v2-81da1338241259e8962963ebd7d25b78_ipico.jpg)](https://zhuanlan.zhihu.com/p/53264096)

**什么是 Scapy?**
--------------

**根据 scapy 官方的定义：**

> Scapy is a Python program that enables the user to send, sniff and dissect and forge network packets. This capability allows construction of tools that can probe, scan or attack networks.  
> In other words, Scapy is a powerful interactive packet manipulation program. It is able to forge or decode packets of a wide number of protocols, send them on the wire, capture them, match requests and replies, and much more. Scapy can easily handle most classical tasks like scanning, tracerouting, probing, unit tests, attacks or network discovery. It can replace hping, arpspoof, arp-sk, arping, p0f and even some parts of Nmap, tcpdump, and tshark).

大意就是：Scapy 是一个强大的，用 Python 编写的交互式数据包处理程序，它能让用户发送、嗅探、解析，以及伪造网络报文，从而用来侦测、扫描和向网络发动攻击。Scapy 可以轻松地处理扫描 (scanning)、路由跟踪(tracerouting)、探测(probing)、单元测试(unit tests)、攻击(attacks) 和发现网络 (network discorvery) 之类的传统任务。它可以代替`hping`,`arpspoof`,`arp-sk`,`arping`,`p0f` 甚至是部分的`Nmap`,`tcpdump`和`tshark` 的功能。

**Scapy 实验运行环境和拓扑：**
--------------------

**本篇的实验运行环境以及网络实验拓扑和《**网络工程师的 Python 之路 --- [初级篇](https://zhuanlan.zhihu.com/p/34932386)**》完全一样，这里简单回顾一下：**

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

1. 安装 Scapy

2. 进入 scapy, 如果你不是 root 账户，需要用 sudo scapy。

![](https://pic3.zhimg.com/v2-71433680da7c51f2b9ad3bdcd0c9b90e_b.jpg)

3. 进入 scapy 后，可以用 ls() 函数来查看 scapy 支持的网络协议, （由于输出内容太长，只截取部分以供参考）。

可以看到网工们耳熟能详的 ARP, BOOTP, Dot1Q, DHCP, DNS, GRE, HSRP, ICMP, IP, NTP, RIP, SNMP, STP, PPPoE, TCP, TFTP, UDP 等等统统都支持。

![](https://pic2.zhimg.com/v2-f2172b67c815d10e49075688a1c8211d_b.jpg)

4. 除了 ls() 外，还可以用 lsc() 函数来查看 scapy 的指令集（函数）。比较常用的函数包括 arpcachepoison（用于 arp 毒化攻击，也叫 arp 欺骗攻击），arping（用于构造一个 ARP 的 who-has 包） ，send(用于发 3 层报文)，sendp（用于发 2 层报文）, sniff（用于网络嗅探，类似 Wireshark 和 tcpdump）, sr（发送 + 接收 3 层报文），srp（发送 + 接收 2 层报文）等等

![](https://pic1.zhimg.com/v2-59d33c736a4b024c6e0ffbee8377c9c8_b.jpg)

5. 这里还可以用使用 ls() 的携带参数模式，比如 ls(IP) 来查看 IP 包的各种默认参数。

![](https://pic3.zhimg.com/v2-269028f6fda424a283c516da2c8880a6_b.jpg)

是不是让你回想起了 IP 报文的格式图？

![](https://pic3.zhimg.com/v2-572ee5e1fd3dc34025e7ec0c5e551cda_b.jpg)

**实验 1**
--------

**实验目的：使用 IP() 函数构造一个目的地址为 192.168.2.11（即拓扑中的交换机 S1）的 IP 报文，然后用 send() 函数将该 IP 报文发送给 S1，在 S1 上开启 debug ip packet 以验证是否收到该报文。**

a. 首先用 IP() 函数构造一个目的地址为 192.168.2.11 的 IP 报文，将它实例化给 ip 这个变量。

```
ip = IP(dst='192.168.2.11')
```

![](https://pic2.zhimg.com/v2-a9449d3b6e181378a389798c1f3cc399_b.jpg)

b. 用 ls(ip) 查看该 IP 报文的内容，可以发现 src 已经变为 192.168.2.1（本机的 IP），dst 变为了 192.168.2.11。 一个最基本的 IP 报文就构造好了。

```
ls(ip)
```

![](https://pic4.zhimg.com/v2-9e81a685e9d1ec66537a9d64e73b0a2b_b.jpg)

c. 构造好了 IP 报文 (src=192.168.2.1, dst=192.168.2.11) 后，我们就可以用 send()这个函数来把它发送出去了，发送给谁呢？当然是 192.168.2.11，也就是我们的 S1。

为了验证 S1 确实接收到了我们发送的报文，首先在 S1 上开启 debug ip packet.

![](https://pic2.zhimg.com/v2-6c8c1220bb65fd30c4df04925759c129_b.jpg)

然后在 scapy 上输入 send(ip, iface='ens33') 将该报文发出去，注意后面的 iface 参数用来指定端口，该参数可选。

```
send(ip,iface='ens33')
```

![](https://pic3.zhimg.com/v2-f68afd1b59ac36a357ae33ff931bb50a_b.jpg)

d. 再回到 S1 上，这时可以看到我们已经抓到了从 192.168.2.1 发来的 IP 报文，注意右下角的 unknown protocol，这是因为该包的 proto 位为 0, 不代表任何协议。

![](https://pic4.zhimg.com/v2-b586e888f41283f13a6602acfbc5390b_b.jpg)![](https://pic4.zhimg.com/v2-104deaba3cf24f72636b2ece55a1d17b_b.jpg)

**实验 2**
--------

**实验目的：除了 send() 外，scapy 还有个 sendp() 函数，两者的区别是前者是发送三层报文，后者则是发送二层报文，实验 2 将演示如何用 sendp() 来构造二层报文。**

  
a. 用 sendp() 配合 Ether() 和 arp() 函数来构造一个 ARP 报文，命令如下

```
sendp(Ether(dst='ff:ff:ff:ff:ff:ff') / ARP(hwsrc = '00:0c:29:72:b2:b5', psrc = '192.168.2.1', hwdst = 'ff:ff:ff:ff:ff:ff', pdst = '192.168.2.11') / 'abc', iface='ens33')
```

这里我们构造了一个源 MAC 地址为 00:0c:29:72:b2:b5, 源 IP 地址为 192.168.2.1, 目标 MAC 地址为 ff:ff:ff:ff:ff:ff，目标 IP 地址为 192.168.2.11，payload 为 abc 的 ARP 报文。

b. 另开一个 putty 客户端，再次进入 scapy，启用 sniff() 来抓包，并将抓包的内容实例化到 data 这个变量上。

```
data = sniff()
```

![](https://pic3.zhimg.com/v2-0d9de4a73990acc4bf9279315a57296e_b.jpg)

另外一边，在交换机 S1，也就是 192.168.2.11 上开启 debug arp，用来验证 S1 从 scapy (192.168.2.1) 收到了该 ARP 包。

![](https://pic2.zhimg.com/v2-364f91c83e0887db31c4ede774b14359_b.jpg)

c. 回到之前的 putty 窗口，用 sendp() 将下面的 ARP 报文发出去

![](https://pic1.zhimg.com/v2-45f3868a5775c1c7fe0ce597b9013b6c_b.jpg)

d. 回到正在抓包的 putty，ctrl+c 结束抓包，然后输入 data.show() 来查看抓到的包，这里可以看到我们刚才发的 ARP 包被抓到了，序列号为 0007。

![](https://pic3.zhimg.com/v2-1325b78d2e3ad827a32383796af8c926_b.jpg)

回到 S1，这里可以看到 S1 收到了从 192.168.2.1 发来的 ARP 报文。

![](https://pic3.zhimg.com/v2-ca9e28811d96b49947f444aac9737572_b.jpg)

e. 因为该 ARP 包的序列号为 0007, 继续用 data[7] 和 data[7].show() 深挖该 arp 报文的内容

![](https://pic3.zhimg.com/v2-12681d6c460c0c97aec22e45810fb80a_b.jpg)

可以看到该报文 ARP 部分的内容和 ARP 报文的结构完全一致

![](https://pic1.zhimg.com/v2-406d7469c3606736a55cff9953742954_b.jpg)

**hardware type(HTPYE) 为 0x0001 的时候，表示 Ethernet**

**protocol type(PTPYE) 为 0x0800 的时候，表示 IPv4**

**hardware length (HLEN) 为 0x06 的时候，表示 MAC 地址长度为 6byte**

**protocol length(PLEN) 为 0x04 的时候，表示 IP 地址长度为 4byte**

**ARP 包有 request 和 response 之分，request 包的 OPER(Opcode) 位为 0x0001 （也就是这里的 who has）, response 包的 OPER 位为 0x0002。**

**最后的 payload 位 (padding) 即为我们自己定制的内容'abc'。**

**实验 3**
--------

**实验目的：从实验 1 和实验 2 的例子可以看出：send() 和 sendp() 函数只能发送报文，而不能接收返回的报文。如果要想查看返回的 3 层报文，需要用到 sr() 函数，实验 3 将演示如何使用 sr() 函数。**

a. 用 sr() 向 S1 发一个 ICMP 包，可以看到返回的结果是一个 tuple（元组），该元组里的元素是两个列表，其中一个列表叫 Results（响应），另一个叫 Unanswered（未响应）。

```
sr(IP(dst = '192.168.2.11') / ICMP())
```

![](https://pic3.zhimg.com/v2-43570445b820a6d6335d206898da7406_b.jpg)

**这里可以看到 192.168.2.11 响应了这个 ICMP 包，所以在 Results 后面的 ICMP: 显示 1。**

b. 如果向一个不存在的 IP，比如 192.168.2.2 发 ICMP 包，那么这时会看到 scapy 在找不到该 IP 的 MAC 地址（因为目标 IP 192.168.2.2 和我们的主机 192.168.2.1 在同一个网段下，这里要触发 ARP 寻找目标 IP 对应的 MAC 地址）的时候，转用广播。当然广播也找不到目标 IP，这里可以 Ctrl+C 强行终止。

```
sr(IP(dst = '192.168.2.2') / ICMP())
```

![](https://pic1.zhimg.com/v2-157bff7fc5047f040cd7cf14c94ec2fc_b.jpg)

**由于没有响应，所以你能看到 Unanswered 后面的 ICMP: 显示了 1.**

c. 我们可以将 sr() 函数返回的元组里的两个元素分别赋值给两个变量，第一个变量叫 ans，对应 Results（响应）这个元素，第二个变量叫 unans，对应 Unanswered（未响应）这个元素。

```
ans, unans = sr(IP(dst = '192.168.2.11') / ICMP())
```

![](https://pic3.zhimg.com/v2-e377ce1681d8b1107f7924d751b6caf6_b.jpg)

d. 这里还可以进一步用 show(), summary(), nsummary() 等方法来查看 ans 的内容，这里可以看到 192.168.2.1 向 192.168.2.11 发送了 echo-request 的 ICMP 包，192.168.2.11 向 192.168.2.1 回了一个 echo-reply 的 ICMP 包。

![](https://pic2.zhimg.com/v2-807276306b6e01a63a4abe43aa6a9461_b.jpg)

e. 如果想要查看该 ICMP 包更多的信息，还可以用 ans[0]（ans 本身是个列表）来查看，因为这里我们只向 192.168.2.11 发送了一个 echo-request 包，所以用 [0] 来查看列表里的第一个元素。

```
ans[0]
```

![](https://pic2.zhimg.com/v2-74859815a11b5bfb2f9dbb63aea684f9_b.jpg)

可以看到 ans[0] 本身又是一个包含了两个元素的元组，我们可以继续用 ans[0][0] 和 ans[0][1] 查看这两个元素。

```
ans[0][0]
ans[0][1]
```

![](https://pic3.zhimg.com/v2-28b57652a3593908569b38bf9cee8396_b.jpg)

**实验 4**
--------

**实验目的：实验 3 讲到了 sr(), 它是用来接收返回的 3 层报文。实验 4 将使用 srp() 来接收返回的 2 层报文。**

a. 用 srp() 配合 Ether() 和 ARP() 构造一个 arp 报文，二层目的地址为 ff:ff:ff:ff:ff:ff，三层目的地址为 192.168.2.0/24, 因为我们是向整个 / 24 网络发送 arp, 耗时会很长，所以这里用 timeout = 5，表示将整个过程限制在 5 秒钟之内完成，最后的 iface 参数前面讲过就不解释了。

```
ans, unans = srp(Ether(dst = "ff:ff:ff:ff:ff:ff") / ARP(pdst = "192.168.2.0/24"), timeout = 5, iface = "ens33")
```

![](https://pic2.zhimg.com/v2-9f17903879397020f15f6a7ffd52acad_b.jpg)

b. 我们的实验环境里有 5 台交换机，S1 到 S5 的管理 IP 都在 192.168.2.0/24 这个范围，从上图可以看到我们收到了 5 个 answers，符合我们的实验环境，下面用 ans.summary() 来具体看看到底是哪 5 个 IP 响应了我们的'who has'类型的 arp 报文。

```
ans.summary()
```

![](https://pic3.zhimg.com/v2-562e3e246ad1eeeb0217dc3f4ffd6d42_b.jpg)

这里可以看到 192.168.2.11 （S1）, 192.168.2.12（S2）, 192.168.2.13 （S3）， 192.168.2.14 （S4）， 192.168.2.15 （S5）响应了我们的'who has'类型的 arp 报文，并且能看到它们各自对应的 MAC 地址。

c. 用 unans.summary() 来查看那些没有给予我们'who has'类型 arp 报文回复的 IP 地址

```
unans.summary()
```

![](https://pic3.zhimg.com/v2-a07fb188aac3eab5271409443ca19b3a_b.jpg)

可以看到询问其他 IP 的'who has'类型 arp 报文没有人响应。

**实验 5**
--------

**实验目的：使用 tcp() 函数构造四层报文，理解和应用 RandShort()，RandNum() 和 Fuzz() 函数。**

a. 实验开始前，首先在 S1 上启用 HTTP 服务，打开 TCP 80 端口，并开启 debug ip tcp packet。

![](https://pic2.zhimg.com/v2-4d593609b69b69daa2e5a382027b42b1_b.jpg)

b. 在 scapy 上使用 ip() 和 tcp() 函数来构造一个目的地 IP 为 192.168.2.11（即 S1），源端口为 30，目的端口为 80 的 TCP SYN 报文。

```
ans, unans = sr(IP(dst = "192.168.2.11") / TCP(sport = 30, dport = 80, flags = "S"))
```

![](https://pic1.zhimg.com/v2-182e4ef34fda2709a6460d7b88f1e014_b.jpg)

c. TCP SYN 报文发送后，回到 S1 上，可以看到 S1 已经收到了该报文，而且 S1 向 scapy 主机回复了一个 ACK 报文。

![](https://pic4.zhimg.com/v2-5f81f6ec32246148368bd37aa38bf8c3_b.jpg)

d. 在 scapy 上输入 ans[0] 继续验证从主机发出的包，以及从 S1 收到的包。

```
ans[0]
```

![](https://pic2.zhimg.com/v2-1462984654aa03fbe78930512b70b7e9_b.jpg)

e. TCP 端口号除了手动指定外，还可以使用 RandShort(), RandNum() 和 Fuzz() 这几个函数来让 scapy 帮你自动生成一个随机的端口号，通常可以用作 sport(源端口号)。

首先来看 RandShort()，RandShort() 会在 1-65535 的范围内随机生成一个 TCP 端口号，将上面的 sport = 30 替换成 sport = RandShort() 即可使用。

```
ans, unans = sr(IP(dst = "192.168.2.11") / TCP(sport = RandShort(), dport = 80, flags = "S"))
```

![](https://pic4.zhimg.com/v2-5d7992051475ff584ade8ebd549b0143_b.jpg)

**这里可以看到 RandShort() 替我们随机生成了 13904 这个 TCP 源端口号**

f. 如果你想指定 scapy 生成端口号的范围，可以使用 RandNum()，比如你只想在 1000-1500 这个范围内生成端口号，可以使用 RandNum(1000,1500) 来指定，举例如下：

```
ans, unans = sr(IP(dst = "192.168.2.11") / TCP(sport = RandNum(1000,1500), dport = 80, flags = "S"))
```

![](https://pic3.zhimg.com/v2-d47d833bccbe78435435c59d05e6d1ea_b.jpg)

**这里 RandNum() 帮我们生成了 1008 这个源端口号**

**由于我们指定的范围是 1000-1500，很有可能和一些知名的端口号重复，这个时候会出现 sport 显示的不是端口号，而是具体的网络协议名字的情况，比如这里重复上面的命令再次构造一个 TCP 包：**

![](https://pic4.zhimg.com/v2-cb7666777d0e370c7b4ef6a283185c47_b.jpg)

**这时 sport = blueberry_lm, 不再是具体的端口号**。在 google 查询一下，blueberry_lm 对应的 TCP 端口号为 1432，说明 RandNum() 帮我们随机生成了 1432 这个源端口号。

g. 最后来讲下 fuzz() 函数，前面的 RandShort() 和 RandNum() 都是写在 sport 后面的（当然也可以写在 dport 后面用来随机生成目的端口号），用 fuzz() 的话则可以省略 sport 这部分，fuzz() 会帮你检测到你漏写了 sport，然后帮你随机生成一个 sport 也就是源端口号。

使用 fuzz() 的命令如下：

```
ans, unans = sr(IP(dst = "192.168.2.11") / fuzz(TCP(dport = 80, flags = "S")))
```

![](https://pic4.zhimg.com/v2-dbbf7528c9b1bd8303c420f21b7a5f8b_b.jpg)

**这里看到 fuzz() 函数已经替我们随机生成了 39246 这个源端口号**

未完待续