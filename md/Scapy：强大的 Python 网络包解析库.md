> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0MTY5MzEwMA==&mid=2247488828&idx=1&sn=abdbd2bb8d4c424a197d78d4e20ac1b6&chksm=fb2750ffcc50d9e9b070307a2be8fbdcbf72ce837dec809863bdf748fd127d47ccaaf76b13ec&scene=132#wechat_redirect)

随着 Python 越来越流行，在安全领域的用途也越来越多。比如可以用 requests 模块撰写进行 Web 请求工具；用 sockets 编写 TCP 网络通讯程序；解析和生成字节流可以使用 struct 模块。而要解析和处理网络包在网络安全领域更加普遍，时常我们会使用 tcpdump 和 wireshark（tshark）。但是如果要自己写程序进行处理，则需要更灵活的语言包（库），这就是本文要介绍的 Scapy。

**概述**
======

Scapy 是一个用于对底层网络数据包的 Python 模块和交互式程序，该程序对底层包处理进行了抽象打包，使得对网络数据包的处理非常简便。该类库可以在在网络安全领域有非常广泛用例，可用于漏洞利用开发、数据泄露、网络监听、入侵检测和流量的分析捕获的。Scapy 与数据可视化和报告生成集成，可以方便展示起结果和数据。

Scapy 的基本理念是提出一个基于领域特定语言，从而轻松快速地进行有线格式（Wire Format）管理。

**安装运行**
========

Scapy 可以通过 pip 安装:

```
pip install scapy
```

也可以通过发行版的包管理器安装，比如 yum，但是其版本可能太老已经过时。

也可以通过直接从官方仓库 clone 源码安装：

```
git clone github /secdev/scapy
```

然后，可以可以简单地运行：

```
cd scapy
./run_scapy
```

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLQeibDc8BsiajbWNe3E1SE3D6WNSicT4T6ia6elVpWa5nn7Dr3Zc9gone9g/640?wx_fmt=png)

**用法示例**
========

**解析 PCAP 抓包**
==============

用 Scapy 做的最简单的事情就是读取 PCAP 文件。让我们下载 Wireshark 的 sip-rtp-opus-hybrid.pcap 示例 PCAP 数据包为例子：

用 rdpcap() 函数引入 PCAP 文件，读取其内容的函数：

```
>>> pkts = rdpcap("sip-rtp-opus-hybrid.pcap")
>>> pkts
<sip-rtp-opus-hybrid.pcap: TCP:0 UDP:7 ICMP:0 Other:0>
```

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLePuhTslfBhCwAuN9WqPflgGviaj16EgUw1W8S0HhlukqhkviaKbmgxVA/640?wx_fmt=png)

为了更详细读取 PCAP 文件中的数据，可以使用 PcapReader 从打开的文件句柄中迭代地读取数据包，一次一个包，bing 实例化的对象：

```
>>> fd = open("sip-rtp-opus-hybrid.pcap", "rb")
>>> reader = PcapReader(fd)
>>> reader
<scapy.utils.PcapReader at 0x7f913c7c24e0>
>>> for CC in reader:
...: print(CC)
...:
```

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlL4SbnAmQZUUicWjNPG5vibhREyNLhJlC9HbeMChj11MPnGX4781A5NY1w/640?wx_fmt=png)

```
>>> fd.close()
```

如上面所示，每个数据包都以有线格式提供。Scapy 将每个数据包以网络层的堆栈。Scapy 层对象对应于网络协议及其格式。

获取第一个数据包并检查 IP 层是否可用：

```
>>> first= CC[0]
>>> first.haslayer(IP)
True
>>> IP in first
True
```

要解析来自特定层的数据包，可按想要的层对其进行索引，并让 Scapy 打印所有字段：

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlL9HwGL5dP8gMeKpqIqGpbc09DVu7VZEkSkk428yiciboldxgdCwo01Duw/640?wx_fmt=png)

要以十六进制打印数据包，可以使用 hexdump() 功能：

>>> hexdump(first)

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLu576mENCAiadtOEYuNYibqCUibeGZ0bPicYUzDmJZdLO9vUMbNF5Vsvrow/640?wx_fmt=png)

为了完全解析和完美地输出一个数据包，需要调用 show() 方法：

>>> first.show()

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLLfWicJib7843KiacJmByFXS0icOPffZMc0TtCLzYickryUHp9iaicibyyiatYbA/640?wx_fmt=png)

可以看到，上面未能有效地解析 SI 负载。这是因为 Scapy 主要处理二进制协议 网络堆栈的较低部分，而 SIP 不是。但是可以引入第三方模块来解析一些应用层协议，比如 HTTP 协议。

**实时抓包解析**
==========

比如可以读取带有预先捕获的数据包的 PCAP 文件，如果要做一些数据包嗅探，如果系统准备好在混杂模式下使用网络接口，可以调用 sniff() 从网卡获取一些数据包的函数：

```
>>> for CC in sniff(count=5):
...: CC.show()
...:
```

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLcBDrAmVsy3uSKl6Y6pY1BJDgww6WCTnU6RNmEVAKI5RbwibAv5kCalA/640?wx_fmt=png)

Scapy 中也可以使用和 Wireshark（tshark）、tcpdump 相同 BPF 语法来过滤嗅探到的数据包和许多其他工具支持：

```
>>> for CC in sniff(filter="udp", count=5):
...: CC.show()
...:
```

要将捕获的数据包保存到 PCAP 文件中以供进一步分析，可以使用 wrpcap() 函数来导出到文件：

```
>>> capture = sniff(filter="udp", count=5)
>>> capture
<Sniffed: TCP:0 UDP:5 ICMP:0 Other:0>
>>> wrpcap("udp.pcap", capture)
```

**发送 ping 包**
=============

除了可以嗅探（捕获和解析）网络数据包，但 Scapy 也支持生成数据包进行各种主动欺骗：网络扫描、服务器探测、通过发送攻击系统格式错误的请求等等。

下面尝试 ping 一个服务器，涉及到要给服务发送一个 ICMP 数据包：

```
>>> CC = IP(dst="XXX") / ICMP()
>>> CC.show()
```

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLFRvewafsHnRzMbemickaJYdibIPFLFQFE7KbFdiauC9cK7zOjPkog6rzw/640?wx_fmt=png)

然后调用 sr1() 函数，可以发送一个 ICMP 数据包（即 ping），等待返回数据包返回：

```
>>> rr=sr1(CC)
Begin emission:
Finished sending 1 packets.
...*
Received 4 packets, got 1 answers, remaining 0 packets
>>> rr
```

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLLRbw5jDlq101icfd2kWiaRdJBiaxwKibMfTvdNsXkj1FpT6YibRz1hNTibVA/640?wx_fmt=png)

上面得到了正确的 ICMP 回复。

为了发送多个数据包和接收响应（例如实现 ping 扫描），可以用 sr() 功能。发送多个数据包，但等待单个响应。还可以用 sr1_flood() 功能。

**网络协议层乱序**
===========

Scapy 通过重载了 Python / 运算符来实现层堆叠，不再不强制按照网络层顺序执行，以达到以预期人为顺序执行（这在某些测试和应用中很有用）。

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLJxBMTSibCG5yHGjaHywiaUwNmWkSOicoW0KNBhvb0ibNOgjeLG6zRuEg5g/640?wx_fmt=png)

```
>>> CC=ICMP() / UDP() / IP() / IP()
>>> CC
<ICMP |<UDP |<IP frag=0 proto=ipencap |<IP |>>>>
>>> CC.show()
###[ ICMP ]###
type= echo-request
code= 0
chksum= None
id= 0x0
seq= 0x0
###[ UDP ]###
sport= domain
dport= domain
len= None
chksum= None
###[ IP ]###
version= 4
ihl= None
tos= 0x0
len= None
id= 1
flags=
frag= 0
ttl= 64
proto= ipv4
chksum= None
src= 127.0.0.1
dst= 127.0.0.1
\options\
###[ IP ]###
version= 4
ihl= None
tos= 0x0
len= None
id= 1
flags=
frag= 0
ttl= 64
proto= hopopt
chksum= None
src= 127.0.0.1
dst= 127.0.0.1
\options\
>>> hexdump(CC)
WARNING: No IP underlayer to compute checksum. Leaving null.
0000 08 00 F7 65 00 00 00 00 00 35 00 35 00 30 00 00 ...e.....5.5.0..
0010 45 00 00 28 00 01 00 00 40 04 7C CF 7F 00 00 01 E..(....@.|.....
0020 7F 00 00 01 45 00 00 14 00 01 00 00 40 00 7C E7 ....E.......@.|.
0030 7F 00 00 01 7F 00 00 01
```

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLC8YXHHJwdR1icnTJkjQZ1D1AKxIs4s07p0zXAnFzMTX9RjkWCziaNEnw/640?wx_fmt=png)

设计成这样，主要是为了可以生成任意的网络数据包（故意损坏的），用来进行漏洞测试研究或利用。当然对于对这一块不熟悉的用户，**强烈建议不要轻易尝试**，以免造成问题。

**数据可视化**
=========

Scapy 也支持通过 PyX（需要预先安装模块）对数据进行可视化。可以输出为一个数据包或数据包列表的图形 (PostScript/PDF 格式)：

```
>>> xxCC[404].pdfdump(layer_shift=1)
>>> xxCC[404].psdump("/tmp/xxCC.eps",layer_shift=1)
```

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLGgDn9WpAIfUKQxObklnC1m8aNbFc1KlUfCJiadW5dTZvrjtovOee5bA/640?wx_fmt=png)

**模糊测试**
========

利用函数 fuzz() 可以利用快速构建生成随机测试值利用模糊模板并循环发送进行测试。以下示例中，IP 层正常，UDP 和 NTP 层被 fuzz。UDP 校验和将正确，UDP 目标端口将被 NTP 重载为 123，并且 NTP 版本将被强制为 4，所有其他端口将被随机化：

```
send(IP(dst="target")/fuzz(UDP()/NTP(version=4)),loop=1)
................^C
Sent 16 packets.
```

**总结**
======

抛砖引玉，我们在此介绍了一些基本的 Scapy 用途，当然这只是 scapy 庞大功能中的冰山一角，更多的用法请参考官方文档。

据虫虫所知目前有些工具已经使用了 scapy 包：

Trackerjacker: WiFi 网络映射器

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLbURVCoKDbX990yicicyHR6SiaH98bnRFT7gmQV4qplib6aCOvPhZjHKmFg/640?wx_fmt=png)

Wifiphisher: Wifi 接入点创建工具

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLmwRoSI5vhPibIXz5yNvW5qme6QgHotBSWEugG03Rz2HDicKqdibF4zibSQ/640?wx_fmt=png)

Sshame：SSH 公钥暴力破解器

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlLRJIQBxtQPLv2BnehOu4eJ8HG40Nkc7sU2I47H6icQsLLCwPibAp7TSvw/640?wx_fmt=png)

ISF：工业系统的利用框架。

![](https://mmbiz.qpic.cn/mmbiz_png/hMIkz6ibKC9s0dC5udzTDjyuYCzacSXlL1NIj3iauQmAGtyZwNSnUWbibkTl33Kr0zD7mt571sxLEIS6vvf8vdzMA/640?wx_fmt=png)

还有一些更特殊的用途则需要各位 hacker 来进一步发掘。