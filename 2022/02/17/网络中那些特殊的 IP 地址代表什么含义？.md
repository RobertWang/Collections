> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650533013&idx=3&sn=3d406bc4c678b14b135fb9480ec258fa&chksm=83ba8971b4cd006743afaed0b37702551ca4bef3a811acc7e990e12a912f9723decae51e2309&mpshare=1&scene=1&srcid=0206eawOyzeObesfg6ThkAqe&sharer_sharetime=1644116060161&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> **文章来源 ：****弱电工程师的圈子**  

严格说来，这个奇葩的地址 0.0.0.0 已经不是一个真正意义上的 IP 地址了。它表示的是这样一个集合：也就是说；所有不清楚的主机和目的网络。这里的 “不清楚” 是指在本机的路由表里没有特定条目指明如何到达。对本机来说，它就是一个 “收容所”，所有不认识的“三无” 人员，一律送进去。如果你在网络设置中设置了缺省网关，那么计算机系统会自动产生一个目的地址为 0.0.0.0 的缺省路由。

**2、255.255.255.255 限制广播地址。**

对本机来说，这个地址指本网段内 (同一广播域) 的所有主机。

然而它的意思很明确，使用人类语言来说意思就是 “这里的所有计算机都注意了” 这个地址不能被路由器所转发。

**3、127.0.0.1 本机地址**

主要用于测试。用汉语表示，就是 “我自己”。在 Windows 系统中，这个地址有一个别名“Localhost”。寻址这样一个地址，是不能把它发到网络接口的。除非出错，否则在传输介质上永远不应该出现目的地址为“127.0.0.1” 的数据包。

**4、224.0.0.1 组播地址**

注意它和广播的区别。从 224.0.0.0 到 239.255.255.255 都是这样的地址。224.0.0.1 特指所有主机，224.0.0.2 特指所有路由器。这样的地址多用于一些特定的程序以及多媒体程序。如果你的主机开启了 IRDP(Internet 路由发现协议），使用组播功能功能，那么你的主机路由表中应该有这样一条路由。

**5、169.254.x.x**

如果你的主机使用了 DHCP 功能自动获得一个 IP 地址，那么当你的 DHCP 服务器发生故障，或响应时间太长而超出了一个系统规定的时间，计算机操作系统会为你分配这样一个地址。如果你发现你的主机 IP 地址是一个诸如此类的地址，很不幸的是，现在你的网络不能正常运行了。

**6、10.x.x.x；172.16.0.0---172.31.255.254；192.168.x.x；私有地址**

这些地址被大量用于企业内部网络中。一些宽带路由器，也往往使用 192.168.1.1 作为缺省地址。私有网络由于不与外部互连，因而可能使用随意的 IP 地址。保留这样的地址供其使用是为了避免以后接入公网时引起地址混乱。使用私有地址的私有网络在接入 Internet 时，要使用地址翻译 (NAT)，将私有地址翻译成公用合法地址。在 Internet 上，这类地址是不能出现的。对一台网络上的主机来说，它可以正常接收的合法目的网络地址有三种：本机的 IP 地址、广播地址以及组播地址。

二、IP 地址的分类

1、A 类地址

A 类 IP 地址：由 1 个字节的网络地址和 3 个字节的主机地址，网络地址的最高位必须是 “0”。

如：0XXXXXXX.XXXXXXXX.XXXXXXXX.XXXXXXXX（X 代表 0 或 1）

A 类 IP 地址范围：1.0.0.1---126.255.255.254

A 类 IP 地址中的私有地址和保留地址：

① 10.X.X.X 是私有地址（所谓的私有地址就是在互联网上不使用，而被用在局域网络中的地址）。

B 类 IP 地址的私有地址和保留地址：

① 172.16.0.0---172.31.255.254 是私有地址

② 169.254.X.X 是保留地址。如果你的 IP 地址是自动获取 IP 地址，

而你在网络上又没有找到可用的 DHCP 服务器。就会得到其中一个 IP。

191.255.255.255 是广播地址，不能分配。

3、C 类地址

C 类 IP 地址：由 3 个字节的网络地址和 1 个字节的主机地址，网络地址的最高位必须是 “110”。

如：110XXXXX.XXXXXXXX.XXXXXXXX.XXXXXXXX（X 代表 0 或 1）

类 IP 地址范围：192.0.0.1---223.255.255.254。

C 类地址中的私有地址：

D 类地址：不分网络地址和主机地址，它的第 1 个字节的前四位固定为 1110。

如：1110XXXX.XXXXXXXX.XXXXXXXX.XXXXXXXX（X 代表 0 或 1）

D 类地址范围：224.0.0.1---239.255.255.254

5、E 类地址

E 类地址：不分网络地址和主机地址，它的第 1 个字节的前四位固定为 1111。

如：1111XXXX.XXXXXXXX.XXXXXXXX.XXXXXXXX（X 代表 0 或 1）

很多朋友可能在算 ip 地址的时候，对二进制转换成十进制，或十进制转换成二制比较不理解，弱电君就以自己理解的方式来换算进制之间的转换。

例如：

我们的现有 ip 地址为：192.168.1.1 (ip 地址是十进制)

我们现在把它转化为二进制：

1、先列出 2 进制每一位数的权值（即 2 的位数 - 1 次方），因为 IP 地址每组数最大是 255，例子中只列到第 8 位。

2、把 10 进制数参照权值进行分解，如：192=128+64, 那么对应的第 8 和第 7 位为 1，其它位为 0，转换成二进制：11000000

转换示例如图：

![](https://mmbiz.qpic.cn/mmbiz_jpg/BaicgvSuO8QZXQJWOpRT3PESmrHRx308YMfrGBItnANWnSmsUvegjMAvZ68RePwvE9zwZQiaVG1R3iaaWBRdQ3XBA/640?wx_fmt=jpeg)

结果就是：

192=11000000

168=10101000

1=00000001

最后的 ip 地址 192.168.1.1：

二进制是：

11000000.10101000.00000001.00000001

二进制转化成十进制，反着来一次就行了。