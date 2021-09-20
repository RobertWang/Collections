> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7008563603431227405)*   定位分类

1.  static：静态定位 ———— 静态定位就是元素的默认定位方式，无定位的意思，一般就是默认不显示的，这类定位用的非常少，基本上都是默认不显示
    
2.  relative：相对定位 ———— 相对定位就是元素 相对于它原来标准流中的位置。如果换位置的话，会再相对于它原来的位置做偏移，并不是最初的位置。**相对定位原来在标准流区域位置继续占有，后面的盒子以然会以标准流的方式对待它**
    
3.  absolute：绝对定位 ———— 绝对定位一般和相对定位搭配使用。**绝对定位中只参考父盒子，如果父盒子有一个相对定位的话，绝对定位子盒子会随着父盒子移动。但是如果父级没有相对定位，会逐级向上查找，查找到谁就会依赖它。如果都没有的话会依赖文档移动位置。绝对定位的元素是不会保留它原来的位置**
    
4.  fixed：固定定位 ———— 固定定位就是固定再某一处，不随着其他元素的移动而移动
    

```
// 静态定位（一般不用）
position: static;

// 相对定位（重要）
position: relative;

// 绝对定位（重要）
position: absolute;

// 固定定位
position: fixed
复制代码
```

#### 定位口诀

**子绝父相**：子盒子用绝对定位，父盒子用相对定位

这个定位口诀对于新手来说一定一定要记住，在未来的很长一段时间都需要这个口诀支撑你度过 css 定位的时光~ 例如在文章的第一个图中用的就是这个口诀，当父盒子用相对定位的话，子盒子用绝对定位就会跑到父盒子身上，如果父盒子没有用到相对定位，那么子盒子用了绝对定位就会以文档流为基准。 如下图

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e90cb0f4597343469187d2de5d4c0df2~tplv-k3u1fbpfcp-watermark.awebp?)

#### 固定定位

固定定位：

1.  固定定位是完全脱标的，不占用位置
2.  固定定位只认识浏览器的可视窗口
3.  固定定位跟元素没有任何关系，单独使用，不随着滚动条滚动

如下图所示

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26219330f9324ca5a4b6df73eb614e19~tplv-k3u1fbpfcp-watermark.awebp?) ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f031cd1fee34d6fba65c2aced54269e~tplv-k3u1fbpfcp-watermark.awebp?)

#### 拓展：定位盒子居中显示

**在绝对定位中 margin 0 auto; 是无效的，不能让盒子居中显示**，解决方式就是 首先让元素 向左移动 50% 也就是：left: 50%，然后让元素向右移动自身宽度的一般就可以居中显示。

```
// 如果水平居中的话，首先让盒子向左移动 50%
left: 50%;
// 然后让盒子向右移动自身宽度的一般
margin-left: -100px;
复制代码
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fd92a0d13fa46e6aa3a7eebe05ef1c8~tplv-k3u1fbpfcp-watermark.awebp?)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cec90f4f7cb94dd692abbfb83d07371d~tplv-k3u1fbpfcp-watermark.awebp?)

#### 拓展：堆叠顺序

使用定位的时候很可能会出现盒子重叠的情况，如果加了定位的盒子，默认是后来者居上，会压到前面的盒子，所以就需要到层叠属性来调整盒子的权重从而影响盒子的层叠顺序。 **使用 z-index 属性可以调整盒子的堆叠顺序**

1.  属性值：正整数，负数或者 0，默认是 0，越大盒子越往上
2.  如果属性值相同的话，则按照书写的顺序，后来者居上
3.  数字后面不能加单位

**注意：z-index 只能应用于 相对定位、绝对定位、固定定位的元素，其他标准流的元素无效**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb3e3dbaa76c4b1ab80fc3125e12138c~tplv-k3u1fbpfcp-watermark.awebp?)

添加完 z-index 之后 如下图

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0bf763f4628416a825da5e944dc8cc7~tplv-k3u1fbpfcp-watermark.awebp?)

好了，今天这一篇基本上都讲完了，重要的是 定位的内容，定位在未来的工作当中是非常常用和重要的，一定要熟悉和了解定位具体怎么运用，虽然在未来居中显示这个会用到更为简单的 flex 布局，但是对于原来定位中怎么居中显示需要充分的理解。有什么问题欢迎留言讨论~