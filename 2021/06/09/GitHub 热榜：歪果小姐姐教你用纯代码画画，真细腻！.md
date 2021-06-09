> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg3MDYwMzkwMw==&mid=2247484256&idx=1&sn=7e00a0bd96b22ce437de22aaf4b7e5a6&chksm=ce8a0e70f9fd876627280539e036eb4394016a7cc4c77237a92a591d76604e08188c79860f9d&mpshare=1&scene=1&srcid=0609cEvHefvWODasIxJ6MZRX&sharer_sharetime=1623222000191&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

转自量子位

HTML 不是编程语言，但这并不妨碍精通它的大佬玩出花来。

普通的前端，用 HTML+CSS 制作网页，元素简单，工具丰富。

大佬级前端，用 HTML+CSS 绘画，全程不用 PS、AI 这种图形化的图片编辑器，单纯敲一行行代码纯手工绘制。

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveg4scQat4Ol0ylnwr8OECiatmiacGs46IQMIeQsqLnqTRHXpbWSfplluQ/640?wx_fmt=jpeg)

把代码转换之后，就变成了鲜嫩的水果：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxve8rBz4gic4jEaJsUhWiamApCjxsCdtNE1D24RTU2Abem4oib8YNI03Sa5g/640?wx_fmt=jpeg)

或者画出洛可可风格的古典女性肖像：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvehCzTvlNQDN6z1Hw9icVZow8mvUB2EYKvQBZegZLDXt6hzXc9teV5J7A/640?wx_fmt=jpeg)

还有弗拉芒巴洛克肖像风格的人物画像，充满了中世纪的禁欲感：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveSWEbiaAnXLuVB0F4hFibetRqE3svpDJtYr1vNJKhxic5VNQvxTl2KAS4g/640?wx_fmt=jpeg)

现代的也有，比如这位在粉色灯光下的着礼服的妹子：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveFw8QEgbtoM2EiaicX6YFfiaGYnRzw8XGI9p9HSqjpnI8KoqgV506ruaKw/640?wx_fmt=jpeg)

以及充满者 50 年代气息的复古风人物海报：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxve8C7uJpuzBKpkR0b7Mx82UANEzFLevrCVRheJgFFU7Ang8v3a8ibm7sw/640?wx_fmt=jpeg)

曲线、光影、渐变，每个元素都相当复杂。

而且，创作过程中不用 SVG，只用 Atom 文本编辑器和 Chrome 开发者工具。

也就是说，画面上的每一条曲线和渐变、每一处高光和阴影、每一根头发和睫毛、每一片蕾丝和褶皱，都是一行行代码从头敲出来的！

如此精细程度和创造力，让学美术的网友感叹 “学画画不如写代码”，让学计算机的同学觉得 “别人写的这么艺术，一定是我的教科书打开方式不对”。

真・交叉学科大佬。

这个项目也一度登上了 GitHub Trending 排行榜第二名：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveyhN5iavXHWydtdr4S6dnNJEV4szeW0HJUb4ythegefopp5NJ602PJ2g/640?wx_fmt=jpeg)

并且 Issues 里都是诸多用户的膜拜：厉害！崇拜！太棒了！  

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveia6BqhplotmYNumuNBOLDbwQt9VsT6ploQhwYKiaTCKJmE2lqvUg5xEg/640?wx_fmt=jpeg)

它们的作者，是湾区前端大神 Diana Smith 小姐姐，她目前是企业及软件开发商 Atlassian 的一名资深 Web 开发。

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveAUUuB1RJEU0iawXvI6m3MwHveqOgGfofMicQ1ibPZK3vHqGt6f4SxGN8g/640?wx_fmt=jpeg)

绘制过程
----

Diana 在专门讨论 CSS 的网站 CSS-Tricks 写下了详细的教程。

画出这样一个图形分成几步？

![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxve0tictVxYCSGEAVH2CibaanYtCqdLufia2kqmIWicAcibbj0AljH6NvPDicnw/640?wx_fmt=png)

如果不用 CSS，一般都是直接嵌入这个特殊的图形。

如果用 CSS，那么就从黑色矩形开始，然后在两侧加上上两个

与白色背景颜色匹配的边框半径元素。

![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxven6x7ABlfkXd97KSgKBrDTBblpibhKXMZC0Prib6bXFeTF04hKNQUrHfA/640?wx_fmt=png)

先画出一个黑色矩形，然后两边用圆弧遮挡。有了基础形状后，下一步就是给它添上渐变的背景。但是如果用矩形方式填充，得到的效果就是这样的：

![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvetGREnxTDBSpoylFAl28rLvLsmXQZlPhicS6gW8jb2Ozr3yPEs7icOWpg/640?wx_fmt=png)

Diana 的办法是：在保留矩形的同时，加上两个弯曲的 div，把凹进去的部分也填充上。

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveB539D6QN4muJtaCpJTvyN4viaSuEQFb42c0BZfJaUomKPGKDt55kjoA/640?wx_fmt=jpeg)

最后完整的代码是这样的：

```
div{
  width: 500px;
  height: 350px;
  background: #000;
  position: relative;

  &::after, &::before{
    width: 20%;
    height: 100%;    
    position: absolute;
    top: 0;
    z-index:2;
    content: "";    

    background: #1e5799; 
background: -moz-linear-gradient(top, #1e5799 0%, #7db9e8 100%); 
background: -webkit-linear-gradient(top, #1e5799 0%,#7db9e8 100%);
background: linear-gradient(to bottom, #1e5799 0%,#7db9e8 100%); 
filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#1e5799', endColorstr='#7db9e8',GradientType=0 ); 
  }

  &::after{
    border-radius: 100% 0% 0% 100%;   
    right: 0;
  }

  &::before{
    border-radius: 0 100% 100% 0;   
    left: 0;
  }   
}

body{
  background: #1e5799; 
background: -moz-linear-gradient(top, #1e5799 0%, #7db9e8 100%); 
background: -webkit-linear-gradient(top, #1e5799 0%,#7db9e8 100%); 
background: linear-gradient(to bottom, #1e5799 0%,#7db9e8 100%); 
filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#1e5799', endColorstr='#7db9e8',GradientType=0 ); 
}


```

你也可以去这个完成查看 CSS 样式的实际运行效果：

https://codepen.io/jean-jordan/pen/KeKaBw

刚刚我们画的那幅画像不像人的脖子？好的，我们再回到人像画上，Diana 绘制人物的脖子也是类似的过程。

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvevicx5DBu7DLaQM8nScQC3SJhxoGLibeTnjqib8dvnW2o22usXNAP388cQ/640?wx_fmt=jpeg)

在上面这张图里，我们看到了 Diana 如何逐步改形状，最终得到了油画中人物的脖子。

但是仅仅会画各种几何形状，是无法生成艺术品的，Diana 总结了她在绘图中的 5 个重要 CSS 属性。

1、**边界半径**（border-radius）

边界半径是为了让矩形的边角过渡得更自然，对于大多数网页开发者来说，只需一个参数 border-radius，可以设定不同的半径数值。

```
border-radius: 15px 10px 40px 30px / 40px 10px 15px 30px;


```

![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveTz8V26ZEEbl0c7EeyS2xlNL1icZ1aGuX8MPdpfNQvnSlgGFLBP5UvicQ/640?wx_fmt=png)

2、**盒子阴影**（box-shadow）

对多个盒子阴影进行分层是增加深度的最佳方法之一。框阴影将粘附到 html 容器的边缘，也会沿着边界半径定义的边缘。

```
box-shadow: 6px -11px 20px 1px red, -15px -15px 5px -10px blue, inset 5px 5px 35px 10px green;


```

![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveLm6xuGJMe3cMic1ol5lTBYFvh175Btp5TnPqRhZqEB8shhLj6H06H2g/640?wx_fmt=png)  

开发者可以指定模糊半径，以及阴影是向内延伸还是向外延伸。

3、**变形**（transform）

变形的主要方式有：旋转（rotate）、缩放（scale）和倾斜（skew）

```
transform: rotate(-45deg)
transform: scale(0.7, 1.3)
transform: skew(25deg, 30deg);


```

![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveF09Bqtl7kM5U5CXGsB2xgKMagSx9znQBibiaTmrFaRkJm6p3VTpnPQwQ/640?wx_fmt=png)

此外还有透视，让物体产生远小近大的视觉效果，或者是仅仅为画出一个梯形。

```
transform: perspective(10px) rotateY(5deg);


```

![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveR4h9DwbT8OtAoPbXGSw0349IjK6iaoB6qNUibo7RjuzyD8OwoWSnGfYw/640?wx_fmt=png)

4、**线性梯度**（linear-gradient）和**径向梯度**（radial-gradient）

线性梯度用于定义一个方向上的渐变效果，径向梯度用于定义圆和椭圆形的渐变效果

```
background-image: linear-gradient(0deg, blue, transparent 60%),
radial-gradient(circle at 70% 30%, purple, transparent 40%);


```

![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveqPxicGHLPOUiaGeCDfTbIVrUcicGHQiaF6BicJHJfibZRdWd6j2pxzfibib6rg/640?wx_fmt=png)

5、**层叠**（overflow）

层叠是一种将大量杂乱元素填充到一个整齐的包中的方法，可以创建一些有趣的形状。在变形那部分的基础上使用 hidden 参数，可以把边缘遮盖起来。

```
overflow: hidden;


```

![](https://mmbiz.qpic.cn/mmbiz_png/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvejMIGRTN9qIlq2oSP1iaEPnNpZgRkfibZaKiajjJyEQxGbD2GiaOWpO5zkA/640?wx_fmt=png)

以上 5 种元素缺一不可，随便少一种都会产生怪异的效果。

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveE8hHaLKX2pQ7khwSltcsQhltWvWicFibU79cTXWiaX4Bq5st5RkOtxrRg/640?wx_fmt=jpeg)

###### **△**从左至右分别是缺少边界半径、阴影、变形、梯度、层叠的效果（点击查看大图）

不过即使这样，也很有抽象艺术的美感，仿佛在看毕加索的作品。

只适用于 Chrome
-----------

不过，由于这是一个纯个人艺术创作，Diana 小姐姐并不关心浏览器适配性。

因此，这些代码在 Chrome 里可以完美展现，但如果用其他浏览器打开，可能就会出现不一样的效果。

比如，MAC 上的 Safari 浏览器打开，妹子的眼睛就方了：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvewvXKQRkAqts4aiapgoEXFXjATbic61Vq5ticm3c3wicRgkPUEp9HDWqzhg/640?wx_fmt=jpeg)

肩膀上的高光，变成了一个大圈圈：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveWtz6436E6JcAzHPAwdZjcjOwqn2SkYQ2YibAiclPpwnBzph8dXWWRXoA/640?wx_fmt=jpeg)

胸前的礼服上，也被泼了一道墨：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvemEKK0kiauJIltRM8ocZNCKGhpZz89yp1tjMKhCnib81HnFmmPg4cu7lw/640?wx_fmt=jpeg)

如果用早期的 Chrome 打开，会出现惊悚的头身分离的效果：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveafYZ5ZzxADJBJCylFkZ63WDiaov8Rdq5LbvAic7kG12on94eJAy8Ttdg/640?wx_fmt=jpeg)

早期的 Opera 浏览器，打开之后脸方了：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvewCCAFD2dMx4UYmlAyZCAB6eP4g61ibQUBCv2kK4kF2xX9SRXibibQVUiaw/640?wx_fmt=jpeg)

Windows 7 上从 IE 6 到 IE 11，显示出来的都是这个鬼样子：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveqkM7SGwIj9gKIwDtk3IY22YsGVZ0hrdjgOFCPyxic1CPPR0V63eQxeQ/640?wx_fmt=jpeg)

浓重的线条，甚至有点抽象艺术的感觉。

同样是早期 IE，放到 Mac 上也一样鬼畜，这是 IE 5.1.7 的效果：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveFzSrCibxNnia1xxcegQzBDhFNGMw6ruw75JMyYFFcvQBfH6wfMxop3og/640?wx_fmt=jpeg)

还有人试了试，在 Windows 98 系统的 IE 7 浏览器打开，会变成非常像素风的样子：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveRVaWJCA9fmyHVjLibib0XePQdKjT8k4ibkXAdxppjp4IbmLot8vHGJia9A/640?wx_fmt=jpeg)

最恐怖的是三星手机上的夜间模式打开：

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxveYvNNYSg7C4u9tjmEQD9QcJlPQ7UzUzF1oplPtN0iaXC75BRFejJMyYw/640?wx_fmt=jpeg)

连人种都变了啊！

其他的几张画，换个浏览器打开也比较鬼畜。

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvey6qjMvVbDbk16iaoZKOWLXkP4jBj8wzl093Ekm47DGiaVBxOZfUoTYHw/640?wx_fmt=jpeg)

妹子你 bra 里的钢圈出来了啊！

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvepGlPYiaPYJ1lxNTibbfaibh51mGlQicEMNS0pLiacrehIYvqoib0zJ7bhzdw/640?wx_fmt=jpeg)

拉夫领变得透明而有光泽，领口的蕾丝干脆断掉了，仿佛是逃难时期的肖像画。

最后，如果你在 iPhone 上装了 Chrome，出来的也是 Safari 的效果，想看完整效果的话，请在安卓手机或者电脑的 Chrome 上打开。

因此，有不少网友都觉得，这几幅画可以当成浏览器测试项目，一试就能知道内核用的是谁家的。

反向绘图
----

CSS 太难，学不会？不要紧，虽然我们不能把代码变成图片，但是可以把图片变成代码啊。

没错，就是 **ASCII 艺术**，早在 DOS 时期，就有人用命令行界面来显示图片。直到今天已成为一种流行的互联网文化。

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvesSX83PGCowmTF4icWiaWJlCkYyRMMfQsXL5wWmc93B9OzibWAAcJVJp3w/640?wx_fmt=jpeg)

用单色字符来画出世界名画已经不算新鲜事。最近又有个码农开发了一个新的项目 **Primg**，让任何一幅画都可以用质数来表示。

比如蒙拉丽莎，就可以用一个 3 万位的质数二进制方式绘制出来。

![](https://mmbiz.qpic.cn/mmbiz_jpg/uDRkMWLia28ia6IwyQ0NvzYlicibxySVuxvewohQgYIkhjbleoRh8r4NmK4yVfxWQZdIUOgf8L9eEp8P5klaicnMKSQ/640?wx_fmt=jpeg)

传送门：
----

作者的 GitHub：  
https://github.com/cyanharlow

作者博客主页：  
https://diana-adrianne.com/

教程：  
https://css-tricks.com/solving-lifes-problems-with-css/

用质数生成任意 ASCII 艺术：  
https://github.com/geonnave/primg

公众号