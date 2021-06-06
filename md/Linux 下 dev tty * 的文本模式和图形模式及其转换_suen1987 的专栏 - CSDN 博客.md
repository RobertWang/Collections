> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/suen1987/article/details/10999925)

/dev/tty * 可有两种模式：1. 文本模式（就是只能显示英文）；2. 图形模式（可以显示图像）

怎样从 tty * 的文本模式转换到图形模式呢?  

可以利用

open、ioctl  /dev/tty 调用基本接口来做，但是麻烦。

我们可以利用 SVGAlib 库来做，SVGAlib is a low-level graphics library for Linux. It augments the C programming language, which doesn't provide support for graphics.，[http://www.svgalib.org/](http://www.svgalib.org/) 官网会有介绍，但是你用的 linux 的版不一样可能需要补丁，如 redhat 版就需要去找适合红帽的，ubuntu 就需要找适合 ubuntu 版的。

[http://www.svgalib.org/jay/beginners_guide/beginners_guide.html](http://www.svgalib.org/jay/beginners_guide/beginners_guide.html) 这个是使用说明，根据这个编译安装使用。
