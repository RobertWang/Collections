> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651607614&idx=1&sn=6ed684e836828d0d61a92b934c70bb00&chksm=802285ffb7550ce91a84f7af0ca585c1da76a3e3d19c49a3705e24a766f42b867230730b242f&mpshare=1&scene=1&srcid=0721yRTlI3tQqYHso8a05Bis&sharer_sharetime=1658368461675&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

现代 CSS 布局使开发人员只需按几下键就可以编写十分有意义且强大的样式规则。上面的讨论和接下来的帖文研究了 10 种强大的 CSS 布局，它们实现了一些非凡的工作。  

#### 01. 超级居中：place-items: center

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQlhky9YhOfQkIsdIXPibDicOxfrc3FnsBTCUyjFJpTh72CV2lkeSicQibL1w/640?wx_fmt=gif)

对于第一个 “单行” 布局，让我们解决所有 CSS 领域中最大的谜团：居中。我想让您知道，使用 place-items: center 会让此操作比您想象的容易。  

首先指定 grid 作为 display 方法，然后在同一个元素上写入 place-items: center。place-items 是同时设置 align-items 和 justify-items 的快速方法。通过将其设置为 center ， align-items 和 justify-items 都将设置为 center。

这使得内容能够在父级内完美居中，而不管内部大小。

#### 02. 解构煎饼式布局：flex: `<grow>` `<shrink>` `<baseWidth>`

![](https://mmbiz.qpic.cn/mmbiz_gif/lgHVurTfTcyxmRjdKcuADSULxn59Eibxjbg9hCpE0kTzG7hwCqNGUrWSMZRxomMVCHqVXS8RX7ic9gxdib4RVp8EQ/640?wx_fmt=gif)

接下来我们有解构的煎饼！这是营销网站的常见布局，例如，可能有一行 3 个项目，通常带有图像、标题，然后是一些描述产品某些功能的文本。在移动设备上，我们希望它们能够很好地堆叠，并随着我们增加屏幕尺寸而扩展。

通过使用 Flexbox 实现此效果，您不需要在屏幕尺寸发生变化时通过媒体查询来调整这些元素的位置。

flex 简写代表：flex: `<flex-grow>` `<flex-shrink>` `<flex-basis>` 。

正因为如此，如果您想让您的框填充到它们的 `<flex-basis>` 大小，缩小到更小的尺寸，但不拉伸以填充任何额外的空间，请写入：flex: 0 1 `<flex-basis>` 。在这种情况下，您的 `<flex-basis>` 是 150px，所以应该是这样：

```
.parent {
  display: flex;
}

.child {
  flex: 0 1 150px;
}


```

如果您确实希望框在换到下一行时拉伸并填充空间，请将 `<flex-grow>` 设置为 1 ，所以应该是这样：

```
.parent {
  display: flex;
}

.child {
  flex: 1 1 150px;
}


```

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQlhSEKaf25dZzOEapiaIdkkrtM3U7238lwxEwFWdBaysuiaPJ5YlB87vbQ/640?wx_fmt=gif)

现在，当您增加或减少屏幕尺寸时，这些 flex 项目会缩小和增长。  

#### 03. 侧边栏布局：grid-template-columns: minmax(`<min>`, `<max>`) …)

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQl7Dgtd73l8Xw127ZUrmGUGon9dfAkQqIFLbib0jzdoP7ticHCHMlU69Zg/640?wx_fmt=gif)

此演示对网格布局利用了 minmax 函数。我们在这里做的是将最小侧边栏大小设置为 150px ，但在更大的屏幕上，让它伸展出 25% 。侧边栏将始终占据其父级水平空间的 25%，直到 25% 变得小于 150px 。  

将以下值添加为 grid-template-columns 的值：minmax(150px, 25%) 1fr 。在第一列（在这种情况下，侧边栏）的项目其 minmax 为 150px（在 25% ），第二列项目（这里指 main 部分）占据其余的空间作为单一的 1fr 轨道。

```
.parent {
  display: grid;
  grid-template-columns: minmax(150px, 25%) 1fr;
}


```

#### 04. 煎饼堆栈布局：grid-template-rows: auto 1fr auto

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQl7ic2pYicv4RL9g0sGCmrdUqsFDwDMhoz3ltASX4zaDib9YFDziaOuQUYlw/640?wx_fmt=gif)

与 Deconstructed Pancake 不同，当屏幕尺寸改变时，本例不会包含它的子元素。通常称为粘性页脚，这种布局通常用于网站和应用程序，跨多个移动应用程序（页脚通常是工具栏）和网站（单页应用程序通常使用这种全局布局）。  

向组件添加 display: grid 将为您提供一个单列网格，但是主区域的高度将仅与页脚下方的内容一样高。

要使页脚粘在底部，请添加：

```
.parent {
  display: grid;
  grid-template-rows: auto 1fr auto;
}


```

1fr 页眉和页脚内容设置为自动采用其子项的大小，并将剩余空间 (1fr) 应用于主区域，而 auto 调整大小的行将采用其子项的最小内容的大小，以便该内容大小增加，行本身将增长以进行调整。

#### 05. 经典圣杯布局：grid-template: auto 1fr auto / auto 1fr auto

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQlYibdpaQrX29unZGuU9CvAbkclnTWGydX1xOz6Do1vG1bxYXcsyv3sBg/640?wx_fmt=gif)

对于经典的圣杯布局，有页眉、页脚、左侧边栏、右侧边栏和主要内容。类似于以前的布局，但现在有侧边栏！

要使用一行代码编写整个网格，请使用 grid-template 属性。这使您可以同时设置行和列。

属性和值对为：grid-template: auto 1fr auto / auto 1fr auto 。第一个和第二个以空格分隔的列表之间的斜线是行和列之间的分隔符。

```
.parent {
  display: grid;
  grid-template: auto 1fr auto / auto 1fr auto;
}


```

与上一个示例一样，页眉和页脚具有自动调整大小的内容，这里的左侧和右侧边栏会根据其子项的固有大小自动调整大小。但是，这次是水平尺寸（宽度）而不是垂直尺寸（高度）。

#### 06. 12 跨网格：grid-template-columns: repeat(12, 1fr)

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQljoQAq3GX5GqhzGibzl8CJ8kJthZ8ECRmlzQeuNlqEkUgj2ia8EibYDIJw/640?wx_fmt=gif)

接下来我们有另一个经典布局：12 跨网格。您可以使用 repeat() 函数在 CSS 中快速编写网格。对网格模板列使用 repeat(12, 1fr); 将为每个 1fr 提供 12 列。

```
.parent {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
}

.child-span-12 {
  grid-column: 1 / 13;
}


```

现在您有一个 12 列的轨道网格，我们可以将子项放在网格上。一种方法是使用网格线放置它们。例如， grid-column: 1 / 13 将跨越从第一到最后一行（第 13 行）并跨越 12 列。grid-column: 1 / 5; 将跨越前四个列。

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQl6fZkiaicWAKj7ickq7icxhxlno0Q2Vq2WOmdorf2WlRX4SQiceOaNLbZLEQ/640?wx_fmt=gif)

另一种方法是使用 span 关键字。使用 span ，您可以设置起始线，然后设置从该起点跨越的列数。在这种情况下，grid-column: 1 / span 12 将等效于 grid-column: 1 / 13 ，而 grid-column: 2 / span 6 将等效于 grid-column: 2 / 8 。

```
.child-span-12 {
  grid-column: 1 / span 12;
}


```

#### 07. RAM (Repeat, Auto, MinMax): grid-template-columns(auto-fit, minmax(`<base>`, 1fr))

![](https://mmbiz.qpic.cn/mmbiz_gif/lgHVurTfTcyxmRjdKcuADSULxn59EibxjeMWgOwWd8FPlxbIj4VoiaqzZ4EicQiay1hhOtKjXWFqLqUuvz5Mu0HUuw/640?wx_fmt=gif)

对于这第七个示例，结合您已经了解的一些概念来创建具有自动放置且灵活的子项的响应式布局。漂亮整齐。这里要记住的关键点是 repeat 、 auto-(fit|fill) 和 minmax()' ，可以记住它们的首字母缩写词 RAM。

总之，应是这样：

```
.parent {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
}


```

您再次使用 repeat ，但这次使用 auto-fit 关键字而不是显式数值。这可以自动放置这些子元素。这些子元素的基本最小值为 150px ，最大值为 1fr ，这意味着在较小的屏幕上，它们将占据整个 1fr 宽度，当它们达到 150px 宽度时，它们将开始流到同一条线上。

使用 auto-fit ，当它们的水平尺寸超过 150px 时，框将拉伸以填充整个剩余空间。但是，如果您将其更改为 auto-fill ，则当超出 minmax 函数中的基本大小时，它们将不会拉伸：

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQlLNPjqGs1p83q9ATiaOleLkEqfmSEdibR5fjbufoFQv0QHDCoxxCQXibZA/640?wx_fmt=gif)  

```
.parent {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
}


```

#### 08. 排列布局：justify-content: space-between

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQl22x19GPl77vpp03dNulzNSsOT14APjuQ7icich9mj4TEib9pVvvYC6lMw/640?wx_fmt=gif)

对于下一个布局，这里要主要说明的是 justify-content: space-between ，它将第一个和最后一个子元素放置在其边界框的边缘，其余空间均匀分布在元素之间。对于这些卡片，它们被放置在 Flexbox 显示模式中，使用 flex-direction: column 将方向设置为 column。

这会将标题、描述和图像块放在父卡片内的垂直列中。然后，应用 justify-content: space-between 将第一个（标题）和最后一个（图像块）元素锚定到 flexbox 的边缘，并且它们之间的描述性文本以相等的间距放置到每个端点。

```
.parent {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}


```

#### 09. 保持我的风格：clamp(`<min>`, `<actual>`, `<max>`)

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQl75ibETQib59MiaDIPS7CyhRibumwAwWClHHIWGRHgzGPo1Kcus5a1ibCmIg/640?wx_fmt=gif)

这里，我们介绍一些只有少数浏览器支持的技术，但这些技术对布局和响应式 UI 设计有非常令人兴奋的影响。在本演示中，您将使用固定工具设置宽度，如下所示：width: clamp(`<min>`, `<actual>`, `<max>`) 。

这将设置绝对最小和最大尺寸以及实际尺寸。有了值，应该这样：

```
.parent {
  width: clamp(23ch, 60%, 46ch);
}


```

这里的最小尺寸是 23ch 或 23 个字符单元，最大尺寸是 46ch ，46 个字符。字符宽度单位基于元素的字体大小（特别是 0 字形的宽度）。“实际” 尺寸为 50%，代表此元素父宽度的 50%。

在这里， clamp() 函数所做的是使该元素保持 50% 的宽度，直到 50% 大于 46ch （在较宽的视口上）或小于 23ch （在较小的视口上）。您可以看到，当我拉伸和收缩父尺寸时，这张卡片的宽度会增加到其最大限制点并减小到其限制最小点。然后它保持在父级的中心，因为我们已经应用了其他的属性来将它居中。这可以实现更清晰的布局，因为文本不会太宽（超过 46ch ）或太窄（小于 23ch ）。

这也是实现响应式排版的好方法。例如，您可以编写：font-size: clamp(1.5rem, 20vw, 3rem) 。在这种情况下，标题的字体大小将始终保持在 1.5rem 和 3rem 之间，但会根据 20vw 实际值增大和缩小以适应视口的宽度。

这是一种很好的技术，可以通过最小和最大尺寸值确保易读性，但请记住，并非所有现代浏览器都支持它，因此请确保您有回退措施并进行测试。

#### 10. 保持宽高比：aspect-ratio: `<width>` / `<height>`

![](https://mmbiz.qpic.cn/mmbiz_gif/meG6Vo0MevjHibKAlR5lU2mBVoku5ueQlLcPpYkqhIUhltDrh8QIqnl9M9lzFviaasmZWIoj9vCQfXWqvdNE8ibIQ/640?wx_fmt=gif)

最后要介绍的这一布局工具是最具实验性的工具。它最近在 Chromium 84 中被引入 Chrome Canary，Firefox 正在积极努力实现这一点，但目前还没有任何稳定的浏览器版本。

不过，我确实想提及这一点，因为这是一个经常遇到的问题。这只是简单地保持图像的宽高比。

使用 aspect-ratio 属性，当我调整卡片大小时，绿色视觉块保持 16 x 9 的宽高比。我们通过 aspect-ratio: 16 / 9 保持此宽高比。

```
.video {
  aspect-ratio: 16 / 9;
}


```

要在没有此属性的情况下保持 16 x 9 的宽高比，需要使用 padding-top hack 并为其提供 56.25% 的 padding 以设置顶宽比。我们很快就会有一个属性来避免黑客攻击和计算百分比的需要。可以使用 1 / 1 的比例制作正方形，使用 2 / 1 制作 2:1 比例。可以设置任何图像缩放比例。

```
.square {
  aspect-ratio: 1 / 1;
}


```

虽然此功能仍在不断完善中，但它值得了解，因为它解决了许多开发人员面临的冲突，我自己也多次面临，尤其是在视频和 iframe 方面。

#### 结论

感谢您耐心完成对这 10 种强大的 CSS 布局的了解。要了解更多信息，请观看完整视频，并亲自尝试演示。

演示：https://1linelayouts.glitch.me/ 

> 作者：懵懵懂懂学前端
> 
> https://blog.csdn.net/qq_56966124/article/details/124225189

- EOF -

推荐阅读  点击标题可跳转

1、[CSS 国际化](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651604684&idx=2&sn=724c148d0d3d86c7994bdf3ee327afce&chksm=8022910db755181b6bd054879ec12ec470a316ea4029e068defb93419db1ad3d1c0bc28de3db&scene=21#wechat_redirect)

2、[2022 年的 CSS 全览](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651603544&idx=2&sn=227d71118d97e3f4965df535dd3aff8d&chksm=80229599b7551c8f984441b617c98f4439f38bc25196498928a39ab21efdf473a6970f6a5596&scene=21#wechat_redirect)

3、[纯 CSS 实现十个还不错的 Loading 效果](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651605972&idx=2&sn=5d00a4997252d45a1dcdc438498e8f8c&chksm=80229c15b755150370849607e01fb41e5a64fae0b752d2f8851b8720ddf2b5fb75cd5c4c5f98&scene=21#wechat_redirect)

觉得本文对你有帮助？请分享给更多人

推荐关注「前端大全」，提升前端技能

点赞和在看就是最大的支持❤️