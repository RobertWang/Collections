> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg5ODA5MzQ2Mw==&mid=2247489796&idx=1&sn=72a287c9cf89996bc7129942e75fd428&chksm=c06681fcf71108ea94d6a8b02b89cbdc01ebe4d76680b018ac155b80e553e0927ea4817bdacb&scene=21#wechat_redirect)

公众号

CSS 中有个重要的概念 BFC，搞懂 BFC 可以让我们理解 CSS 中某些原本诡异 (??) 的地方。

1. 简介
-----

在解释 BFC 之前，先说一下文档流。我们常说的文档流其实分为**定位流**、**浮动流**、**普通流**三种。而普通流其实就是指 BFC 中的 FC。FC(Formatting Context)，直译过来是格式化上下文，它是页面中的一块渲染区域，有一套渲染规则，决定了其子元素如何布局，以及和其他元素之间的关系和作用。常见的 FC 有 BFC、IFC，还有 GFC 和 FFC。

2. 三种文档流的定位方案
-------------

**常规流 (Normal flow)**

*   在常规流中，盒一个接着一个排列;
    
*   在块级格式化上下文里面， 它们竖着排列；
    
*   在行内格式化上下文里面， 它们横着排列;
    
*   当 position 为 static 或 relative，并且 float 为 none 时会触发常规流；
    
*   对于静态定位 (static positioning)，position: static，盒的位置是常规流布局里的位置；
    
*   对于相对定位 (relative positioning)，position: relative，盒偏移位置由 top、bottom、left、right 属性定义。即使有偏移，仍然保留原有的位置，其它常规流不能占用这个位置。
    

*   左浮动元素尽量靠左、靠上，右浮动同理
    
*   这导致常规流环绕在它的周边，除非设置 clear 属性
    
*   浮动元素不会影响块级元素的布局
    
*   但浮动元素会影响行内元素的布局，让其围绕在自己周围，撑大父级元素，从而间接影响块级元素布局
    
*   最高点不会超过当前行的最高点、它前面的浮动元素的最高点
    
*   不超过它的包含块，除非元素本身已经比包含块更宽
    
*   行内元素出现在左浮动元素的右边和右浮动元素的左边，左浮动元素的左边和右浮动元素的右边是不会摆放浮动元素的
    

**绝对定位 (Absolute positioning)**

*   绝对定位方案，盒从常规流中被移除，不影响常规流的布局；
    
*   它的定位相对于它的包含块，相关 CSS 属性：top、bottom、left、right；
    
*   如果元素的属性 position 为 absolute 或 fixed，它是绝对定位元素；
    
*   对于 position: absolute，元素定位将相对于上级元素中最近的一个 relative、fixed、absolute，如果没有则相对于 body；
    

3. BFC 触发方式
-----------

1.  根元素，即 HTML 标签
    
2.  浮动元素：float 值为 `left`、 `right`
    
3.  overflow 值不为 visible，为 `auto`、 `scroll`、 `hidden`
    
4.  display 值为 `inline-block`、 `table-cell`、 `table-caption`、 `table`、 `inline-table`、 `flex`、 `inline-flex`、 `grid`、 `inline-grid`
    
5.  定位元素：position 值为 `absolute`、 `fixed`
    

**注意** display:table 也可以生成 BFC 的原因在于 Table 会默认生成一个匿名的 table-cell，是这个匿名的 table-cell 生成了 BFC。

4. 约束规则
-------

浏览器对 BFC 区域的约束规则：

1.  生成 BFC 元素的子元素会一个接一个的放置。
    
2.  垂直方向上他们的起点是一个包含块的顶部，两个相邻子元素之间的垂直距离取决于元素的 margin 特性。在 BFC 中相邻的块级元素的外边距会折叠 (Mastering margin collapsing)。
    
3.  生成 BFC 元素的子元素中，每一个子元素左外边距与包含块的左边界相接触（对于从右到左的格式化，右外边距接触右边界），即使浮动元素也是如此（尽管子元素的内容区域会由于浮动而压缩），除非这个子元素也创建了一个新的 BFC（如它自身也是一个浮动元素）。
    

规则解读：

1.  内部的 Box 会在垂直方向上一个接一个的放置
    
2.  内部的 Box 垂直方向上的距离由 margin 决定。（完整的说法是：属于同一个 BFC 的两个相邻 Box 的 margin 会发生折叠，不同 BFC 不会发生折叠。）
    
3.  每个元素的左外边距与包含块的左边界相接触（从左向右），即使浮动元素也是如此。（这说明 BFC 中子元素不会超出他的包含块，而 position 为 absolute 的元素可以超出他的包含块边界）
    
4.  BFC 的区域不会与 float 的元素区域重叠
    
5.  计算 BFC 的高度时，浮动子元素也参与计算
    

5. 作用
-----

BFC 是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面元素，反之亦然。我们可以利用 BFC 的这个特性来做很多事。

### 5.1 阻止元素被浮动元素覆盖

一个正常文档流的 block 元素可能被一个 float 元素覆盖，挤占正常文档流，因此可以设置一个元素的 float、display、position 值等方式触发 BFC，以阻止被浮动盒子覆盖。

使用 BFC 阻止元素被浮动元素覆盖

![](https://mmbiz.qpic.cn/mmbiz_gif/XP4dRIhZqqULvIgcDgXQz5jeYuwdKp6B4II0FYZ6TCxSjQb2SUp1VkoibiaApHA6MxibbQQfxsIKSyJvianI76dbbw/640?wx_fmt=gif)

### 5.2 可以包含浮动元素

通过改变包含浮动子元素的父盒子的属性值，触发 BFC，以此来包含子元素的浮动盒子。

使用 BFC 包含浮动元素

![](https://mmbiz.qpic.cn/mmbiz_gif/XP4dRIhZqqULvIgcDgXQz5jeYuwdKp6BicksVTXjHfnsQZFSQmict5zLf1iapEibWCY1LM9iakxsWoJzrYwfztZgXdQ/640?wx_fmt=gif)

注意，这里触发 BFC 并不能阻止其它形式的脱离文档流的元素覆盖正常流元素。

BFC 内部其他形式脱离文档流 (absolute fixed)

![](https://mmbiz.qpic.cn/mmbiz_png/XP4dRIhZqqULvIgcDgXQz5jeYuwdKp6BrWBbucH9A9RB5scOkHLMM7EfCdibRv9hvkBAFPFCRFd3qQv6nlPoU1Q/640?wx_fmt=png)

### 5.3 阻止因为浏览器因为四舍五入造成的多列布局换行的情况

有时候因为多列布局采用小数点位的 width 导致因为浏览器因为四舍五入造成的换行的情况，可以在最后一列触发 BFC 的形式来阻止换行的发生。比如下面栗子的特殊情况

使用 BFC 阻止多列布局最后一列换行

![](https://mmbiz.qpic.cn/mmbiz_gif/XP4dRIhZqqULvIgcDgXQz5jeYuwdKp6BGvEBVagTASACsqj5d5w6xuRicriahkgldt6kqmbCTp3ObUO4MVeP2jDw/640?wx_fmt=gif)

### 5.4 阻止相邻元素的 margin 合并

属于同一个 BFC 的两个相邻块级子元素的上下 margin 会发生重叠，(设置 writing-mode:tb-rl 时，水平 margin 会发生重叠)。所以当两个相邻块级子元素分属于不同的 BFC 时可以阻止 margin 重叠。这里给任一个相邻块级盒子的外面包一个 div，通过改变此 div 的属性使两个原盒子分属于两个不同的 BFC，以此来阻止 margin 重叠。

使用 BFC 阻止 margin 合并

![](https://mmbiz.qpic.cn/mmbiz_gif/XP4dRIhZqqULvIgcDgXQz5jeYuwdKp6BWLzzgumdicFmAwia1t4oGsCfacL7sFuWLLZ47T0cLocqQxdJ8ibnzT3kw/640?wx_fmt=gif)

网上的帖子大多深浅不一，甚至有些前后矛盾，在下的文章都是学习过程中的总结，如果发现错误，欢迎留言指出~

> 参考：
> 
> 1.  我对 BFC 的理解
>     
> 2.  深入理解 BFC 和 Margin Collapse
>     
> 3.  深入理解 BFC
>     
> 4.  Understanding Block Formatting Contexts in CSS
>     
> 5.  学习 BFC
>     
> 6.  Understanding Block Formatting Contexts in CSS
>     
> 7.  带你彻底掌握 CSS 浮动
>     

### 最后

  

  

如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：  

1.  点个「**在看**」，让更多的人也能看到这篇内容（喜欢不点在看，都是耍流氓 -_-）
    
2.  欢迎加我微信「**qianyu443033099**」拉你进技术群，长期交流学习...
    
3.  关注公众号「**前端下午茶**」，持续为你推送精选好文，也可以加我为好友，随时聊骚。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/2wV7LicL762ZUCR5WEela9H9fDfYic8BAp8ib4cmuicFgACoRwORYGwkBtgUVaILLOjXtlGBnicuM5246MgketktMCg/640?wx_fmt=png)

点个在看支持我吧，转发就更好了