> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650843052&idx=3&sn=3554f749fa2956c07587844819a7869b&chksm=bd0132e28a76bbf484c3076c55ae214cb8698566fd2ee4e39a8a89e6baa20ba2e95c12788232&mpshare=1&scene=1&srcid=0317YPdpvxKe87tJj7QkccV0&sharer_sharetime=1647504545946&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来源 | 大迁世界 （ID：qq449245885）

已获得原公众号的授权转载

修改 placeholder 样式，多行文本溢出，隐藏滚动条，修改光标颜色，水平和垂直居中。这些熟悉的场景啊! 前端开发者几乎每天都会和它们打交道，这里有 20 个 CSS 技巧，让我们一起来看看吧。

### 1. 解决 img 5px 间距的问题

你是否经常遇到图片底部多出`5p`x 间距的问题？不用急，这里有 4 种方法可以解决。

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrU8QJLw0TRf541HVRib6ZMOUvdNPSc5NBoV01LVDAVFYon2VNJdb4uibUA/640?wx_fmt=png)

**方案 1：设置父元素字体大小为 0**

关键代码：

```
.img-container{
  font-size: 0;
}
```

事例地址：https://codepen.io/qianlong/pen/VwrzoyE

**方案 2：将 img 元素设置为 `display: block`**

关键代码：

```
img{
  display: block;
}
```

事例地址：https://codepen.io/qianlong/pen/eYeGONM

**方案 3：将 img 元素设置为 `vertical-align: bottom`**

关键代码：

```
img{
  vertical-align: bottom;
}
```

事例地址：https://codepen.io/qianlong/pen/jOaGNWw

**解决方案 4：给父元素设置 `line-height: 5px`**

关键代码：

```
.img-container{
  line-height: 5px;
}
```

事例地址：https://codepen.io/qianlong/pen/PoOJYzN

### 2. 元素的高度与 window 的高度相同

如何使元素与窗口一样高？答案使用 `height: 100vh;`

事例地址：https://codepen.io/qianlong/pen/xxPXKXe

### 3. 修改 input  placeholder  样式

关键代码：

```
.placehoder-custom::-webkit-input-placeholder {
  color: #babbc1;
  font-size: 12px;
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUvUO2yyIrn2PH7YB9HAd0xvF2ZoicIfUBRs7JOPBibdzfCoNboZvnnwfQ/640?wx_fmt=png)

事例地址：https://codepen.io/qianlong/pen/JjOrPOq

### 4. 使用 `:not` 选择器

除了最后一个元素外，所有元素都需要一些样式，使用 `not` 选择器非常容易做到。

如下图所示：最后一个元素没有底边。

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUXCZgLasm5W1CONsflEK6SYv0uRX1euiccLOE5CiawDL6GEUwDqiazP44w/640?wx_fmt=png)

关键代码

```
li:not(:last-child) {
  border-bottom: 1px solid #ebedf0;
}
```

事例地址：https://codepen.io/qianlong/pen/QWOqLQO

### 5. 使用 flex 布局将一个元素智能地固定在底部

当内容不够时，按钮应该在页面的底部。当有足够的内容时，按钮应该跟随内容。当你遇到类似的问题时，使用 `flex` 来实现智能的布局。

![](https://mmbiz.qpic.cn/mmbiz_gif/LDPLltmNy57P7pF03TKJSWk8XjugqKrUDFfaGICclYSlRpcuiaQ6CJrjLjlibm4jntl7krr5WZQ1HhnldH5nUP5A/640?wx_fmt=gif)

事例地址：https://codepen.io/qianlong/pen/ZEaXzxM

### 6. 使用 `caret-color` 来修改光标的颜色

可以使用 `caret-color` 来修改光标的颜色，如下所示：

```
caret-color: #ffd476;
```

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUwQRatxsQyrNNwG8ovOLbZOFSWibCubdz2glSY0bmceYQud6kIsOKxicg/640?wx_fmt=png)

事例地址：https://codepen.io/qianlong/pen/YzErKvy

### 7. 删除 `type="number"` 末尾的箭头

默认情况下，在`type="number"`的末尾会出现一个小箭头，但有时我们需要将其删除。我们应该怎么做呢？

![](https://mmbiz.qpic.cn/mmbiz_gif/LDPLltmNy57P7pF03TKJSWk8XjugqKrUBzqx3rCVVlQA50RV1jypFWs4K8AMApEicvYgiaexPriaturFVqwbSTjng/640?wx_fmt=gif)

关键代码：

```
.no-arrow::-webkit-outer-spin-button,
.no-arrow::-webkit-inner-spin-button {
  -webkit-appearance: none;
}
```

事例地址：https://codepen.io/qianlong/pen/OJOxLrg

### 8. `outline:none` 删除输入状态线

当输入框被选中时，它默认会有一条蓝色的状态线，可以通过使用 `outline: none` 来移除它。

如下图所示：第二个输入框被移除，第一个输入框没有被移除。

![](https://mmbiz.qpic.cn/mmbiz_gif/LDPLltmNy57P7pF03TKJSWk8XjugqKrULjs1m9cUePY70jz0lohHLRDXNtSctgxLplT0m8ZAy8ZR57cn8fzUibA/640?wx_fmt=gif)

事件地址：https://codepen.io/qianlong/pen/YzErzKG

### 9. 解决 iOS 滚动条被卡住的问题

在苹果手机上，经常发生元素在滚动时被卡住的情况。这时，可以使用如下的 CSS 来支持弹性滚动。

```
body,html{
  -webkit-overflow-scrolling: touch;
}
```

### 10. 绘制三角形

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUUFyb8HMODmnzNFkFRraSSRgqeDl9nht7E0BWAtUdwiaJEnia0UL7z1Fg/640?wx_fmt=png)

```
.box {
  padding: 15px;
  background-color: #f5f6f9;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.triangle {
  display: inline-block;
  margin-right: 10px;
  /* Base Style */
  border: solid 10px transparent;
}
/*下*/
.triangle.bottom {
  border-top-color: #0097a7;
}
/*上*/
.triangle.top {
  border-bottom-color: #b2ebf2;
}
/*左*/
.triangle.left {
  border-right-color: #00bcd4;
}
/*右*/
.triangle.right {
  border-left-color: #009688;
}
```

事例地址：https://codepen.io/qianlong/pen/rNYGNRe

### 11. 绘制小箭头、

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUwNBy7zQ2aiaURx2QVfLdP3FTyAH3pBuPHJd1kUWabv9LpW8rE0rM8ng/640?wx_fmt=png)

关键代码：

```
.box {
  padding: 15px;
  background-color: #ffffff;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.arrow {
  display: inline-block;
  margin-right: 10px;
  width: 0;
  height: 0;
  /* Base Style */
  border: 16px solid;
  border-color: transparent #cddc39 transparent transparent;
  position: relative;
}

.arrow::after {
  content: "";
  position: absolute;
  right: -20px;
  top: -16px;
  border: 16px solid;
  border-color: transparent #fff transparent transparent;
}
/*下*/
.arrow.bottom {
  transform: rotate(270deg);
}
/*上*/
.arrow.top {
  transform: rotate(90deg);
}
/*左*/
.arrow.left {
  transform: rotate(180deg);
}
/*右*/
.arrow.right {
  transform: rotate(0deg);
}
```

事例地址：https://codepen.io/qianlong/pen/ZEaXEEP

### 12. 图像适配窗口大小

![](https://mmbiz.qpic.cn/mmbiz_gif/LDPLltmNy57P7pF03TKJSWk8XjugqKrU379qtaoXupcPotPUseHExMysgic2TalyBfibzuzgiaSEbMxjbgfnwPaaQ/640?wx_fmt=gif)

事例地址：https://codepen.io/qianlong/pen/PoOJoPO

### 13. 隐藏滚动条

第一个滚动条是可见的，第二个滚动条是隐藏的。这意味着容器可以被滚动，但滚动条被隐藏起来，就像它是透明的一样。

![](https://mmbiz.qpic.cn/mmbiz_gif/LDPLltmNy57P7pF03TKJSWk8XjugqKrU73w47uIXoKUur1P57MSAl8Wj7W0oSyiaHcIo8NnaChMmxib44ROPaibSQ/640?wx_fmt=gif)

关键代码：

```
.box-hide-scrollbar::-webkit-scrollbar {
  display: none; /* Chrome Safari */
}
```

事例地址：https://codepen.io/qianlong/pen/yLPzLeZ

### 14. 自定义选定的文本样式

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUlIykFfoPjPSFlTcmlfKcic2CA4uTUkclBDLeIu8t6VkdeaSzobicIZmA/640?wx_fmt=png)

关键代码：

```
.box-custom::selection {
  color: #ffffff;
  background-color: #ff4c9f;
}
```

事例地址：https://codepen.io/qianlong/pen/jOaGOVQ

### 15. 不允许选择文本

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUiakGa4l6qnSuq56ee1Vicr75fRibwsPaWkMwW883smxicgR0ZI3LAaOEIA/640?wx_fmt=png)

关键代码：

```
.box p:last-child {
  user-select: none;
}
```

事例地址：https://codepen.io/qianlong/pen/rNYGNyB

### 16. 将一个元素在水平和垂直方向上居中

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUBGiciaibeRNKQ33gjvgibdIab4L1lEDJ7E1xX8FtH6KnT5YNlUtpWVJQXA/640?wx_fmt=png)

关键代码：

```
display: flex;
align-items: center;
justify-content: center;
```

事例地址：https://codepen.io/qianlong/pen/VwrMwWb

### 17. 单行文本溢出时显示省略号

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrU83g31QBn0hc6uobPIbzg6MqVQqo1ShibtjXZCS9MHDm7Pes4HSrDIzw/640?wx_fmt=png)

关键代码：

```
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  max-width: 375px;
```

事例地址：https://codepen.io/qianlong/pen/vYWeYJJ

### 18. 多行文本溢出时显示省略号

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUqic32TjiboIu5V9PqzsysPm13qkf0DOUfenqCgDlquyR75Rib4TLsQ6uA/640?wx_fmt=png)

关键代码：

```
  overflow: hidden;
  text-overflow: ellipsis;

  display: -webkit-box;
  /* set n lines, including 1 */
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
```

事例地址：https://codepen.io/qianlong/pen/ZEaXEJg

### 19. 使用 "filter:grayscale(1)"，使页面处于灰色模式。

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy57P7pF03TKJSWk8XjugqKrUtw53yPVyTon13ZNyYNfHD0cOHQiaxdUzlJCmR8Ab1OgN5iarRI8VOcHw/640?wx_fmt=png)

关键代码：

```
body{
  filter: grayscale(1);
}
```

* * *

作者：Matt Maribojoc  译者：前端小智  来源：stackabuse 原文：https://javascript.plainenglish.io/20-css-tips-and-tricks-to-make-you-a-better-developer-d80ae5c09617