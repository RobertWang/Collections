> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/64008576)

初识 Scapy--Python 的 Scapy/Kamene 模块学习之路
--------------------------------------

原文地址:

### Scapy/Kamene 是什么?

Scapy 是基于 Python 语言的网络报文处理程序，它可以让用户发送、嗅探、解析、以及伪造网络报文，运用 Scapy 可以进行网路侦测、端口扫描、路由追踪、以及网络攻击。Kamene 是 Scapy 的分支，一开始的 Scapy 并不支持 Python 3，为了支持 Python 3 就诞生了 Scapy3k 这个分支，为了方便区分就把 Scapy3k 更名为了 Kamene。注意：最新的 Scapy 是支持 Python 3 的 (Scapy 2.4.0+)。

Scapy 官网: [https://scapy.net/](https://link.zhihu.com/?target=https%3A//scapy.net/)

Scapy Pypi 包主页: [https://pypi.org/project/scapy/](https://link.zhihu.com/?target=https%3A//pypi.org/project/scapy/)

**注意:** 本系列文章默认你有 Python 基础

### 如何选择 Python 和 Scapy 的版本？

这是 Scapy 版本的兼容列表:

![](https://pic4.zhimg.com/v2-67e1d486a2137beaf4baed16ac3efdff_b.jpg)

现在 (2019 年 4 月) 有相当一部分 Linux 发行版都不预装 Python 2 了而改为 Python 3 (Fedora 29, Ubuntu 19.04)，而且 Python 2 也将在 2020 年停止维护，所以本系列文章完全使用 Python 3(外加我也没有怎么学过 Python 2)。

### 我的实验环境:

*   运行 Scapy 的主机: Ubuntu 19.04 / Kali linux 2019.1a(不需要额外安装 Scapy)
*   端口扫描测试靶机: Metasploittable2-Linux

它们都是 VMware 虚拟机，通过桥接模式连接到本地网络，安装 VMware 虚拟机的方法这里就不介绍了，通过搜索引擎就可以查找到很多资料。当然实验环境可以跟我不一样，但推荐用 Linux 系统运行 Scapy，其他系统的安装方法可以参考官方的[文档](https://link.zhihu.com/?target=https%3A//scapy.readthedocs.io/en/latest/installation.html) (英文)。

### 安装 Scapy/Kamene:

安装 Scapy 之前需要先安装 Python 3 ，有些 Linux 发行版已经安装好了 Python 3 (Ubuntu, Kali)。安装 Python 3 网上也有很多资料了，只要输入你的操作系统加上 Python 3 就可以找到。

安装完 Python 3 之后再建议把 pip 的源改为国内的镜像 (如果你不在国内的话那就不用改了)，改 pip 源可以参考这篇文章: [https://www.cnblogs.com/ZhangRuoXu/p/6370107.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/ZhangRuoXu/p/6370107.html)

编辑配置文件:

mkdir ~/.pip

vi ~/.pip/pip.conf

然后输入一下内容，把 pip 源设为豆瓣的镜像：

```
[global]
timeout = 60
index-url = https://pypi.doubanio.com/simple

```

安装 Scapy:

sudo pip3 install scapy

执行完毕后输入 sudo scapy 验证安装是否成功，出现以下界面就表示安装成功了 (**注意:** Scapy 某些功能需用 root 权限):

sudo scapy

![](https://pic2.zhimg.com/v2-35b627cc5f401e0653079b6faa19b249_b.jpg)

上面白色的警告提示说: 无法导入 matplotlib 和 PyX 模块，没有 IPv6 路由，没有 IPython shell。我们依次安装这几个模块。目前来说我们用不到 IPv6，所以不用管它。

**这就是一个 Python shell，所以可以用 exit() 退出它。**

exit()

安装 matplotlib:

sudo pip3 install matplotlib

安装 PyX:

sudo pip3 install PyX

安装 IPython:

sudo pip3 install IPython

实际上 matplotlib 和 PyX 不安装也可以，后面一般不用，要用到时我会另外再说。但是 IPython 有必要安装一下，安装 IPython 之后你会发现 Python Shell 明显流畅了很多，而且自动补全功能也更加完善。

安装完 Scapy 后我们就开始使用它吧。

### Scapy shell 的使用

进入 Scapy shell:

sudo scapy

进入 Scapy 你可能发现了这和 Python 的 shell 有点像，但是颜色又有点不对。事实上这就是一个 Python Shell 只是更改了颜色选项，然后引入了 Scapy 模块。

知道这原理后我们只需要一行代码就可以构造一个 Ping 的请求包。

ping = IP(dst="[http://www.baidu.com](https://link.zhihu.com/?target=http%3A//www.baidu.com)") / ICMP()

看到这个你可能会问: “你这一行代码这么就构造一个数据包数据了啊！”，先别急我们慢慢来看这个数据包是这么构成的。

我们想要灵活使用 Scapy 需要知道一定的网络知识，下面我们来简单了解一下这些知识。

现在的网络数据都是分层传输的，以 TCP/IP 协议族为例网络可分为 4 层，下图就是 TCP/IP 协议族的层次图。

![](https://pic3.zhimg.com/v2-997b45db2a1b50c94ce6f0baae3a23a2_b.jpg)

最上面的应用层 (Application Layer)，然后传输层 (Transport Layer)，然后网络 (Internet Layer)，最下面一层是数据链路层 (Link/Physical Layer)。

比如说 QQ 这个应用它处在应用层，我们发送一条消息时 QQ 应用会把消息放入自己定义的一个应用协议中，然后交给它的下一层传输层。传输层协议例如 UDP(TCP 会有一点不一样)，会把自己的报文头加在数据之前，然后再交给下一层网络层。网络层的协议例如 IP 协议也会把自己的报文头加在数据之前，然后再交给数据数据链路层。数据链路层的协议例如以太网协议 (Ethernet) 还是会把自己的报文头加在数据之前(以太网协议不仅有头还有尾)，最后再通过网卡发送 Bit 流到网络中。这个过程就就叫封装，当然有封装就有解封装，解封装就是把封装的过程倒过来。下图就是数据传输过程的图片:

![](https://pic1.zhimg.com/v2-98a78e10a04ed67fb1443188616554d8_b.jpg)

知道了数据传输的过程，那些 TCP, UDP, IP, Ethernet 是什么东西啊？它们都是网络协议，网络协议就是一些标准，如果你不遵守协议或者协议和别人不一样的话那你就没办法和别人通讯。就像说不同语言的两个人没办法正常沟通一样。

我这里只是笼统概括了一下如果想深入学习的话可以看这篇文章: [https://wizardforcel.gitbooks.io/network-basic/content/0.html](https://link.zhihu.com/?target=https%3A//wizardforcel.gitbooks.io/network-basic/content/0.html)，也可以买一些相关书籍来看。

Scapy 的主要的功能就是操作各种协议的报文，所以我们接下来会学习到一些协议，但是我不会一下子讲太多协议，而是我们需要什么协议就学什么。

这里提一下报文是什么，报文就是按照一定规则组成的数据，方便构造和解析。报文头就是数据最前面的标识信息，报文 = 报文头 + 数据

好了了解这些基础知识我们继续看刚刚构造的数据包

Scapy 中的斜杠’ / ‘就是用来区分网络层次的，一般来说把底层的协议写在左边把高层的协议写在右边，就像下面这样:

数据链路层协议 / 网络层协议 / 传输层协议 / 应用层协议及数据

下面是 TCP/IP 协议族关系表:

![](https://pic4.zhimg.com/v2-12348205529ca03b554dd7b1dd53c497_b.jpg)

一般来说我们不需要关心数据链路层的协议 Scapy 会为我们自动配置。

Scapy 定义了很多函数和类，通过这些函数和类我们就可以构造数据包。这里的 IP 可以用来生成一个 IP 的报文头。

IP()

![](https://pic4.zhimg.com/v2-f1a6011ac5bba937aff9f50de4814023_b.jpg)

show() 方法获取报文字段信息 (在 Python shell 中下划线’_’表示上条语句执行结果)

_.show()

![](https://pic2.zhimg.com/v2-2564158ec881ffa1e9c97dc91ccfc419_b.jpg)

这里简单介绍一下 IP 协议。IP 协议就是通过 IP 地址来标识互联网上的主机，我们通过 IP 地址就可以访问到互联网上的主机 (非 NAT)。IP 协议有 2 个版本 IPv4 和 IPv6，目前主流的还是 IPv4，但是已经有很多地方在使用 IPv6 了，可能过个几年 IPv6 就是主流了。

长度 (len)，校验和(chksum)、协议(proto) 等字段 Scapy 都会自动计算，所以我们只有 3 个字段需要注意一下，ttl、src、dst。

*   ttl: 生存时间值 (Time To Live)，当数据包每进行进行一次转发时 TLL 值就会减一，TTL 为 0 时数据包就会被丢弃，TTL 的主要作用是避免 IP 包在网络中的无限循环和收发，节省了网络资源，并能使 IP 包的发送者能收到告警消息 (路由追踪原理)。
*   src: 源地址 (Source Address) 数据发送者的地址
*   dst: 目的地址 (Destination Address) 数据接收者的地址

所以这段代码就是设置目标地址:

IP(dst="[http://www.baidu.com](https://link.zhihu.com/?target=http%3A//www.baidu.com)")

有基础的朋友可能会说：“这明明是域名，不是 IP 地址!“。对，这确实是域名，但是 Scapy 功能非常强大，当你把 dst 赋值为域名时 Scapy 会自动调用一个名为 Net 的函数来解析域名。

![](https://pic2.zhimg.com/v2-f2be3abdf66569d3724e6bb5b2157e0d_b.jpg)

还可以指定掩码位数:

list(Net('[http://www.baidu.com/30](https://link.zhihu.com/?target=http%3A//www.baidu.com/30)'))

![](https://pic3.zhimg.com/v2-2c22e5448e0c74cda460157f161c4812_b.jpg)

然后我们再来看看 ICMP 协议:

ICMP(互联网控制消息协议) 是互联网协议族的核心协议之一。它用于 TCP/IP 网络中发送控制消息，提供可能发生在通信环境中的各种问题反馈，通过这些信息，使管理者可以对所发生的问题作出诊断，然后采取适当的措施解决 (维基百科)。ping 就是基于 ICMP 协议的。

生成一个 ICMP 报文头:

ICMP()

![](https://pic3.zhimg.com/v2-e447fb78d6cd60b0c93054d775e7561a_b.jpg)

ICMP 只有 5 个字段这些字段分别的意思是 (维基百科):

*   type：ICMP 的类型, 标识生成的错误报文。
*   code：进一步划分 ICMP 的类型, 该字段用来查找产生错误的原因。例如，ICMP 的目标不可达类型可以把这个位设为 1 至 15 等来表示不同的意思。
*   chksum：校验码部分, 这个字段包含有从 ICMP 报头和数据部分计算得来的，用于检查错误的数据，其中此校验码字段的值视为 0。
*   id：这个字段包含了 ID 值，在 Echo Reply 类型的消息中要返回这个字段。
*   Seq：这个字段包含一个序号，同样要在 Echo Reply 类型的消息中要返回这个字段。

以下是 ICMP type code 的对应表:

![](https://pic4.zhimg.com/v2-47be23fd72b94b23b059879295e51ff7_b.jpg)

一般来说只用记住这 2 个，type 为 8、code 为 0 是 ping 的请求，type 为 0、code 为 0 是 ping 的响应。

知道这些之后我们就可以构造一个 ICMP 请求报文头。注意 Scapy 会为每个报文设定默认值，调用 ICMP 函数默认就是构造一个 ICMP 请求，所以下面两条是等价的:

ICMP()

ICMP(type=8, code=0)

也可以查看对象的 type，code 属性值

![](https://pic3.zhimg.com/v2-e4dd6fcb3b8375e2616e209c163cbff2_b.jpg)

通过 ls 函数就可以查看默认值

ls(<协议名>)

![](https://pic3.zhimg.com/v2-c206109662cb0eb66715214f4cacf806_b.jpg)

也可以同时查看实际值和默认值

ls(IP(dst="[http://www.baidu.com](https://link.zhihu.com/?target=http%3A//www.baidu.com)"))

没有括号的是实际值，有括号的是默认值

![](https://pic4.zhimg.com/v2-559c136ad27f47cd4dab412f943c50d7_b.jpg)

好了现在我们再回头看刚刚构造的 ping 请求包就应该可以明白其中的意思了:

ping = IP(dst="192.168.40.1") / ICMP()

**注意:** 为了方便演示我把地址换成了自己的网关

ICMP 函数首先构造一个 ping 请求报文头，然后 IP 函数构造一个 IP 报文头设置目标为’192.168.40.1’，最后赋值给变量 ping。

构造完数据包之后那我们就来发送数据包。

resultpkt = sr1(ping)

sr1 函数发送一个数据包，再接收一个数据包，然后返回响应的数据包。发送成功会显示形如下面信息。

![](https://pic4.zhimg.com/v2-db6899af9a8798e443e1182ee060e867_b.jpg)

Received 3 packets 意思是: 收到了 3 个包

got 1 answers 意思是: 得到了一个应答。也就是我们需要的包。

rermaining 0 packets 意思是: 剩余没有应答得包。

直接在命令行输入变量名就可以输出接收到的包信息了。

![](https://pic1.zhimg.com/v2-39d6902eb9c7f6362bc9febefd9a7308_b.jpg)

好了得到回显数据之后就可以通过一些方法来访问其中的数据了。

通过下标访问每个报文数据:

这里可能有一点难理解，简单解释一下就是：对于 IP 协议来说 ICMP 报文头和 Padding 实际上就是一堆数据，当我们获取 IP 报文时它会把 IP 报文头和数据一起输出。对于获取 ICMP 报文也是如此，输出 ICMP 报文头和 Padding 数据 **(报文实际上就是报文头加数据)**。

通过属性的形式获取字段

resultpkt[0].src

![](https://pic4.zhimg.com/v2-2511b95f10b0620e61eef68b2a26925b_b.jpg)

resultpkt[1].type

![](https://pic2.zhimg.com/v2-a52821b50f772f66dcf34774a5f598cd_b.jpg)

可以加下标也可以不加，但是推荐加上。例如我们想访问 ICMP 的校验和 (chksum)，不加标访问的是 IP 的校验和加下标[1] 才可以访问 ICMP 的校验和

![](https://pic4.zhimg.com/v2-10804c733d982da7054e17134ef162d3_b.jpg)

通过 fields 属性以字典的形式获取报文头

获取 IP 报文头:

resultpkt[0].fields

![](https://pic1.zhimg.com/v2-6a5274657131aae472b62d751f0ae544_b.jpg)

获取 ICMP 报文头 :

resultpkt[0].fields

![](https://pic2.zhimg.com/v2-9398406c9132730cdc058f9d97383705_b.jpg)

获取 Padding 数据:

resultpkt[2].fields

![](https://pic3.zhimg.com/v2-c48b6a214df147b870b679a6c25ca866_b.jpg)

既然是字典就可以通过键获取对应的值。

获取 IP 报文头的源地址:

resultpkt[0].fields['src']

![](https://pic1.zhimg.com/v2-f1c00b93153417dc23cfec56901de5ac_b.jpg)

获取 IP 报文头的目标地址:

resultpkt[0].fields['dst']

![](https://pic3.zhimg.com/v2-d86e013a8ce420f36b59620317473f8a_b.jpg)

获取 ICMP 报文头的 type 值和 code 值:

resultpkt[0].fields['type']

resultpkt[0].fields['code']

![](https://pic2.zhimg.com/v2-eaa9ed2d08c867558cf3ba3575aaefb1_b.jpg)

还有一种高级的方法，通过协议下标来访问报文头:

![](https://pic4.zhimg.com/v2-aa2b7794beb37f922a60728015695dd7_b.jpg)

就是把数字下标改为了对应协议的下标功能和数字下标基本一样，但是用协议下标可以增加程序的灵活性和可读性。

好了这就是 Scapy shell 的基本用法并，接下来我们讲讲 Scapy 数据包收发机制。

### Scapy 数据包收发机制

Scapy 数据包发送函数:

*   sr1: 发送一个数据包并接收一个相匹配的数据包
*   srp1: 与 sr1 一样但是关注数据链路层 (以后讲 ARP 协议时会用)
*   sr: 发送数据包并接收相匹配的数据包
*   srp: 与 sr 一样但是关注数据链路层 (以后讲 ARP 协议时会用)
*   send: 只发送数据包不接收数据包
*   sendp: 与 send 一样但是关注数据链路层 (以后讲 ARP 协议时会用)

上面提到了匹配这个概念接下来会讲解。

下图就是 Scapy sr 函数发收流程图:

![](https://pic1.zhimg.com/v2-489e23e74b6096685d72204a228bf178_b.jpg)

解释一下就是: Scapy sr 函数会把数据包批量发送到网络，再获取接收到的所有包 (包括其他应用的数据包)，再用发送的包和接收包的进行匹配(通过 IP, 端口等) 把发送的数据包和它相匹配的应答包组成一个元组放入一个列表中(和列表很相似的对象)，再把没有应答的包放入一个列表中(发送了数据包但是没有配到对应的接收包)，最后把两个列表合成一个元组。(没有匹配到的数据包丢弃)

通过实例理解一下

生成一组数据包然后通过 sr 函数发送，timeout 参数设置超时时间默认单位为秒，这里设置超时时间为 10 秒钟。

1.  result = sr(IP(dst="10.72.1.0/24") / ICMP(), timeout=10)

![](https://pic2.zhimg.com/v2-ae5e31c9d8f882ef2bac2dedd5ab0461_b.jpg)

直接输入变量名或用 type 函数就可以看到返回的数据是个元组

![](https://pic4.zhimg.com/v2-5028e67355f0f2658c2ce4a06e2d8507_b.jpg)

通过下标访问 results 中的数据

![](https://pic3.zhimg.com/v2-f004b4f195fa8edf9915e80bb13278ea_b.jpg)

上图通过 result[0]，获取有响应的包的列表，用 len 函数获取长度，用 display 方法来展示数据。

result[0] 里的每个素都是元组，每个元组都是由发送的包和它相匹配的应答包组成。

![](https://pic1.zhimg.com/v2-e5f6c0a69e0f25524235dac3eaab2510_b.jpg)

分别访问发送的包和接收的包

![](https://pic3.zhimg.com/v2-6818d148034f1db25e09f440f83b2466_b.jpg)

对于单个数据包的操作就和就前面的一样了

通过 result[1] 访问没有相应的请求数据表列表，里面的元素就是单个数据包。

![](https://pic4.zhimg.com/v2-2563866882b68ae54799e266587b55b3_b.jpg)

清楚理解 Scapy 的发收包机制很重要，以后的学习都是基于这个。

接下来带来一个干货。

### 开发一个 ping 扫描工具

下面代码的 Git 仓库地址: [https://github.com/starunity/Scapy-instance](https://link.zhihu.com/?target=https%3A//github.com/starunity/Scapy-instance)

直接亮代码:

```
#!/usr/bin/env python3

from scapy.all import *

def pingscan(ip):
   """
   pingscan(ip)
   Ping the incoming IP.
   ip: Pass in an IP like 192.168.1.0 or 192.168.1.0/24
   """
   answer, uanswer = sr( \
       IP(dst=ip) / ICMP(), \
       timeout=10, verbose=False \
   )
 
   alive = []
 for send, recv in answer:
 if recv[ICMP].type == recv[ICMP].code == 0:
           alive.append(recv[IP].src)

 return alive


if __name__ == '__main__':
   ip = input("Enter IP address:")
   result = pingscan(ip)
 for i in result:
 print("{} is alive.".format(i))

```

第 13 行的 sr 函数里面的 verbose 参数是用来开启或关闭发收包详情的

默认开启的效果:

![](https://pic2.zhimg.com/v2-1b941c7dc667f503e286c40c0bd4b7a1_b.jpg)

关闭的效果:

![](https://pic1.zhimg.com/v2-c7f6f4e486f60e38f6a91c67285694d0_b.jpg)

执行结果如下:

![](https://pic4.zhimg.com/v2-efbeabb32b2ac3605c6e254ce40e64d3_b.jpg)

好了这篇为到这里就结束了，接下来我还会写一些和 Scapy 相关的内容。如果喜欢的话可以点个赞、打个赏，如果有写的不对或有需要改进的地方可以在下面留言。