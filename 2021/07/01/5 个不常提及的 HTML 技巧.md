> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651578211&idx=2&sn=96655771339196d756a4df468916e18e&chksm=802508a2b75281b47a34565a07f0aea0fa532db2f29ed180c2eba066769fcec6f5f430204eac&mpshare=1&scene=1&srcid=0701CQv4lT6iIg6YJrMETNSs&sharer_sharetime=1625123707490&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

↓推荐关注↓

> 2021 年你需要知道的 HTML 标签和属性

Web 开发人员都在广泛的使用 HTML。无论你使用什么框架或者选择哪个后端语言，框架在变，但是 HTML 始终如一。尽管被广泛使用，但还是有一些标签或者属性是大部分开发者不熟知的。虽然现在有很多的模版引擎供我们使用，但是我们还是需要尽可能的熟练掌握 HTML 内容，就像 CSS 一样。

在我看来，最好尽可能使用 HTML 特性来实现我们的功能，而不是使用 JavaScript 实现相同的功能，尽管我承认编写 HTML 可能会是重复的和无聊的。

尽管许多开发人员每天都在使用 HTML，但他们并没有尝试改进自己的项目，也没有真正利用 HTML 的一些鲜为人知的特性。

下面这 5 个通过 HTML 标签 / 属性实现的功能我觉得需要了解一下：

* * *

### **图片懒加载**

图片懒加载可以帮助提升网站的性能和响应能力。图片懒加载可以避免立即加载那些不在屏幕中立即显示的图片素材，当用户滚动临近图片时再去开始加载。

换言之，当用户滚动到图片出现时再进行加载，否则不加载。这就降低了屏幕内容展示过程中的图片素材的请求数量，提升了站点性能。

往往我们都是通过 javascript 来实现的，通过监听页面滚动事件来确定加载对应的资源。但是，在不完全考虑兼容性的场景下，我们其实可以直接通过 HTML 来直接实现。

> 注：本篇的提到的标签和属性的兼容性需要大家根据实际场景来选取是否使用

可以通过为图片文件添加 loading="lazy" 的属性来实现:

```
<img src="image.png" loading="lazy" alt="lazy" width="200" height="200" />


```

* * *

### **输入提示**

当用户在进行输入搜索功能时，如果能够给出有效的提示，这会大大提升用户体验。输入建议和自动完成功能现在到处可见，我们可以使用 Javascript 添加输入建议，方法是在输入框上设置事件侦听器，然后将搜索到的关键词与预定义的建议相匹配。

其实，HTML 也是能够让我们来实现预定义输入建议功能的，通过

```
<label for="country">请选择喜欢的国家:</label>
<input list="countries" >
<datalist id="countries">
  <option value="UK">
  <option value="Germany">
  <option value="USA">
  <option value="Japan">
  <option value="India">
  <option value=“China”>
</datalist>


```

* * *

### **Picture 标签**

你是否遇到过在不同场景或者不同尺寸的设备上面的时候，图片展示适配问题呢？我想大家都遇到过。

针对只有一个尺寸的图片素材的时候，我们往往可以通过 CSS 的 object-fit 属性来进行裁切适配。但是有些时候需要针对不同的分辨率来显示不同尺寸的图片的场景的时候，我们是否可以直接通过 HTML 来实现呢？

HTML 提供了标签，允许我们来添加多张图片资源，并且根据不同的分辨率需求来展示不同的图片。

```
<picture>
  <source media="(min-width:768px)" srcset="med_flower.jpg">
  <source media="(min-width:495px)" srcset="small_flower.jpg">
  <img src="high_flower" style="width: auto;" />
</picture>


```

我们可以定义不同区间的最小分辨率来确定图片素材，这个标签的使用有些类似

* * *

### **Base URL**

当我们的页面有大量的锚点跳转或者静态资源加载时，并且这些跳转或者资源都在统一的域名的场景时，我们可以通过标签来简化这个处理。

例如，我们有一个列表需要跳转到微博的不同大 V 的主页，我们就可以通过设置来简化跳转路径

```
<head>
  <base href="https://www.weibo.com/" target="_blank">  
</head>
<body>
  <a href="jackiechan">成龙</a>
  <a href="kukoujialing">贾玲</a>
</body>


```

标记必须具有 href 和 target 属性。

* * *

### **页面重定向（刷新）**

当我们希望实现一段时间后或者是立即重定向到另一个页面的功能时，我们可以直接通过 HTML 来实现。

我们经常会遇到有些站点会有这样一个功能，“5s 后页面将跳转”。这个交互可以嵌入到 HTML 中，直接通过标签，设置 http-equiv="refresh" 来实现

```
<meta http-equiv="refresh" content="4; URL='https://google.com' />


```

这里 content 属性指定了重定向发生的秒数。值得一提的是，尽管谷歌声称这种形式的重定向和其他的重定向方式一样可用，但是使用这种类型的重定向其实并不是那么的优雅，往往会显得很突兀。

因此，最好在某些特殊的情况下使用它，比如在长时间用户不活动之后再重定向到目标页面。

* * *

### **后记**

HTML 和 CSS 是非常强大的，哪怕我们仅仅使用这两种技术也能创建出一些奇妙的网站。虽然它们的使用量很大很普遍，还是有很多的开发者并没有真正的深入了解他们，还有很多的内容需要我们深入的去学习和理解，实践，有很多的技巧等待着我们去发现。

_原文地址：_

_https://js.plainenglish.io/5-html-tricks-nobody-is-talking-about-a0480104fe19_

> 转自：SegmentFault - 冷星

推荐阅读  点击标题可跳转

1、[前端面试总结——http、html 和浏览器篇](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651554664&idx=1&sn=60715e69d7aa725ae238f026274e0c88&chksm=802554a9b752ddbf91b8a86ef91d0ad313fc2898c2b41e3dfc7695526b444c7fd7d95cfcba24&scene=21#wechat_redirect)

2、[一代传奇！经典播放器 Winamp 的开源实现（HTML5+JS）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651577151&idx=2&sn=708ae36246429b2c64e18f321f4aa546&chksm=80250cfeb75285e802ab782baaebc7afca03836b2d87399534335ad900cca415295bf536266d&scene=21#wechat_redirect)

3、[那些不常见，但却非常实用的 css 属性](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651559733&idx=1&sn=c3119f9ec0bb024a609a41d577c01518&chksm=802540f4b752c9e20826b10d50b3f8eaf21509f52bf725650cf4a883278f85587c3a4cc38607&scene=21#wechat_redirect)

公众号