> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/austin1000/article/details/100042405)

### 文章目录

*   [1 scapy 简介](#1_scapy_1)
*   [2 安装和运行 scapy](#2_scapy_3)
*   [3 查看当前网络配置](#3__18)
*   *   [3.1 conf 概览](#31_conf_21)
    *   [3.2 查看网卡及路由](#32__148)
    *   *   [3.2.1 查看网卡](#321__149)
        *   [3.2.2 查看路由](#322__228)
        *   [3.3.3 查看默认网卡](#333__288)

1 scapy 简介
==========

scapy 是一个 [python](https://so.csdn.net/so/search?from=pc_blog_highlight&q=python) 语言写的，用来操作 TCP/IP 数据包的库，基本涵盖了 wireshark 的主要功能，例如抓包、ping、traceroute、嗅探、扫描，但由于其可以按照自己的意愿来拼接和 “无中生成”TCP/IP 数据包中的内容，因此还可以实现 attack 的部分功能，并可以移植到任意平台运行。

2 安装和运行 scapy
=============

scapy 官网上有安装教程，不再赘述。建议在 venv 的虚拟环境下安装 Scapy 的 basic 包，不影响主 python 环境。本文是在 windows10+python3.7 环境下，安装的 scapy2.4.3 basic 包。另在 windows 下使用 scapy 需要安装 npcap 软件。

运行 venv\Scripts 下运行 activate 进入虚拟环境，再运行 scapy。  
图中 INFO 错误是 scapy 的附加功能，需要依赖一些三方包，不安装也不影响 scapy 核心功能的使用。  
![](https://img-blog.csdnimg.cn/20190823170852258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2F1c3RpbjEwMDA=,size_16,color_FFFFFF,t_70)  
scapy 的默认主题太暗，建议改成亮色，conf.color_theme=BrightTheme()

```
 conf.color_theme=BrightTheme()

```

默认主题效果：  
![](https://img-blog.csdnimg.cn/20190823171246125.png)  
BrightTheme 主题效果：  
![](https://img-blog.csdnimg.cn/20190823171302446.png)

3 查看当前网络配置
==========

venv/Lib/site-packages/scapy/config.py 下有一个 Conf 类，主要存储了 scapy 最主要的一些配置，比如 scapy 版本、主题颜色、网卡、路由、是否使用 npcap、可以在 scapy 交互环境中使用哪些命令等。在 scapy 交互环境中可以直接输入 conf 来查看相应的内容。

3.1 conf 概览
-----------

<table><thead><tr><th>命令</th><th>作用</th></tr></thead><tbody><tr><td>conf</td><td>显示 conf 所有配置</td></tr><tr><td>conf.iface</td><td>主网卡</td></tr><tr><td>conf.route</td><td>获取主路由</td></tr><tr><td>conf.commands</td><td>可在交互环境中使用的命令集合</td></tr><tr><td>lsc()</td><td>同 conf.commands</td></tr></tbody></table>

详解：

conf 本质是 Conf 类的实例：

```
>>> type(conf)
scapy.config.Conf

```

Conf 类中包含有大量配置，部分配置如下：

```
>>> conf
ASN1_default_codec = <ASN1Codec BER[1]>
AS_resolver = <scapy.as_resolvers.AS_resolver_multi object at 0x0000029E42AC...
BTsocket   = <BluetoothRFCommSocket: read/write packets on a connected L2CAP...
L2listen   = <L2pcapListenSocket: read packets at layer 2 using libpcap>
L2socket   = <L2pcapSocket: read/write packets at layer 2 using only libpcap>
L3socket   = <L3pcapSocket: read/write packets at layer 3 using only libpcap>
L3socket6  = functools.partial(<L3pcapSocket: read/write packets at layer 3 ...
USBsocket  = None
auto_crop_tables = True
auto_fragment = True
bufsize    = 65536

```

当前主网卡 conf.iface

```
>>> conf.iface
<NetworkInterface [Intel(R) Ethernet Connection (5) I219-V #2] {XXXX}>

```

当前路由表 conf.route（部分）

```
>>> conf.route
Network          Netmask          Gateway         Iface                                        Output IP        Metric
0.0.0.0          0.0.0.0          25.255.255.254  ZeroTier One Virtual Port                    172.28.33.102    10034
0.0.0.0          0.0.0.0          10.11.91.254    Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     25
10.11.91.0       255.255.255.0    0.0.0.0         Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     281
10.11.91.161     255.255.255.255  0.0.0.0         Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     281
10.11.91.255     255.255.255.255  0.0.0.0         Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     281
127.0.0.0        255.0.0.0        0.0.0.0         Npcap Loopback Adapter                       127.0.0.1        281
127.0.0.1        255.255.255.255  0.0.0.0         Npcap Loopback Adapter                       127.0.0.1        281

```

可在交互环境中使用的命令 conf.commands（或者输入 lsc())

```
>>> conf.commands
IPID_count          : Identify IP id values classes in a list of packets
arpcachepoison      : Poison target's cache with (your MAC,victim's IP) couple
arping              : Send ARP who-has requests to determine which hosts are up
arpleak             : Exploit ARP leak flaws, like NetBSD-SA2017-002.
bind_layers         : Bind 2 layers on some specific fields' values.
bridge_and_sniff    : Forward traffic between interfaces if1 and if2, sniff and return
chexdump            : Build a per byte hexadecimal representation
computeNIGroupAddr  : Compute the NI group Address. Can take a FQDN as input parameter
corrupt_bits        : Flip a given percentage or number of bits from a string
corrupt_bytes       : Corrupt a given percentage or number of bytes from a string
defrag              : defrag(plist) -> ([not fragmented], [defragmented],
defragment          : defragment(plist) -> plist defragmented as much as possible
dhcp_request        : Send a DHCP discover request and return the answer
dyndns_add          : Send a DNS add message to a nameserver for "name" to have a new "rdata"
dyndns_del          : Send a DNS delete message to a nameserver for "name"
etherleak           : Exploit Etherleak flaw
explore             : Function used to discover the Scapy layers and protocols.
fletcher16_checkbytes: Calculates the Fletcher-16 checkbytes returned as 2 byte binary-string.
fletcher16_checksum : Calculates Fletcher-16 checksum of the given buffer.
fragleak            : --
fragleak2           : --
fragment            : Fragment a big IP datagram
fuzz                :
getmacbyip          : Return MAC address corresponding to a given IP address
getmacbyip6         : Returns the MAC address corresponding to an IPv6 address
hexdiff             : Show differences between 2 binary strings
hexdump             : Build a tcpdump like hexadecimal view
hexedit             : Run hexedit on a list of packets, then return the edited packets.
hexstr              : Build a fancy tcpdump like hex from bytes.
import_hexcap       : Imports a tcpdump like hexadecimal view
is_promisc          : Try to guess if target is in Promisc mode. The target is provided by its ip.
linehexdump         : Build an equivalent view of hexdump() on a single line
ls                  : List  available layers, or infos on a given layer class or name.
neighsol            : Sends and receive an ICMPv6 Neighbor Solicitation message
overlap_frag        : Build overlapping fragments to bypass NIPS
promiscping         : Send ARP who-has requests to determine which hosts are in promiscuous mode
rdpcap              : Read a pcap or pcapng file and return a packet list
report_ports        : portscan a target and output a LaTeX table
restart             : Restarts scapy
send                : Send packets at layer 3
sendp               : Send packets at layer 2
sendpfast           : Send packets at layer 2 using tcpreplay for performance
sniff               :
split_layers        : Split 2 layers previously bound.
sr                  : Send and receive packets at layer 3
sr1                 : Send packets at layer 3 and return only the first answer
sr1flood            : Flood and receive packets at layer 3 and return only the first answer
srbt                : send and receive using a bluetooth socket
srbt1               : send and receive 1 packet using a bluetooth socket
srflood             : Flood and receive packets at layer 3
srloop              : Send a packet at layer 3 in loop and print the answer each time
srp                 : Send and receive packets at layer 2
srp1                : Send and receive packets at layer 2 and return only the first answer
srp1flood           : Flood and receive packets at layer 2 and return only the first answer
srpflood            : Flood and receive packets at layer 2
srploop             : Send a packet at layer 2 in loop and print the answer each time
tcpdump             : Run tcpdump or tshark on a list of packets.
tdecode             :
traceroute          : Instant TCP traceroute
traceroute6         : Instant TCP traceroute using IPv6
traceroute_map      : Util function to call traceroute on multiple targets, then
tshark              : Sniff packets and print them calling pkt.summary().
wireshark           :
wrpcap              : Write a list of packets to a pcap file

```

3.2 查看网卡及路由
-----------

### 3.2.1 查看网卡

<table><thead><tr><th>命令</th><th>作用</th></tr></thead><tbody><tr><td>get_windows_if_list()</td><td>获取所有网卡</td></tr><tr><td>IFACES / ifaces</td><td>get_windows_if_list() 的全局变量</td></tr><tr><td>IFACES.reload() / ifaces.reload()</td><td>网卡发生变化时，刷新 IFACES</td></tr></tbody></table>

详解：  
venv/Lib/site-packages/scapy/arch/windows/__init__.py 文件 get_windows_if_list() 用来获取网卡列表：

```
>>> get_windows_if_list()
[{'name': '有线网',
  'win_index': 27,
  'description': 'Intel(R) Ethernet Connection (5) I219-V #2',
  'guid': '{XX}',
  'mac': 'xx:xx:xx:xx:xx:xx',
  'ipv4_metric': 25,
  'ipv6_metric': 25,
  'ips': ['fe80::XXXX:XXXX:XXXX:XXXX', '10.11.91.161']},
 {'name': 'Npcap Loopback Adapter',
  'win_index': 22,
  'description': 'Npcap Loopback Adapter',
  'guid': '{XX}',
  'mac': 'xx:xx:xx:xx:xx:xx',
  'ipv4_metric': 25,
  'ipv6_metric': 25,
  'ips': ['fe80::XXXX:XXXX:XXXX:XXXX', '169.254.140.26']},
 {'name': 'Wifi',
  'win_index': 31,
  'description': 'Intel(R) Dual Band Wireless-AC 8265 #2',
  'guid': '{XX}',
  'mac': 'xx:xx:xx:xx:xx:xx',
  'ipv4_metric': 25,
  'ipv6_metric': 25,
  'ips': ['fe80::XXXX:XXXX:XXXX:XXXX', '169.254.62.191']}]

```

NetworkInterfaceDict 中，用 NetworkInterface 将 get_windows_if_list() 进行了封装，并在 windows/__init__.py 中进行了如下初始化，因此可使用 IFACES 或者 ifaces 查看网卡列表：

```
IFACES = ifaces = NetworkInterfaceDict()  #NetworkInterfaceDict的无参构造函数不包含任何有用信息
IFACES.load() #这里是真正加载本地网卡的，因此如果网卡列表发生了变化，需要手工重新调用下ifaces.reload()

```

```
>>> ifaces
INDEX  IFACE                                                 IP               MAC
27     Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     XXX
22     Npcap Loopback Adapter                       127.0.0.1        00:00:00:00:00:00
18     SVN Adapter V1.0                             169.254.112.118  XXX
29     Microsoft Wi-Fi Direct Virtual Adapter #3    169.254.173.211  XXX
6      Microsoft Wi-Fi Direct Virtual Adapter #4    169.254.192.230  XXX
21     Bluetooth Device (Personal Area Network) #2  169.254.227.190  XXX
17     TAP-Windows Adapter V9                       169.254.26.68    XXX
31     Intel(R) Dual Band Wireless-AC 8265 #2       169.254.62.191   XXX
14     ZeroTier One Virtual Port                    172.28.33.102    XXX
10     VMware Virtual Ethernet Adapter for VMnet8   192.168.15.1     XXX
23     VMware Virtual Ethernet Adapter for VMnet1   192.168.220.1    XXX
-2     [Unknown] NdisWan Adapter                    None             ff:ff:ff:ff:ff:ff
-3     [Unknown] NdisWan Adapter                    None             ff:ff:ff:ff:ff:ff
-1     [Unknown] NdisWan Adapter                    None             ff:ff:ff:ff:ff:ff

```

如果在使用过程中网卡列表发生了变化，需要手动调用 ifaces.reload()

```
class NetworkInterfaceDict(UserDict):
        def reload(self):
        """Reload interface list"""
        self.restarted_adapter = False
        self.data.clear()
        if conf.use_pcap:
            # Reload from Winpcapy
            from scapy.arch.pcapdnet import load_winpcapy
            load_winpcapy()
        self.load()  # reload函数实际上最后也是通过调用load重新加载网卡列表
        # Reload conf.iface
        conf.iface = get_working_if()  #reload函数会同时刷新默认网卡，这里目前有点问题，详见3.2.3节

```

### 3.2.2 查看路由

<table><thead><tr><th>命令</th><th>作用</th></tr></thead><tbody><tr><td>read_routes()</td><td>查看 ipv4 路由</td></tr><tr><td>Route()</td><td>对 read_routes() 的封装</td></tr><tr><td>conf.route</td><td>Route 的全局对象</td></tr><tr><td>conf.route.route(dst=“www.baidu.com”)</td><td>获取去百度的路由，如果 dst=None 的话返回默认路由</td></tr><tr><td>conf.route.resync()</td><td>如果网络发生了变化，用来刷新 conf.route</td></tr></tbody></table>

venv/Lib/site-packages/scapy/arch/windows/__init__.py 中，read_routes() 用来获取 ipv4 路由

```
>>> read_routes()
[(0,  //dest 以十进制显示
  0,  //netmask 以十进制显示
  '10.11.91.254',   //nexthop
  <NetworkInterface [Intel(R) Ethernet Connection (5) I219-V #2] {XXX}>,  //iface
  '10.11.91.161',  //ip
  25), //metric
 (0,
  0,
  '25.255.255.254',
  <NetworkInterface [ZeroTier One Virtual Port] {XXX}>,
  '172.28.33.102',
  10034),
 (168516352,
  4294967040,
  '0.0.0.0',
  <NetworkInterface [Intel(R) Ethernet Connection (5) I219-V #2] {XXX}>,
  '10.11.91.161',
  281)]

```

在 venv/Lib/site-packages/scapy/route.py 文件中对 ipv4 的 route 命令进行了封装成了 Route 类

```
class Route:
    def __init__(self):
        self.resync()

    def resync(self):
        from scapy.arch import read_routes
        self.invalidate_cache()
        self.routes = read_routes()

```

```
>>> Route()
Network          Netmask          Gateway         Iface                                        Output IP        Metric
0.0.0.0          0.0.0.0          10.11.91.254    Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     25
0.0.0.0          0.0.0.0          25.255.255.254  ZeroTier One Virtual Port                    172.28.33.102    10034
10.11.91.0       255.255.255.0    0.0.0.0         Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     281
10.11.91.161     255.255.255.255  0.0.0.0         Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     281

```

conf.route 初始化时 = Route()，如果网络发生变化，需要手动调用 conf.route.resync() 刷新路由

```
conf.route = Route()

```

### 3.3.3 查看默认网卡

<table><thead><tr><th>命令</th><th>作用</th></tr></thead><tbody><tr><td>conf.route.resync()</td><td>刷新路由</td></tr><tr><td>conf.iface</td><td>conf.iface = conf.route.route(‘0.0.0.0’)[0] ，默认路由对应的网卡</td></tr></tbody></table>

详解：  
conf.iface 被初始化为 conf.iface = iface = conf.route.route(None, verbose=0)[0]，之后如果网卡发生了变化，需要手动指定

```
conf.route.resync()  #这一步不可少，必须先刷新conf.route
conf.route.route(None, verbose=0)[0] # 从conf.route中获取默认路由对应的接口

```

或者使用 ifaces 的 dev_from_index(INDEX) 方法手动指定：

```
>>> ifaces
INDEX  IFACE                                                 IP               MAC
27     Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     XXX 
22     Npcap Loopback Adapter                       127.0.0.1        00:00:00:00:00:00
18     SVN Adapter V1.0                             169.254.112.118  XXX
29     Microsoft Wi-Fi Direct Virtual Adapter #3    169.254.173.211  XXX
6      Microsoft Wi-Fi Direct Virtual Adapter #4    169.254.192.230  XXX
21     Bluetooth Device (Personal Area Network) #2  169.254.227.190  XXX
17     TAP-Windows Adapter V9                       169.254.26.68    XXX
31     Intel(R) Dual Band Wireless-AC 8265 #2       169.254.62.191   XXX
14     ZeroTier One Virtual Port                    172.28.33.102    XXX
10     VMware Virtual Ethernet Adapter for VMnet8   192.168.15.1     XXX
23     VMware Virtual Ethernet Adapter for VMnet1   192.168.220.1    XXX
-2     [Unknown] NdisWan Adapter                    None             ff:ff:ff:ff:ff:ff
-3     [Unknown] NdisWan Adapter                    None             ff:ff:ff:ff:ff:ff
-1     [Unknown] NdisWan Adapter                    None             ff:ff:ff:ff:ff:ff
>>> conf.iface=ifaces.dev_from_index(27)  

```

PS：  
scapy 有一个函数`get_working_if()`也可以返回网卡，大多数情况下是正常的，但在有多个 route 的 mask 为 0.0.0.0 时有可能返回错误的结果，原因是该函数调用的路由表中 netmask 最小的网卡，如下获取到 Zerotier 虚拟网卡，原因详见 [win10 下 scapy get_working_if() 不能获得正确的网卡原因分析](https://blog.csdn.net/austin1000/article/details/100775993)

```
>>> Route()
Network          Netmask          Gateway         Iface                                        Output IP        Metric
0.0.0.0          0.0.0.0          10.11.91.254    Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     25
0.0.0.0          0.0.0.0          25.255.255.254  ZeroTier One Virtual Port                    172.28.33.102    10034
10.11.91.0       255.255.255.0    0.0.0.0         Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     281
10.11.91.161     255.255.255.255  0.0.0.0         Intel(R) Ethernet Connection (5) I219-V #2   10.11.91.161     281
>>> get_working_if()
<NetworkInterface [ZeroTier One Virtual Port] {XXX}>

```

```
def get_working_if():
    try:
        iface = min(conf.route.routes, key=lambda x: x[1])[3]  #这里有点问题，详见https://blog.csdn.net/austin1000/article/details/100775993 

```