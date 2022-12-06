> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzM0MjcyMQ==&mid=2650152996&idx=3&sn=e4254213d4af1d959694a58e05c2c1b7&chksm=bed9fb4a89ae725c6984cea6a0b453d8b2706537cb1e27019253b53cf525be05aed0bd22b5e8&mpshare=1&scene=1&srcid=0721KGqyiGYVOZM5bm2LwcoT&sharer_sharetime=1658412578216&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

    ip2region v2.0 的 xdb 数据管理引擎已经发布，并且已经完成了 3 种语言的 maker 以及 8 种语言的 binding 实现，在此简单地描述下 v1.0 的缺陷，算是告别这个工作了近 8 年的管理引擎，详细介绍下 v2.0 xdb 的数据结构以及查询的过程。

一、v1.0 db 格式的缺陷

二、v2.0 xdb 格式的设计  

    ip2region 现在自带的原始数据都已经接近 70 万行了，更别提上百万行自定义数据的场景的使用者了，也就是上面提到的 bug 其实已经存在一段时间了，要从根本上解决这个问题我还是果断选择了重新设计一个数据存储格式。

    xdb 存储管理格式已经稳定且测试效果非常好，不仅可以支持**亿级别的 IP 数据段**，还将查询时间缩短到了 **10 微秒级别**，代码和使用文档请到 Github 或者 Gitee 浏览，这里主要介绍 xdb 数据格式的结构，整个结构按照文件指针从小到大分为如下四个层段（提示：阅读过程中不要过于纠结，往后看完，结合后面的查询过程分析会更容易理解或者理解的更清晰）：

![](https://mmbiz.qpic.cn/mmbiz_png/KtlVMTxZZ4Maydgcr3lMhEiaRIHKEV9CgXbXVpwOOPGoAgrMYGV0VY9iaXQLibxh1LaUjFJ9VLt1ibribe0Rwj9s9lQ/640?wx_fmt=png)

第一层 - Header 段：

    header 段主要是记载一些头信息，这个段固定占据 256 字节的空间，目前实际上只用到 16 个字节，剩下的全部为 0 的预留空间，便于后期扩展需要，目前的具体空间使用情况如下：

![](https://mmbiz.qpic.cn/mmbiz_png/KtlVMTxZZ4Maydgcr3lMhEiaRIHKEV9CgbHcG8mt9lGbj7WAGNiadQRnic7eApWNjJg6LnBN9uxBeb801SfNDMMUA/640?wx_fmt=png)

题外思考，有哪些可能的空间扩展需求呢？例如：  

*   如果想要验证 xdb 数据的完整性，可以在 16 到 31 之间的 16 字节中存储 header 段外的数据的 MD5 哈希值，便于确认文件是否在网络传输或者项目打包过程中受损。
    
*   增加其他的加速查询的信息等等。  
    

第二层 - Vector 索引段：  

    vector 索引段是二分索引的二级索引，本质上是一个 256 × 256 的二维数组，等同于将全部的 IP 数据段按照 IP 的前两个字节拆分成 256 × 256 个分区，这样利用一次固定 IO 操作就可以很大程度的减少后续的二分查找的 IO 操作次数，数据行数越多，这个索引的加速效果越明显，我统计过大部分的 IP 查询可以两次 IO 操作完成，Vector 索引空间表结构如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/KtlVMTxZZ4Maydgcr3lMhEiaRIHKEV9CgtWXZnPTpHn6CEibhZGhV8Jd3B9ENAibnUKmeVic06ZyotXdps8N4PNffg/640?wx_fmt=png)

    如图所示，本质上就是一个 256 × 256 的二维数组，每个 Vector 索引项的结构，也就是上面的 256 × 256 的每个单元格的结构如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/KtlVMTxZZ4Maydgcr3lMhEiaRIHKEV9Cgqna88eggObRABiaiccBM3cPpvltgCicBvd4V1AneLsPJYpA7vIquGtESA/640?wx_fmt=png)

    开始位置指向区域二分索引的开始位置，结束位置指向区域二级索引的结束位置，也就是 vector 索引将整个二分索引分成了 256 × 256 个分区，便于更快的减少二分索引的查找范围，从而加速查询。每个单元格的空间是固定的，所以整个 vector 索引段占据的空间为：256 × 256 × 8 = 524288 Bytes = 512 KiB，这个空间是固定的与原始的 IP 数据行数无关。

第三层 - 地域信息段：

    这层写入的是 data/ip.merge.txt 原始数据中每行数据的起始和结束 ip 后的地域信息，对于 xdb 格式，你可以完全替换为自己的地域信息，原始文件是什么 xdb 里面就存储什么，例如如下三个连续的 IP 数据段：

```
0.0.0.0|0.255.255.255|0|0|0|内网IP|内网IP
1.0.0.0|1.0.0.255|澳大利亚|0|0|0|0
1.0.1.0|1.0.3.255|中国|0|福建省|福州市|电信

```

会依次写入如下三个连续的地域信息：  

```
0|0|0|内网IP|内网IP
澳大利亚|0|0|0|0
中国|0|福建省|福州市|电信

```

    地域信息原始文件是什么就会写入什么，字符串使用的是 utf-8 编码，生成程序会自动去重，同样的地域信息只会写入一份，写入前会记载当时文件的指针存储到下面的二分索引的数据指针空间中，所以这一层的空间占用是没法确定的，最终写入了多少数据就占据多少空间。

第四层 - 二分索引空间段：

    二分索引层是查询的核心，搜索过程的核心就是在二分索引中进行拆半查找，二分索引的空间表结构如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/KtlVMTxZZ4Maydgcr3lMhEiaRIHKEV9CgXEtibCXkR7ddkhiaemAfAlOmh1Dz94FicGbzEquIEmOjIODJeZ76kNKaA/640?wx_fmt=png)

    每个二分索引项占据 14 个字节，详细字段和空间分布如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/KtlVMTxZZ4Maydgcr3lMhEiaRIHKEV9CgFkjZGykiaNSOzicGNOribvDb7myjEjVKcyjib463uvj3jLfLfgFD2ZGHZw/640?wx_fmt=png)

    所以二分索引的空间占用也是没法确定，但是可以通过原始的 IP 数据段行数来计算，例如默认的 data/ip.merge.txt 原始数据有 683591 行，则二分索引空间占用为：683591 × 14 = 9570274 Bytes = 9.12 MiB，占据了 11MiB 的整个 data/ip2region.xdb 的绝大部分空间，这个二分索引空间是还可以继续压缩的，我现在的原则是查询速度第一，空间压缩这块觉得没这个必要，有需要的小伙伴可以自己压缩下。

    综合上面的描述，整个 xdb 文件的空间结构以及层级之间的大致关系描述如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/KtlVMTxZZ4Maydgcr3lMhEiaRIHKEV9CgEPYNLxpNj4vzbGtWdicjv49J7RWLsUcUpwBibTxXzEmjbhrYTKE32O7w/640?wx_fmt=png)

三、查询过程的分析

    有了上述对整个 xdb 文件的基础结构我们就可以详细的探讨 xdb 的搜索过程了，搜索过程的深入也自然会让你对整个 xdb 文件的结构有更清晰的认识，整个搜索过程分为如下四步：

第一步：将 IP 转为整数  

    计算机里面任何数据都可以转换为数字，这是一句废话？整个数字电路就是基于二进制的，每个查询 binding 都有一个类似的 ip2long 的函数，用于把字符串 IP 转换为整数 IP，IP(v4) 本身就是一个 4 字节的整数，IP 的字符串输出形式就是四个字节转换成整数然后用点串接起来：  

```
var ip = "120.24.78.129"
// 把字符串 ip 按照 . 拆分，此处略过错误检测
var ps = ip.split(".")
// 把切分得到四个数字转换成整数，然后进行位运行就好。
var ip_int = (toInt(ps[0]) << 24) | (toInt(ps[1]) << 16) | (toInt(ps[2]) << 8) | toInt(ps[3]))

```

第二步：从 vector 索引中获取二分索引段

    第一步确保我们收到的参数是一个整数，至少是 4 字节，例如 java 这类没有无符号概念的语言需要使用 8 字节的长整，不然会出现溢出的情况。vector 索引检索的逻辑很简单，具体如下：

```
// 常量定义
// 每个 vector 索引项的字节数
var VectorIndexSize = 8
// vector 索引的列数
var VectorIndexCols = 256
// vector 索引段整个的字节数
var VectorIndexLength = 512KiB
// 假设 vectorIndex 为加载的向量索引
// loadVectorIndex api 返回的数据，本质上是一维的 byte 数组。
var vectorIndex = byte[VectorIndexLength]
// 第一个字节的整数 (&0xFF，确保取值在 0 ~ 255)
var il0 = (ip_int >> 24) & 0xFF
// 第二个字节的整数 (&0xFF，确保取值在 0 ~ 255)
var il1 = (ip_int >> 16) & 0xFF
// 计算得到 vector 索引项的开始地址。
// 这里可以对比上述的 vector table 结构进行理解
var idx = il0 * VectorIndexCols * VectorIndexSize + il1 * VectorIndexSize
// 区域二分索引的开始地址
// idx 开始读取 4 个字节，用小端字节序解码得到一个整数
var sPtr = getInt(vectorIndex, idx)
// 二分区域索引的结束地址
// idx + 4 处读取 4 个字节，用小端字节序解码得到一个整数
var ePtr = getInt(vectorIndex, idx + 4)

```

    每个 searcher binding 都有加载 vectorIndex 的 api，本质上是一个一维的 byte 数组，然后利用 IP 地址的前两个字节到 vectorIndex 中快速的得到这个分区的二分索引的开始和结束地址，这也是向量索引的核心，利用一次检索将二分索引分成 256 × 256 个分区，急剧减少后续二分搜索的扫描范围，从而减少运行时的 IO 操作数，加速查询过程。

第三步：在二分索引中进行拆半查找

     第二步得到了区域二分索引的开始和结束地址之后，就可以在这个区域索引之间使用拆半搜索去得到具体的可以定位地域数据的信息了，具体的拆半搜过过程如下：

```
// 常量定义
// 二分索引项的字节数
var SegmentIndexSize = 14
// 二分搜索低位
var l = 0
// 二分搜索高位
// 上第一步得到的结束索引位置减去开始索引位置
// 再除以每个索引的字节大小就是这次搜索要扫描的索引的个数了。
var h = (ePtr - sPtr) / SegmentIndexSize
// 数据长度
var dataLen = 0
// 数据地址
var dataPtr = 0
// 开始拆半搜索得到地域数据长度和位置信息
var buff[SegmentIndexSize]
for l << h {
  // 得到中间的索引项
  var m = (l + h) >> 1
  // 计算中间索引项的指针地址
  // 在起始地址 sPtr 上加上 m 的相对地址即可
  var p = sPtr + m * SegmentIndexSize
  // 从 p 位置开始读取 SegmentIndexSize = 14 个字节到 buff
  // 得到一个完整的上述描述的二分索引项，不过为了减少不必要的操作
  // 我们是按需要解码，此处 buff 为 p 开始的 14 个 byte 的数据
  read(p, buff)
   // 前面 4 个字节是起始 IP
  var sip = getInt(buff, 0) 
  if ip_int < sip {
      // 目标比 sip 小，也就是在 p 位置的左边
      h = m - 1
  } else {
      // 4 ~ 7 之间的 4 个字节是结束 IP 地址
      var eip = getInt(buff, 4)
      if ip_int > eip {
          // 目标 ip 比 eip 大，也就是在 p 位置的右边
          l = m + 1
      } else {
          // 搜索命中
          // 目标 ip 正好在 sip 和 eip 之间
          // 也就是找到目标 ip 了
          // 8 ~ 10 两个字节是地域数据的长度
          dataLen = getShort(buff, 8)
          // 10 ~ 13 的 4 个字节是地域数据的地址
          dataPtr = getInt(buff, 10)
      }
  }
}

```

    经过上面的搜索过程之后，我们会得到地域数据的长度 dataLen 和地址 dataPtr。

第四步：获取地域信息并完成搜索  

    通过第三步得到的 dataLen 和 dataPtr 我们就可以轻松的得到地域数据了，从 dataPtr 位置读取 dataLen 个字节就是地域数据，具体代码如下：  

```
// 将文件指针移动到 dataPtr
handle.seek(dataPtr)
// 当前位置读取 dataLen 个字节
var buff[dataLen]
handle.read(buff)
// 把 buff 解码成 utf-8 字符串返回
return string(buff, "utf-8")

```

    到此整个 xdb 的结构以及查询过程就讲解完成了，一开始看结构分析的时候不一定会完全理解，再结合后面的查询逻辑就可以比较清晰的明白整个 xdb 的结构了，有问题欢迎 ip2region 技术交流群讨论，至于 xdb 的生成过程分析会下一个篇章介绍。  

END  

  

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/dkwuWwLoRK9CibiaQfOcCloBxyJQyeIl8tdbboS6EcmeZ1XJamPhfAgbXDMLJ11XItCSnTE7eFHWcnBrBzTftj8A/640?wx_fmt=jpeg)

* * *

  

这里有最新开源资讯、软件更新、技术干货等内容

点这里 ↓↓↓ 记得 关注✔ 标星⭐ 哦~