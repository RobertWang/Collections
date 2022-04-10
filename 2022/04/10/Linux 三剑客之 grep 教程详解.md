> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5NDU4Mzg1Mw==&mid=2247484069&idx=1&sn=d95ee0f43dba0c1a12a367b01147281f&chksm=c01c17e7f76b9ef18719a13cbb5cb891fde9a144afda8f8752fef18e6f0ea9f5db02fa914edf&scene=21#wechat_redirect)

Linux 最重要的三个命令在业界被称为**三剑客**，它们是：`awk`、`sed`、`grep`。sed 已经在**[上篇](https://mp.weixin.qq.com/s?__biz=MzU3NzQ4OTE1MA==&mid=2247483780&idx=1&sn=a2c5b8e289b4d6d707dac866fbbc3aa9&scene=21#wechat_redirect)**中讲过，本文要讲的是 `grep` 命令。

我们在使用 Linux 系统中，grep 命令的使用尤为频繁，熟练掌握 grep 的常见用法，能够极大地提高你的工作效率。

grep 命令是一种强大的文本搜索工具，它能使用正则表达式，按照指定的模式去匹配，并把匹配的行打印出来。需要注意的是，grep 只支持匹配而不能替换匹配的内容，替换的功能可以由 sed 来完成。

整体上 grep 还是比较简单的，文中不会详细列举所有的选项和参数，会以多个具体示例来说明 grep 的使用方法和场景，帮助你快速学会 grep 的常见用法。

示例实战
----

废话不说了，直接实战。文章中的示例 需要一个样例文件，文件内容如下：

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDNVgelEAUqOLSBtP9LdL9lwsmfahcbNO6BFxaANMbuia984EvJZmUiaBg/640?wx_fmt=png)

_1._ 把包含 syslog 的行过滤出来

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDhDiaDyicVcBHOsfMCQAicHRwiaarhOUFepXpVDHMVhicEic6gLVV9HTfUVtw/640?wx_fmt=png)

_2._ 把以 ntp 开头的行过滤出来

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDO7Hd5CumtYRTnc9UeiafxYf6tsOybmblVb992icACPicvbNYHlsKfTzbg/640?wx_fmt=png)

_3._ 把匹配 ntp 的行以及下边的两行过滤出来

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDgSTWoicmP9HvKfqOiaIJnt3wgaibH7YckBjW8K05r2wwA2w7iaDRYicBzqQ/640?wx_fmt=png)

_4._ 把包含 syslog 及上边的一行过滤出来

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wD5ZKktibqjdTYIQMP2YYe3ibM9cb6qpjBQpJfBzg6eQAFcmO61TqNBtgA/640?wx_fmt=png)

_5._ 把包含 syslog 以及上、下一行内容过滤出来

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDMicpGtgQDnl65nwcnsgic61l5Ozy1H19Lb9zFHiaYahjxoiaMABnibGXH3w/640?wx_fmt=png)

_6._ 过滤某个关键词，并输出行号

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDxiaKkicfZe3Kou1anQZJgEY5tD3qcjFKPcvU1UwS8hPFqGBHwQZH38Zg/640?wx_fmt=png)

_7._ 过滤**不包含**某关键词，并输出行号

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDmJLkMRh2mobhSCCqyQHbMC0ucN9t3flfhCXHsJujticEPu8nicn7Vghg/640?wx_fmt=png)

_8._ 删除掉空行

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDSUzm2tBVtNvplasZ24ia0icuwuSBNOjZo4shNRnyoJDj3koATmcZLibwA/640?wx_fmt=png)

_9._ 过滤包含 root 或 syslog 的行

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDH22JXVhhS8Bj2pFZQTdMZXgSuAwy7RaaYL3FU8UNvhiaEt7JJLh88sg/640?wx_fmt=png)

_10._ 查看当前目录中包含某关键词的所有文件（这个很有用）

![](https://mmbiz.qpic.cn/mmbiz_png/yVibDjicRT1VvsLOFebIsPXOnTrdW1f7wDOfaOexBic9wUEzd6S9nmwpg01ic1ibEJyGWOKWjClCuiaBzHOB611kZhmg/640?wx_fmt=png)

简单总结
----

通过了一些简单案例操作，我们应该已经熟悉了 grep 的常见用法，下边再来简单总结 grep 的常见选项，相信在实战练习后再来总结应该会有更好的学习效果。

*   `-A`：除了匹配行，额外显示该行之**后**的 N 行
    
*   `-B`：除了匹配行，额外显示该行之**前**的 N 行
    
*   `-C`：除了匹配行，额外显示该行**前后**的 N 行
    
*   `-c`：统计匹配的行数
    
*   `-e`：**实现多个选项间的逻辑 or 关系**
    
*   `-E`：**支持扩展的正则表达式**
    
*   `-F`：相当于 fgrep
    
*   `-i`：**忽略大小写**
    
*   `-n`：显示匹配的行号
    
*   `-o`：仅显示匹配到的字符串
    
*   `-q`：安静模式，不输出任何信息，脚本中常用
    
*   `-s`：不显示错误信息
    
*   `-v`：**显示不被匹配到的行**
    
*   `-w`：显示整个**单词**
    
*   `--color`：以颜色突出显示匹配到的字符串
    

与 grep 相似的工具还有 `egrep`、`fgrep`，实用性并不强，其功能完全可以通过 grep 的扩展参数来实现，所以就不再扩展。