> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/94699579)

在这个年代，你要是不懂一点点 css3 的知识，你都不好意思说你是个美工。美你妹啊，请叫我前端工程师好不好。呃。。好吧，攻城尸。。。呵呵，作为一个攻城尸，没有点高端大气上档次的东西怎么能行呢，那么 css3 的动画就绝对是值得你拥有了，虽说 IE9 以及更早版本的 IE 浏览器都不支持 css3 动画，但是 IE6-8 浏览器已是江河日下，使用谷歌浏览器、火狐浏览器、IE10 + 浏览器以及移动端浏览器等这些支持 css3 动画的浏览器的人数越来越多，所以如果很简单的就能让一部分人获得更好的用户体验，那何乐而不为呢。

从广义上来讲，css3 动画可以分为两种。

**过渡动画**

第一种叫过渡（transition）动画，就是从初始状态过渡到结束状态这个过程中所产生的动画。所谓的状态就是指大小、位置、颜色、变形（transform）等等这些属性。css 过渡只能定义首和尾两个状态，所以是最简单的一种动画。

要想使一个元素产生过渡动画，首先要在这个元素上用 transition 属性定义动画的各种参数。可定义的参数有

transition-property：规定对哪个属性进行过渡

transition-duration：定义过渡的时间，默认是 0

transition-timing-function：定义过渡动画的缓动效果，如淡入、淡出等，默认是 ease

transition-delay：规定过渡效果的延迟时间，即在过了这个时间后才开始动画，默认是 0

![](https://pic2.zhimg.com/v2-ae78e7d1a7a9765d561aa7aef1581ff9_b.jpg)

为了书写方便，也可以把这四个属性按照以上顺序简写在一个 transition 属性上：

![](https://pic1.zhimg.com/v2-f0a28a35df68b19959a402a3505adba4_b.png)

如果是使属性的默认值，则可以省略：

![](https://pic1.zhimg.com/v2-daea5094944075f03c7161dff9245a24_b.png)

相当于：

![](https://pic3.zhimg.com/v2-9da964b6dbc285fa237e65061abe169a_b.png)

如果想要同时过渡多个属性，可以用逗号隔开，如：

![](https://pic4.zhimg.com/v2-6045055f4cc131d4e227c1b7c2690247_b.png)

使用 transtion 属性只是规定了要如何去过渡，要想让动画发生，还得要有元素状态的改变。如何改变元素状态呢，当然就是在 css 中给这个元素定义一个类（:hover 等伪类也可以），这个类描述的是过渡动画结束时元素的状态。

![](https://pic2.zhimg.com/v2-c5aa636b37a2626b9dd6f1198a936eb9_b.jpg)

这样，当我们把鼠标移动到 div 上的时候，div 的状态发生了变化，就能看到宽度从 100 到 400，高度从 100 到 400，背景颜色从黑到红的，过渡时间为 3 秒的过渡效果了。

除了使用 hover 等系统提供的伪类外，我们也可以随意的定义自己的类，然后想要过渡时就通过 js 把类加到元素上面：

![](https://pic1.zhimg.com/v2-206329070d43c237181fdcef9fd540f8_b.jpg)

**关键帧动画**

第二种叫做关键帧（keyframes）动画。不同于第一种的过渡动画只能定义首尾两个状态，关键帧动画可以定义多个状态，或者用关键帧来说的话，过渡动画只能定义第一帧和最后一帧这两个关键帧，而关键帧动画则可以定义任意多的关键帧，因而能实现更复杂的动画效果。

关键帧动画的定义方式也比较特殊，它使用了一个关键字 @keyframes 来定义动画。具体格式为：

@keyframes 动画名称 {

时间点 {元素状态}

时间点 {元素状态}

…

例如：

![](https://pic2.zhimg.com/v2-b91ec8487ad1829b5b2f54ffb241e859_b.jpg)

这段代码定义了一个名为 demo, 且有 5 个关键帧的动画。0% ，10% 等这些表示的是时间点，是相对于整个动画的持续时间来说的，时间点之后的花括号里则是元素的状态属性集合，描述了这个元素在这个时间点的状态，动画发生时，就是从第一个状态到第二个状态进行过渡，然后从第二个状态到第三个状态进行过渡，直到最后一个状态。一般来说，0% 和 100% 这两个关键帧是必须要定义的。

关键帧的书写方式很灵活，一行可以写多个关键帧。

![](https://pic3.zhimg.com/v2-040faf0c183d7be5113ed011491324b2_b.jpg)

甚至它们之间的空格也是可以不要的。

现在我们知道了怎么去定义一个关键帧动画了，那怎么去实现这个动画呢？其实很简单，只要把这个动画绑定到某个要进行动画的元素上就行了。

把动画绑定到元素上，我们可以使用 animation 属性。animation 属性有以下这些：

![](https://pic3.zhimg.com/v2-c7b2b19af543ddc5adf1b35173802ede_b.jpg)

像前面讲的 transition 属性一样，也可以把这些 animation 属性简写到一个 animation 中，使用默认值的也可以省略掉。但 animation-play-state 属性不能简写到 animation 中。

![](https://pic4.zhimg.com/v2-86f2b855ad586f215774eae19c77b80f_b.png)

只要像这样把定义好的动画绑定到元素上，就能实现关键帧动画了，而不是像第一种过渡动画那样，需要一个状态的改变才能触发动画。

![](https://pic3.zhimg.com/v2-7f7feabaed34830d6a8fe789c4218cd6_b.jpg)

注意，为了达到最佳的浏览器兼容效果，在实际书写代码的时候，还必须加上各大浏览器的私有前缀

![](https://pic3.zhimg.com/v2-9a9ff411c6b1226722a9784af16b4282_b.jpg)

**animate.css 的使用**

[animate.css](https://link.zhihu.com/?target=https%3A//daneden.me/animate/) 是一个 css3 动画库，可以到 [github](https://link.zhihu.com/?target=https%3A//github.com/daneden/animate.css) 上去下载，里面预设了很多种常用的动画，[可以先在本页看下演示效果](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/2050/p/3409129.html%23demo)，使用也很简单，因为它是把不同的动画绑定到了不同的类里，所以我们想要使用哪种动画的时候，只需要简单的把那个相应的类添加到元素上就行了：

首先在 head 中引入下载的 animate.css 文件

![](https://pic3.zhimg.com/v2-ae8ee36f42c1148b8c1da66a5dfbeeba_b.png)

然后你想要哪个元素进行动画，就给那个元素添加上 animated 类 以及特定的动画类名，animated 是每个要进行动画的元素都必须要添加的类。

假设使用 jquery，要给一个 id 为 demo 的元素添加一个摇动的动画, 因为摇动的动画类名为 shake，所以代码是这样的：

![](https://pic4.zhimg.com/v2-ccd75e8ab55010d5b241b17c9097417b_b.jpg)

这样载入页面，元素就能动起来了。你也可以在动画完成后，把动画类移除，以便可以再次进行同一个动画。

![](https://pic3.zhimg.com/v2-f0ae58130759cb0222c0abce2529e436_b.jpg)

至于动画的配置参数，比如动画持续时间，动画的执行次数等等，你可以在你的的元素上自行定义，覆盖掉 animate.css 里面所定义的就行了。

注意这些属性还要记得加上各浏览器的前缀。

总之是很灵活的，说到底不就是一个 css 文件吗，一看就懂的，你在里面想怎么整就怎么整，不想用它提供的类名，就在里面改掉就行了。如果你只想用里面的部分动画，也可以把那些要使用的动画分离出来，它的[官网也提供了这样的功能](https://link.zhihu.com/?target=https%3A//daneden.me/animate/build)。

下面展示了 animate.css 里面提供的所有动画，动画名就是类名，你想使用哪个动画，加上这个类名就行了。

shake flash swing bounce tada wobble pulse

flip flipInX flipOutX flipInY flipOutY

fadeOutfadeOutUpfadeOutDownfadeOutLeftfadeOutRightfadeOutUpBigfadeOutDownBigfadeOutLeftBigfadeOutRightBig

slideInDownslideInLeftslideInRightslideOutUpslideOutLeftslideOutRight

bounceInbounceInDownbounceInUpbounceInLeftbounceInRight

bounceOutbounceOutDownbounceOutUpbounceOutLeftbounceOutRight

rotateInrotateInDownLeftrotateInDownRightrotateInUpLeftrotateInUpRight

rotateOutrotateOutDownLeftrotateOutDownRightrotateOutUpLeftrotateOutUpRight

lightSpeedInlightSpeedOuthingerollInrollOut

相信很多人在刚接触前端或者中期时候总会遇到一些问题及瓶颈期，如学了一段时间没有方向感或者坚持不下去一个人学习枯燥乏味有问题也不知道怎么解决，对此我整理了一些资料 喜欢我的文章想与更多资深大牛一起讨论和学习的话 欢迎加入我的学习交流群  

[907694362​jq.qq.com](https://link.zhihu.com/?target=https%3A//jq.qq.com/%3F_wv%3D1027%26k%3D59jir0A)