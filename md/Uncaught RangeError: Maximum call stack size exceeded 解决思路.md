> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/telwanggs/p/5717653.html)

今天突然碰到这样的 JavaScript 错误：Uncaught RangeError: Maximum call stack size exceeded

这个翻译过来就是堆栈溢出了。

1、原因：有小类到大类的递归查询导致溢出

2、解决方法思想：

A、在做递归查询时候由大类到小类去查询

B、匹配结果后及时 return 退出，防止过多查询

代码是商业机密就不贴出来了