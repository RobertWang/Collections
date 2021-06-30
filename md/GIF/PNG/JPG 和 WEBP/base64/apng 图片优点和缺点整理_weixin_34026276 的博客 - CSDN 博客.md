> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_34026276/article/details/92672958)

　[GIF](http://so.wtoutiao.com/cse/search?s=4457292908549467373&entry=1&q=GIF)/[PNG](http://so.wtoutiao.com/cse/search?s=4457292908549467373&entry=1&q=PNG)/JPG/[WEBP](http://so.wtoutiao.com/cse/search?s=4457292908549467373&entry=1&q=WEBP)/[APNG](http://so.wtoutiao.com/cse/search?s=4457292908549467373&entry=1&q=APNG) 都是属于[位图](http://so.wtoutiao.com/cse/search?s=4457292908549467373&entry=1&q=%E4%BD%8D%E5%9B%BE)（位图 , 务必区别于[矢量图](http://so.wtoutiao.com/cse/search?s=4457292908549467373&entry=1&q=%E7%9F%A2%E9%87%8F%E5%9B%BE)）；

　GIF/PNG 和 JPG 这三种格式的图片被广泛应用在现今的互联网中, gif 曾在过去互联网初期慢速的情况下几乎是做到了大一统的地位, 而现如今随着互联网技术应用和硬件条件的提高,[png](http://so.wtoutiao.com/cse/search?s=4457292908549467373&entry=1&q=png) 和 jpg 格式的图片越来越多的被应用, gif 昔日的辉煌一去不复, [webp](http://so.wtoutiao.com/cse/search?s=4457292908549467373&entry=1&q=webp) 图片格式现在还不普及：

  GIF（Graphics Interchange Format）
==================================

![](http://static.oschina.net/uploads/img/201510/01014530_j1Zm.gif)

　　GIF [图形交换格式](http://so.wtoutiao.com/cse/search?s=4457292908549467373&entry=1&q=%E5%9B%BE%E5%BD%A2%E4%BA%A4%E6%8D%A2%E6%A0%BC%E5%BC%8F)是一种位图图形文件格式, 以 8 位色（即 256 种颜色）重现真彩色的图像。它实际上是一种压缩文档, 采用 LZW 压缩算法进行编码, 有效地减少了图像文件在网络上传输的时间。它是目前广泛应用于网络传输的图像格式之一。

优点

　　1. 优秀的压缩算法使其在一定程度上保证图像质量的同时将体积变得很小。  
　　2. 可插入多帧, 从而实现动画效果。  
　　3. 可设置透明色以产生对象浮现于背景之上的效果。

缺点

　 由于采用了 8 位压缩, 最多只能处理 256 种颜色（2 的 8 次方）, 故不宜应用于真彩图像。

   PNG（Portable Network Graphics）
=================================

　　便携式网络图片（Portable Network Graphics）, 简称 PNG, 是一种无损数据压缩位图图形文件格式。PNG 格式是无损数据压缩的, PNG 格式有 8 位、24 位、32 位三种形式, 其中 8 位 PNG 支持两种不同 的透明形式（索引透明和 alpha 透明）,24 位 PNG 不支持透明, 32 位 PNG 在 24 位基础上增加了 8 位透明通道（32-24===8）, 因此可展现 256 级透明程度。

　　PNG 这种类型的图片就是为了取代 GIF 图片而生的, 除了 GIF 不支持动画的优势, 能用 PNG 的地方就用 PNG, 原因是压缩比高, 色彩好；

![](http://static.oschina.net/uploads/img/201510/01014530_ON55.png)

优点

　　* 支持 256 色调色板技术以产生小体积文件  
　　* 最高支持 48 位真彩色图像以及 16 位灰度图像。  
　　* 支持 Alpha 通道的半透明特性。  
　　* 支持图像亮度的 gamma 校正信息。  
　　* 支持存储附加文本信息, 以保留图像名称、作者、版权、创作时间、注释等信息。  
　　* 使用无损压缩。  
　　* 渐近显示和流式读写, 适合在网络传输中快速显示预览效果后再展示全貌。  
　　* 使用 CRC 循环冗余编码防止文件出错。  
　　* 最新的 PNG 标准允许在一个文件内存储多幅图像。

缺点

　　但也有一些软件不能使用适合的预测, 而造成过分臃肿的 PNG 文件。

让 IE6 透明的小技巧：

　　如上图所示, IE6 支持全透明, 不支持半透明, 所以我们在 PS 到处透明图片的时候可以把图片设置为 **png8**, 在 PS 的生成图片是记得把 **png 透明的选项**勾选上, 测试代码：

运行下面代码

> ```
> <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> <html xmlns="http://www.w3.org/1999/xhtml"> <head> <meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> <title>无标题文档</title> </head> <body> <style> body{ background:#eee; } p{ position:absolute; } p.p1{ left:200px; top:140px; } p.p2{ left:500px; top:140px; } img{ position:relative; } </style> <p class="p1"> text </p> <p class="p2"> text </p> <img src="http://images.cnitblog.com/blog2015/497865/201505/022343328802481.png" /> </body> </html>
> 
> ```

View Code

效果图：

![](http://static.oschina.net/uploads/img/201510/01014531_Pwkb.png)

    JPG（Joint Photographic Experts Group）
=========================================

　　JPEG 是一种针对相片影像而广泛使用的一种失真压缩标准方法。JPEG 的压缩方式通常是破坏性资料压缩（lossy compression）, 意即在压缩过程中图像的品质会遭受到可见的破坏。

优点

　　JPEG/JFIF 是最普遍在万维网（World Wide Web）上被用来储存和传输照片的格式。**JPEG 在色调及颜色平滑变化**的相片或是写实绘画（painting）上可以达到它最佳的效果。在这种情况下, 它通常比完全无失真方法作得更好, 仍然可以产生非常好看的影像（事实上它会比其他一般的方法像是 GIF 产生更高品质的影像, 因为 GIF 对于线条绘画（drawing）和图示的图形是无失真, 但针对全彩影像则需要极困难的量化）。

缺点

　　它并不适合于线条绘图（drawing）和其他文字或图示（iconic）的图形, 因为它的压缩方法用在这些图形的型态上, 会得到不适当的结果；

　　给个活生生的例子：一张照片在 Instagram 反复上传下载 90 次之后....(在博客园找了半小时, link), 在最后 jpg 图片完全变样了;

![](http://static.oschina.net/uploads/img/201510/01014531_giFJ.jpg)

WEBP 图片格式：

　　2010 年谷歌推迟的图片格式, 专门用来在 web 中使用, 压缩率只有 jpg 的 2/3 或者更低； 第一个版本的 webp 图片格式是有损的, 新版本中 webp 图片是无损的。

　　相对于 png 图片, webp 比 png 小了 45%, 但是缺点是你压缩的时候需要的时间更久了；

![](http://static.oschina.net/uploads/img/201510/01014531_DZye.jpg)

优点：

　　体积小巧；

缺点 ：

　　兼容性不太好, 只有 opera, 和 chrome 支持;

　　但是有个插件可以让所有浏览器都支持 webp 格式, 利用了 flash 把 webp 图片转换为浏览器可以识别的图片格式;  
　　WebP 插件打包下载：http://www.etherdream.com/WebP/WebP.zip  
　　WebP 插件在线 Demo：http://www.etherdream.com/WebP/  
　　WebP 插件源码下载：http://code.google.com/intl/zh-CN/speed/webp

额外的信息：

　　前面如果有看清楚的有写 png 和 gif 是无损压缩, 但是实际上通过**作图工具**导出的 png 或者 gif 图片明明很模糊的啊, 为什么呢？

运行下面代码

> ```
> 因为gif是8位的压缩,"8位"是指图片所能表现的颜色深度, 一个8位图像仅最多只能支持256种不同颜色(一个多余256种颜色的图片若用gif图片保存会出现失真, 相对于jpg图片来讲, gif有他所能表示色彩的极限, 如果原图中色彩太多了就悲剧了, 所以所谓的无损是相对于jpg格式会对图片进行压缩的一种说法);
>     png的图片有8为有24为有32位, 当然实际上24位和32位的png图片颜色看起来更加鲜艳自然;
> ```

    base64
==========

　　Base64 是网络上最常见的用于传输 8Bit 字节代码的编码方式之一。Base64 编码可用于在 HTTP 环境下传递较长的标识信息, 直接把 base64 当成是**字符串方式**的**数据**就好了

　　利用 Base64 的不可读性，可以加密字符串，标准浏览器的 window 下有两个方法，分别是 window.btoa 和 window.atob，（IE67 下虽然不支持，但是可以用 vbscript 模拟，参考司徒正美大牛（简称司牛）的地址，如下：

运行下面代码

> ```
> <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
> <html xmlns="http://www.w3.org/1999/xhtml">
> <head>
> <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
> <title>无标题文档</title>
> </head>
> <body>
>     <script> var str = "nono::::"; var result = "";
>         console.log( result = window.btoa(str) ); //bm9ubzo6Ojo= console.warn( window.atob( result ) ); //nono:::: </script>
> </body>
> </html>
> ```

![](http://static.oschina.net/uploads/img/201510/01014531_xOdz.png)

优点：

　　1：减少了 http 请求；

　　2：数据就是图片；

缺点：

　　1：如果图片稍微有点大，这个字符串会很长很长；

　　2：IE6,7 你懂得；

　　如何获取图片对应的 base64 字符串呢？

　　　　1：使用代码获取：

运行下面代码

> ```
> var reader = new FileReader(), htmlImage;
> reader.onload = function(e) {
>     htmlImage = "<img src=""+ e.target.result +"" />"; // 这里e.target.result就是base64编码 }
> reader.readAsDataURL(file);
> ```

　　　　2：在 webkit 内核浏览器有个挺方便的技巧， 你打开发者工具， 选中图片， 那么右侧就会出现对应图片的 base64 ，你把这个字符串复制一下，在字符串前面加上 **data:images/gif;base64**, 然后直接复制到浏览器的地址栏， 打开就会显示这副图片;

：

![](http://static.oschina.net/uploads/img/201510/01014531_J1AR.png)

　　base64 图片的 DEMO:

运行下面代码

> ```
> <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> <html xmlns="http://www.w3.org/1999/xhtml"> <head> <meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> <title>无标题文档</title> </head> <body> <img src="data:images/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAFo9M/3AAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyBpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMC1jMDYwIDYxLjEzNDc3NywgMjAxMC8wMi8xMi0xNzozMjowMCAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNSBXaW5kb3dzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjM4RjcyMUE5NEFDNzExRTA5RjMxODI4RjU2OTNEMzNCIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjM4RjcyMUFBNEFDNzExRTA5RjMxODI4RjU2OTNEMzNCIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6MzhGNzIxQTc0QUM3MTFFMDlGMzE4MjhGNTY5M0QzM0IiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6MzhGNzIxQTg0QUM3MTFFMDlGMzE4MjhGNTY5M0QzM0IiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz5fTzGYAAADg0lEQVR42mJgAAIlRY3/TMryegp3VX8xAAQQg76BlQFIlOG9Fv9/TXmt/wABBOa4uwX8BwF9Q4v9DCC1n798+//r95//B9rN/wMEEFiFtpbxf1l5BQcwR0VRux+kXEle9T+Iz9htqP2/WEuQ4e2jZwy2L/kdGTZ3WYANvPXk7X8pcVkFgAACa5OXVxOwtA75DwOKQO2yCooKIDkWEDFf/P/7Jw92MjAcEGFguPmJof4fO4O4iOR9zwcMjAxqSob7KzLj/z81V/7/7dv3/79///7/7efv/5G2Wv/lZVT2M4JM+H8n6P+1tR8Y7muXMvD/fc2gz17BcHk/F4N15x1GgABigAEbW8/16iq6/xUV1fbDxIDOgTDMzD3fHz245z8zM/N/EK0or/IfpoAJxHj78rEAAxJgROZUMTA47OAX+P+rm/3//zUs/392sP5fJCb2fzoPQwPIBEZjY8f/256fZeA1lGfgBHr6/4d3DNeeMjNEsEsyXL5yghEU+v9/Pnv2/0mg0/8fP37+//fv3/8X7778l5VS/g9zg+Ctr38ZxFbuZGBjY2P4/O0Xw4/P7xn+/v0lCA7JGYb3DbQ/mDEcORzHwK4dxiD7rJVBXpeBYbn9e1DUHwAIUCa5+zIUgFH8tLf1qFZvQ1yVktvWo14VJBLEIwYxSCqWRmJobV2kEf+A2CwVIRbhDiYiGEjEwILFwCAS8ahHRGl6tW7r3roPbRMN8S3f9J2c73fOH8OO2pahCtq5GWUjYMNPEL94KJDTD0OlUiAqovfh/pb5xyk9dmudp87RrvR0DWQCTTPzT81n9k/ANrpKqax0rPwWyDoYJ/XsDkmTR74bUP0prLkVgBADIiGEX+MovhHhXLahMfqKLiFq9XEIZgW6zXb/tIkPdAdZXNRYsE7lwBIMQ9YmoZZzYCTUoD4VOAUJSyYzdiNx5uDx2psRaKzvCEiS5C8vNWBh1A1uZhIaXT7K6HJoKTOSNc34stih0NUg7LVoa2oDodWgoNDAnJ+deDUJjksX93C0j+ql3SNQrc8ikmuEafMoyycVHWIJAUk+geHBFqxtHR/ynx9ktjZznXqyw02zRcU2hBoCoC0lKNTlQZJl8Ekxc/x8dwnzxQTeYgIu966tY/tc8A/EDZfeU++QXakekBIrv59eEdu+4ziz2K7ztFoll4okSIMO76EX9XbPKsf8pPANFgKR/lchulcAAAAASUVORK5CYIIvKiAgfHhHdjAwfGIyZTA0ZTk5NjI2MWI5OWRkOTRkZjNlZjg2YWMzM2ZiICov" /> </body> </html>
> 
> ```

　　APNG
======

　　这东西是 mozilla 搞出来的， 它是 24 位的，而且也是动图，可以容纳 **1680** 万种颜色，也是为了取代 GIF，但是.... 也就火狐支持，IE10 和 chrome，safari 全部不行， 如果说 gif 图片是卡片机的话， APNG 就是单反， 测试浏览器是否支持 apng 格式;

转载于: https://my.oschina.net/huangcongcong/blog/513035