> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903649605320711)

本文作者：[高峰](https://link.juejin.cn?target=http%3A%2F%2Fverymuch.site%2F "http://verymuch.site/")，360 奇舞团前端工程师，W3C 性能工作组成员，同时参与 WOT 工作组的学习。

我们都知道对于网站来说，性能至关重要，CSS 作为页面渲染和内容展现的重要环节，影响着用户对整个网站的第一体验。因此，与其相关的性能优化是不容忽视的。

对于性能优化我们常常在项目完成时才去考虑，经常被推迟到项目的末期，甚至到暴露出严重的性能问题时才进行性能优化，相信大多数人对此深有体会。

笔者认为，为了更多地避免这一情况，首先要重视起性能优化相关的工作，将其贯穿到整个产品设计与开发中。其次，就是了解性能相关的内容，在项目开发过程中，自然而然地进行性能优化。最后，也是最最重要的，那就是从现在开始实施优化。

推荐大家阅读下奇舞周刊之前推的[《嗨，送你一张 Web 性能优化地图》](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FC2Zx3KPNPkgj-aHnOY43Iw "https://mp.weixin.qq.com/s/C2Zx3KPNPkgj-aHnOY43Iw")1 这篇文章，能够帮助大家对性能优化需要做的事以及需要考虑的问题形成一个整体的概念。

本文将会详细介绍 CSS 性能优化相关的技巧，笔者将它们分为**实践型**和**建议型**两类，共 8 个小技巧。实践型技巧能够快速地应用在项目中，能够很好地提升性能，也是笔者经常使用的，建议大家尽快在项目中实践。建议型技巧中，有的可能对性能影响并不显著，有的平时大家也并不会那么用，所以笔者不会着重讲述，读者们可以根据自身情况了解一下即可。

在正式开始之前，需要大家对于[浏览器的工作原理](https://link.juejin.cn?target=https%3A%2F%2Fwww.html5rocks.com%2Fen%2Ftutorials%2Finternals%2Fhowbrowserswork%2F "https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/") 2 有些一定的了解，需要的小伙伴可以先简单了解下。

下面我们开始介绍**实践型的 4 个优化技巧**，先从首屏关键 CSS 开始。

1. 内联首屏关键 CSS（Critical CSS）
---------------------------

性能优化中有一个重要的指标——**首次有效绘制**（First Meaningful Paint，简称 FMP）即指页面的首要内容（primary content）出现在屏幕上的时间。这一指标影响用户看到页面前所需等待的时间，而 ** 内联首屏关键 CSS（即 Critical CSS，可以称之为首屏关键 CSS）** 能减少这一时间。

大家应该都习惯于通过 link 标签引用外部 CSS 文件。但需要知道的是，将 CSS 直接内联到 HTML 文档中能使 CSS 更快速地下载。而使用外部 CSS 文件时，需要在 HTML 文档下载完成后才知道所要引用的 CSS 文件，然后才下载它们。所以说，**内联 CSS 能够使浏览器开始页面渲染的时间提前**，因为在 HTML 下载完成之后就能渲染了。

既然内联 CSS 能够使页面渲染的开始时间提前，那么是否可以内联所有的 CSS 呢？答案显然是否定的，这种方式并不适用于内联较大的 CSS 文件。因为[初始拥塞窗口](https://link.juejin.cn?target=https%3A%2F%2Ftylercipriani.com%2Fblog%2F2016%2F09%2F25%2Fthe-14kb-in-the-tcp-initial-window%2F "https://tylercipriani.com/blog/2016/09/25/the-14kb-in-the-tcp-initial-window/") 3 存在限制（TCP 相关概念，通常是 14.6kB，压缩后大小），如果内联 CSS 后的文件超出了这一限制，系统就需要在服务器和浏览器之间进行更多次的往返，这样并不能提前页面渲染时间。因此，我们应当**只将渲染首屏内容所需的关键 CSS 内联到 HTML 中**。

既然已经知道内联首屏关键 CSS 能够优化性能了，那下一步就是如何确定首屏关键 CSS 了。显然，我们不需要手动确定哪些内容是首屏关键 CSS。Github 上有一个项目 [Critical CSS](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ffilamentgroup%2FcriticalCSS "https://github.com/filamentgroup/criticalCSS")4，可以将属于首屏的关键样式提取出来，大家可以看一下该项目，结合自己的构建工具进行使用。当然为了保证正确，大家最好再亲自确认下提取出的内容是否有缺失。

不过内联 CSS 有一个缺点，内联之后的 CSS 不会进行缓存，每次都会重新下载。不过如上所说，如果我们将内联后的文件大小控制在了 14.6kb 以内，这似乎并不是什么大问题。

如上，我们已经介绍了为什么要内联关键 CSS 以及如何内联，那么剩下的 CSS 我们怎么处理好呢？建议使用外部 CSS 引入剩余 CSS，这样能够启用缓存，除此之外还可以异步加载它们。

2. 异步加载 CSS
-----------

CSS 会阻塞渲染，在 CSS 文件请求、下载、解析完成之前，浏览器将不会渲染任何已处理的内容。有时，这种阻塞是必须的，因为我们并不希望在所需的 CSS 加载之前，浏览器就开始渲染页面。那么将首屏关键 CSS 内联后，剩余的 CSS 内容的阻塞渲染就不是必需的了，可以使用外部 CSS，并且异步加载。

那么如何实现 CSS 的异步加载呢？有以下四种方式可以实现浏览器异步加载 CSS。

第一种方式是使用 JavaScript 动态创建样式表 link 元素，并插入到 DOM 中。

```
// 创建link标签
const myCSS = document.createElement( "link" );
myCSS.rel = "stylesheet";
myCSS.href = "mystyles.css";
// 插入到header的最后位置
document.head.insertBefore( myCSS, document.head.childNodes[ document.head.childNodes.length - 1 ].nextSibling );
复制代码
```

第二种方式是将 link 元素的`media`属性设置为用户浏览器不匹配的媒体类型（或媒体查询），如`media="print"`，甚至可以是完全不存在的类型`media="noexist"`。对浏览器来说，如果样式表不适用于当前媒体类型，其优先级会被放低，会在不阻塞页面渲染的情况下再进行下载。

当然，这么做只是为了实现 CSS 的异步加载，别忘了在文件加载完成之后，将`media`的值设为`screen`或`all`，从而让浏览器开始解析 CSS。

```
<link rel="stylesheet" href="mystyles.css" media="noexist" onload="this.media='all'">
复制代码
```

与第二种方式相似，我们还可以通过`rel`属性将`link`元素标记为`alternate`可选样式表，也能实现浏览器异步加载。同样别忘了加载完成之后，将`rel`改回去。

```
<link rel="alternate stylesheet" href="mystyles.css" onload="this.rel='stylesheet'">
复制代码
```

上述的三种方法都较为古老。现在，[rel="preload"](https://link.juejin.cn?target=https%3A%2F%2Fwww.w3.org%2FTR%2Fpreload%2F "https://www.w3.org/TR/preload/")5 这一 Web 标准指出了如何异步加载资源，包括 CSS 类资源。

```
<link rel="preload" href="mystyles.css" as="style" onload="this.rel='stylesheet'">
复制代码
```

注意，`as`是必须的。忽略`as`属性，或者错误的`as`属性会使`preload`等同于`XHR`请求，浏览器不知道加载的是什么内容，因此此类资源加载优先级会非常低。`as`的可选值可以参考上述标准文档。

看起来，`rel="preload"`的用法和上面两种没什么区别，都是通过更改某些属性，使得浏览器异步加载 CSS 文件但不解析，直到加载完成并将修改还原，然后开始解析。

但是它们之间其实有一个很重要的不同点，那就是**使用 preload，比使用不匹配的`media`方法能够更早地开始加载 CSS**。所以尽管这一标准的支持度还不完善，仍建议优先使用该方法。

该标准现在已经是候选标准，相信浏览器会逐渐支持该标准。在各浏览器的支持度如下图所示。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/8/1/164f3b1b1df0672d~tplv-t2oaga2asx-watermark.awebp)

从上图可以看出这一方法在现在的浏览器中支持度不算乐观，不过我们可以通过 [loadCSS](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ffilamentgroup%2FloadCSS%2Ftree%2Fv2.0.1%23loadcss "https://github.com/filamentgroup/loadCSS/tree/v2.0.1#loadcss")6 进行 polyfill，所以支持不支持，这都不是事儿。

3. 文件压缩
-------

性能优化时有一个最容易想到，也最常使用的方法，那就是文件压缩，这一方案往往效果显著。

文件的大小会直接影响浏览器的加载速度，这一点在网络较差时表现地尤为明显。相信大家都早已习惯对 CSS 进行压缩，现在的构建工具，如 webpack、gulp/grunt、rollup 等也都支持 CSS 压缩功能。压缩后的文件能够明显减小，可以大大降低了浏览器的加载时间。

4. 去除无用 CSS
-----------

虽然文件压缩能够降低文件大小。但 CSS 文件压缩通常只会去除无用的空格，这样就限制了 CSS 文件的压缩比例。那是否还有其他手段来精简 CSS 呢？答案显然是肯定的，如果压缩后的文件仍然超出了预期的大小，我们可以试着**找到并删除代码中无用的 CSS**。

一般情况下，会存在这两种无用的 CSS 代码：一种是不同元素或者其他情况下的重复代码，一种是整个页面内没有生效的 CSS 代码。对于前者，在编写的代码时候，我们应该尽可能地提取公共类，减少重复。对于后者，在不同开发者进行代码维护的过程中，总会产生不再使用的 CSS 的代码，当然一个人编写时也有可能出现这一问题。而这些无用的 CSS 代码不仅会增加浏览器的下载量，还会增加浏览器的解析时间，这对性能来说是很大的消耗。所以我们需要找到并去除这些无用代码。

当然，如果手动删除这些无用 CSS 是很低效的。我们可以借助 [Uncss](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Funcss%2Funcss "https://github.com/uncss/uncss")7 库来进行。Uncss 可以用来移除样式表中的无用 CSS，并且支持多文件和 JavaScript 注入的 CSS。

前面已经说完了实践型的 4 个优化技巧，下面我们介绍下**建议型的 4 个技巧**。

1. 有选择地使用选择器
------------

大多数朋友应该都知道 **CSS 选择器的匹配是从右向左进行的**，这一策略导致了不同种类的选择器之间的性能也存在差异。相比于`#markdown-content-h3`，显然使用`#markdown .content h3`时，浏览器生成渲染树（render-tree）所要花费的时间更多。因为后者需要先找到 DOM 中的所有`h3`元素，再过滤掉祖先元素不是`.content`的，最后过滤掉`.content`的祖先不是`#markdown`的。试想，如果嵌套的层级更多，页面中的元素更多，那么匹配所要花费的时间代价自然更高。

**不过现代浏览器在这一方面做了很多优化，不同选择器的性能差别并不明显，甚至可以说差别甚微。此外不同选择器在[不同浏览器中的性能表现](https://link.juejin.cn?target=https%3A%2F%2Fbenfrain.com%2Fcss-performance-revisited-selectors-bloat-expensive-styles%2F%23h-H1_1 "https://benfrain.com/css-performance-revisited-selectors-bloat-expensive-styles/#h-H1_1") 8 也不完全统一，在编写 CSS 的时候无法兼顾每种浏览器**。鉴于这两点原因，我们在使用选择器时，只需要记住以下几点，其他的可以全凭喜好。

1.  **保持简单，不要使用嵌套过多过于复杂的选择器。**
2.  **通配符和属性选择器效率最低，需要匹配的元素最多，尽量避免使用。**
3.  **不要使用类选择器和 ID 选择器修饰元素标签，如`h3#markdown-content`，这样多此一举，还会降低效率。**
4.  **不要为了追求速度而放弃可读性与可维护性。**

如果大家对于上面这几点还存在疑问，笔者建议大家选择以下几种 CSS 方法论之一（[BEM](https://link.juejin.cn?target=https%3A%2F%2Fen.bem.info%2Fmethodology%2Fquick-start%2F "https://en.bem.info/methodology/quick-start/")9，[OOCSS](https://link.juejin.cn?target=http%3A%2F%2Foocss.org%2F "http://oocss.org/")10，[SUIT](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsuitcss%2Fsuit%2Fblob%2Fmaster%2Fdoc%2Fnaming-conventions.md "https://github.com/suitcss/suit/blob/master/doc/naming-conventions.md")11，[SMACSS](https://link.juejin.cn?target=https%3A%2F%2Fsmacss.com%2F "https://smacss.com/")12，[ITCSS](https://link.juejin.cn?target=https%3A%2F%2Fwww.xfive.co%2Fblog%2Fitcss-scalable-maintainable-css-architecture%2F "https://www.xfive.co/blog/itcss-scalable-maintainable-css-architecture/")13，[Enduring CSS](https://link.juejin.cn?target=http%3A%2F%2Fecss.io%2F "http://ecss.io/")14 等）作为 CSS 编写规范。使用统一的方法论能够帮助大家形成统一的风格，减少命名冲突，也能避免上述的问题，总之好处多多，如果你还没有使用，就赶快用起来吧。

> ### Tips：为什么 CSS 选择器是从右向左匹配的？
> 
> CSS 中更多的选择器是不会匹配的，所以在考虑性能问题时，需要考虑的是如何在选择器不匹配时提升效率。从右向左匹配就是为了达成这一目的的，通过这一策略能够使得 CSS 选择器在不匹配的时候效率更高。这样想来，在匹配时多耗费一些性能也能够想的通了。

2. 减少使用昂贵的属性
------------

在浏览器绘制屏幕时，**所有需要浏览器进行操作或计算的属性相对而言都需要花费更大的代价**。当页面发生重绘时，它们会降低浏览器的渲染性能。所以在编写 CSS 时，我们应该尽量减少使用昂贵属性，如`box-shadow`/`border-radius`/`filter`/ 透明度 /`:nth-child`等。

当然，并不是让大家不要使用这些属性，因为这些应该都是我们经常使用的属性。之所以提这一点，是让大家对此有一个了解。当有两种方案可以选择的时候，可以优先选择没有昂贵属性或昂贵属性更少的方案，如果每次都这样的选择，网站的性能会在不知不觉中得到一定的提升。

3. 优化重排与重绘
----------

在网站的使用过程中，某些操作会导致样式的改变，这时浏览器需要检测这些改变并重新渲染，其中有些操作所耗费的性能更多。我们都知道，当 FPS 为 60 时，用户使用网站时才会感到流畅。这也就是说，我们需要在 16.67ms 内完成每次渲染相关的所有操作，所以我们要尽量减少耗费更多的操作。

### 3.1 减少重排

重排会导致浏览器重新计算整个文档，重新构建渲染树，这一过程会降低浏览器的渲染速度。如下所示，有很多操作会触发重排，我们应该避免频繁触发这些操作。

1.  改变`font-size`和`font-family`
2.  改变元素的内外边距
3.  通过 JS 改变 CSS 类
4.  通过 JS 获取 DOM 元素的位置相关属性（如 width/height/left 等）
5.  CSS 伪类激活
6.  滚动滚动条或者改变窗口大小

此外，我们还可以通过 [CSS Trigger](https://link.juejin.cn?target=https%3A%2F%2Fcsstriggers.com%2F "https://csstriggers.com/")15 查询哪些属性会触发重排与重绘。

值得一提的是，某些 CSS 属性具有更好的重排性能。如使用`Flex`时，比使用`inline-block`和`float`时重排更快，所以在布局时可以优先考虑`Flex`。

### 3.2 避免不必要的重绘

当元素的外观（如 color，background，visibility 等属性）发生改变时，会触发重绘。在网站的使用过程中，**重绘是无法避免的**。不过，浏览器对此做了优化，它会将多次的重排、重绘操作合并为一次执行。不过我们仍需要**避免不必要的重绘**，如页面滚动时触发的 hover 事件，可以在滚动的时候禁用 hover 事件，这样页面在滚动时会更加流畅。

此外，我们编写的 CSS 中动画相关的代码越来越多，我们已经习惯于使用动画来提升用户体验。我们在编写动画时，也应当参考上述内容，减少重绘重排的触发。除此之外我们还可以通过[硬件加速](https://link.juejin.cn?target=https%3A%2F%2Fwww.sitepoint.com%2Fintroduction-to-hardware-acceleration-css-animations%2F "https://www.sitepoint.com/introduction-to-hardware-acceleration-css-animations/") 16 和 [will-change](https://link.juejin.cn?target=https%3A%2F%2Fdrafts.csswg.org%2Fcss-will-change%2F "https://drafts.csswg.org/css-will-change/")17 来提升动画性能，本文不对此展开详细介绍，感兴趣的小伙伴可以点击链接进行查看。

最后需要注意的是，用户的设备可能并没有想象中的那么好，至少不会有我们的开发机器那么好。我们可以借助 Chrome 的开发者工具进行 CPU 降速，然后再进行相关的测试，降速方法如下图所示。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/8/1/164f3b1b11c7828b~tplv-t2oaga2asx-watermark.awebp)

如果需要在移动端访问的，最好将速度限制更低，因为移动端的性能往往更差。

4. 不要使用 @import
---------------

最后提一下，不要使用 @import 引入 CSS，相信大家也很少使用。

不建议使用 @import 主要有以下两点原因。

首先，使用 @import 引入 CSS 会影响浏览器的并行下载。使用 @import 引用的 CSS 文件只有在引用它的那个 css 文件被下载、解析之后，浏览器才会知道还有另外一个 css 需要下载，这时才去下载，然后下载后开始解析、构建 render tree 等一系列操作。这就导致浏览器无法并行下载所需的样式文件。

其次，多个 @import 会导致下载顺序紊乱。在 IE 中，@import 会引发资源文件的下载顺序被打乱，即**排列在 @import 后面的 js 文件先于 @import 下载，并且打乱甚至破坏 @import 自身的并行下载**。

所以不要使用这一方法，使用 link 标签就行了。

总结
--

至此，我们介绍完了 CSS 性能优化的 4 个实践型技巧和 4 个建议型技巧，在了解这些技巧之后，CSS 的性能优化从现在就可以开始了。不要犹豫了，尽快开始吧。

致谢
--

特别感谢 @anjia(安佳)、@刘宇晨、@hxl(黄小璐)、@刘观宇的辛苦审校，感谢你们对于文章结构和内容提出的宝贵建议。

参考文章
----

> 1.  [Efficiently Rendering CSS](https://link.juejin.cn?target=https%3A%2F%2Fcss-tricks.com%2Fefficiently-rendering-css%2F%23article-header-id-4 "https://css-tricks.com/efficiently-rendering-css/#article-header-id-4")
> 2.  [How to write CSS for a great performance web application](https://link.juejin.cn?target=https%3A%2F%2Fblog.nextzy.me%2Fhow-to-write-css-for-a-great-performance-web-application-edf75bb8a8cc "https://blog.nextzy.me/how-to-write-css-for-a-great-performance-web-application-edf75bb8a8cc")
> 3.  [CSS performance revisited: selectors, bloat and expensive styles](https://link.juejin.cn?target=https%3A%2F%2Fbenfrain.com%2Fcss-performance-revisited-selectors-bloat-expensive-styles%2F "https://benfrain.com/css-performance-revisited-selectors-bloat-expensive-styles/")
> 4.  [Avoiding Unnecessary Paints](https://link.juejin.cn?target=https%3A%2F%2Fwww.html5rocks.com%2Fen%2Ftutorials%2Fspeed%2Funnecessary-paints%2F "https://www.html5rocks.com/en/tutorials/speed/unnecessary-paints/")
> 5.  [Five CSS Performance Tools to Speed up Your Website](https://link.juejin.cn?target=https%3A%2F%2Fwww.sitepoint.com%2Ffive-css-performance-tools-speed-website%2F "https://www.sitepoint.com/five-css-performance-tools-speed-website/")
> 6.  [How and Why You Should Inline Your Critical CSS](https://link.juejin.cn?target=https%3A%2F%2Fwww.sitepoint.com%2Fhow-and-why-you-should-inline-your-critical-css%2F "https://www.sitepoint.com/how-and-why-you-should-inline-your-critical-css/")
> 7.  [Render blocking css](https://link.juejin.cn?target=https%3A%2F%2Fdevelopers.google.com%2Fweb%2Ffundamentals%2Fperformance%2Fcritical-rendering-path%2Frender-blocking-css%3Fhl%3Dzh-cn "https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-blocking-css?hl=zh-cn")
> 8.  [Modern Asynchronous CSS Loading](https://link.juejin.cn?target=https%3A%2F%2Fwww.filamentgroup.com%2Flab%2Fasync-css.html "https://www.filamentgroup.com/lab/async-css.html")
> 9.  [Preload](https://link.juejin.cn?target=https%3A%2F%2Fwww.w3.org%2FTR%2Fpreload%2F "https://www.w3.org/TR/preload/")