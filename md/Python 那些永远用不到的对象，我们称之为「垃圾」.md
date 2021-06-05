> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTU5OTI4MA==&mid=2247499975&idx=1&sn=884e5174c0875e6d7995e3b795926e70&chksm=fbafe198ccd8688e910de598a672dafe160194bf3457f94b7e4d5ca135ccfbd9ae47886e26b0&mpshare=1&scene=1&srcid=06050Q7R9rvnyMAQKT8MPHHO&sharer_sharetime=1622870157269&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

  

我们知道使用 Python 可以创建对象，当我们去引用它的时候，系统会开辟一个内存空间存放对象，不过可能有些对象我们用完之后，永远再也不会去使用了，那这对象不能一直留在内存里边吧，对象已经废了，也就成为了「垃圾」，垃圾要清理掉，内存才能腾出位置给别的程序使用。

那么：

Python是怎么回收垃圾的？
---------------

### 先来了解一下「引用次数」，在 Python 的内置模块 sys 有一个 getrefcount 方法，通过它我们可以得到对象被引用的次数：

![图片](https://mmbiz.qpic.cn/mmbiz_png/J2icnQspGlaKqS0EK7CrCAnBNeTmEOSpX46Ybm91Bdjn1qn4FHW6sHl2XJOdiaynHaEt3Pw0lAib7sib124TIWuEww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

比如我们定义这样一个「s」：

![图片](https://mmbiz.qpic.cn/mmbiz_png/J2icnQspGlaKqS0EK7CrCAnBNeTmEOSpXVLoIAus2dFQmic7k5GLn86cMiafyNblbm7micLHhvS3LLKf6W3AiaD82ww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里得到的结果为 2 次引用，其中一次是 s，一次是 getrefcount。

接下来我们看看这样的代码：

![图片](https://mmbiz.qpic.cn/mmbiz_png/J2icnQspGlaKqS0EK7CrCAnBNeTmEOSpX8a9yt6PHxec3SDYdGBLQYyhkjic9HkuLxmhWL0L8aHTKjutZyfAa1lA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当我们执行完 fxxk 方法之后，在它下面的 print 调用会报错，也就是说我们无法再去引用对象 s 了，因为它已经被回收。

当执行完 fxxk() 之后，对象 s 的引用数量为 0，而在 Python 的垃圾回收算法中，一个主要的点就是，当对象的引用数量为 0，说明这个对象已经成为「垃圾」了，Python 会将这个对象回收掉，从而释放资源。

通过统计引用次数来释放资源，是相对高效可行的，不过也有存在这样的现象：对象之间相**互循环引用**，会导致引用数量为一直不为 0，那么这样的垃圾是回收不了的，这就可能会造成内存泄漏。

所以在 Python 新版本中，补充了垃圾回收机制算法——**标记清除法和分代收集。**

所谓标记清除，就是遍历所有对象，通过链表逐个对象追踪标记到的这些对象是可达的，那么剩下那些对象就是不可达的，说明它们是垃圾，回收掉，这样就可以避免对象循环引用而没办法回收的问题了。

因为每次标记清除的时候，肯定是占用系统资源的，这时候就有人想到，是不是可以分代收集，也就是说，分成三代，把第一次遍历的对象视为第 0 代，那么第一次遍历完活下来的对象，就把它们放入第二代，第二代就不会被那么「严格」的去扫描，如果第二次遍历，第二代的对象又存活，那么就放入第三代，在第三代里面的对象就更「安全」了。

因为对象活得越久，说明它越不是「垃圾」。

以上，我们说的 Python 垃圾回收机制，都是自动的，不用我们亲自来清理垃圾。

那如果我们想自己手动来回收垃圾，可不可以实现呢？

答案是肯定的。

Python手动回收垃圾
------------

我们可以通过 del 命令来删除对象的引用，然后使用 Python 的 gc 模块，调用 collect 方法就可以实现。

现在来写个例子给你参考一下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/J2icnQspGlaKqS0EK7CrCAnBNeTmEOSpXPZiaib2IQ6pbmIVBmoDEfXBOPRibbplYwNkNk2giaNXWErribqqwDcYdq1Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

代码很简单，这里我们创建了 handsomeb 对象，见一个女生爱一个。。。

主要的是最后两行代码，当我们使用 del handsomeb 的时候，实际上就是将 handsomb 这个对象给删除释放掉，不过这个时候还存在那些和 handsomeb 的关联对象 Girl 们，她们已经没有什么用了，我们可以使用 gc.collect()，将和 handsomeb 关联的对象们都给释放掉，这样就实现了手动回收垃圾。

你还可以到这里了解更多相关内容：

https://docs.python.org/zh-cn/3.8/library/gc.html

ok，以上就是小帅b今天给你带来的分享，关于 Python 的垃圾回收问题，面试也常常会被问到，希望对你有帮助。