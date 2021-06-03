> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwNjMxMTA5Mw==&mid=2651346577&idx=1&sn=960cd8d8c4d03b4def324ea9ec01b311&chksm=80f3ab4bb784225daed9943f24691e0d1d71173ac9700f68c77f66f64b162c75c905212bd196&scene=21#wechat_redirect)

计算机起源于美国，上个世纪，他们对英语字符与二进制位之间的关系做了统一规定，并制定了一套字符编码规则，这套编码规则被称为 ASCII 编码

由于字符编码不同，计算机在不同国家之间的交流变得很困难，经常会出现乱码的问题，比如：对于同一个二进制数据，不同的编码会解析出不同的字符

### Unicode 简介

Unicode 是国际标准字符集，它将世界各种语言的每个字符定义一个唯一的编码，以满足跨语言、跨平台的文本信息转换

Unicode 字符集的编码范围是 0x0000 - 0x10FFFF , 可以容纳一百多万个字符， 每个字符都有一个独一无二的编码，也即每个字符都有一个二进制数值和它对应，这里的二进制数值也叫 码点 , 比如：汉字 "中" 的 码点是 0x4E2D, 大写字母 A 的码点是 0x41, 具体字符对应的 Unicode 编码可以查询 Unicode 字符编码表

### 字符集和字符编码

字符集是很多个字符的集合，例如 GB2312 是简体中文的字符集，它收录了六千多个常用的简体汉字及一些符号，数字，拼音等字符

字符编码是 字符集的一种实现方式，把字符集中的字符映射为特定的字节或字节序列，它是一种规则

比如：Unicode 只是字符集，UTF-8、UTF-16、UTF-32 才是真正的字符编码规则

### Unicode 字符存储

Unicode 是一个符号集， 它只规定了每个符号的二进制值，但是符号具体如何存储它并没有规定

前面提到, Unicode 字符集的编码范围是 0x0000 - 0x10FFFF，因此需要 1 到 3 个字节来表示

那么，对于三个字节的 Unicode 字符，计算机怎么知道它表示的是一个字符而不是三个字符呢 ？

如果所有字符都用三个字节表示，那么对于那些一个字节就能表示的字符来说，有两个字节是无意义的，对于存储来说，这是极大的浪费，假如 , 一个普通的文本, 大部分字符都只需一个字节就能表示，现在如果需要三个字节才能表示，文本的大小会大出三倍左右

因此，Unicode 出现了多种存储方式，常见的有 UTF-8、UTF-16、UTF-32，它们分别用不同的二进制格式来表示 Unicode 字符

UTF-8、UTF-16、UTF-32 中的 "UTF" 是 "Unicode Transformation Format" 的缩写，意思是 "Unicode 转换格式"，后面的数 字表明至少使用多少个比特位来存储字符, 比如：UTF-8 最少需要 8 个比特位也就是一个字节来存储，对应的， UTF-16 和 UTF-32 分别需要最少 2 个字节 和 4 个字节来存储

### UTF-8 编码

UTF-8: 是一种变长字符编码，被定义为将码点编码为 1 至 4 个字节，具体取决于码点数值中有效二进制位的数量

UTF-8 的编码规则:

0.  对于单字节的符号，字节的第一位设为 0，后面 7 位为这个符号的 Unicode 码。因此对于英语字母，UTF-8 编码和 ASCII 码是相同的, 所以 UTF-8 能兼容 ASCII 编码，这也是互联网普遍采用 UTF-8 的原因之一
    

2.  对于 n 字节的符号（ n > 1），第一个字节的前 n 位都设为 1，第 n + 1 位设为 0，后面字节的前两位一律设为 10 。剩下的没有提及的二进制位，全部为这个符号的 Unicode 码
    

下表是 Unicode 编码对应 UTF-8 需要的字节数量以及编码格式

<table data-source-line="69" width="NaN"><thead><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">Unicode 编码范围 (16 进制)</th><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">UTF-8 编码方式 (二进制)</th></tr></thead><tbody><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">000000 - 00007F</td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)" data-style="color: red; font-size: 16px; box-sizing: border-box;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">0xxxxxxx&nbsp;<span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)" data-style="color: red; box-sizing: border-box;">ASCII 码</span></span></span></td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">000080 - 0007FF</td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">110xxxxx&nbsp;<span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">10xxxxxx</span></span></span></span></td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">000800 - 00FFFF</td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)" data-style="color: red; font-size: 16px; box-sizing: border-box;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">1110xxxx&nbsp;<span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)" data-style="color: red; font-size: 16px; box-sizing: border-box;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">10xxxxxx&nbsp;<span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)" data-style="color: red; font-size: 16px; box-sizing: border-box;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">10xxxxxx</span></span></span></span></span></span></td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">01 0000 - 10 FFFF</td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">11110xxx&nbsp;<span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">10xxxxxx&nbsp;<span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">10xxxxxx&nbsp;<span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">10xxxxxx</span></span></span></span></span></span></span></span></td></tr></tbody></table>

表格中第一列是 Unicode 编码的范围，第二列是对应 UTF-8 编码方式，其中红色的二进制 "1" 和 "0" 是固定的前缀, 字母 x 表示可用编码的二进制位

根据上面表格，要解析 UTF-8 编码就很简单了，如果一个字节第一位是 0 ，则这个字节就是一个单独的字符，如果第一位是 1 ，则连续有多少个 1 ，就表示当前字符占用多少个字节

下面以 "中" 字 为例来说明 UTF-8 的编码，具体的步骤如下图， 为了便于说明，图中左边加了 `1，2，3，4` 的步骤编号

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmkErWEOtFlvP4ABPVvBhrNGYknEUy5xtyEZneBdn8hibGu6BsrIj6ncqRP3bJNAZhicyvWCqV16vxfA/640?wx_fmt=png)

首先查询 "中" 字的 Unicode 码 0x4E2D, 转成二进制, 总共有 16 个二进制位， 具体如上图 步骤 1 所示

通过前面的 Unicode 编码和 UTF-8 编码的表格知道，Unicode 码 0x4E2D 对应 000800 - 00FFFF 的范围，所以, "中" 字的 UTF-8 编码 需要 3 个字节，即格式是 1110xxxx 10xxxxxx 10xxxxxx

然后从 "中" 字的最后一个二进制位开始，按照从后向前的顺序依次填入格式中的 x 字符，多出的二进制补为 0， 具体如上图 步骤 2、步骤 3 所示

于是，就得到了 "中" 的 UTF-8 编码是 11100100 10111000 10101101, 转换成十六进制就是 0xE4B8AD， 具体如上图 步骤 4 所示

### UTF-16 编码

UTF-16 也是一种变长字符编码, 这种编码方式比较特殊, 它将字符编码成 2 字节 或者 4 字节

具体的编码规则如下:

0.  对于 Unicode 码小于 0x10000 的字符， 使用 2 个字节存储，并且是直接存储 Unicode 码，不用进行编码转换
    

2.  对于 Unicode 码在 0x10000 和 0x10FFFF 之间的字符，使用 4 个字节存储，这 4 个字节分成前后两部分，每个部分各两个字节，其中，前面两个字节的前 6 位二进制固定为 110110，后面两个字节的前 6 位二进制固定为 110111, 前后部分各剩余 10 位二进制表示符号的 Unicode 码 减去 0x10000 的结果
    

3.  大于 0x10FFFF 的 Unicode 码无法用 UTF-16 编码
    

下表是 Unicode 编码对应 UTF-16 编码格式

<table data-source-line="116" width="NaN"><thead><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">Unicode 编码范围 (16 进制)</th><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">具体 Unicode 码 (二进制)</th><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">UTF-16 编码方式 (二进制)</th><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">字节</th></tr></thead><tbody><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">0000 0000 - 0000 FFFF</td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">xxxxxxxx xxxxxxxx</td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">xxxxxxxx xxxxxxxx</td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">2</td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">0001 0000 - 0010 FFFF</td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">yy yyyyyyyy xx xxxxxxxx</td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">110110yy yyyyyyyy&nbsp;<span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">110111xx xxxxxxxx</span></span></span></span></td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">4</td></tr></tbody></table>

表格中第一列是 Unicode 编码的范围，第二列是 具体 Unicode 码的二进制 ( 第二行的第二列表示的是 Unicode 码 减去 0x10000 后的二进制 ) , 第三列是对应 UTF-16 编码方式，其中红色的二进制 "1" 和 "0" 是固定的前缀, 字母 x 和 y 表示可用编码的二进制位， 第四列表示 编码占用的字节数

前面提到过，"中" 字的 Unicode 码是 4E2D, 它小于 0x10000，根据表格可知，它的 UTF-16 编码占两个字节，并且和 Unicode 码相同，所以 "中" 字的 UTF-16 编码为 4E2D

我从 Unicode 字符表网站 找了一个老的南阿拉伯字母, 它的 Unicode 码是: 0x10A6F , 可以访问 https://unicode-table.com/cn/10A6F/ 查看字符的说明, Unicode 码对应的字符如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmkErWEOtFlvP4ABPVvBhrNGbGnG7HtFuZSWfhG1iauGWr1icVq2UuWPJEIDKaBOWscqiayf91FFQ6CfA/640?wx_fmt=png)

下面以这个 老的南阿拉伯字母的 Unicode 码 0x10A6F 为例来说明 UTF-16 4 字节的编码，具体步骤如下，为了便于说明，图中左边加了 `1，2，3，4 、5`的步骤编号

![](https://mmbiz.qpic.cn/mmbiz_jpg/a2ibvjLwErmkErWEOtFlvP4ABPVvBhrNG7DxqFrCmfpmDnadC5kAWsbcDHSWQhkNtOJ3Vicxicq2TWuGPMl7O5Zibw/640?wx_fmt=jpeg)

首先把 Unicode 码 0x10A6F 转成二进制, 对应上图的 步骤 1

然后把 Unicode 码 0x10A6F 减去 0x10000, 结果为 0xA6F 并把这个值转成二进制 00 00000010 10 01101111，对应上图的 步骤 2

然后 从二进制 00 00000010 10 01101111 的最后一个二进制为开始，按照从后向前的顺序依次填入格式中的 x 和 y 字符，多出的二进制补为 0， 对应上图的 步骤 3、 步骤 4

于是，就计算出了 Unicode 码 0x10A6F 的 UTF-16 编码是 11011000 00000010 11011110 01101111 , 转换成十六进制就是 0xD802DE6F， 对应上图的 步骤 5

### UTF-32 编码

UTF-32 是固定长度的编码，始终占用 4 个字节，足以容纳所有的 Unicode 字符，所以直接存储 Unicode 码即可，不需要任何编码转换。虽然浪费了空间，但提高了效率。

### UTF-8、UTF-16、UTF-32 之间如何转换

前面介绍过，UTF-8、UTF-16、UTF-32 是 Unicode 码表示成不同的二进制格式的编码规则，同样，通过这三种编码的二进制表示，也能获得对应的 Unicode 码，有了字符的 Unicode 码，按照上面介绍的 UTF-8、UTF-16、UTF-32 的编码方法 就能转换成任一种编码了

### UTF 字节序

最小编码单元是多字节才会有字节序的问题存在，UTF-8 最小编码单元是一字节，所以 它是没有字节序的问题，UTF-16 最小编码单元是 2 个字节，在解析一个 UTF-16 字符之前，需要知道每个编码单元的字节序

比如：前面提到过，"中" 字的 Unicode 码是 4E2D, "ⵎ" 字符的 Unicode 码是 2D4E， 当我们收到一个 UTF-16 字节流 4E2D 时，计算机如何识别它表示的是字符 "中" 还是 字符 "ⵎ" 呢 ?

所以，对于多字节的编码单元，需要有一个标记显式的告诉计算机，按照什么样的顺序解析字符，也就是字节序，字节序分为 大端字节序 和 小端字节序

下面以 0x4E2D 为例来说明大端和小端，具体参见下图:

数据是从高位字节到低位字节显示的，这也更符合人们阅读数据的习惯，而内存地址是从低地址向高地址增加

所以，字符 0x4E2D 数据的高位字节是 4E，低位字节是 2D

按照大端字节序的高位字节保存内存低地址端的规则，4E 保存到低内存地址 0x10001 上，2D 则保存到高内存地址 0x10002 上

对于小端字节序，则正好相反，数据的高位字节保存到内存的高地址端，低位字节保存到内存低地址端的，所以 4E 保存到高内存地址 0x10002 上，2D 则保存到低内存地址 0x10001 上

### BOM

BOM 是 byte-order mark 的缩写，是 "字节序标记" 的意思, 它常被用来当做标识文件是以 UTF-8、UTF-16 或 UTF-32 编码的标记

在 Unicode 编码中有一个叫做 "零宽度非换行空格" 的字符 (ZERO WIDTH NO-BREAK SPACE), 用字符 FEFF 来表示

对于 UTF-16 ，如果接收到以 FEFF 开头的字节流， 就表明是大端字节序，如果接收到 FFFE， 就表明字节流 是小端字节序

UTF-8 没有字节序问题，上述字符只是用来标识它是 UTF-8 文件，而不是用来说明字节顺序的。"零宽度非换行空格" 字符 的 UTF-8 编码是 EF BB BF, 所以如果接收到以 EF BB BF 开头的字节流，就知道这是 UTF-8 文件

下面的表格列出了不同 UTF 格式的固定文件头

<table data-source-line="199" width="NaN"><thead><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">UTF 编码</th><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">固定文件头</th></tr></thead><tbody><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">UTF-8</td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">EF BB BF</td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">UTF-16LE</td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">FF FE</td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">UTF-16BE</td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">FE FF</td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">UTF-32LE</td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">FF FE 00 00</td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">UTF-32BE</td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">00 00 FE FF</td></tr></tbody></table>

根据上面的 固定文件头，下面列出了 "中" 字在文件中的存储 (包含文件头)

<table data-source-line="213" width="NaN"><thead><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">编码</th><th data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-top-width: 1px; border-color: rgb(223, 226, 229); word-break: break-all; text-align: left;">固定文件头</th></tr></thead><tbody><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)">Unicode 编码</span></td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;">0X004E2D</td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)">UTF-8</span></td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">EF BB BF&nbsp;4E 2D</span></span></td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)">UTF-16BE</span></td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)" data-style="color: red; box-sizing: border-box;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">FE FF&nbsp;4E 2D</span></span></td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)">UTF-16LE</span></td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">FF FE&nbsp;2D 4E</span></span></td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; background-color: rgb(255, 255, 255); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)">UTF-32BE</span></td><td data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)" data-style="color: red; box-sizing: border-box;"><span data-darkmode-bgcolor-16227015820379="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(255, 255, 255)" data-darkmode-color-16227015820379="rgb(255, 23, 0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">00 00 FE FF&nbsp;00 00 4E 2D</span></span></td></tr><tr data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; background-color: rgb(246, 248, 250); border-top: 1px solid rgb(198, 203, 209);"><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)">UTF-32LE</span></td><td data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-style="box-sizing: border-box; padding: 6px 13px; border-color: rgb(223, 226, 229); word-break: break-all;"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)"><span data-darkmode-bgcolor-16227015820379="rgb(189, 190, 192)" data-darkmode-original-bgcolor-16227015820379="#fff|rgb(246, 248, 250)" data-darkmode-color-16227015820379="rgb(255,0,0)" data-darkmode-original-color-16227015820379="#fff|rgb(255,0,0)">FF FE 00 00&nbsp;2D 4E 00 00</span></span></td></tr></tbody></table>

### 常见的字符编码的问题

*   Redis 中文 key 的显示
    

有时候我们需要向 redis 中写入含有中文的数据，然后在查看数据，但是会看到一些其他的字符，而不是我们写入的中文

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmkErWEOtFlvP4ABPVvBhrNG2VoHpfMog6Lq5q2CJ7NTaPD9iaP4hYANmSdtgC9VuBltFBN3UHAd0AA/640?wx_fmt=png)

上图中，我们向 redis 写入了一个 "中" 字，通过 get 命令查看的时候无法显示我们写入的 "中" 字

这时候加一个 --raw 参数，重新启动 redis-cli 即可，也即 执行 redis-cli --raw 命令启动 redis 客户端，具体的如下图所示

*   MySQL 中的 utf8 和 utf8mb4
    

```
mysql> show create table test\G
*************************** 1. row ***************************
       Table: test
Create Table: CREATE TABLE `test` (
  `name` char(32) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)


```

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmkErWEOtFlvP4ABPVvBhrNGSZoaIwDZrzs8WllZibfC7OJgoa8bpZsXaNloQMgUPBRPH1sok3sa3MA/640?wx_fmt=png)

从上图可以看出，插入 "中" 字 成功，插入 0x10A6F 字符失败，错误提示无效的字符串，\xF0\X90\XA9\xAF 正是 0x10A6F 字符的 UTF-8 编码，占用 4 个字节, 因为 MySQL 的 utf8 编码最多只支持 3 个字节，所以插入会失败

把 `test` 表的字符集改成 `utf8mb4` , 排序规则 改成 `utf8bm4_unicode_ci`, 具体如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmkErWEOtFlvP4ABPVvBhrNGndFwD1A441vhc8vyxgXJObiat3ueda7LcniaAxegjnLZTJF3l9vibicRdg/640?wx_fmt=png)

字符集和排序方式修改之后，再次插入 0x10A6F 字符， 结果是成功的，具体执行结果如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/a2ibvjLwErmkErWEOtFlvP4ABPVvBhrNGQrAqzshHPric9F3wqHmxGKskwF99ClscjkmgNmAiahWr1iaia2UznLz5MA/640?wx_fmt=png)

上图中，`set names utf8mb4` 是为了测试方便，临时修改当前会话的字符集，以便保持和 服务器一致，实际解决这个问题需要修改 `my.cnf` 配置中 服务器和客户端的字符集

### 小结

本文从字符编码的历史介绍了 Unicode 出现的原因，接着介绍了 Unicode 字符集中 三种不同的编码方式：UTF-8、UTF-16、UTF-32 以及它们的的编码方法，紧接着介绍了 字节序、BOM ，最后讲到了字符集在 MySQL 和 Redis 应用中常见的问题以及解决方案 ，更多关于 Unicode 的介绍请参考 Unicode 的 RFC 文档