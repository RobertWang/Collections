> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [linus.blog.csdn.net](https://linus.blog.csdn.net/article/details/117433052)

> 今天一个读者朋友给我发的一段代码，这段代码让他有了疑惑。代码如下：#include"stdio.h"intmain(){typedefunion{...

今天一个读者朋友给我发的一段代码，这段代码让他有了疑惑。

代码如下：

他的几个测试代码以及输出

![](https://img-blog.csdnimg.cn/img_convert/00bc182c9142ca21da767c3f3e394bd9.png) ![](https://img-blog.csdnimg.cn/img_convert/b86a0dbd8c71b533121834ecacf84971.png) ![](https://img-blog.csdnimg.cn/img_convert/6f4ad5718c771978d4d40ccd33431248.png)

这里说一个问题，**我们从 printf 上看到的不一定我们想看到的，所以我们需要去变量的内存地址一探究竟，一定要了解内存的布局，对内存有所了解。**

上面注释的代码，在我的电脑中运行的结果不相同，所以要看 printf 的准确输出，应该初始化变量`a`。

使用 gdb 来查看地址，可以准确看到变量内存中的数据。

![](https://img-blog.csdnimg.cn/img_convert/7d65058cb492c5debabdf4492ea003da.png) ![](https://img-blog.csdnimg.cn/img_convert/bfb81fd9a2f741d84984cbcc56c43b5d.png) ![](https://img-blog.csdnimg.cn/img_convert/2c3bcd6c5dc0d9dd39061bd33c398546.png)

这个问题在之前的文章说过，这里再重新提一下

*   **大端模式（Big-endian）**，是指数据的高字节，保存在内存的低地址中，而数据的低字节，保存在内存的高地址中
    
*   **小端模式（Little-endian）**，是指数据的高字节保存在内存的高地址中，而数据的低字节保存在内存的低地址中
    

我们用这个再来看看我们的程序

`j[0]`在低地址，`j[1]`在高地址，这个没有什么意见吧？

内存就是一个尺子????，它是不断变长的，所以这个地址也是慢慢变大的，没有任何问题吧。

![](https://img-blog.csdnimg.cn/img_convert/356b5b3f6688606d5982f7dacce0b3d1.png)

然后，我们可以看看现在的输出，从上面的输出可以看到输出`100`，也就是`j[1]`在高地址，`j[0]`在低地址，那这个计算机就是`小端模式`。

也可以通过查看内存地址来确认

![](https://img-blog.csdnimg.cn/img_convert/7f6a9b2d235d5ef5a70c73251b2048a8.png)

[计算机验证大小端](https://mp.weixin.qq.com/s?__biz=MzA5NTM3MjIxMw%3D%3D&chksm=90411dd4a73694c2e2605d1d3a23fa99418c8f2520cf1126de93362ac1e4c4aabaa796202ca1&idx=1&lang=zh_CN&mid=2247486222&scene=21&sn=3d597bd3fd6c2cdc0124421efbdba0db&token=448152620#wechat_redirect)

* * *