> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650807139&idx=5&sn=e5f2f0cf90cf8a642c19b738bd9842c7&chksm=bd018e2d8a76073bbb9e9730e7fc60a8b2064d04db411b7d51b6cf61a53a45bbd90fee916afa&mpshare=1&scene=1&srcid=0625qtGpr8h8PgiYK427vEM8&sharer_sharetime=1624561921725&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

译者 | 弯月  

于是，我看到了下面这个问题：怎样将字节数输出成人类可读的格式？也就是说，怎样将 123,456,789 字节输出成 123.5MB？

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4qHXibUicmuttH3p0nVibHPrOJrpb7q89MicMlYfiaw83eC1tAyT15rPibV0RWoibxZ6qyv0icVibl8wBDsw/640?wx_fmt=png)

隐含的条件是，结果字符串应当在 1~999.9 的范围内，后面跟一个适当的表示单位的后缀。

这个问题已经有一个答案了，代码是用循环写的。基本思路很简单：尝试所有尺度，从最大的 EB（10^18 字节）开始直到最小的 B（1 字节），然后选择小于字节数的第一个尺度。用伪代码来表示的话大致如下：

```
suffixes   = [ "EB", "PB", "TB", "GB", "MB", "kB", "B" ]magnitudes = [ 1018, 1015, 1012, 109, 106, 103, 100 ]i = 0while (i < magnitudes.length && magnitudes[i] > byteCount)i++printf("%.1f %s", byteCount / magnitudes[i], suffixes[i])

```

通常，如果一个问题已经有了正确答案，并且有人赞过，别的回答就很难赶超了。不过这个答案有一些问题，所以我依然有机会超过它。至少，循环还有很大的清理空间。

  

1

**这只是一个代数问题！**  

=================

然后我就想到，kB、MB、GB…… 等后缀只不过是 1000 的幂（或者在 IEC 标准下是 1024 的幂），也就是说不需要使用循环，完全可以使用对数来计算正确的后缀。

根据这个想法，我写出了下面的答案：

```
public static String humanReadableByteCount(long bytes, boolean si) {int unit = si ? 1000 : 1024;if (bytes < unit) return bytes + " B";int exp = (int) (Math.log(bytes) / Math.log(unit));String pre = (si ? "kMGTPE" : "KMGTPE").charAt(exp-1) + (si ? "" : "i");return String.format("%.1f %sB", bytes / Math.pow(unit, exp), pre);}

```

当然，这段代码并不是太好理解，而且 log 和 pow 的组合的效率也不高。但我没有使用循环，而且没有任何分支，看起来很干净。

这段代码的数学原理很简单。字节数表示为 byteCount = 1000^s，其中 s 表示尺度。（对于二进制记法则使用 1024 为底。）求解 s 可得 s = log_1000(byteCount)。

API 并没有提供 log_1000，但我们可以用自然对数表示为 s = log(byteCount) / log(1000)。然后对 s 向下取整（强制转换为 int），这样对于大于 1MB 但不足 1GB 的都可以用 MB 来表示。

此时如果 s=1，尺度就是 kB，如果 s=2，尺度就是 MB，以此类推。然后将 byteCount 除以 1000^s，并找出正确的后缀。

接下来，我就等着社区的反馈了。我并不知道这段代码后来成了被复制粘贴最多的代码。

2

**对于贡献的研究**  

==============

到了 2018 年，一位名叫 Sebastian Baltes 的博士生在《Empirical Software Engineering》杂志上发表了一篇论文，题为《Usage and Attribution of Stack Overflow Code Snippets in GitHub Projects》。该论文的主旨可以概括成一点：人们是否在遵守 Stack Overflow 的 CC BY-SA 3.0 授权？也就是说，当人们从 Stack Overflow 上复制粘贴时，会怎么注明来源？

作为分析的一部分，他们从 Stack Overflow 的数据转出中提取了代码片段，并与公开的 GitHub 代码库中的代码进行匹配。论文摘要如是说：

> We present results of a large-scale empirical study analyzing the usage and attribution of non-trivial Java code snippets from SO answers in public GitHub (GH) projects.  

（本文对于在公开的 GitHub 项目中使用来自 Stack Overflow 上有价值的代码片段的情况以及来源注明情况进行了大规模的经验分析，并给出了结果。）  

（剧透：绝大多数人并不会注明来源。）

论文中有这样一张表格：

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4qHXibUicmuttH3p0nVibHPr23ZialhNp5k31qJmLicBAMZp5U3rgEaWgx8pfgvx2y0BMFcrqZNrVs0Q/640?wx_fmt=png)

id 为 3758880 的答案正是我八年前贴出的答案。此时该答案已经被阅读了几十万次，拥有上千个赞。

在 GitHub 上随便搜索一下就能找到数千个 humanReadableByteCount 函数：

![](https://mmbiz.qpic.cn/mmbiz_png/Pn4Sm0RsAuj4qHXibUicmuttH3p0nVibHPrb17tn5esBh9U6V3DgZuC1xhwtcRe8UQtob0wKicmekZmkVWXV0BMofA/640?wx_fmt=png)

你可以用下面的命令看看自己有没有无意中用到：

```
$ git grep humanReadableByteCount

```

3

**问题**  

=========

你肯定在想：这段代码有什么问题：

再来看一次：

```
public static String humanReadableByteCount(long bytes, boolean si) {int unit = si ? 1000 : 1024;if (bytes < unit) return bytes + " B";int exp = (int) (Math.log(bytes) / Math.log(unit));String pre = (si ? "kMGTPE" : "KMGTPE").charAt(exp-1) + (si ? "" : "i");return String.format("%.1f %sB", bytes / Math.pow(unit, exp), pre);}

```

在 EB（10^18）之后是 ZB（10^21）。是不是因为 kMGTPE 字符串的越界问题？并不是。long 的最大值为 2^63-1，大约是 9.2x10^18，所以 long 绝对不会超过 EB。

是不是 SI 和二进制的混合问题？不是。前一个版本的确有这个问题，不过很快就修复了。

是不是因为 exp 为 0 会导致 charAt(exp-1) 出错？也不是。第一个 if 语句已经处理了该情况。exp 值至少为 1。

是不是一些奇怪的舍入问题？对了……

4

**许多 9**  

===========

这段代码在 1MB 之前都非常正确。但当输入为 999,999 时，它（在 SI 模式下）会给出 “1000.0 kB”。尽管 999,999 与 1,000x1000^1 的距离比与 999.9x1000^1 的距离更小，但根据问题的定义，有效数字部分的 1,000 是不正确的。正确结果应为 "1.0 MB"。

据我所知，原帖下的所有 22 个答案（包括一个使用 Apache Commons 和 Android 库的答案）都有这个问题（或至少是类似的问题）。

那么怎样修复呢？首先，我们注意到指数（exp）应该在字节数接近 1x1,000^2（1MB）时，将返回结果从 k 改成 M，而不是在字节数接近 999.9x1000^1（999.9k）时。这个点上的字节数为 999,950。类似地，在超过 999,950,000 时应该从 M 改成 G，以此类推。

为了实现这一点，我们应该计算该阈值，并当 bytes 大于阈值时增加 exp 的结果。（对于二进制的情况，由于阈值不再是整数，因此需要使用 ceil 进行向上取整）。

```
if (bytes >= Math.ceil(Math.pow(unit, exp) * (unit - 0.05)))exp++;

```

5

**更多的 9**  

============

但是，当输入为 999,949,999,999,999,999 时，结果为 1000.0 PB，而正确的结果为 999.9 PB。从数学上来看这段代码是正确的，那么问题除在何处？

此时我们已经达到了 double 类型的精度上限。

**关于浮点数运算**
-----------

根据 IEEE 754 的浮点数表示方法，接近 0 的数字非常稠密，而很大的数字非常稀疏。实际上，超过一半的值位于 - 1 和 1 之间，而且像 Long.MAX_VALUE 如此大的数字对于双精度来说没有任何意义。用代码来表示就是  

```
double a = Double.MAX_VALUE;double b = a - Long.MAX_VALUE;System.err.println(a == b);  // prints true

```

有两个计算是有问题的：

*   String.format 参数中的触发
    
*   对 exp 的结果加一时的阈值
    

当然，改成 BigDecimal 就行了，但这有什么意思呢？而且改成 BigDecimal 代码也会变得更乱，因为标准 API 没有 BigDecimal 的对数函数。

**缩小中间值**
---------

对于第一个问题，我们可以将 bytes 值缩小到精度更好的范围，并相应地调整 exp。由于最终结果总要取整的，所以丢弃最低位有效数字也无所谓。

```
if (exp > 4) {bytes /= unit;exp--;}

```

**调整最低有效比特**
------------

对于第二个问题，我们需要关心最低有效比特（999,949,99...9 和 999,950,00...0 等不同幂次的值），所以需要使用不同的方法解决。

首先注意到，阈值有 12 种不同的情况（每个模式下有六种），只有其中一种有问题。有问题的结果的十六进制表示的末尾为 D00。如果出现这种情况，只需要调整至正确的值即可。

```
long th = (long) Math.ceil(Math.pow(unit, exp) * (unit - 0.05));if (exp < 6 && bytes >= th - ((th & 0xFFF) == 0xD00 ? 51 : 0))exp++;

```

由于需要依赖于浮点数结果中的特定比特模式，所以需要使用 strictfp 来保证它在任何硬件上都能运行正确。

6

**负输入**  

==========

尽管还不清楚什么情况下会用到负的字节数，但由于 Java 并没有无符号的 long，所以最好处理复数。现在，-10,000 会产生 - 10000 B。

引入 absBytes 变量：

```
long absBytes = bytes == Long.MIN_VALUE ? Long.MAX_VALUE : Math.abs(bytes);

```

表达式如此复杂，是因为 - Long.MIN_VLAUE == LONG.MIN_VALUE。以后有关 exp 的计算你都要使用 absBytes 来代替 bytes。

7

**最终版本**  

===========

下面是最终版本的代码：

```
// From: https://programming.guide/worlds-most-copied-so-snippet.htmlpublic static strictfp String humanReadableByteCount(long bytes, boolean si) {int unit = si ? 1000 : 1024;long absBytes = bytes == Long.MIN_VALUE ? Long.MAX_VALUE : Math.abs(bytes);if (absBytes < unit) return bytes + " B";int exp = (int) (Math.log(absBytes) / Math.log(unit));long th = (long) Math.ceil(Math.pow(unit, exp) * (unit - 0.05));if (exp < 6 && absBytes >= th - ((th & 0xFFF) == 0xD00 ? 51 : 0)) exp++;String pre = (si ? "kMGTPE" : "KMGTPE").charAt(exp - 1) + (si ? "" : "i");if (exp > 4) {bytes /= unit;exp -= 1;}return String.format("%.1f %sB", bytes / Math.pow(unit, exp), pre);}

```

这个答案最初只是为了避免循环和过多的分支的。讽刺的是，考虑到各种边界情况后，这段代码比原答案还难懂了。我肯定不会在产品中使用这段代码。

8

**总结**  

=========

*   Stack Overflow 上的代码就算有几千个赞也可能有问题。
    
*   要测试所有边界情况，特别是对于从 Stack Overflow 上复制粘贴的代码。
    
*   浮点数运算很难。
    
*   复制代码时一定要注明来源。别人可以据此提醒你重要的事情。
    

原文链接：https://programming.guide/worlds-most-copied-so-snippet.html

声明：本文由 CSDN 翻译，转载请注明来源。