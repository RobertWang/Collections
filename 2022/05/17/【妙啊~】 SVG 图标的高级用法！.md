> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/gygs9Jh6TkRhKzzij-qE3Q)

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgG2w9kGPPwgBqJ5iaIicsMlHUU3GJe1Ay1NETAFLKzoeDJ3KMS5t4ZOPHLRumPYwdXLicu6QS6qQX14Q/640?wx_fmt=png)

 **前言** 

SVG 格式图标在软件界面中有广泛应用，它与生俱来的矢量属性，使其在高分辨率屏幕上的表现非常完美。SVG 是一个基于 XML 标记语言的开放网络标准格式，拥有跨设备多平台的兼容效果。前面我们有分享过一篇关于 WPS 图标的文章《[探索 WPS 3000 个图标设计背后的故事](http://mp.weixin.qq.com/s?__biz=MzI5NTYyOTUwMw==&mid=2247484279&idx=1&sn=dfa73c4352579505dbbf33931902fbd1&chksm=ec51e4a7db266db1f15d879c05bdc47aa025b621f8c0f0c56efdb53a2cdaff73069c823ef503&scene=21#wechat_redirect)》，得到了很多网友的积极反馈。相信界面设计的小伙伴们都很熟悉 SVG 格式了，这次，我有一个棒的想法想分享给大家：利用 SVG 图标套色，来完成不同界面的适配。

 **什么是 SVG 图标套色？** 

图标套色的简单来说就是利用 SVG 格式的文本属性，使用 XML 格式标准，在 SVG 文档中增加 CSS 样式，通过修改 CSS 样式属性，精准控制 SVG 格式图标颜色，通过修改透明度控制图形显示与否，从而变换图标的风格外观。  

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5cn96cTe8FLuQBDBFLnicHQzngse2sjQIk0IGO40icibnxuYEWlTpnQKHlw/640?wx_fmt=png)

▲ 基本原理：修改 SVG 的样式，生成不同风格的图标

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgG2w9kGPPwgBqJ5iaIicsMlHUGFlwrOpv3iaVptM4PeaZYncopWicWmze0iaGrQLeGbyfeXtkpiabcVUBkg/640?wx_fmt=png)

**用处一：颜色适配**

这里有几个插件，都用到了 “保存”、“打印” 这些功能。因为主题色不同，即使是同样外形的图标，还是需要根据主题色的不同输出适配各个插件的图标。采用图标套色的方法，就可以避免这类图标资源的重复输出。

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5cPiayue83qJpPgtBM7Sib8NMpqBpPUgyjkJHyA8fibX4h5j0gcf3hicCyWw/640?wx_fmt=png)

▲ 相同功能需要两套不同主题色的图标

**用处二：皮肤适配**

现在多数软件一般都有皮肤功能，不同用户需求，衍生出风格各异的皮肤，各种颜色，深浅不一，一套图标满足不了所有，为了视觉效果需要对每个皮肤输出特定颜色、风格的图标。图标数量如果很多，投入的成本将随皮肤数量呈几何倍增加，图标套色就可以很好的解决这类问题，只需要通过修改图标颜色和风格即可适配。

![](https://mmbiz.qpic.cn/mmbiz_gif/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5c5QNCs98pV9yjUduvOzMibPIDxxiclyZUW2AUTUZdiaVM4Tsq9ic1Pxg4CA/640?wx_fmt=gif)

▲ 通过修改映射配置，可以得到不同颜色的图标

 **套色方法** 

我们先看看图标套色之后的效果：

![](https://mmbiz.qpic.cn/mmbiz_gif/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5cdM4qXaD1ag9AOHdmJyJlV4aYJjB6JSb2Sw7b2tic4Eb4A4iavGnqu8Aw/640?wx_fmt=gif)  

▲ 修改映射配置，可以得到线、面不同风格的图标

简单来说，实现这种效果有下方五个步骤:

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgG2w9kGPPwgBqJ5iaIicsMlHUjekHvIJWHgVsb5sseY3xySqPGZehmYBY8aOsckqLxOnEiaaPYLlicFTg/640?wx_fmt=png)

▲ 套色方法五个步骤

以下方几个图标来做示例：

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5cqlURc8NwmzNMKnH94kAqSyFib2HJsbFr47fjxV758u7UTzu9zTRNGyQ/640?wx_fmt=png)

▲ SVG 示例图标

**第一步，确定图标线、面风格**

设计师将图标线、面风格确定下来，并保证两者效果上可以兼容，即轮廓一致。

![](https://mmbiz.qpic.cn/mmbiz_gif/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5cHfCuhvYtLkS8Xs4DovT3J6pNnpZNP0TrBt3avibY5iae7jq244A9OUNQ/640?wx_fmt=gif)

▲ 同时兼容线、面两种风格效果

**第二步，定义图标颜色**

在确定了图标的风格之后，将图标中用到的 7 种颜色，根据一深一浅再拆分为 14 种（具体几种颜色可根据图标设计需要来定），深色用于填充线性图形，浅色用于填充面性图形。

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5cpbicDAsickdTDCkWfkt4gUdICSIibnBjxjO5Wo0UCvozWSNOMPhdDxnRw/640?wx_fmt=png)

▲ 根据线、面风格需要，定义图标的颜色

**第三步，给颜色定义样式名**

给 14 种颜色，分别定义好 CSS 样式名（样式名遵循 CSS 规则即可）。

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5ctl0zxe2opGplkxxHJI9ys8rxr5HxicQlGE84q1mGicsLn0a6zgg97aKQ/640?wx_fmt=png)

▲ 给颜色定义样式名

**第四步，给 SVG 图标添加 CSS 内部样式**

SVG 格式图标默认是没有 CSS 样式，需要手动将 CSS 内部样式添加到 SVG 文档中，并将 SVG 路径颜色与 CSS 样式名关联起来。

![](https://mmbiz.qpic.cn/mmbiz_gif/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5cqOptaIERf4d1myDBQseDoHufU6xwvPFzeMR0qvQp93y9cyuMKICZPw/640?wx_fmt=gif)

▲ 给 SVG 添加 CSS 样式

**第五步，样式属性配置机制**

添加内部样式之后，需要开发小哥哥在软件中增加对 SVG 图标 CSS 样式属性的映射机制。修改映射机制配置文件中 CSS 样式属性，就可以控制图标风格变化。

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5cfF54oNBjuQcMIxoDiaDhNlIFDe7Q0oVdK7gPy929bBicVA50Hkl6OicUg/640?wx_fmt=png)

▲ 修改配置代码即可改变图标颜色

完成了以上五个步骤，通过修改软件中的映射机制配置文件，就可以改变图标风格。

 ****应用案例**** 

了解了步骤方法，我们以 WPS 为例来讲解图标套色在实际案例中的应用：

**案例一：**前面有提到我们的四大组件，WPS 由文字、表格、演示、PDF 四组件组成。每个组件都有各自的主题色，文字主题色为蓝色，表格绿色，演示橘黄色，PDF 红色。多数图标都含有主题色，但外形是一样的，因各组件主题色不同而导致了很多图标的重复输出。

套色用处之一的颜色适配，可以让图标变色以适应不同的组件色，避免图标的重复。

![](https://mmbiz.qpic.cn/mmbiz_gif/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5c7ghGPSbyDxxWiblYFnN6M4YuRMWN3McwIYKT8P5fPn3QHtOZUutibqLg/640?wx_fmt=gif)

▲ 不同主题色图标的变换效果

**案例二：**WPS 有推出多个风格各异的皮肤，因为图标数量的关系，无法每个皮肤都输出一套图标，目前只能使用默认的线性图标。也因时间和维护成本而导致图标风格的单一。

套色用处之二的皮肤适配，能使图标改变风格以适应不同的皮肤，既能满足图标风格多样，又能满足时间和维护成本的可控。

![](https://mmbiz.qpic.cn/mmbiz_gif/ia8ZHRc55hgEFhia6ViaBQ0rkH6zERKHiaHy5uCaiald27KhInfzJxSicUYUicKtJqB5rzibczkg0bdbvowP9PYgkXhtyg/640?wx_fmt=gif)

▲ 不同风格图标的变换效果

**案例三：**深色模式的配色与浅色模式大相径庭，图标使用的颜色也截然不同，适配需要输出两套图标资源，因不同深浅色模式而导致的图标资源重复输出。WPS 组件功能区的图标有几千个，输出和维护都很费精力。

套色用处之二的皮肤适配，在深浅色模式下也能适用，改变图标颜色以适应不同的深浅色模式，避免图标的重复输出。

![](https://mmbiz.qpic.cn/mmbiz_gif/ia8ZHRc55hgEa0yxNvtaLGhG06JkLxu5cKFcWM1DRCDmLLDhSZU9ya324YHjec7X6n63y7bdV2WShSZibWnChOuQ/640?wx_fmt=gif)

▲ 深浅色模式下图标的变换效果

 **总结** 

通过以上的案例不难发现，SVG 图标套色技术的价值，主要有以下几个方面：

![](https://mmbiz.qpic.cn/mmbiz_png/ia8ZHRc55hgEFhia6ViaBQ0rkH6zERKHiaHyj3ZA8rA5FDWx1AWbnS4ibFASh9w6iaVfnAGLReRsPEtqy2lY4KAotcDQ/640?wx_fmt=png)

**1. 时间和维护成本的降低**

利用图标套色方法，设计师只需要输出一套图标资源，就完成了多组件、多风格、深浅色模式适配。开发小哥哥也可以删掉适配用的冗余代码，提升了图标的管理和软件运行效率；  

**2. 个性化需求的满足**

后期可以增加自定义扩展，让用户配置图标风格，更好地满足个性化需求；  

**3. 服务器资源的节约**

更少图标资源，更小压缩包，更少空间和宽带的占用。

采用新技术，帮助设计、开发更好地完成多场景适配，降低了整体流程的执行难度，为项目节省了大量时间，避免过多精力投入在重复工作中，为最终完成目标创造了有利条件。同时也解放了生产力，有了更多的时间可以去关注高价值的项目。

工作中经常能遇到重复的内容，这都有提升和优化的空间，寻找更高效的方法，让工作变得轻松简单。  

记得关注我们，下次还有更多好玩、有趣的内容分享给你～