> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651586879&idx=2&sn=35c77a0b1ab94dd913cb38b54ca89ce0&chksm=8022d6feb7555fe8199684a8044540e4b947d766b0284830396a243b5d66a34f4c0279f19bda&mpshare=1&scene=1&srcid=12065sIr1orrL7pHyThfCV7C&sharer_sharetime=1638763332669&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

前几天在逛社区时，偶然发现一个非常赏心悦目得 **小鸟飞翔效果**，并且是纯 `css` 实现的。

这不由让我大感惊奇， `css` 的世界可谓是渊博如海，让人又爱又恨。

我是一只小小小小鸟，想要飞呀飞却飞也飞不高，我寻寻觅觅寻寻觅觅一个，温暖的怀抱~~~ 🐤🐥🐥🐤

🍂 进入正题，接下来咱们就来一起实现自由的小鸟🐦吧。

学习本文，你可以学习到以下知识🍔:

*   `border` 绘制简单几何图形
    
*   `transform` 与 `animation` 知识
    
*   收获一只可爱的飞鸟
    

📕 border 属性学习
--------------

`border` 是个非常神奇、非常强大的属性，通过设置四个方向的 `border` 可以实现很多简单的几何图形。

下面用几个例子带大家学习 `border` 的渲染机制:

1.  ```
    设置 `div` 的 `width` 与 `height` 为 `0`，设置四个方向的 `border` 颜色不同。
    ```
    

```
border-left: 30px solid red;
border-right: 30px solid blue;
border-top: 30px solid green;
border-bottom: 30px solid pink;
```

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0QwFgrAnln0vmvjiaN8nIWXVvSt3w5ZialjoOwzUhtMDEWzqHCicZxtmgkg/640?wx_fmt=jpeg)

border.png

2.  分别设置 `height: 20px;width: 0` `width: 20px;height:0` 和 `width: 20px;height:20px`
    

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0Qo0qB3rxjYmuGcQ1Okfiby5Cp43dx9MgUwC8IMHypAZUG4XYlLIDmGibQ/640?wx_fmt=jpeg)

borderwh.png

> 上图按照步骤 2 的顺序设置

3.  不设置 `border-top`
    

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0Q66WIRYuCziaRt8dU9yP7NicDB9v0WwVibKMXVgWfx9av9wtH6YGGGk3Gw/640?wx_fmt=jpeg)

border-top.png

> 左侧图为设置 height: 20px 不设置 border-top；右侧图为设置 width: 20px 不设置 border-top

### 📄 border 绘制简单图形总结

通过上面三个案例，我们可以总结出 `border` 属性绘制简单图形的规律:

1.  `border` 共分四个方向，每个方向绘制其中一块，如果宽高都为 `0` ，每个方向绘制成三角形；否则绘制成梯形。
    
2.  `border` 的宽度为绘制梯形的高
    
3.  `height` 影响 `border-left` 和 `border-right` ，`width` 影响 `border-top` 和 `border-bottom` (例如示例 2 设置)
    
4.  当一个方向未绘制时， `border` 会只绘制对立方向部分，并且只会绘制一般 (例如示例 3 设置，会被腰斩)
    

🐣 绘制鸟儿
-------

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0QTic85DLHhMQ1GpFITG0Crsk1cZyKrEWTV5j5h38qj0fwUSajtSBDnkQ/640?wx_fmt=jpeg)

bird.png

由上面图可以看到小鸟共有四个部位: 头部、主体、两翼、尾巴，都由简单的几何图形构成。所以都可以通过 `border` 实现，下面来实现以下头部和两翼部分。

### 🐱‍🚀 头部实现

头部是个三角形，直接看代码

```
/* 设置 bottom 显示， left/right 为透明 */
border-right: 15px solid transparent;
border-bottom: 30px solid #6b83fa;
border-left: 15px solid transparent;
```

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0QcQ6VstxtlHV3RHmZLOBvNiahONOhkT3K7xeP0ozETufM6yJLr4mtkLQ/640?wx_fmt=jpeg)

birdhead.png

### 🛫 实现右翼

右翼由两部分，分别是直角梯形和直角三角形。通过 `border` 部分示例 3 可知，如果不设置 `border-top` ，`border-left/right` 的设置效果也会被腰斩，形成一个直角边，咱们就利用这个特性，就可以完成直角三角形和直角梯形的设置。

1.  ```
    直角三角形实现
    ```
    

```
/* 设置 border-left， borde-bottom 为透明 */
border-left: 30px solid #c0a9e0;
border-bottom: 30px solid transparent;
```

2.  ```
    直角梯形实现
    ```
    

```
/* 设置 height 拉长 left 形成梯形效果 */
height: 30px;
border-left: 30px solid #8567f0;
border-bottom: 10px solid transparent;
```

### 🎈 鸟儿组装

1.  鸟儿组装
    

通过 `position: relative` 和 `position: absolute` 将图形组装起来，组装后的小鸟如下图

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0Qpbxcufyb9VCZJ3QyiaKcLzjVILP5WslmKovKAoG3nqNs44mNFiaNxiaog/640?wx_fmt=jpeg)

bird1.gif

因为只是通过定位组装小鸟，小鸟的所有部位处于同一平面，看起来十分生硬。

2.  让鸟儿更立体
    

通过 `transform: rotate` 和 `transform-origin` 属性将小鸟的头部，尾部，两翼旋转一下，形成立体感

例如头部:

```
/* 设置 transform 的基点 */
transform-origin: 50% 100%;
/* 沿 x 轴偏移，负值逆时针旋转*/
transform: rotateX(-20deg);
```

经过 `rotate` 变换后的小鸟，是不是就生动多了。

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0QE27jHlBtSxxMCTrGJFo5La2fLQaW21N3SK6joSXF7VY0jicvGYib0sFQ/640?wx_fmt=jpeg)

bird2.gif

🌀 实现风
------

风的实现并不难，是通过 div 标签与 :before 配合实现。

```
.wind {
    position: absolute;
    width: 4px;
    height: 200px;
    /* 风是移动的，通过 overflow 可以限制风的长度 */
    overflow: hidden;
}
.wind:before {
    before {
    content: "";
    position: absolute;
    width: 4px;
    height: 300px;
    background: rgba(100, 200, 207, 0.3);
    transform: translateY(-300px);
    -webkit-animation: wind 2331ms 1268ms linear infinite; 
    animation: wind 2331ms 1268ms linear infinite;
}
```

就可以实现 `wind` ，之后依次按照上述实现 `10` 组 `wind` ，实现后的效果如下:

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0Q37l0hx2CPly7DTAOPkDicgo1D7a3FjxId5KcRLCMGkknecmHVtxqX0w/640?wx_fmt=jpeg)

wind.png

`10` 组 `wind` 重叠在一起了，使用 `transform: translate3d` 依次设置 `wind` 位置，不断调整，最终实现效果如下:

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0QvjoicshtibNBn4vicicVTFSbIHhwbM84M9zoicOPcIEAIXBzibQ7AFichvCGg/640?wx_fmt=jpeg)

wind3d.gif

🎡 让风动起来
--------

风动起来的效果实现原理非常简单：.wind 设置 height: 200px ，超出 200px 会隐藏， :before 伪类中可以使用 animation 属性，移动 wind 的 translateY

```
@-webkit-keyframes wind {
    0% {
        transform: translateY(-300px);
    }
    100% {
        transform: translateY(200px);
    }
}
.wind:before {
    transform: translateY(-300px);
    /* 设置每个风的延迟时间和持续时间都不相同 */
    animation: wind 2249ms 3544ms linear infinite;
}
```

风就动起来了，现在看起来效果就很赏心悦目了，就差飞翔的鸟儿了。

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0QylTTa3pmgXsR3qnPWicuClYz6eKZl48FKiaLE03RbxCB6CsY3yicbzbRw/640?wx_fmt=jpeg)

wind3dmove.gif

🚀 飞翔吧小鸟
--------

再让鸟儿飞起来之前，我们首先来设置一些基本属性

### 初始化 3D

1.  设置 `transform-style`: `preserve-3d` 开启 `3D` 渲染
    
2.  设置 `perspective`: 设置视距，让飞翔的鸟儿看起来更符合人类视觉效果
    
3.  设置 `drop-shadow`: 真实世界的飞鸟怎么能没有影子那。
    

### 两翼动画

让鸟儿飞起来最重要的就是两翼动起来，分别设置两个动画 `wingRight` 和 `windLeft`

```
/* 两翼翅膀的时针相反，因此动画是反的 */
@keyframes wingRight {
    0% {
        transform: rotateY(40deg);
    }
    100% {
        transform: rotateY(-40deg);
    }
}
@-webkit-keyframes wingLeft {
    0% {
        transform: rotateY(-40deg);
    }
    100% {
        transform: rotateY(40deg);
    }
}
```

设置完动画后，还有至关重要的一步，要设置 `transform-origin` ，以右翼为例子，翅膀应该沿红框标识线旋转

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0QzXnThfZK4Lg9ex3rEx03IKqMic9SLnJyHricWHNUSZ20H72GaKRTBUMw/640?wx_fmt=jpeg)

birdfly.png

`transform-origin` 默认值为 `50% 50% 0`，很明显是无法满足旋转要求的，根据浏览器坐标系的知识，因此设置右翼 `transform-origin` 为 `0 0` ，同理设置左翼为 `100% 0`

### 整体动画

光在一个平面中飞翔不免有几分无聊和乏味，作为天空中的精灵，鸟儿应该自由的飞翔。给鸟儿添加绕 `z` 轴 `360°` 旋转的动画。

![](https://mmbiz.qpic.cn/mmbiz/pfCCZhlbMQRnwkIRuQiaFpt4j3P3iaSU0QrXP4icbrFXkKRnDeFayAM2Yib8ZPlkcnAwPJK6tK2pfBTGCkuxDZFzpA/640?wx_fmt=jpeg)

flybird.gif

🛕 源码仓库
-------

传送门: flybird   
https://github.com/zcxiaobao/zc-demos/blob/main/display/paper-bird/paperBird.html

如果感觉有帮助的话，别忘了点个赞👍。

> 作者：战场小包
> 
> https://juejin.cn/post/7032876592544088101

- EOF -