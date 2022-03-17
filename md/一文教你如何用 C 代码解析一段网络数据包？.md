> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651164879&idx=1&sn=4b887c7325cb109108ec82c49e2edf9d&chksm=80644190b713c886a1c5793d6439030afd19a6f964bd36b93e1db1706e24e8db361a6d50f0b9&scene=21#wechat_redirect)

本文的目的是通过随机截取的一段网络数据包，然后根据协议类型来解析出这段内存。

学习本文需要掌握的基础知识：

1.  网络协议
    
2.  C 语言
    
3.  Linux 操作
    
4.  抓包工具的使用
    

其中抓包工具的安装和使用见下文：

《[一文包你学会网络数据抓包](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651164876&idx=2&sn=770d8c1d55b0263a64b7aa522077ed8c&chksm=80644193b713c88537eed42a490eef13ad6454bbb00f8d2a093a574b04e788329f85c853de6f&scene=21#wechat_redirect)》

一、截取一个网络数据包
-----------

通过抓包工具，随机抓取一个 tcp 数据包

![](https://mmbiz.qpic.cn/mmbiz_jpg/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIn2J4DosvhWcom7wZQibJ0GS4waR30LlNNabbg1Zzic68xMKAENA8bc0w/640?wx_fmt=jpeg)科莱抓包工具解析出的数据包信息如下：![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIa5gQ3DiaQicYMfuvX2hiaJCDicIn8ibLDTFIvZiajlpQjJ5Vfr0blwuYtTgA/640?wx_fmt=png)数据包的内存信息：![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIRb4qEz97zAu5GR2vuz3BdGZ83aoESsqXeg6y6pZcR9EL5nFsr7Sb3w/640?wx_fmt=png)数据信息可以直接拷贝出来：![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIlCQiaBGH5kh7DeESEscCACSHPjxjaPPB9y3WVmamKTcwpsrG0yl1M5Q/640?wx_fmt=png)

二、用到的结构体
--------

下面，一口君就手把手教大家如何解析出这些数据包的信息。

我们可以从 Linux 内核中找到协议头的定义

*   以太头：
    

```
drivers\staging\rtl8188eu\include\if_ether.h 
```

```
struct ethhdr {
 unsigned char h_dest[ETH_ALEN]; /* destination eth addr */
 unsigned char h_source[ETH_ALEN]; /* source ether addr */
 unsigned short h_proto;  /* packet type ID field */
};
```

*   IP 头
    

```
 include\uapi\linux\ip.h 
```

```
struct iphdr {
#if defined(__LITTLE_ENDIAN_BITFIELD)  //小端模式
 __u8 ihl:4,
  version:4;
#elif defined(__BIG_ENDIAN_BITFIELD)    //大端模式
 __u8 version:4,
  ihl:4;
#endif
 __u8 tos;
 __u16 tot_len;
 __u16 id;
 __u16 frag_off;
 __u8 ttl;
 __u8 protocol;
 __u16 check;
 __u32 saddr;
 __u32 daddr;
 /*The options start here. */
};
```

tcp 头

```
include\uapi\linux\tcp.h
```

```
struct tcphdr {
 __be16 source;
 __be16 dest;
 __be32 seq;
 __be32 ack_seq;
#if defined(__LITTLE_ENDIAN_BITFIELD)
 __u16 res1:4,
  doff:4,
  fin:1,
  syn:1,
  rst:1,
  psh:1,
  ack:1,
  urg:1,
  ece:1,
  cwr:1;
#elif defined(__BIG_ENDIAN_BITFIELD)
 __u16 doff:4,
  res1:4,
  cwr:1,
  ece:1,
  urg:1,
  ack:1,
  psh:1,
  rst:1,
  syn:1,
  fin:1;
#else
#error "Adjust your <asm/byteorder.h> defines"
#endif 
 __be16 window;
 __sum16 check;
 __be16 urg_ptr;
};
```

因为协议头长度都是按照标准协议来定义的，

所以以太长度是 14，IP 头长度是 20，tcp 头长度是 20，

各个协议头对应的内存空间如下：

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKInjXDTxM85kWjEtzbicjyVcjibaiaoKCwibOuvJKbslicVM093YBib4ia6wgOg/640?wx_fmt=png)

三、解析以太头
-------

```
#define MAC_ARG(p) p[0],p[1],p[2],p[3],p[4],p[5]
```

```
 struct ethhdr *ethh;
 unsigned char *p = pkt;
 
 ethh = (struct ethhdr *)p;

 printf("h_dest:%02x:%02x:%02x:%02x:%02x:%02x \n", MAC_ARG(ethh->h_dest));
 printf("h_source:%02x:%02x:%02x:%02x:%02x:%02x \n", MAC_ARG(ethh->h_source));
 printf("h_proto:%04x\n",ntohs(ethh->h_proto));
```

> 注意，数据包中的数据是网络字节序，如果要提取数据一定要注意字节序问题 ethh->h_proto 是 short 类型，占 2 个字节，所以存储到本地需要使用函数 ntohs 其中：n：network 网络字节序 h：host       主机字节序 s：short     2 个字节 l：long       4 个字节 ntohl()  ：4 字节网络字节序数据转换成主机字节序 htons() ：2 字节主机字节序数据转换成网络字节序 ntohs() ：2 字节网络字节序数据转换成主机字节序 htonl() ：4 字节主机字节序数据转换成网络字节序

当执行下面这条语句时，

```
ethh = (struct ethhdr *)p;
```

结构体指针变量 **eth** 的成员对应关系如下：

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIEbMsvUPicbZ7icYcXOvgVcuicA7ZQS8cF0143JJwBcV949DdvrmYNnkJw/640?wx_fmt=png)最终打印结果如下：![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKITO6LVnk3bZLJ4FaIC5qSUVYSnl4cPRDzn6IHSDDEqkZV6PP1QW1ribA/640?wx_fmt=png)

四、解析 ip 头
---------

解析 ip 头思路很简单，

就是从 pkt 头开始偏移过以太头长度 (14 字节) 就可以找到 IP 头，

解析代码如下：

```
#define IP_ARG(p)  p[0],p[1],p[2],p[3]
```

```
 /*
  解析IP头
 */
 if(ntohs(ethh->h_proto) == 0x0800)
 {
 
  iph = (struct iphdr *)(p + sizeof(struct ethhdr));

  q = (unsigned char *)&(iph->saddr);
  printf("src ip:%d.%d.%d.%d\n",IP_ARG(q));

  q = (unsigned char *)&(iph->daddr);
  printf("dest ip:%d.%d.%d.%d\n",IP_ARG(q));
 }
```

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIpULtevsTsN6dYY7hOIUAib8MCoo420a86C4heNaZCeyrX7GeJjK3ASg/640?wx_fmt=png)

Iiph

最终解析结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKI3Pxfq4ZpOpLLvyMMDibiacHaaIhzIeZUun2eNS7vJhqNxw5aXP3fov2A/640?wx_fmt=png)可以看到我们正确解析出了 IP 地址，结果与抓包工具分析出的数据保持了一致。

其中 protocol 字段表示了 ip 协议后面的额协议类型，常见的值如下：

<table data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)"><thead data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)"><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">数值</th><th data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); min-width: 85px; text-align: left;">描述</th></tr></thead><tbody data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)"><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">0</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">保留字段，用于 IPv6(跳跃点到跳跃点选项)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">1</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Internet 控制消息 (ICMP)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">2</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Internet 组管理 (IGMP)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">3</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">网关到网关 (GGP)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">4</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">1P 中的 IP(封装)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">6</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">传输控制 (TCP)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">7</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">CBT</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">8</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">外部网关协议 (EGP)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">9</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">任何私有内部网关 (Cisco 在它的 IGRP 实现中使用) (IGP)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">10</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">BBNRCC 监视</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">11</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">网络语音协议</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">12</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">PUP</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">13</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">ARGUS</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">14</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">EMCON</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">15</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">网络诊断工具</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">16</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">混乱 (Chaos)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">17</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">用户数据报文 (UDP)</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">41</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">1Pv6</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">58</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">1Pv6 的 ICMP</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">59</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">1Pv6 的无下一个报头</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">60</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">IPv6 的信宿选项</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">89</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">OSPF IGP</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">92</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">多播传输协议</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">94</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">IP 内部的 IP 封装协议</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">95</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">可移动网络互连控制协议</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">96</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">旗语通讯安全协议</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">97</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">IP 中的以太封装</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">98</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">封装报头</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">100</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">GMTP</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">101</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Ipsilon 流量管理协议</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">133～254</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">未分配</td></tr><tr data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">255</td><td data-darkmode-color-16475456566708="rgb(163, 163, 163)" data-darkmode-original-color-16475456566708="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16475456566708="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16475456566708="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">保留</td></tr></tbody></table>

五、解析 tcp 头
----------

查找 tcp 头思路很，

就是从 pkt 头开始偏移过以太头长度 (14 字节)、和 IP 头长度（20 字节）就可以找到 tcp 头，

```
 switch(iph->protocol)
  {
   case 0x1:
    //icmp
    break;
   case 0x6:
    //tcp    
    tcph = (struct tcphdr *)(p + sizeof(struct ethhdr) + sizeof(struct iphdr));
    printf("source:%d dest:%d \n",ntohs(tcph->source),ntohs(tcph->dest); 

    break;
   case 0x11:
    //udp
    
    break;
  }
```

结构体与内存对应关系![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIW8lZ0EW0OZSYlTgpup6cRkZQdzsQFm2LGSc7W1JTiauV8e43Bozr79A/640?wx_fmt=png)

打印结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIueXraFZEsYGdpe4xicsh8Rd0bjEjFhHAzvicRKwjuywdQ05OMZh7Xicjw/640?wx_fmt=png)

六、学会用不同格式打印这块内存
---------------

在实际项目中，可能我们解析的并不是标准的 TCP/IP 协议数据包，

可能是我们自己的定义的协议数据包，

只要掌握了上述方法，

所有的协议分析都能够手到擒来！

有时候我们还需要打印对方发送过来的数据帧内容，

往往我们会以 16 进制形式将所有数据打印出来，

这样是最有利于我们分析数据内容的。

### 1. 按字节打印

代码如下：

```
 for(i=0;i<400;i++)
 {
  printf("%02x ",pkt[i]);
  if(i%20 == 19)
  {
   printf("\n");
  }
 }
```

![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIybNdjQqicQwj4nRBZIERrJ4OP4IAKK0ibh5cRPV4ne2ndwOswNice1T2Q/640?wx_fmt=png)

2. 按 short 类型分析一段内存
-------------------

我们接收数据时，虽然使用一个 unsigned char 型数组，

但是有时候对方发送过来的数据可能是 2 个字节的数组，

那我们只需要用 short 类型的指针，指向内存的头，

然后就可以通过该指针访问到对方发送的数据，

**这个时候一定要注意字节序问题，**

不同场景可能不一样，所以一定要具体问题具体分析，

本例因为是网络字节序数据转换成主机字节序，

所以需要转换字节序。

```
//转变short型字节序
void indian_reverse(unsigned short arr[],int num)
{
 int i;
 unsigned short temp;

 for(i=0;i<num;i++)
 {
  temp = 0;

  temp = (arr[i]&0xff00)>>8;
  temp |= (arr[i]&0xff)<<8;
  arr[i] = temp;
 }
}
main()
{
 unsigned short spkt[200];
 
 ………………
 memcpy(spkt,pkt,sizeof(pkt));

 indian_reverse(spkt,ARRAY_SIZE(spkt));
 
 for(i=0;i<200;i++)
 {
  printf("%04x ",spkt[i]);
  if(i%10 == 9)
  {
   printf("\n");
  }
 }
 ………………
}
```

结果如下：![](https://mmbiz.qpic.cn/mmbiz_png/icRxcMBeJfcicM97SaPBSfnKEibnTmuUaKIlL6dz1QpziblEFlicLAxgNwOdicFcBDNibr1hzV0Bfp8IVTSVoAjCPNgrQ/640?wx_fmt=png)

好了，这个例子掌握了，那么网络就算入门了，快操练起来吧！

完整代码请公众号后台回复：**数据包解析**

- EOF -

推荐阅读  点击标题可跳转

1、[编写可移植 C/C++ 程序的一些要点](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651164733&idx=1&sn=564a54552227f8b1d5821d31620a5687&chksm=80644162b713c8747a5de1db76181995033e771f51663a54df5d3812274f4adaec2993f60188&scene=21#wechat_redirect)

2、[C++ 内存管理（建议收藏）](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651164732&idx=1&sn=23dbe725d1f7e8b502a14b23908214be&chksm=80644163b713c875a27c785a81ebbbbed2b152a7309a059acdf2aa26a67b9e6ce21934bcc3e7&scene=21#wechat_redirect)

3、[Modern C++ 有哪些能真正提升开发效率的语法糖？](http://mp.weixin.qq.com/s?__biz=MzAxNDI5NzEzNg==&mid=2651164730&idx=2&sn=77fb20f5fae3bc3852f5c3d18c502e36&chksm=80644165b713c87342c0056a0f3863462a40f6e8d46ebe1cb17de69334214649b5e249ac3bac&scene=21#wechat_redirect)