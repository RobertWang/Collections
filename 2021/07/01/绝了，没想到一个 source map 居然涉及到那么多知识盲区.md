> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651578211&idx=1&sn=a6faf578f4101c552556a6c705bb3d96&chksm=802508a2b75281b4dead3c8e29687259c4693b6cf584cbeeba6065ea7146e684e7f7c5014abb&mpshare=1&scene=1&srcid=0701XryMnLjO6evxsKwEfeoa&sharer_sharetime=1625123894085&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

Source map 想必大家都不陌生。线上的代码多是压缩后的，如果线上有报错却只能调试那个代码多半是个噩梦。因此我们需要有一个桥梁帮助我们搭建起源代码及压缩后代码的联系，source map 就是起了这个作用。

以下是 MDN 对于 source map 的解释：

> 调试原始源代码会比浏览器下载的转换后的代码更加容易。source map[1] 是从已转换的代码映射到原始源的文件，使浏览器能够重构原始源并在调试器中显示重建的原始源。

但是不知道各位读者有没有对 source map 的原理产生过疑问？笔者列出了四个疑问，不知道各位是不是也存在过这样的问题：

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib1W6VwQ03NU6clT1alBexqegiaDsCmicyZQuEdiclLpPN0RD6C5nLAXnPDdTSR5BYWLiclr3jtKQfia5Og/640?wx_fmt=png)

Source map 四问

接下来的内容会逐步为读者解答这四问。

source map 文件是否影响网页性能
---------------------

这个答案肯定是不会影响，否则构建相关的优化就肯定会涉及到对于 source map 的处理了，毕竟 source map 文件也不小。

其实 source map 只有在打开 dev tools 的情况下才会开始下载，相信大部分用户都不会去打开这个面板，所以这也就不是问题了。

这时可能会有读者想说：哎，但是我好像从来没有在 Network 里看到 source map 文件的加载呀？其实这只是浏览器隐藏了而已，如果大家使用抓包工具的话就能发现在打开 dev tools 的时候开始下载 source map 了。

source map 存在标准嘛？
-----------------

source map 是存在一个标准的，为 Google 及 Mozilla 的工程师制定，文档地址 [2]。正是因为存在这份标准，各个打包器及浏览器才能生成及使用 source map，否则就乱套了。

各个打包器基本都基于该库 [3] 来生成 source map，当然也存在一些魔改的方案，但是标准都是统一的。

通过上面的库生成出来的 source map 格式大致如下，大家也可以对比各个打包器的产物，格式及内容大部分都是一致的：

```
{
  version: 3,
  file: "min.js",
  names: ["bar", "baz", "n"],
  sources: ["one.js", "two.js"],
  sourceRoot: "http://example.com/www/js/",
  mappings: "CAAC,IAAI,IAAM,SAAUA,GAClB,OAAOC,IAAID;CCDb,IAAI,IAAM,SAAUE,GAClB,OAAOA"
}


```

接下来笔者介绍下重要字段的作用：

*   version：顾名思义，指代了版本号，目前 source map 标准的版本为 3，也就是说这份 source map 使用的是第三版标准产出的
    
*   file：编译后的文件名
    
*   names：一个优化用的字段，后续会在 mappings 中用到
    
*   sources：多个源文件名
    
*   mappings：这是最重要的内容，表示了源代码及编译后代码的关系，但是先略过这块，下文中会详细解释
    

另外大部分应用都是由 webpack 来打包的，可能有些读者会发现 webpack 的 source map 产出的字段于上面的略微有些不一致。

这是因为 webpack 魔改了一些东西，但是底下还是基于这个库实现的，只是变动了一些不涉及核心的字段，具体代码 [4]。

浏览器怎么知道源文件和 source map 的关系？
---------------------------

这里我们以 webpack 做个实验，通过 webpack5 对于以下代码进行打包：

```
// index.js
const a = 1
console.log(a);


```

当我们开启 source map 选项以后，产物应该为两个文件，分别为 `bundle.js` 以及 `bundle.js.map`。

查看 `bundle.js` 文件以后我们会发现代码中存在这一一段注释：

```
console.log(1);
//# sourceMappingURL=bundle.js.map


```

`sourceMappingURL` 就是标记了该文件的 source map 地址。

当然除此之外还有别的方式，通过查阅 MDN 文档 [5] 发现还可以通过 response header 的 `SourceMap: <url>` 字段来表明。

source map 是如何对应到源代码的？
----------------------

这是 source map 最核心的功能，也是最涉及知识盲区的一块内容。

大家应该还记得上文中没介绍的 `mapping` 字段吧，接下来我们就来详细了解这个字段的用处。

我们还是以刚才打包的文件为例，来看看产出的 source map 长啥样（去掉了无关紧要的）：

```
{
  sources:["webpack://webpack-source-demo/./src/index.js"],
  names: ['console', 'log'],
  mappings: 'AACAA,QAAQC,IADE',
}


```

首先 `mappings` 的内容其实是 Base64 VLQ 的编码表示。

内容由三部分组成，分别为：

*   英文，表示源码及压缩代码的位置关联
    
*   逗号，分隔一行代码中的内容。比如说 `console.log(a)` 就由 `console` 、`log` 及 `a` 三部分组成，所以存在两个逗号。
    
*   分号，代表换行
    

逗号和分号想必大家没啥疑问，但是对于这几个英文内容应该会很困惑。

其实这就是一种压缩数字内容的编码方式，毕竟源代码可能很庞大，用数字表示行数及列数的话 source map 文件将也会很庞大，因此选用 Base 64 来代表数字用以减少文件体积。

比如说 `A` 代表了数字 0，`C` 代表了数字 1 等等，有兴趣的读者可以通过该网站 [6] 了解映射关系。

了解了这层编码的映射关系，我们再来聊聊这一串串英文到底代表了什么。

其实这每串英文中的字母都代表了一个位置：

1.  压缩代码的第几列
    
2.  哪个源代码文件，毕竟可以多个文件打包成一个，对应 `sources` 字段
    
3.  源代码第几行
    
4.  源代码第几列
    
5.  `names` 字段里的索引
    

这时读者可能有个疑惑，为啥没有压缩代码的第几行表示？这是因为压缩后的代码就一行，所以只需要表示第几列就行了。

* * *

**更新：有读者询问 Base64 表达的数字是有上限的，如果需要表示的数字很大的话该怎么办。实际上除了每个分号中的第一串英文是用来表示代码的第几行第几列的绝对位置之外，后面的都是相对于之前的位置来做加减法的。**

* * *

了解完以上知识以后，我们就来根据上文的内容解析下 `AACAA` 的具体含义吧，通过该网站 [7] 我们可以知道 `AACAA` 对应了 `[0,0,1,0,0]`，这里需要注意的是数字都从 0 开始，笔者表述的时候会自动加一，毕竟代码第零行听起来怪怪的。

1.  压缩代码的第一列
    
2.  第一个源代码文件，也就是 `index.js` 文件了
    
3.  源代码第二行了
    
4.  源代码的第一列
    
5.  `names` 数组中的第一个索引，也就是 `console`
    

通过以上的解析，我们就能知道 `console` 在源代码及压缩文件中的具体位置了。

但是为什么 source map 会知道编译后的代码具体在什么位置呢？这里就要用到 AST 了。让我们打开网站 [8] 输入 `console.log(a)` 后观察右边的内容，你应该会发现如图所示的数据：

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib1W6VwQ03NU6clT1alBexqeKhicZZrxp0C9Jx3086SvcF4z8497oBuGPMcFl4GtaCiar5WSwS0wdfUw/640?wx_fmt=png)

image-20210516214636867

因为 source map 是由 AST 产出的，所以我们能用上 AST 中的这个数据。

source map 的应用
--------------

一般来说 source map 的应用都是在监控系统中，开发者构建完应用后，通过插件将源代码及 source map 上传至平台中。一旦客户端上报错误后，我们就可以通过该库 [9] 来还原源代码的报错位置（具体 API 看文档即可），方便开发者快速定位线上问题。

最后
--

source map 是我们日常中经常用到的东西，但是直到学习这块内容的时候才知道居然涉及到了那么多的知识盲区。

### 参考资料

[1]

source map: _https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/_

[2]

文档地址: _https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit_

[3]

该库: _https://github.com/mozilla/source-map_

[4]

具体代码: _https://github.com/webpack/webpack-sources/blob/master/lib/SourceMapSource.js_

[5]

MDN 文档: _https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/SourceMap_

[6]

该网站: _https://www.murzwin.com/base64vlq.html_

[7]

该网站: _https://www.murzwin.com/base64vlq.html_

[8]

网站: _https://astexplorer.net/_

[9]

该库: _https://github.com/mozilla/source-map_

- EOF -

推荐阅读  点击标题可跳转

1、[SourceMap 知多少：介绍与实践](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651561935&idx=1&sn=44076f34ba21ee97bdc3fae609335981&chksm=8025480eb752c1181e82f3893f08b6592864576ceefd14726bff069b636ddbc7f0a3643f035a&scene=21#wechat_redirect)

2、[前端面试官：你知道 source-map 的原理吗？](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651558030&idx=2&sn=0bdae1330ec5506f18149ad81c026de7&chksm=8025474fb752ce592b88c5fc355c0bceed3b9f39b758fbe1c06e4ae159e6a862ce2b8ec29bfe&scene=21#wechat_redirect)

3、[送给前端 er 一份 HTTP 基础知识大图](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651575799&idx=2&sn=7f0558d84d97d5ced1905cbd8e7b1853&chksm=80250236b7528b2045da42866cc967568d154dd2a9b12dc7994be6bda4821963d773f26a3bf3&scene=21#wechat_redirect)

公众号