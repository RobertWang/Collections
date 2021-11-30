> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1638257440&ver=3467&signature=bkumz*eMttjLLYIdj0px0ri6SyKAcMzUKgj*M-MaSpN-Akx0*MM9CB23V5DbEIWCuYj2FFUfeTYuJjiyWcshiU329d1CNtwvVcqjgDBogVynACw6y1k0nXjkb0sN63qK&new=1)

主机探测
----

### TCP SYN Ping

*   发送仅设置了 SYN 的空 TCP 数据包。
    
*   SYN/ACK 或 RST 响应表示机器已启动并正在运行。
    

### TCP ACK Ping

*   发送仅设置了 ACK 位的空 TCP 数据包。
    
*   未经请求的 ACK 数据包应通过 RST 进行响应，RST 显示一台机器。
    
*   SYN-ping 和 ACK-ping 看起来可能是多余的，但是大多数无状态防火墙不会过滤未经请求的 ACK 数据包，所以最好同时使用这两种 ping 技术。
    

```
>>> ans, unans = sr(IP(dst='60.205.177.90-105')/TCP(dport=80, flags='A'))
Begin emission:
Finished sending 16 packets.
.*.******....................................................................................................................................................................^C
Received 173 packets, got 7 answers, remaining 9 packets
>>> ans.summary(lambda s:s[1].sprintf("{IP: %IP.src% is alive}"))
 60.205.177.91 is alive
 60.205.177.94 is alive
 60.205.177.95 is alive
 60.205.177.97 is alive
 60.205.177.100 is alive
 60.205.177.101 is alive
 60.205.177.102 is alive


```

### UDP Ping

*   将 UDP 数据包发送给给定的端口（无论是否带有有效载荷），协议特定的有效载荷会使扫描更加有效。
    
*   选择最有可能关闭的端口（开放的 UDP 端口可能会收到空数据包，但会忽略它们）。
    
*   ICMP 端口不可达表示机器是启动的。
    

```
>>> ans, unans = sr(IP(dst='60.205.177.100-254')/UDP(dport=90),timeout=0.1)
Begin emission:
Finished sending 155 packets.
..******..*****...
Received 18 packets, got 11 answers, remaining 144 packets
>>> ans.summary(lambda s:s[1].sprintf("%IP.src% is unreachable"))
60.205.177.106 is unreachable
60.205.177.108 is unreachable
60.205.177.107 is unreachable
60.205.177.111 is unreachable
60.205.177.125 is unreachable
60.205.177.172 is unreachable
60.205.177.191 is unreachable
60.205.177.203 is unreachable
60.205.177.224 is unreachable
60.205.177.242 is unreachable
60.205.177.244 is unreachable


```

### ARP Ping

*   在同一网络 / LAN 上探测存活主机时，可以使用 ARP Ping。
    
*   更快，更可靠，因为它仅通过 ARP 在第 2 层上运行。
    
*   ARP 是任何第 2 层通信的骨干协议
    

> *   由于在 IPv6 中没有 ARP 协议，所以在 IPv6 上层定义了 NDP 协议实现 ARP 的地址解析，冲突地址检测等功能以及 IPV6 的邻居发现功能。
>     

```
>>>  ans,unans=srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst="172.17.51.0/24"),timeout=2)
Begin emission:
Finished sending 256 packets.
*******************************************************************************.***********************************************************************************...........................
Received 190 packets, got 162 answers, remaining 94 packets
>>> ans.summary(lambda r: r[0].sprintf("%Ether.src% %ARP.pdst%") )
00:16:3e:0c:d1:ad 172.17.51.0
00:16:3e:0c:d1:ad 172.17.51.1
00:16:3e:0c:d1:ad 172.17.51.2
00:16:3e:0c:d1:ad 172.17.51.3
00:16:3e:0c:d1:ad 172.17.51.4
00:16:3e:0c:d1:ad 172.17.51.5
00:16:3e:0c:d1:ad 172.17.51.6
00:16:3e:0c:d1:ad 172.17.51.7


```

### ICMP Ping

*   ICMP 扫描涉及无处不在的_ping 程序_发送的标准数据包。
    
*   向目标 IP 发送一个 ICMP 类型 8（回显请求）数据包，收到一个 ICMP 类型 0（回显应答）的包表示机器存活。
    
*   现在许多主机和防火墙阻止这些数据包，因此基本的 ICMP 扫描是不可靠的。
    
*   ICMP 还支持时间戳请求和地址掩码请求，可以显示计算机的可用性。
    

```
>>>  ans,unans=sr(IP(dst="60.205.177.168-180")/ICMP())
>>> ans.summary(lambda s:s[0].sprintf("{IP: %IP.dst% is alive}"))
 60.205.177.168 is alive
 60.205.177.169 is alive
 60.205.177.171 is alive
 60.205.177.172 is alive
 60.205.177.175 is alive
 60.205.177.174 is alive
 60.205.177.176 is alive
 60.205.177.179 is alive
 60.205.177.178 is alive
 60.205.177.180 is alive


```

服务发现（端口扫描）
----------

### TCP 连接扫描

找了个网图（ 侵删）这里展示一下 tcpdump 抓到的握手包

```
192.168.2.1.35555 > 192.168.2.12.4444: Flags [S] seq=12345   
192.168.2.12.4444 > 192.168.2.1.35555: Flags [S.],  seq=9998 ack=12346
192.168.2.1.35555 > 192.168.2.12.4444: Flags [.] seq=12346 ack=9999  


```

IP 与端口号之间以'.'分隔，ACK 用'.'表示，SYN 用'S'表示，而 [S.] 则表示 SYN+ACK

#### 在 Scapy 中制作三次握手包

**第 1 步 - 将客户端的 SYN 发送到侦听服务器**

*   使用源 IP 地址和目标 IP 地址制作一个 IP 头。
    
*   制作一个 TCP 标头，在其中生成 TCP 源端口，设置服务器侦听的目标端口，设置 TCP 的 flag SYN，并生成客户端的 seq。
    

```
ip=IP(src="192.168.2.53", dst="60.205.177.168")
syn_packet = TCP(sport=1500, dport=80, flags="S", seq=100)


```

**第 2 步 - 监听服务器的响应（SYN-ACK）**

*   保存服务器的响应。
    
*   获取服务器的 TCP 序列号，并将该值加 1。
    

```
synack_packet = sr1(ip/syn_packet)
my_ack = synack_packet.seq+1


```

*   IP 标头与初始 SYN 数据包具有相同的源和目标。
    
*   TCP 报头具有与 syn 数据包相同的 TCP 源端口和目标端口，仅设置 ACK 位，由于 SYN 数据包消耗一个序列号，因此将客户端的 ISN 递增 1，将确认值设置为递增的服务器的序列号值。
    

```
ack_packet = TCP(sport=1500, dport=80, flags="A", seq=101, ack=my_ack)
send(ip/ack_packet)


```

```
#!/usr/bin/python

from scapy.all import *
# 构建payload
get='GET / HTTP/1.0\n\n'
#设置目的地址和源地址
ip=IP(src="192.168.2.53",dst="60.205.177.168")
# 定义一个随机源端口
port=RandNum(1024,65535)
# 构建SYN的包
SYN=ip/TCP(sport=port, dport=80, flags="S", seq=42)
# 发送SYN并接收服务器响应（SYN,ACK）
SYNACK=sr1(SYN)
#构建确认包
ACK=ip/TCP(sport=SYNACK.dport,dport=80,flags="A",seq=SYNACK.ack,ack=SYNACK.seq+1)/get
#发送ack确认包
reply,error=sr(ACK)
# 打印响应结果
print(reply.show())


```

### SYN 扫描

#### 在单个主机，单个端口上进行 SYN 扫描

*   使用`sr1`功能发送并响应数据包
    
*   使用`sprintf`方法在响应中打印字段。（“SA” 标志表示开放的端口，“ RA” 标志表示关闭的端口）
    

```
>>> syn_packet = IP(dst='60.205.177.168')/TCP(dport=22,flags='S')
>>> rsp=sr1(syn_packet)
Begin emission:
Finished sending 1 packets.
..*
Received 3 packets, got 1 answers, remaining 0 packets
>>> rsp.sprintf("%IP.src%  %TCP.sport%  %TCP.flags%")
'60.205.177.168  ssh  SA'


```

#### 在单个主机，多个端口上进行 SYN 扫描

```
>>> ans,unans=sr(IP(dst="60.205.177.168")/TCP(dport=(20,22),flags="S"))
Begin emission:
Finished sending 3 packets.
..*..**
Received 7 packets, got 3 answers, remaining 0 packets
>>> ans.summary(lambda s:s[1].sprintf("%TCP.sport%  %TCP.flags%" ))
ftp_data  RA
ftp  RA
ssh  SA


```

#### 对多个主机，多个端口进行 SYN 扫描

*   `make_table`接受三个值，行，列和表数据。（在下面的示例中，目标 IP 位于 x 轴上，目标端口位于 y 轴上，响应中的 TCP 标志是表格数据）
    

```
>>> ans,unans = sr(IP(dst=["60.205.177.168-170"])/TCP(dport=[20,22,80],flags="S"))
Begin emission:
Finished sending 9 packets.
..*..**..*.................................................................................................................................................................................................................................................^C
Received 251 packets, got 4 answers, remaining 5 packets
>>> ans.make_table(lambda s: (s[0].dst, s[0].dport,s[1].sprintf("%TCP.flags%")))
   60.205.177.168 60.205.177.169 
20 RA             -              
22 SA             -              
80 SA             SA 


```

### Fin 扫描

#### 端口开放

```
>>> fin_packet = IP(dst='60.205.177.168')/TCP(dport=4444,flags='F')
>>> resp = sr1(fin_packet)
Begin emission:
Finished to send 1 packets.
^C
Received 0 packets, got 0 answers, remaining 1 packets


```

#### 端口关闭

```
>>> fin_packet = IP(dst='60.205.177.168')/TCP(dport=4399,flags='F')
>>> resp = sr1(fin_packet)
>>> resp.sprintf('%TCP.flags%')
'RA'


```

### NULL 扫描

#### 端口关闭

```
>>> null_scan_resp = sr1(IP(dst="60.205.177.168")/TCP(dport=4399,flags=""),timeout=1)
>>> null_scan_resp.sprintf('%TCP.flags%')
'RA'


```

### Xmas 扫描

#### 端口关闭

```
>>> xmas_scan_resp=sr1(IP(dst="60.205.177.168")/TCP(dport=4399,flags=”FPU”),timeout=1)
Begin emission:
.Finished sending 1 packets.
*
Received 2 packets, got 1 answers, remaining 0 packets
>>> xmas_scan_resp.sprintf('%TCP.flags%')
'RA'


```

### UDP 扫描

```
>>> udp_scan=sr1(IP(dst="60.205.177.168")/UDP(dport=53),timeout=1))


```

跟踪路由
----

*   跟踪路由技术基于 IP 协议的设计方式。IP 标头中的 TTL 值被视为跳数限制。每当路由器收到要转发的数据包时，它将 TTL 减 1 并转发数据包。当 TTL 达到 0 时，路由器将向源计算机发送答复，表示数据包已被丢弃。
    
*   各种工具背后的技术是相同的，但是实现它们的方式略有不同。Unix 系统使用 UDP 数据报文，而 Windows tracert 则发送 ICMP 请求，Linux 的 tcptraceroute 使用 TCP 协议。
    

### 使用 ICMP 进行路由跟踪

```
>>> ans,unans=sr(IP(dst="49.232.152.189",ttl=(1,10))/ICMP())
Begin emission:
Finished sending 10 packets.
*****.**........................................................................................................^C
Received 112 packets, got 7 answers, remaining 3 packets
>>> ans.summary(lambda s:s[1].sprintf("%IP.src%"))
10.36.76.142
10.54.138.21
10.36.76.13
45.112.216.134
103.216.40.18
9.102.250.221
10.102.251.214


```

### 使用 tcp 进行路由跟踪

```
>>> ans,unans=sr(IP(dst="baidu.com",ttl=(1,10))/TCP(dport=53,flags="S"))
Begin emission:
Finished sending 10 packets.
*********......................^C
Received 31 packets, got 9 answers, remaining 1 packets
>>> ans.summary(lambda s:s[1].sprintf("%IP.src% {ICMP:%ICMP.type%}"))
10.36.76.142 time-exceeded
10.36.76.13 time-exceeded
10.102.252.130 time-exceeded
117.49.35.150 time-exceeded
10.102.34.237 time-exceeded
111.13.123.150 time-exceeded
218.206.88.22 time-exceeded
39.156.67.73 time-exceeded
39.156.27.1 time-exceeded


```

Scapy 包含一个内置的 traceroute() 函数可以实现与上面相同的功能

```
>>> traceroute("baidu.com")
Begin emission:
Finished sending 30 packets.
************************
Received 24 packets, got 24 answers, remaining 6 packets
   220.181.38.148:tcp80 
2  10.36.76.13     11   
3  10.102.252.34   11   
4  117.49.35.138   11   
5  116.251.112.185 11   
6  36.110.217.9    11   
7  36.110.246.201  11   
8  220.181.17.150  11   
14 220.181.38.148  SA   
15 220.181.38.148  SA   
16 220.181.38.148  SA   
17 220.181.38.148  SA   
18 220.181.38.148  SA   
19 220.181.38.148  SA   
20 220.181.38.148  SA   
21 220.181.38.148  SA   
22 220.181.38.148  SA   
23 220.181.38.148  SA   
24 220.181.38.148  SA   
25 220.181.38.148  SA   
26 220.181.38.148  SA   
27 220.181.38.148  SA   
28 220.181.38.148  SA   
29 220.181.38.148  SA   
30 220.181.38.148  SA   
(<Traceroute: TCP:17 UDP:0 ICMP:7 Other:0>,
 <Unanswered: TCP:6 UDP:0 ICMP:0 Other:0>


```

### 使用 DNS 跟踪路由

我们可以通过在 traceroute() 函数的 l4 参数中指定完整的数据包来执行 DNS 跟踪路由

```
>>> ans,unans=traceroute("60.205.177.168",l4=UDP(sport=RandShort())/DNS(qd=DNSQR(q)))
Begin emission:
****Finished sending 30 packets.
.................
Received 21 packets, got 4 answers, remaining 26 packets
  60.205.177.168:udp53 
1 10.2.0.1        11   
2 114.242.29.1    11   
4 125.33.185.114  11   
5 61.49.143.2     11 

```

**公众号：运维开发故事**

**github：https://github.com/orgs/sunsharing-note/dashboard**