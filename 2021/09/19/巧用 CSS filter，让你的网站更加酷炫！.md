> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7002829486806794276) /* 使用单个滤镜 （如果传入的参数是百分数，那么也可以传入对应的小数：40% --> 0.4）*/ filter: blur(5px); filter: brightness(40%); filter: contrast(200%); filter: drop-shadow(16px 16px 20px blue); filter: grayscale(50%); filter: hue-rotate(90deg); filter: invert(75%); filter: opacity(25%); filter: saturate(30%); filter: sepia(60%); /* 使用多个滤镜 */ filter: contrast(175%) brightness(3%); /* 不使用任何滤镜 */ filter: none; 复制代码

官方 demo：[MDN](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FCSS%2Ffilter "https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter")

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9410f8b9bf4b4909ae5ed38d1efde624~tplv-k3u1fbpfcp-watermark.awebp)

滤镜在日常开发中是很常见的，比如使用`drop-shadow`给不规则形状添加阴影；使用`blur`来实现背景模糊，以及毛玻璃效果等。

下面我们将进一步使用`CSS filter`实现一些动画效果，让网站交互更加酷炫，同时也加深对`CSS filter`的理解。一起开始吧！

（ 下面要使用到的 动画 和 伪类 知识，在 [CSS 的 N 个编码技巧](https://juejin.cn/post/6924206099193135111 "https://juejin.cn/post/6924206099193135111") 中都有详细的介绍，这里就不重复了，有需要的朋友可以前往查看哦。 ）

电影效果
====

滤镜中的`brightness`用于调整图像的明暗度。默认值是`1`；小于`1`时图像变暗，为`0`时显示为全黑图像；大于`1`时图像显示比原图更明亮。

我们可以通过调整 `背景图的明暗度` 和 `文字的透明度` ，来模拟电影谢幕的效果。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b843c216405b47938613fe576026b199~tplv-k3u1fbpfcp-watermark.awebp)

```
<div class="container">
  <div class="pic"></div>
  <div class="text">
    <p>如果生活中有什么使你感到快乐，那就去做吧</p>
    <br>
    <p>不要管别人说什么</p>
  </div>
</div>
复制代码
```

```
.pic{
    height: 100%;
    width: 100%;
    position: absolute;
    background: url('./images/movie.webp') no-repeat;
    background-size: cover;
    animation: fade-away 2.5s linear forwards;    //forwards当动画完成后，保持最后一帧的状态
}
.text{
    position: absolute;
    line-height: 55px;
    color: #fff;
    font-size: 36px;
    text-align: center;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
    opacity: 0;
    animation: show 2s cubic-bezier(.74,-0.1,.86,.83) forwards;
}
    
@keyframes fade-away {    //背景图的明暗度动画
    30%{
        filter: brightness(1);
    }
    100%{
        filter: brightness(0);
    }
}
@keyframes show{         //文字的透明度动画
    20%{
        opacity: 0;
    }
    100%{
        opacity: 1;
    }
}
复制代码
```

模糊效果
====

在下面的单词卡片中，当鼠标`hover`到某一张卡片上时，其他卡片背景模糊，使用户焦点集中到当前卡片。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b580f7a573d9497dbc6497b6cf66364c~tplv-k3u1fbpfcp-watermark.awebp)

`html`结构：

```
<ul class="cards">
    <li class="card">
      <p class="title">Flower</p>
      <p class="content">The flowers mingle to form a blaze of color.</p>
    </li>
    <li class="card">
      <p class="title">Sunset</p>
      <p class="content">The sunset glow tinted the sky red.</p>
    </li>
    <li class="card">
      <p class="title">Plain</p>
      <p class="content">The winds came from the north, across the plains, funnelling down the valley. </p>
    </li>
 </ul>
复制代码
```

实现的方式，是将背景加在`.card`元素的伪类上，当元素不是焦点时，为该元素的伪类加上滤镜。

```
.card:before{
    z-index: -1;
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    border-radius: 20px;
    filter: blur(0px) opacity(1);
    transition: filter 200ms linear, transform 200ms linear;
}
/*
     这里不能将滤镜直接加在.card元素，而是将背景和滤镜都加在伪类上。
     因为，父元素加了滤镜，它的子元素都会一起由该滤镜改变。
     如果滤镜直接加在.card元素上，会导致上面的文字也变模糊。
*/
复制代码
```

```
//通过css选择器选出非hover的.card元素，给其伪类添加模糊、透明度和明暗度的滤镜 

.cards:hover > .card:not(:hover):before{    
  filter: blur(5px) opacity(0.8) brightness(0.8);
}
复制代码
```

```
//对于hover的元素，其伪类增强饱和度，尺寸放大

.card:hover:before{
  filter: saturate(1.2);  
  transform: scale(1.05);
}
复制代码
```

褪色效果
====

褪色效果可以打造出一种怀旧的风格。下面这组照片墙，我们通过`sepia`滤镜将图像基调转换为深褐色，再通过降低 饱和度`saturate` 和 色相旋转`hue-rotate` 微调，模拟老照片的效果。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59281a5ea70544e1bd75414762e38aba~tplv-k3u1fbpfcp-watermark.awebp)

```
.pic{
    border: 3px solid #fff;
    box-shadow: 0 10px 50px #5f2f1182;
    filter: sepia(30%) saturate(40%) hue-rotate(5deg);
    transition: transform 1s;
}
.pic:hover{
    filter: none;
    transform: scale(1.2) translateX(10px);
    z-index: 1;
}
复制代码
```

灰度效果
====

怎样让网站变成灰色？在 html 元素上加上`filter: grayscale(100%)`即可。

`grayscale(amount)`函数将改变输入图像灰度。`amount` 的值定义了灰度转换的比例。值为 `100%` 则完全转为灰度图像，值为 `0%` 图像无变化。若未设置值，默认值是 `0`。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d1d8e11b1614065a2a7ec7719e03a5f~tplv-k3u1fbpfcp-watermark.awebp)

融合效果
====

要使两个相交的元素产生下面这种融合的效果，需要用到的滤镜是`blur`和`contrast`。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63cd6ad60de742f1a382de1ffe5e8362~tplv-k3u1fbpfcp-watermark.awebp)

```
<div class="container">
  <div class="circle circle-1"></div>
  <div class="circle circle-2"></div>
</div>
复制代码
```

```
.container{
  margin: 50px auto;
  height: 140px;
  width: 400px;
  background: #fff;   //给融合元素的父元素设置背景色
  display: flex;
  align-items: center;
  justify-content: center;
  filter: contrast(30);    //给融合元素的父元素设置contrast
}
.circle{
  border-radius: 50%;
  position: absolute;
  filter: blur(10px);    //给融合元素设置blur
}
.circle-1{
  height: 90px;
  width: 90px;
  background: #03a9f4;
  transform: translate(-50px);
  animation: 2s moving linear infinite alternate-reverse;
}
.circle-2{
  height: 60px;
  width: 60px;
  background: #0000ff;
  transform: translate(50px);
  animation: 2s moving linear infinite alternate;
}
 @keyframes moving {    //两个元素的移动
  0%{
    transform: translate(50px)
  }
  100%{
    transform: translate(-50px)
  }
}
复制代码
```

**实现融合效果的技术要点：**

1.  `contrast`滤镜应用在融合元素的父元素（`.container`）上，且父元素必须设置`background`。
2.  `blur`滤镜应用在融合元素（`.circle`）上。

`blur`设置图像的模糊程度，`contrast`设置图像的对比度。当两者像上面那样组合时，就会产生神奇的融合效果，你可以像使用公式一样使用这种写法。

在这种融合效果的基础上，我们可以做一些有趣的交互设计。

*   加载动画：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/165fcd62e189433a8e4700cf88d8ceca~tplv-k3u1fbpfcp-watermark.awebp) `html`和`css`如下所示，这个动画主要通过控制子元素`.circle`的尺寸和位移来实现，但是由于父元素和子元素都满足 “融合公式” ，所以当子元素相交时，就出现了融合的效果。

```
<div class="container">
  <div class="circle"></div>
  <div class="circle"></div>
  <div class="circle"></div>
  <div class="circle"></div>
  <div class="circle"></div>
</div>
复制代码
```

```
.container {
  margin: 10px auto;
  height: 140px;
  width: 300px;
  background: #fff;       //父元素设置背景色
  display: flex;
  align-items: center;
  filter: contrast(30);   //父元素设置contrast
}
.circle {
  height: 50px;
  width: 60px;
  background: #1aa7ff;
  border-radius: 50%;
  position: absolute;
  filter: blur(20px);    //子元素设置blur
  transform: scale(0.1);
  transform-origin: left top;
}
.circle{
  animation: move 4s cubic-bezier(.44,.79,.83,.96) infinite;
}
.circle:nth-child(2) {
  animation-delay: .4s;
}
.circle:nth-child(3) {
  animation-delay: .8s;
}
.circle:nth-child(4) {
  animation-delay: 1.2s;
}
.circle:nth-child(5) {
  animation-delay: 1.6s;
}
@keyframes move{     //子元素的位移和尺寸动画
  0%{
    transform: translateX(10px) scale(0.3);
  }
  45%{
    transform: translateX(135px) scale(0.8);
  }
  85%{
    transform: translateX(270px) scale(0.1);
  }
}
复制代码
```

*   酷炫的文字出场方式：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cebab5c1c76c474298d0e1d003b1c121~tplv-k3u1fbpfcp-watermark.awebp) 主要通过不断改变`letter-spacing`和`blur`的值，使文字从融合到分开：

```
<div class="container">
  <span class="text">fantastic</span>
</div>
复制代码
```

```
.container{
  margin-top: 50px;
  text-align: center;
  background-color: #000;
  filter: contrast(30);
}
.text{
  font-size: 100px;
  font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
  letter-spacing: -40px;
  color: #fff;
  animation: move-letter 4s linear forwards;  //forwards当动画完成后，保持最后一帧的状态
}
@keyframes move-letter{
  0% {
    opacity: 0;
    letter-spacing: -40px;
    filter: blur(10px);
  }
  25% {
    opacity: 1;
  }
  50% {
    filter: blur(5px);
  }
  100% {
    letter-spacing: 20px;
    filter: blur(2px);
  }
}
复制代码
```

水波效果
====

filter 还可以通过 URL 链接到 SVG 滤镜元素，[SVG 滤镜元素 MDN](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FSVG%2FElement%2Ffilter "https://developer.mozilla.org/en-US/docs/Web/SVG/Element/filter") 。

下面的水波纹效果就是基于 SVG 的`feTurbulence`滤镜实现的，原理参考了 [说说 SVG 的 feTurbulence 滤镜](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F366438535 "https://zhuanlan.zhihu.com/p/366438535") 和 [SVG feTurbulence 滤镜深入介绍](https://link.juejin.cn?target=https%3A%2F%2Fwww.zhangxinxu.com%2Fwordpress%2F2020%2F10%2Fsvg-feturbulence "https://www.zhangxinxu.com/wordpress/2020/10/svg-feturbulence")，有兴趣的朋友可以深入阅读。

> `feTurbulence滤镜`借助`Perlin`噪声算法模拟自然界真实事物那样的随机样式。它接收下面 5 个属性：
> 
> *   `baseFrequency`表示噪声的基本频率参数，频率越高，噪声越密集。
> *   `numOctaves`就表示倍频的数量，倍频的数量越多，噪声看起来越自然。
> *   `seed`属性表示 feTurbulence 滤镜效果中伪随机数生成的起始值，不同数量的`seed`不会改变噪声的频率和密度，改变的是噪声的形状和位置。
> *   `stitchTiles`定义了 Perlin 噪声在边框处的行为表现。
> *   `type`属性值有`fractalNoise`和`turbulence`，模拟随机样式使用`turbulence`。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/536a053c044f43da9dc2913d4175806e~tplv-k3u1fbpfcp-watermark.awebp)

在这个例子，两个`img标签`使用同一张图片，将第二个`img标签`使用`scaleY(-1)`实现垂直方向的镜像翻转，模拟倒影。

并且，对倒影图片使用`feTurbulence`滤镜，通过动画不断改变`feTurbulence`滤镜的`baseFrequency`值实现水纹波动的效果。

```
<div class="container">
  <img src="images/moon.jpg">
  <img src="images/moon.jpg" class="reflect">
</div>

<!--定义svg滤镜，这里使用的是feTurbulence滤镜-->
<svg width="0" height="0">
    <filter id="displacement-wave-filter">
    
      <!--baseFrequency设置0.01 0.09两个值，代表x轴和y轴的噪声频率-->  
      <feTurbulence baseFrequency="0.01 0.09">
        
        <!--这是svg动画的定义方式，通过动画不断改变baseFrequency的值，从而形成波动效果-->
        <animate attribute
        dur="20s" keyTimes="0;0.5;1" values="0.01 0.09;0.02 0.13;0.01 0.09"
        repeatCount="indefinite" ></animate>
        
      </feTurbulence>
      <feDisplacementMap in="SourceGraphic" scale="10" /> 
    </filter>
</svg>
复制代码
```

```
.container{
   height: 520px;
   width: 400px;
   display: flex;
   clip-path: inset(10px);
   flex-direction: column;
}
img{
  height: 50%;
  width: 100%;
}
.reflect {
  transform: translateY(-2px) scaleY(-1);
  //对模拟倒影的元素应用svg filter
  //url中对应的是上面svg filter的id
  filter: url(#displacement-wave-filter);  
}
复制代码
```

抖动效果
====

在上面的水波动画中改变的是`baseFrequency`值，我们也通过改变`seed`的值，实现文字的抖动效果。 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86c3ebeda5e84915a9db230e5dc5d371~tplv-k3u1fbpfcp-watermark.awebp)

```
<div>
  <p class="shaky">Such a joyful night!</p>
</div>
<svg width="0" height="0">
    <filter id="displacement-text-filter">
    
      <!--定义feTurbulence滤镜-->
      <feTurbulence baseFrequency="0.02" seed="0">
      
       <!--这是svg动画的定义方式，通过动画不断改变seed的值，形成抖动效果-->
        <animate attribute
        dur="1s" keyTimes="0;0.5;1" values="1;2;3"
        repeatCount="indefinite" ></animate>
      </feTurbulence>
      <feDisplacementMap in="SourceGraphic" scale="10" /> 
    </filter>
  </svg>
复制代码
```

```
.shaky{
  font-size: 60px;
  filter: url(#displacement-text-filter);   //url中对应的是上面svg filter的id
}
复制代码
```

参考
==

[【CSS】水滴动画｜水滴融合效果](https://link.juejin.cn?target=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2Fav584118838%2F "https://www.bilibili.com/video/av584118838/")

[你所不知道的 CSS 滤镜技巧与细节](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Fcoco1s%2Fp%2F7519460.html "https://www.cnblogs.com/coco1s/p/7519460.html")

[说说 SVG 的 feTurbulence 滤镜](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F366438535 "https://zhuanlan.zhihu.com/p/366438535")

[SVG feTurbulence 滤镜深入介绍](https://link.juejin.cn?target=https%3A%2F%2Fwww.zhangxinxu.com%2Fwordpress%2F2020%2F10%2Fsvg-feturbulence "https://www.zhangxinxu.com/wordpress/2020/10/svg-feturbulence")