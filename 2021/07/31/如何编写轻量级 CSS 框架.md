> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488286&idx=1&sn=08d7d117bac2c520e9847c03c70f576e&chksm=ce0da49df97a2d8b7c0fc13620eef9542e5bbec1de7d85454aedc9eb64483985bca7ff1ab0bc&scene=21#wechat_redirect)

作者 | 叙帝利  

链接 | www.cnblogs.com/nzbin/p/7073601.html

前言

这篇文章我已经酝酿了半年之久，或者说拖沓了这么久吧。想说的东西很多，却又无从说起。如今轻量级框架如雨后春笋，层出不穷。我想每个人都应该归纳总结工作中的常见需求，编写一套适合自己的 CSS 框架。

在之前的文章中，我提到了面向对象的 CSS（比如 BEM、OOCSS、SMACSS，详见  http://vanseodesign.com/css/dry-principles/）。这是一种思想，并不涉及具体的 CSS 问题，主要是类命名的策略！现在仍然有很多人对于前端框架的认识还停留在表面，认为 Bootstrap 是后端人员专用，前端没必要等等。

我不知道这种说法从何而来，我最开始也不喜欢使用框架，或许和很多人的想法一样，畏惧新知识、害怕难以驾驭、遇到问题的时候无法解决等等。最关键的一点是很多人认为框架的样式是固定的，修改起来太麻烦，还不如自己根据设计图写起来方便。

为什么使用框架
-------

为什么使用框架？答案显而易见，效率。除此之外，使用框架或者研究框架的意义还有很多，比如面向对象思想的具体实现。在上一家公司工作的时候，开始的几个项目我也是用最原始的方法书写 CSS 。项目之中最让我头疼的就是类的命名。我想大多数人都是根据功能去命名，这就造成了很多的冗余，相同的组件可能写很多次。简单举一个例子，如下图，个人中心的登录界面。

![](https://mmbiz.qpic.cn/mmbiz_jpg/oTKHc6F8tsgRCQ3hqozibIqMRXgJSuX0wuY2NO2cyQzEIYWLtKHFf0iakFaDarvjx9v3vEhNy3EDv8JDoV8v8K5Q/640)

很多人包括我刚开始的时候可能会选择下面的类命名及布局方式，其通用性非常差：

```
<div>
    <div>
　　　　<img src="..." />
　　</div>
    <div>
　　　　<a href="...">请点击登录</a>
　　</div>
</div>
```

然而了解 Bootstrap 的人应该一眼就发现上图就是一个 media 对象，无非一些小细节需要调整一下：

```
<div>
    <div>
        <img src="..." />
    </div>
    <div>
        <a href="...">请点击登录</a>
    </div>
</div>
```

为了让文字与图片居中对齐，我们可以使用 Bootstrap 的`.media-middle`的辅助类。如果在工作中还要根据需求自定义一些辅助类调整细节，当然这是一个移动端的例子，可以选择移动端框架相关的 media 对象。

另外，在项目改版的时候，原始的方法的修改更是惨不忍睹，可以说是噩梦，冗长的 CSS 文件、混乱的功能划分、类名、色值等等。最后也只能硬着头皮一点一点修改。那一刻我才体会到框架的意义以及前端工具的重要性。我从工作中总结出，**要么你可以熟练的使用某一个框架，要么就自己实现一个框架**。

前端框架对比
------

目前市面上前端框架主要分重量级与轻量级。重量级主要有 Bootstrap、Semantic、UIkit、Foundation 等，轻量级有 Pure、Skeleton、Miligram 等。经常关注前端动态的工程师会发现轻量级框架每年都层出不穷。在我上面提到的主流轻量级框架之外还有很多类似的框架。

我一直问自己，为什么要重复造轮子。经过研究，我发现这些轻量级框架其实大多都不能胜任工作需求，而且模仿的痕迹很重，基本上都或多或少的有 Bootstrap 的影子。那么这些轻量级框架有没有意义呢？当然有。但是就我个人观点，选择轻量级框架反倒不如自己实现一个框架。因为大多轻量级框架就像是工作总结，是根据自己的业务需求实现的。所以大多不具有通用性。

前端框架的对比主要以 Bootstrap、Semantic、UIkit 为主，因为我个人感觉这三个最具有代表性，而且设计风格各有特色。Foundation 也有很多大公司在用，但以我个人观点，无论是框架的易用性还是设计风格，相比其它几个框架稍逊一筹。

**其中 Bootstrap 和 Semantic 是面向对象的最好体现。**

我先说一下 Bootstrap 的优势，不是设计风格，不是模块，不是特效，而是栅格，响应式栅格。Bootstrap 的栅格在与其它框架对比中占有绝对优势，无论是栅格的划分还是类名的风格都堪称经典。如果读者有心看一下 Bootstrap 的 Less 源文件，就会感受到 Bootstrap 对于响应式栅格的独具匠心。其实在 Bootstrap 之前也有很多栅格方案，但是给人的感觉就是不够利索，类名繁琐不好记。而后来的很多框架，尤其轻量级的框架大多都有 Bootstrap 的影子。

下面我们通过对比几个框架的栅格和按钮来看一下类命名的策略。

**Bootstrap**

```
<div>
  <div></div>
  <div></div>
</div>
<button class="btn btn-primary" type="submit">Button</button>
```

**Semantic**

```
<div>
  <div></div>
  <div></div>
</div>
<button>Primary</button>
```

**Foundation**

```
<div>
  <div></div>
  <div></div>
</div>
<button type="button">Primary</button>
```

**UIkit**

```
<div>
    <div></div>
    <div></div>
</div>
<button type="button">Primary</button>
```

**Pure**

```
<div>
    <div></div>
    <div></div>
</div>
<button>A Primary Button</button>
```

通过上面的对比，大家应该已经发现了这些框架的命名策略的不同。不可否认，Bootstrap 的命名最经典。

之前在网上看到有人讨论关于框架的易用性，有人说 Bootstrap 的类名太长，然而通过上面几个框架的对比，Bootstrap 的类并不繁琐，而且用预处理器编写框架时嵌套会比较灵活。

Semantic 的类名最简洁，通过多个定语的修饰组成一句话，确实很有意思。但是过多的修饰类在编写框架时会稍显凌乱，有利有弊，因人而异吧。

Foundation 的栅格应该是最丰富的，策略上类似 Bootstrap，只是对公共属性进行了拆分，大家也可以看看其中的具体细节。

UIkit 和 Pure 的策略相同，都加了前缀以区分其它框架，但是很显然类名过于冗长了。我在编写框架时也这样想过，但是最终放弃了这种方式。

关于 CSS 预处理器
-----------

CSS 预处理器早已不是什么新鲜事，但是真正能够在工作中运用的人有多少呢？熟练使用预处理器特性的人又有多少呢？

我之前工作的时候也没有用预处理器，因为不用，所以也意识不到预处理器的好处。主要是觉得麻烦，因为要使用编译器编译一下，还不如直接写 CSS 方便。但是在项目维护的时候就意识到预处理器的好处。

后来在几个项目中尝试了预处理器，但是对于模块化的写法不太明确。预处理器作为工具，可以实现模块化编写 CSS，那么应该如何划分模块？另外，预处理器有很多特性，但是大多数人刚开始只用到变量和嵌套，其它的特性几乎很少用到。我相信在自己动手实现一个轻量级框架的过程中，我们可以对预处理器有一个全面的了解。

目前流行的预处理器有 Less，Sass，Stylus 三个，选择哪个完全是看自己的习惯。我最开始因为 Bootstrap 了解的 Less，但是因为习惯选择了 Sass，其次 Sass 的功能要更全面一些。

无论是工作还是自己写项目，都要搭建一个项目环境，也就是安装一系列的 npm 包。相比刀耕火种的开发方式，使用工具开发的前期准备过程稍显麻烦，然而一旦环境建好，后期的开发将会游刃有余。

Miligram 这个轻量级框架在 Github 上有很高人气，但是说实话，用处并不大。不过这个框架的构建方式非常值得学习。虽然 CSS 对于很多人来说很简单，但是真要去写一个框架，还是非常棘手，这时候就需要借鉴一些优秀的框架。

编写框架大致会用到的 npm 如下：

```
--autoprefixer
--node-sass
--npm-run-all
--rimraf
--onchange
```

其实最主要的就是一个 node-sass，其它的都是辅助 CSS 文件的生成修改，大家感兴趣的话可以去 npm 官网搜索这些插件，了解具体用法，如有不懂可以给我留言，我就不啰嗦了。

编写轻量级框架
-------

终于到了本篇文章的重头戏。

简单介绍一下，我给自己编写的框架取名 Snack，原意 “快餐”，主要表达简单之意。虽然是轻量级框架，但我并不想拿轻量级做为噱头，毕竟体量轻意味着某些功能的缺失以及疏漏。这个框架的意义更多的是交流学习，我试图借鉴其它框架的优秀之处，尽量简化类名，以及尝试探索一些更通用的组件。

大多数的轻量级框架只是 CSS 框架，不涉及 JS 部分，主要用于网页的布局。我之所以打算自己编写框架，是因为工作中重复的东西太多，通过框架可以很好的将这些零散组件整合到一起。另一方面，写个小项目，学点新知识是一件趣事。

编写框架是去年想做的事情，但因为时间原因，拖了很久。写框架之初我曾陷入一个误区，我打算设计一些比较前卫的样式，立体的按钮、浮动的面板等，比如下图中的风格。

![](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsgRCQ3hqozibIqMRXgJSuX0waLPvyiaBsUy8U7navoFYMOa6nvzwN2Jyf010eYF6QNuwicocOU9Jl0ibw/640)

https://dribbble.com/shots/524593-Soft-Interface-Black

但是在断断续续编写框架的过程中，我逐渐找到了方向，上图的样式只是一种皮肤，编写框架之初不应该把重点放在这上面。当然，好的 UI 设计也是框架成功的一部分。

### 模块划分

编写框架的第一步就是要确定框架应该包含哪些模块。因为是轻量级框架，所以模块肯定没有重量级框架那么全面，只有核心的一些组件。通过比较一些轻量级框架以及工作总结，大致常用的模块包括栅格、媒体、按钮、排版、表单、表格、面板以及辅助工具。

在常用的这几个组件中，需要重点关注的是栅格、表单及面板，媒体组件也很重要，但是自由发挥的空间不大，我直接用了 Bootstrap 的媒体组件。

### 命名策略

首先是类命名的层次与结构。类命名一直是我比较纠结的地方，刚开始工作的时候为了起一个见名知意又简洁的类名总是抓耳挠腮。我在编写框架时尽量避免与 Bootstrap 的类名重叠，但也不能完全避免。对比其他框架会发现，这种情况不可避免的会出现，毕竟类名会有一定的规律性以及层次性。在这一点上我比较喜欢 Bootstrap 的风格。下面和 Bootstrap 的表单做一个对比。

Bootstrap 的表单结构及类名

```
--div.form-horizontal
  --div.form-group
    --label.control-label
    --input.form-control
```

Snack 的表单结构及类名

```
--div.form-row
  --div.form-item
     --label.form-label
     --input.form-field
```

这个表单结构整体而言还算不错，只是个别地方需要修改。有一些框架不给`input`等元素起类名，而是给父元素一个类名，个人对这种做法表示疑问，不起类名会降低框架编写及使用的灵活性。

第二个策略是组件的修饰，比如按钮及面板都存在多个语境（颜色、大小等），在这一点上我编写框架时做了一些简化，风格上有些 Semantic 的影子。

```
<button>primary</button>
<table>...</table>
<div class="boxes primary">...</div>
```

关于修饰类的策略是一个仁者见仁智者见智的问题，至于哪种方法更好，还需要在编写框架的过程中摸索。

### 栅格系统

演示示例: https://nzbin.github.io/snack/#grid

任何框架必须建立在栅格的基础上才能灵活布局。我在前面提到了 Bootstrap 的精华就是栅格系统。栅格系统的编写需要使用预处理器的循环功能，否则就要做无谓的重复劳动了。我遇到过一些轻量级框架是用 Less 编写的，其栅格系统就没有用循环，这样的源码稍显唐突，可能是作者对 Less 的循环功能不熟，当然 Less 本身的循环比较弱，用起来有些别扭。

关于预处理器的循环，可以参照我之前翻译的 《CSS 预处理器中的循环》，比较详细地对比了三种流行预处理器的循环功能。简单说一下，Less 没有循环，但可以用递归实现，而 Sass 和 Stylus 有真循环。

我编写的栅格系统也是默认 12 列，但是后来发现 12 列的栅格缺少最常用的列宽（比如 10%、20%、30% 等），比如下面 CodePen 展示的例子用 12 列栅格是无法完成的，所以我又添加了 10 列栅格，但仍然无法面面俱到，不过已经很灵活了。

栅格的使用和 Bootstrap 是一样的，除了 12 列栅格外，10 列栅格以及均分栅格都要添加`.cols-`类

```
<!-- 默认 12 列栅格，所以省略 cols-12 -->
<div>
    <div></div>
    <div></div>
</div>
<!-- 10 列栅格 -->
<div>
    <div></div>
    <div></div>
</div>
```

这个栅格并没有响应式，只有一个断点，小屏手机上的话所有栅格都会单行显示。一方面，这样的设计符合大多数轻量级框架的初衷；另一方面，我打算再写一个针对移动端的框架，毕竟 Web 端和移动端的风格差距较大，按照业务需求分开会更好。不过最近我更改了源文件，为响应式预留了扩展方式。

### 表单

演示示例: https://nzbin.github.io/snack/#forms

在上面的命名策略中已经展示了 Snack 表单的基本结构，基本表单除了结构之外，样式上并没有太多可以讨论的地方。在此说一下表单中`checkbox`的结构调整，先看一下 Bootstrap 的`checkbox`结构。

```
<!-- checkbox -->
<div>
  <label>
    <input type="checkbox" value=""> checkbox
  </label>
</div>
<!-- checkbox-inline  -->
<label>
  <input type="checkbox" value="option1"> checkbox
</label>
```

以上两种结构不能有偏差，稍有偏差样式就会错乱，灵活性较差。其次我在想两种结构能不能整合在一起，增强灵活性。想了很久，找到了方法，Snack 结构如下：

```
<!-- checkbox -->
<div>
  <label>
    <input type="checkbox" value=""> checkbox
  </label>
</div>
<!-- checkbox-inline -->
<div>
  <label>
    <input type="checkbox" value=""> checkbox
  </label>
</div>
```

也可以将样式直接加到`label`标签上。另外，如果将`input`移到`label`标签外也是没有问题的，如下：

```
<!-- checkbox -->
<div>
  <input type="checkbox" value="">
  <label for="checkbox1">checkbox</label>
</div>
<!-- checkbox-inline -->
<div>
  <input type="checkbox" value="">
  <label for="inlineCheckbox1">checkbox</label>
</div>
```

这种结构有一个好处，就是可以自定义`input`样式，详见下面的 CodePen 的`scss`文件。`radio`的设置和`checkBox`是一样的。

### 辅助类

辅助类是一系列类的组合，比如字号大小、颜色值、padding、margin 以及左右浮动等。在一些 Bootstrap 搭建的后台管理系统中尤为常见，这样布局起来就会比较灵活。以下是一个边框的辅助类。

```
.border-left-right {
  border-left: 1px solid #eee;
  border-right: 1px solid #eee;
}
.border-top-bottom {
  border-top: 1px solid #eee;
  border-bottom: 1px solid #eee;
}
.border-left {
  border-left: 1px solid #eee;
}
.border-right {
  border-right: 1px solid #eee;
}
.border-top {
  border-top: 1px solid #eee;
}
.border-bottom {
  border-bottom: 1px solid #eee;
}
```

关于助类的更多内容可以搜索阅读这篇文章《如何编写通用的 Helper Class》

### 盒组件

演示示例: https://nzbin.github.io/snack/#boxes

盒组件是我整个框架中比较满意的一个模块。之所以要做这个组件主要是觉得 Bootstrap 的 list 组件和 panel 组件可以整合到一起。当然，这样的做法有利有弊。盒组件在后台管理系统的布局中表现的尤为突出。

其命名也是多种多样，比如 panel、widget、portlet、ibox、card 等，每个后台管理系统框架都会对这个组件进行深度开发，可见其在布局上的重要性。给一个组件起一个合适的类名也很关键，想了很久，最后用了`box`的类名，当然一般情况下尽量不要用`box`，因为这个类名比较宽泛。下面的 CodePen 模拟了 Bootstrap 的 list 及 panel 组件。

### 主题

给框架添加主题是一件有趣的事情。Snack 的默认主题是白色，因为喜欢黑色，最后添加了暗夜主题，编写主题只需改变组件的颜色。演示文档 的页面用了暗夜主题，点击上方的红色按钮可以切换主题。

总结
--

如果大家问我那个框架更好，我会毫不犹豫的选择 Bootstrap。在工作中可以根据需求的难易进行框架选择，如果业务比较重，最好根据 Bootstrap 进行二次开发；反之，可以选择一些轻量级框架，最好还是根据自己的需求造轮子，如果大家愿意选择或是借鉴我的框架，那会是我的荣幸。

Github: https://github.com/nzbin/snack

Docs:  https://nzbin.github.io/snack

推荐阅读

_**1.**_ [8 个学习编程的好地方](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488267&idx=1&sn=829fad61666ce40d955bb530b3ce3306&chksm=ce0da488f97a2d9e1577c323841b929c20e2f3174089d305eb7427d9f4308b06f9cd1aee3f9c&scene=21#wechat_redirect)

_**2.**_ [如何同步多个 Git 远程仓库](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488267&idx=2&sn=5233c1a5644c287067239fee16db1004&chksm=ce0da488f97a2d9e015f453d0950a8b3df1750347a697190af648865d1ab8b385a7d9c1f37f1&scene=21#wechat_redirect)

_**3.**_ [为什么要放弃 JSP ？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488247&idx=1&sn=8f40c70e93f789211b262735ff5a0bb3&chksm=ce0da574f97a2c62d213b3a4bac5bb72dd3d34764579be08ba78836f6cb8e3205faaa12cd980&scene=21#wechat_redirect)

_**4. [](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488238&idx=1&sn=253a237ec16ac1bea64a37ddc934867f&chksm=ce0da56df97a2c7b82c8849142c365a1ec01f010aa4d7982b842bd5a26c6ef312196d4c72e53&scene=21#wechat_redirect)** [](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488238&idx=1&sn=253a237ec16ac1bea64a37ddc934867f&chksm=ce0da56df97a2c7b82c8849142c365a1ec01f010aa4d7982b842bd5a26c6ef312196d4c72e53&scene=21#wechat_redirect)_ [你还在从零搭建项目 ？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488238&idx=1&sn=253a237ec16ac1bea64a37ddc934867f&chksm=ce0da56df97a2c7b82c8849142c365a1ec01f010aa4d7982b842bd5a26c6ef312196d4c72e53&scene=21#wechat_redirect)

_**5.**_ [外行人都能看懂的 Spring Cloud](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488222&idx=1&sn=bbf8d140906aaf26c88d9b1dfb99d654&chksm=ce0da55df97a2c4b56f2cf54be5e3f18a7d8e87177311276d827bd7c24d0515f37bbcea4bcfd&scene=21#wechat_redirect)

_**6.**_ [如何分析一条 SQL 的性能 ?](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488106&idx=2&sn=51ea9f478004170e5c10c428b4a405c8&chksm=ce0da5e9f97a2cffe0b7af071e7fbce5a8b6fee8d0e8a485cbf114dd0cc39dc1469c0c88e4ea&scene=21#wechat_redirect)

_**7.**_ [还不懂 Java 中的多线程 ？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488085&idx=1&sn=66d41311ea6c4eeb4418a007e5d67b27&chksm=ce0da5d6f97a2cc0e4ea693f403adbd11614eedc9fbb2e3e7e6c51c3b9d03c60e8844ec6eb9d&scene=21#wechat_redirect)

_**8.**_ [为什么老外不愿意用 MyBatis？](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247488085&idx=3&sn=e107207958249561e7c6590e8b3abe92&chksm=ce0da5d6f97a2cc02a045987af65f05a6d28a02841280fff4179426fd61812c1e6db95a6e1db&scene=21#wechat_redirect)