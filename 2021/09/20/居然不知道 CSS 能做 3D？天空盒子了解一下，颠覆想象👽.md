> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6997697496176820255) 可以这个链接来查看，three.js 来实现的，戳👇[three.js 全景图 DEMO 链接](https://link.juejin.cn?target=https%3A%2F%2Fthreejs.org%2Fexamples%2F%3Fq%3Dpano%23webgl_panorama_cube "https://threejs.org/examples/?q=pano#webgl_panorama_cube")。

其实我们通过 CSS3 也能实现类似的效果，而且性能上更好，兼容性更好，支持低端机型。

是不是很惊讶，CSS 居然也能做这种事情？

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/451c824ce87c49deb3eaf13fd4096080~tplv-k3u1fbpfcp-watermark.awebp)

好了，放放手上的事情，花 10 多分钟专心致志🐶，羽飞老师的课开始了。

注意⚠️：建议 PC 端观摩，因为有挺多例子需要查看后理解更好，不过也不太影响，为手机同学准备了比较多的 gif 图，准备地好疲乏🥱。

由于本文重点在最后章，文中借用了一些 DEMO 方便快速带入，可能有所纰漏，欢迎各位大佬拍砖🧱、吐槽💬。

〇 背景
====

17 年双十一前夕，其实也前不了多少天（大家都懂），产品找到我，说要做它，赶在双十一前上线，然后就有了它🐶。

开门见山，直接甩上成品给大家看看。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d76343de0674a0d93eb39cef83afd7e~tplv-k3u1fbpfcp-watermark.awebp)

那...... 我就开动啦。我们先看看成品是长啥样的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/562beb11423244f7823666ec92aa91b5~tplv-k3u1fbpfcp-watermark.awebp)

可以查看这个，👇[CSS 全景图 DEMO 链接](https://link.juejin.cn?target=https%3A%2F%2Fyun.dui88.com%2Ftuia%2Fjunhehe%2Findex.html "https://yun.dui88.com/tuia/junhehe/index.html")。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55365be5dccf48a2af7871c8dc52d3ce~tplv-k3u1fbpfcp-watermark.awebp)

或者通过如上 CSS 全景图 DEMO 二维码进行尝试。

如果是 “尊贵” 的苹果手机用户🐶，在 iOS13 以上需要允许陀螺仪才可，如下图，**得点击屏幕授权通过**。iOS13 之前都是默认开启的，苹果真的是一点不考虑向下兼容🥲，有点霸道呀。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62ee047906644a45a9c84ecce98f0854~tplv-k3u1fbpfcp-watermark.awebp)

扯远了扯远了，收。

这个时候大家就可以通过旋转手机或拖拽来查看整个全景图了。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0aa24854c1b145b0b27c1655076311ba~tplv-k3u1fbpfcp-watermark.awebp)

是不是还挺神奇的？不是？

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a6ff938cb2d4dbc9d7b44d7a5cd8f34~tplv-k3u1fbpfcp-watermark.awebp)

还是不是？🐶。🦢🦢🦢，不能向苹果学习🐶。

回来回来，接下来讲讲原理，先看看前置知识点。

〇 前置知识
======

看问题先看全貌，我们先来了解下如题中所提的天空盒子是什么概念。

天空盒子
----

天空盒子其实通俗的理解，可以理解如果把你放到天空中，上下前后左右都是蓝色的天空。而这个天空可以简单的用六边形来实现。

如下图所示，六边组成了一个封闭空间。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c741130414244db7a00d8e47f3ec6e96~tplv-k3u1fbpfcp-watermark.awebp)

如果把你放到这个空间里，然后把每个空间的墙壁弄成天蓝色，而且每面都是纯蓝天色，这样你就分辨不出自己是不是在天上，还是只是在一个封闭的天空盒子里。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc90292015ca453eb08be9487af64303~tplv-k3u1fbpfcp-watermark.awebp)

细思极恐，让人想到了缸中之脑，没听过的同学可以看看[百度百科的缸中之脑解释](https://link.juejin.cn?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E7%25BC%25B8%25E4%25B8%25AD%25E4%25B9%258B%25E8%2584%2591%2F6185744 "https://baike.baidu.com/item/%E7%BC%B8%E4%B8%AD%E4%B9%8B%E8%84%91/6185744")。

好了，回归主题👻。这样一个天空盒子就形成了一个全景空间图。

那 CSS 是要怎么才能实现一个天空盒子呢？我们继续。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3060ae34129c42d5be9f37300c303af4~tplv-k3u1fbpfcp-watermark.awebp)

CSS 3D 坐标系
----------

先来了解一下坐标系的概念。

从二维 “反降维” 到三维，需要理解下这个坐标系。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8f8a2251fc64760a5d7f01ffbb0e330~tplv-k3u1fbpfcp-watermark.awebp)

我们可以看到增加一个 Z 纬度的线，平面就变 3D 了。

这里需要注意的是 CSS3D 中，上下轴是 Y 轴，左右轴是 X 轴，前后轴是 Z 轴。可以简单理解为在原有竖着的面对我们的平面中，在 X 和 Y 轴中间强行插入一根直线，与 Y 轴和 X 轴都成 90 度，这根直线就是 Z 轴。

通过上面的处理，这样就形成了一个空间坐标系。

这有什么用呢？

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b66e632014342c7924d09b57196c20b~tplv-k3u1fbpfcp-watermark.awebp)

大家可能有点懵逼，感觉二维都没搞定，突然要搞三维了。

可以先看看这个 3D 坐标系的 DEMO，👇[链接在此](https://link.juejin.cn?target=https%3A%2F%2Fyun.tuia.cn%2Ftuia%2Fshare%2Fdemo.html "https://yun.tuia.cn/tuia/share/demo.html")，可以先随意把玩把玩。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79ea7aacc61347e5a99ca68dbbf6103f~tplv-k3u1fbpfcp-watermark.awebp)

可以看到途中绿色线就是 Z 轴，红色就是 X 轴，蓝色就是 Z 轴。

多玩一玩就有点感觉啦，是不是感觉逐渐有了 3D 空间的感觉。

没有？

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ca20c8c6b654ad09d3ecdb392582ef4~tplv-k3u1fbpfcp-watermark.awebp)

其他同学们，不要他了，我们继续。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b6627ae155045c7a43765c4b56eaab9~tplv-k3u1fbpfcp-watermark.awebp)

不管你了，辛苦做了好久的 DEMO🐶。继续继续。

如果想深入了解此 CSS 3D 坐标系演示的 DEMO，源码可以查看这里，👇[链接在此](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ffly0o0%2Fcss3d-demo%2Ftree%2Fmain%2Fcss-3d "https://github.com/fly0o0/css3d-demo/tree/main/css-3d")。

说到 CSS 3D，肯定离不开 CSS3D transform，下面开始学习。

CSS 3D transform
----------------

3D transform 字面意思翻译过来为三维变换。

### 3D rotate

我们先从 rotate 3d（旋转）开始，这个能辅助我们理解 3D 坐标系。

**rotate X**

单杠运动员，如果正面对着我们，就是可以理解为围着 X 转。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9618810d27247feb5e49a402526c45a~tplv-k3u1fbpfcp-watermark.awebp)

**rotate Y**

围着钢管转，就可以理解为围着 Y 轴在转。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7984e0c9b43449bbd05957c6059ac2a~tplv-k3u1fbpfcp-watermark.awebp)

**rotate Z**

如果我们正面对着摩天轮，其实摩天轮就在围着 Z 轴在做运动，中间那个白点，可以理解为 Z 轴从这个圆圈穿透过去的点。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4a52b44bcba4bba992c2671d0cdc5e1~tplv-k3u1fbpfcp-watermark.awebp)

如果还没理解的同学，可以通过之前的 CSS3D DEMO，👇[链接在此](https://link.juejin.cn?target=http%3A%2F%2Fyun.tuia.cn%2Ftuia%2Fshare%2Fdemo.html "http://yun.tuia.cn/tuia/share/demo.html")，辅助理解 3D rotate。

理解了 3D rotate 后，可以辅助我们理解三维坐标系。下面我们开始讲解 perspective，有一些理解的难度哦。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d1c4fa0be734268bd072198f634ae45~tplv-k3u1fbpfcp-watermark.awebp)

### perspective

perspective 是做什么用的呢？字面意思是视角、透视的意思。

有一种我们从小到大看到的想象，可能我们都并不在意了，就是现实生活中的透视。比如同样的电线杆，会进高远低。其实这个现象是有一些规律的：近大远小、近实远虚、近宽远窄。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f60dde766dec44539d01363d0061af85~tplv-k3u1fbpfcp-watermark.awebp)

因此在素描、建筑的行业，都会通过一种透视的方式来表达现实世界的 3D 模型。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98091175a7fd40fd911a456df4a10c15~tplv-k3u1fbpfcp-watermark.awebp)

而我们在计算机世界怎么表达 3D 呢？

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a59b4595c38f4865a20ef26e83cc8b4f~tplv-k3u1fbpfcp-watermark.awebp)

上方图可以辅助大家理解 3D 的透视 perspective，黄色的是电脑或手机屏幕，红色是屏幕里的方块。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/344717c0275f4067b83b71fe296cade1~tplv-k3u1fbpfcp-watermark.awebp)

再看看上面这个二维图，可以看到，perspective: 800，代表 3D 物体距离屏幕（中间那个平面）是 800px。

这里还有个概念，perspective-origin，可以看到上面 perspective-origin 是 50% 50%，可以理解为眼睛视角的中心点，分别在 x 轴、y 轴（x 轴 50%，y 轴 50%）交叉处。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2d844b52d3d4656a17d098780554730~tplv-k3u1fbpfcp-watermark.awebp)

没事没事，如果上面这些还不够你理解的，可以看看下面这张图。再不懂就不管你了🐶。

「下图来自：[CSS 3D - Scrolling on the z-axis | CSS | devNotes](https://link.juejin.cn?target=https%3A%2F%2Fvinceumo.github.io%2FdevNotes%2FCSS%2Fcss-3d-scrolling-on-the-z-axis%2F "https://vinceumo.github.io/devNotes/CSS/css-3d-scrolling-on-the-z-axis/")」 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d7283da7cdf4eca857e5c4ad2fa15f6~tplv-k3u1fbpfcp-watermark.awebp)

上图里的 Z 就是 Z 轴的值。Z 轴如果是正数的离屏幕更近，如果是负数离屏幕更远。

而 Z 轴的远近和 translateZ 分不开，下面来讲解 translateZ。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fbecf11831a4104a70a3a904eaddc88~tplv-k3u1fbpfcp-watermark.awebp)

### translateZ

这个属性可以帮助我们理解 perspective。

可以通过 translate 的 DEMO 进行把玩把玩，有助于理解，戳👇[DEMO 链接在此](https://link.juejin.cn?target=https%3A%2F%2Fyun.tuia.cn%2Ftuia%2Fshare%2Fdemo.html "https://yun.tuia.cn/tuia/share/demo.html")。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9168cca9ffab490bb51b298886a4556f~tplv-k3u1fbpfcp-watermark.awebp)

translateZ 实现了 CSS3D 世界空间的近大远小。

看一下这个例子，平面上的 translateZ 的变换，戳👇[DEMO 链接在此](//yun.dui88.com/tuia/junhehe/demo/translateZ.html "//yun.dui88.com/tuia/junhehe/demo/translateZ.html")。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84da1349e0b048b98d3f7f0b93fcce5c~tplv-k3u1fbpfcp-watermark.awebp)

比如，我们设置元素 perspective 为 201px，则其子元素的 translateZ 值越小，则看着越小；如果 translateZ 值越大，则看着越大。当 translateZ 为 200px 的时候，该元素会撑满屏幕，当超过 201px 时候，该元素消失了，跑到我们眼睛后面了。

平面上的 translateZ 感受完了，来试试三维下的，看看这个 DEMO，戳👇[链接在此](//yun.dui88.com/tuia/junhehe/demo/3d-translateZ.html "//yun.dui88.com/tuia/junhehe/demo/3d-translateZ.html")。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/711eed87b0e04451bde31e656b298cd0~tplv-k3u1fbpfcp-watermark.awebp)

上图中，如果把 perspective 往左拖，可以发现 front 面会离我们越来越远，如果往右拖，反之。

通过这么一节，基本 translateZ 的作用，大家应该都能理解到位了，还没有？回头看看🐶。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bba23ffa48149b2aa68b9e024403904~tplv-k3u1fbpfcp-watermark.awebp)

### 模拟现实 3D 空间

其实计算机的 3D 世界就是现实 3D 世界的模拟。而和计算机的 3D 世界中，构建 3D 空间概念很相近的现实场景，是摄像。我们可以考虑一下如果你去拍照，会有几个要素？

第一个：镜头，第二个：拍摄的环境的空间，第三个：要拍摄的物件。

「下图来自[搞懂 CSS 3D，你必须理解 perspective（视域）](https://link.juejin.cn?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1485924 "https://cloud.tencent.com/developer/article/1485924")」

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d04aac70d1b4ad7973b15a94447c656~tplv-k3u1fbpfcp-watermark.awebp)

而在 CSS 的 3D 世界，我们也需要去模仿这三要素。我们用三层 div 来表示，第一层是摄像镜头、第二层是立体空间或也可叫舞台，第三层是立体空间内的元素。

大致的 HTML 代码如下。

```
<div class="camera">
    <div class="space">
        <div class="box"> 
        </div>
    </div>
</div>
复制代码
```

下面就是真枪实弹地干了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f57392076364bf9bf688527cbcfd196~tplv-k3u1fbpfcp-watermark.awebp)

〇 实现天空盒子
========

已经知道了足够的前置知识，我们来简单实现一下天空盒子。

六面盒子
----

需要生成前后、左右、上下六个面。首先我们想一下第一面前面应该怎么放？

### 前面墙

假设我们在天空盒子（是一个正方体 1024px*1024px），我们在正方体里面的中心点，那我们要往前面的墙上贴一张图，需要做什么？

我们回顾下坐标系。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d804fd4898104c0e97a0a1492b7f01ed~tplv-k3u1fbpfcp-watermark.awebp)

你可以想象自己站在 x 轴和 y 轴交叉的中心点，即你在正方体的中心点。则你的前面的墙就是在 z 为 - 512px 处，因为是前面，我们无需对这个墙进行旋转。

```
<html>
<head>
  <title>CSS3D天空盒子</title>
  <style>
    html,
    body {
      margin: 0;
      overflow: hidden;    
      background-color: #ccc;
    }
    .camera {
      perspective: 512px;
      perspective-origin:50% 50%;
    }
    .space {
      width: 1024px;
      height: 1024px;
      margin: 0 auto;
      transform-style: preserve-3d;
    }
    .space img {
      width: 1024px;
      height: 1024px;
      position: absolute;
    }
    .space .front {
      /* 正面的图无需旋转 */
      transform: rotateZ(0) rotateY(0) rotateZ(0) translateZ(-512px);
    }
  </style>
</head>

<body>
  <div class="camera" id="camera">
    <div class="space">
      <img class="front" src="//yun.dui88.com/tuia/junhehe/skybox/front.jpg" alt="" />
    </div>
  </div>
</body>
</html>
复制代码
```

生成如下页面，[演示代码地址](//yun.dui88.com/tuia/junhehe/step1-front.html "//yun.dui88.com/tuia/junhehe/step1-front.html")：。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0c7d7d3dfac43268b8cb94dc628a4bb~tplv-k3u1fbpfcp-watermark.awebp)

可以看到第一张图被放在了前面。

### 左面墙

从前面墙放上一张图，然后转向左面墙，需要几步走？

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/381331c3cb8547638e91d04fe2ff9fd3~tplv-k3u1fbpfcp-watermark.awebp)

第一步，需要让平面与前面的墙垂直，这个时候我们需要把左面的图绕着 Y 轴旋转 90 度。

左面墙的图本应该放在 X 轴的 - 512px 位置，但由于做了旋转，所以左面墙对应的坐标系也做了绕着 Y 轴向下旋转了 90 度。如果我们想把左侧的图放到对应的位置，我们需要让其在 Z 轴的 - 512px 位置。

因此代码如下。

```
<html>
<head>
  <title>CSS3D天空盒子</title>
  <style>
    html,
    body {
      margin: 0;
      overflow: hidden;    
      background-color: #ccc;
    }
    .camera {
      perspective: 512px;
      perspective-origin:50% 50%;
    }
    .space {
      width: 1024px;
      height: 1024px;
      margin: 0 auto;
      transform-style: preserve-3d;
    }
    .space img {
      width: 1024px;
      height: 1024px;
      position: absolute;
    }
    .space .front {
      /* 正面的图无需旋转 */
      transform: rotateZ(0) rotateY(0) rotateZ(0) translateZ(-512px);
    }
    .space .left {
      transform: rotateY(90deg) translateZ(-512px);
    }
  </style>
</head>
<body>
  <div class="camera" id="camera">
    <div class="space">
      <img class="front" src="//yun.dui88.com/tuia/junhehe/skybox/front.jpg" alt="" />
      <img class="left" src="//yun.dui88.com/tuia/junhehe/skybox/left.jpg" alt="" />
    </div>
  </div>
</body>
</html>
复制代码
```

生成的页面如下，[演示代码地址](//yun.dui88.com/tuia/junhehe/step2-left.html "//yun.dui88.com/tuia/junhehe/step2-left.html")。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb87c87d92af4bfea23c51c0943974a9~tplv-k3u1fbpfcp-watermark.awebp)

可以看到左面墙确实生成在了前面墙的左侧。

### 底面

类似前面墙、左面墙，我们把底面，做了绕着 X 轴旋转 90 度，然后沿着 Y 轴走 - 512px。

代码如下。

```
<html>
<head>
  <title>CSS3D天空盒子</title>
  <style>
    html,
    body {
      margin: 0;
      overflow: hidden;    
      background-color: #ccc;
    }
    .camera {
      perspective: 512px;
      perspective-origin:50% 50%;
    }
    .space {
      width: 1024px;
      height: 1024px;
      margin: 0 auto;
      transform-style: preserve-3d;
    }
    .space img {
      width: 1024px;
      height: 1024px;
      position: absolute;
    }
    .space .front {
      /* 正面的图无需旋转 */
      transform: rotateZ(0) rotateY(0) rotateZ(0) translateZ(-512px);
    }
    .space .left {
      transform: rotateY(90deg) translateZ(-512px);
    }
    .space .bottom {
      transform: rotateX(90deg) translateZ(-512px);
    }
  </style>
</head>
<body>
  <div class="camera" id="camera">
    <div class="space">
      <img class="front" src="//yun.dui88.com/tuia/junhehe/skybox/front.jpg" alt="" />
      <img class="left" src="//yun.dui88.com/tuia/junhehe/skybox/left.jpg" alt="" />
      <img class="bottom" src="//yun.dui88.com/tuia/junhehe/skybox/bottom.jpg" alt="" />
    </div>
  </div>
</body>
</html>
复制代码
```

生成页面如下，[演示代码地址](//yun.dui88.com/tuia/junhehe/step3-bottom.html "//yun.dui88.com/tuia/junhehe/step3-bottom.html")。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5686b23ed1914e77b257b28f117960b1~tplv-k3u1fbpfcp-watermark.awebp)

可以看到我们底部也有了，看看所有面集成后是什么样。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74664d8ffe304bb984381e2d4e393bec~tplv-k3u1fbpfcp-watermark.awebp)

### 所有面

类似上面的操作，我们把六个面补全，下面我们就把六个面都集合起来。

```
<html>
<head>
  <title>CSS3D天空盒子</title>
  <style>
    html,
    body {
      overflow: hidden;    
      margin: 0;
    }
    .camera {
      perspective: 512px;
      perspective-origin:50% 50%;
    }
    .space {
      width: 1024px;
      height: 1024px;
      margin: 0 auto;
      transform-style: preserve-3d;
    }
    .space img {
      width: 1024px;
      height: 1024px;
      position: absolute;
    }
    .space .front {
      /* 正面的图无需旋转 */
      transform: rotateZ(0) rotateY(0) rotateZ(0) translateZ(-512px);
    }
    .space .back {
      transform: rotateY(180deg) translateZ(-512px);
    }
    .space .left {
      transform: rotateY(90deg) translateZ(-512px);
    }
    .space .right {
      transform: rotateY(-90deg) translateZ(-512px);
    }
    .space .bottom {
      transform: rotateX(90deg) translateZ(-512px);
    }
    .space .top {
      transform: rotateX(-90deg) translateZ(-512px);
    }
  </style>
</head>
<body>
  <div class="camera" id="camera">
    <div class="space">
      <img class="front" src="//yun.dui88.com/tuia/junhehe/skybox/front.jpg" alt="" />
      <img class="back" src="//yun.dui88.com/tuia/junhehe/skybox/back.jpg" alt="" />
      <img class="left" src="//yun.dui88.com/tuia/junhehe/skybox/left.jpg" alt="" />
      <img class="right" src="//yun.dui88.com/tuia/junhehe/skybox/right.jpg" alt="" />
      <img class="bottom" src="//yun.dui88.com/tuia/junhehe/skybox/bottom.jpg" alt="" />
      <img class="top" src="//yun.dui88.com/tuia/junhehe/skybox/top.jpg" alt="" />
    </div>
  </div>
</body>
</html>
复制代码
```

生成页面如下，[演示代码地址](//yun.dui88.com/tuia/junhehe/step4-all.html "//yun.dui88.com/tuia/junhehe/step4-all.html")。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bda6d0c2b7884695aec8d528a7ae08d1~tplv-k3u1fbpfcp-watermark.awebp)

我们发现看不到后方墙（背面墙）。所以我们打算把整个场景转起来。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c41aab536ac4ee389121f22d58f5d24~tplv-k3u1fbpfcp-watermark.awebp)

盒子旋转
----

怎么才能把盒子进行旋转？这里需要对六面墙所在的场景，也即是它们上一层的元素。

我们给. cube 加上一个动画效果，绕着 Y 轴钢管舞🐶，回忆起前置知识里的钢管舞没？

```
<html>
<head>
  <title>CSS3D天空盒子</title>
  <style>
    html,
    body {
      overflow: hidden;
      margin: 0;
    }
    .camera {
      perspective: 512px;
      perspective-origin:50% 50%;
      
    }
    .space {
      width: 1024px;
      height: 1024px;
      margin: 0 auto;
      transform-style: preserve-3d;
    }
    .space img {
      width: 1024px;
      height: 1024px;
      position: absolute;
    }
    .space .front {
      /* 正面的图无需旋转 */
      transform: rotateZ(0) rotateY(0) rotateZ(0) translateZ(-512px);
    }
    .space .back {
      transform: rotateY(180deg) translateZ(-512px);
    }
    .space .left {
      transform: rotateY(90deg) translateZ(-512px);
    }
    .space .right {
      transform: rotateY(-90deg) translateZ(-512px);
    }
    .space .bottom {
      transform: rotateX(90deg) translateZ(-512px);
    }
    .space .top {
      transform: rotateX(-90deg) translateZ(-512px);
    }
    @keyframes rot {
      0% {
        transform: rotateY(0deg)
      }

      10% {
        transform: rotateY(90deg)
      }

      25% {
        transform: rotateY(90deg)
      }

      35% {
        transform: rotateY(180deg)
      }

      50% {
        transform: rotateY(180deg)
      }

      60% {
        transform: rotateY(270deg)
      }

      75% {
        transform: rotateY(270deg)
      }

      85% {
        transform: rotateY(360deg)
      }

      100% {
        transform: rotateY(360deg)
      }
    }
    /*为立方体加上帧动画*/
    .space {
      animation: rot 8s ease-out 0s infinite forwards;
    }
  </style>
</head>
<body>
  <div class="camera" id="camera">
    <div class="space">
      <img class="front" src="//yun.dui88.com/tuia/junhehe/skybox/front.jpg" alt="" />
      <img class="back" src="//yun.dui88.com/tuia/junhehe/skybox/back.jpg" alt="" />
      <img class="left" src="//yun.dui88.com/tuia/junhehe/skybox/left.jpg" alt="" />
      <img class="right" src="//yun.dui88.com/tuia/junhehe/skybox/right.jpg" alt="" />
      <img class="bottom" src="//yun.dui88.com/tuia/junhehe/skybox/bottom.jpg" alt="" />
      <img class="top" src="//yun.dui88.com/tuia/junhehe/skybox/top.jpg" alt="" />
    </div>
  </div>
</body>
</html>
复制代码
```

生成页面动画效果如下，这次用的手机拍摄的更真实一些😂，虽然有点糊，[演示代码地址](//yun.dui88.com/tuia/junhehe/step5-animate.html "//yun.dui88.com/tuia/junhehe/step5-animate.html")。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be9d1faf148e4e5fa63824de6fb08a45~tplv-k3u1fbpfcp-watermark.awebp)

既然能自动旋转，我们是不是可以考虑用手动旋转呢？

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76be36c23058479d874286b927b89366~tplv-k3u1fbpfcp-watermark.awebp)

手动旋转
----

大概原理，就是手动拖拽（手机是 touchmove，PC 是 mousemove），拖拽过去走的多少路程，计算出角度，然后把这个角度通过 DOM 设置（这个过程通过 requestAnimationFrame 不停地轮询设置）。

启动手动拖拽的代码。

```
var curMouseX = 0;
var curMouseY = 0;
var lastMouseX = 0;
var lastMouseY = 0;

if (isAndroid || isiOS) {
  document.addEventListener('touchstart', mouseDownHandler);
  document.addEventListener('touchmove', mouseMoveHandler);
} else {
  document.addEventListener('mousedown', mouseDownHandler);
  document.addEventListener('mousemove', mouseMoveHandler);
}

function mouseDownHandler(evt) {
  lastMouseX = evt.pageX || evt.targetTouches[0].pageX;
  lastMouseY = evt.pageY || evt.targetTouches[0].pageY;
}

function mouseMoveHandler(evt) {
  curMouseX = evt.pageX || evt.targetTouches[0].pageX;
  curMouseY = evt.pageY || evt.targetTouches[0].pageY;
}
复制代码
```

具体的不分析了，不是本次的重点。有兴趣的可以直接看代码深入。

且由于我们想使用在手机上，因此做了 rem 的适配，适配在手机端。

生成页面动画效果如下，[演示代码地址](//yun.dui88.com/tuia/junhehe/step6-touch.html "//yun.dui88.com/tuia/junhehe/step6-touch.html")。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aedd0237a12f4e14972f1592167f8bbe~tplv-k3u1fbpfcp-watermark.awebp)

上面是手机录制的旋转视频。既然我们能通过手触旋转，那我们肯定也可以进行陀螺仪旋转。

陀螺仪旋转
-----

大致原理也是如上，把手动拖拽换成了陀螺仪旋转，然后计算旋转角度。

启动陀螺仪的代码。

```
window.addEventListener('deviceorientation', motionHandler, false)
function motionHandler(event) {
  var x = event.beta;  
  var y = event.gamma;
}
复制代码
```

自开头所说，陀螺仪在 IOS13 + 下需要授权。

```
var isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); // ios??
if (isiOS) {
  permission()
} 

function permission () {
  if ( typeof( DeviceMotionEvent ) !== "undefined" && typeof( DeviceMotionEvent.requestPermission ) === "function" ) {
    // (optional) Do something before API request prompt.
    DeviceMotionEvent.requestPermission()
      .then( response => {
      // (optional) Do something after API prompt dismissed.
      if ( response == "granted" ) {
        window.addEventListener( "devicemotion", (e) => {
          // do something for 'e' here.
        })
      }
    })
      .catch( console.error )
  } else {
    alert( "请使用手机浏览器" );
  }
}
 
复制代码
```

下面是手机录制展示陀螺仪的例子，生成页面动画效果如下，[演示代码地址](//yun.dui88.com/tuia/junhehe/step7-orientation.html "//yun.dui88.com/tuia/junhehe/step7-orientation.html")。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f255b2e4aeb4cf5b17cea51be5dbefa~tplv-k3u1fbpfcp-watermark.awebp)

这里想深入的同学，可以看一下代码，和上面一样不是本文的重点就不分析了。

有没有感觉写了这么多代码，感觉跟写纯 JS 操作 DOM 似的，有没有类似 JQuery 之类的库呢？

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a2240acd7f246ea8590069b9aaa1117~tplv-k3u1fbpfcp-watermark.awebp)

css3d-engine
------------

上面只是实现了平行旋转，要实现任意角度旋转，我们是基于 css3d-engine 做了实现。

这一节只是带过，理解了大概的原理后，结合例子去学习这个库还是非常快的。

### 部分示例代码

文章第一个 DEMO 就是以这个库为基础进行实践的，地址在这里：[github.com/shrekshrek/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fshrekshrek%2Fcss3d-engine "https://github.com/shrekshrek/css3d-engine")。

创建 stage，stage 是舞台，是整个场景的根。

```
var s = new C3D.Stage();  
复制代码
```

创建一个天空盒子的例子，控制各面的素材。

```
//创建1个立方体放入场景
var c = new C3D.Skybox();
c.size(1024).position(0, 0, 0).material({
  front: {image: "images/cube_FR.jpg"},
  back: {image: "images/cube_BK.jpg"},
  left: {image: "images/cube_LF.jpg"},
  right: {image: "images/cube_RT.jpg"},
  up: {image: "images/cube_UP.jpg"},
  down: {image: "images/cube_DN.jpg"},
}).update();
s.addChild(c);
复制代码
```

### Tween 制作动效

第一个 DEMO 中动效，是通过 Tween.js 实现的，地址在这里：[github.com/sole/tween.…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsole%2Ftween.js "https://github.com/sole/tween.js")。

为什么 DOM 元素会有动效，也是因为属性值的变化，而 Tween 可以控制属性值在一段时间内按规定的规律变化。

下面是一个 Tween 的示例。

```
var coords = { x: 0, y: 0 };
var tween = new TWEEN.Tween(coords)
	.to({ x: 100, y: 100 }, 1000)
	.onUpdate(function() {
		console.log(this.x, this.y);
	})
	.start();

requestAnimationFrame(animate);

function animate(time) {
	requestAnimationFrame(animate);
	TWEEN.update(time);
}
复制代码
```

在最后再体验一下整个处理好后的 DEMO，重新感受一下。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0da12c28e644694aa3f6ab0d12d2b03~tplv-k3u1fbpfcp-watermark.awebp)

具体的完整版 DEMO 的源码在此，有兴趣的可以深入研究，由于是之前早几年做的 DEMO，代码比较乱，还请见谅，地址在此：[github.com/fly0o0/css3…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ffly0o0%2Fcss3d-demo%2Ftree%2Fmain%2Fh5-pano "https://github.com/fly0o0/css3d-demo/tree/main/h5-pano")。

〇 尾声
====

还是回应下开头，希望大家多拍砖、建议，谢谢😘。

结语
--

**回顾作者往期高赞文章，可能有意想不到的收获！**

*   《[一篇讲透自研的前端错误监控](https://juejin.cn/post/6987681953424080926 "https://juejin.cn/post/6987681953424080926")》，720 + 点赞量
    
*   《[你不可能知道的骨架屏玩法🐶](https://juejin.cn/post/6994678354200756238 "https://juejin.cn/post/6994678354200756238")》，650 + 点赞量
    
*   《[你可能不知道的动态组件玩法🍉](https://juejin.cn/post/6992483283187531789 "https://juejin.cn/post/6992483283187531789")》，390 + 点赞量
    

**😘点赞 + 评论 + 转发😘**，原创一篇不易，可能要肝好几个凌晨😂，求鼓励写更多的文章。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80df0301a3874e82a8808389075a0361~tplv-k3u1fbpfcp-watermark.awebp)

扩展阅读
----

> [【拇指期刊 vol.8】案例二：CSS3 动画之 3D 动画 - 数英](https://link.juejin.cn?target=https%3A%2F%2Fwww.digitaling.com%2Farticles%2F24608.html "https://www.digitaling.com/articles/24608.html")
> 
> [Perspective · Intro to CSS 3D transforms](https://link.juejin.cn?target=https%3A%2F%2F3dtransforms.desandro.com%2Fperspective "https://3dtransforms.desandro.com/perspective")
> 
> [Css3 Transform 的一些例子](https://link.juejin.cn?target=https%3A%2F%2Fcodepen.io%2Fvineethtrv%2Fpen%2FXKKEgM "https://codepen.io/vineethtrv/pen/XKKEgM")
> 
> [CSS 3D Panorama（全景） - 淘宝造物节技术剖析 - SegmentFault 思否](https://link.juejin.cn?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000006880856 "https://segmentfault.com/a/1190000006880856")
> 
> [「CSS 3D 专题」搞懂 CSS 3D，你必须理解 perspective（视域）这个属性 - 云 + 社区 - 腾讯云](https://link.juejin.cn?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1485924 "https://cloud.tencent.com/developer/article/1485924")
> 
> [CSS Perspective, WHW(what?, how?, when?) | by Neel Dedhia | Medium | Medium](https://link.juejin.cn?target=https%3A%2F%2Fmedium.com%2F%40neeldedhia%2Fcss-perspective-whw-286262a505d8 "https://medium.com/@neeldedhia/css-perspective-whw-286262a505d8")