> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651582910&idx=1&sn=40b350e2eec3dadba9b8333ac2d38ecf&chksm=8025267fb752af6970bd23d858cc2c77d867b4c7acd45b92119c56ada4167bca5f74fc0bc46f&scene=21#wechat_redirect)

前言
==

偶然接触到 CSS 的 3D 属性, 就萌生了一种做 **3D 游戏**的想法.

了解过 css3D 属性的同学应该都了解过`perspective`、`perspective-origin`、`transform-style: preserve-3d`这个三个属性值, 它们构成了 CSS 的 3d 世界.

同时, 还有`transform`属性来对 3D 的节点进行平移、缩放、旋转以及拉伸.

属性值很简单, 在我们平时的 web 开发中也很少用到.

**那用这些 CSS3D 属性可以做 3D 游戏吗?**

当然是可以的.

即使只有沙盒, 也有**我的世界**这种神作.

今天我就来带大家玩一个从未有过的全新 3D 体验.

废话不多说, 我们先来看下效果:

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/H8M5QJDxMHpHuEytHHLgSon79xuAoHrquqBqGgqNYeOtbtSiaJGufvI2HJN6DH87IAhBI7vkdMLyRS2e6vN2PCA/640?wx_fmt=gif)

我们要完成这个**迷宫大作战**, 需要完成以下步骤:

1.  创建一个 3D 世界
    
2.  写一个 3D 相机的功能
    
3.  创建一座 3D 迷宫
    
4.  创建一个可以自由运动的玩家
    
5.  在迷宫中找出一条最短路径提示
    

我们先来看下一些前置知识.

做一款 CSS3D 游戏需要的知识和概念
====================

CSS3D 坐标系
---------

在 css3D 中, 首先要明确一个概念, **3D** 坐标系.  
使用**左手坐标系**, 伸出我们的左手, 大拇指和食指成 **L** 状, 其他手指与食指垂直, 如图:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHpHuEytHHLgSon79xuAoHrqCPxicib6CY59RlvNpEaECkMh9IvOqra84B1b8CTIW1KvOvORTmlGkpVQ/640?wx_fmt=png)

  

大拇指为 X 轴, 食指为 Y 轴, 其他手指为 Z 轴.  
这个就是 CSS3D 中的坐标系.

透视属性
----

`perspective`为 css 中的透视属性.

这个属性是什么意思呢, 可以把我们的眼睛看作观察点, 眼睛到目标物体的距离就是视距, 也就是这里说的透视属性.

大家都知道, _「透视」+「2D」= 「3D」_.

```
perspective: 1200px;
-webkit-perspective:  1200px;
```

3D 相机
-----

在 3D 游戏开发中, 会有相机的概念, 即是人眼所见皆是相机所见.  
在游戏中场景的移动, 大部分都是移动相机.  
例如赛车游戏中, 相机就是跟随车子移动, 所以我们才能看到一路的风景.  
在这里, 我们会使用 CSS 去实现一个伪 3d 相机.

变换属性
----

在 CSS3D 中我们对 3D 盒子做平移、旋转、拉伸、缩放使用`transform`属性.

*   translateX 平移 X 轴
    
*   translateY 平移 Y 轴
    
*   translateZ 平移 Z 轴
    
*   rotateX 旋转 X 轴
    
*   rotateY 旋转 Y 轴
    
*   rotateZ 旋转 Z 轴
    
*   rotate3d(x,y,z,deg) 旋转 X、Y、Z 轴多少度
    

> 注意:  
> 这里「先平移再旋转」和「先旋转再平移」是不一样的  
> 旋转的角度都是角度值.

矩阵变换
----

我们完成游戏的过程中会用到矩阵变换.  
在 js 中, 获取某个节点的`transform`属性, 会得到一个矩阵, 这里我打印一下, 他就是长这个样子:

```
var _ground = document.getElementsByClassName("ground")[0];
var bg_style = document.defaultView.getComputedStyle(_ground, null).transform;
console.log("矩阵变换---->>>",bg_style)
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHpHuEytHHLgSon79xuAoHrqLwOZCtIu16QSibmoP79hDUJzHSYRjHXzniabE6OLqm03bQb1d3tQaY5Q/640?wx_fmt=png)

  

**那么我们如何使用矩阵去操作 transform 呢?**

在线性变换中, 我们都会去使用矩阵的相乘.  
CSS3D 中使用 4*4 的矩阵进行 3D 变换.  
下面的矩阵我均用二维数组表示.  
例如`matrix3d(1,0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,1)`可以用二维数组表示:

```
[
    [1, 0, 0, 0],
    [0, 1, 0, 0],
    [0, 0, 1, 0],
    [0, 0, 0, 1]
]
```

平移即使使用原来状态的矩阵和以下矩阵相乘, dx, dy, dz 分别是移动的方向 x, y, z.

```
[
    [1, 0, 0, dx],
    [0, 1, 0, dy],
    [0, 0, 1, dz],
    [0, 0, 0, 1]
]
```

绕 X 轴旋转𝞱, 即是与以下矩阵相乘.

```
[
    [1, 0, 0, 0],
    [0, cos𝞱, sin𝞱, 0],
    [0, -sin𝞱, cos𝞱, 0],
    [0, 0, 0, 1]
]
```

绕 Y 轴旋转𝞱, 即是与以下矩阵相乘.

```
[
    [cos𝞱, 0, -sin𝞱, 0],
    [0, 1, 0, 0],
    [sin𝞱, 0, cos𝞱, 0],
    [0, 0, 0, 1]
]
```

绕 Z 轴旋转𝞱, 即是与以下矩阵相乘.

```
[
    [cos𝞱, sin𝞱, 0, 0],
    [-sin𝞱, cos𝞱, 0, 0],
    [0, 0, 1, 0],
    [0, 0, 0, 1]
]
```

具体的矩阵的其他知识这里讲了, 大家有兴趣可以自行下去学习.  
我们这里只需要很简单的旋转应用.

开始创建一个 3D 世界
============

我们先来创建 UI 界面.

*   相机 div
    
*   地平线 div
    
*   棋盘 div
    
*   玩家 div(这里是一个正方体)
    

> 注意  
> 正方体先旋转在平移, 这种方法应该是最简单的.  
> 一个平面绕 X 轴、Y 轴旋转 180 度、±90 度, 都只需要平移 Z 轴.  
> 这里大家试过就明白了.

我们先来看下 html 部分:

```
    <div class="camera">
        <!-- 地面 -->
        <div class="ground">
            <div class="box">
                <div class="box-con">
                    <div class="wall">z</div>
                    <div class="wall">z</div>
                    <div class="wall">y</div>
                    <div class="wall">y</div>
                    <div class="wall">x</div>
                    <div class="wall">x</div>
                    <div class="linex"></div>
                    <div class="liney"></div>
                    <div class="linez"></div>
                </div>
                <!-- 棋盘 -->
                <div class="pan"></div>
            </div>
        </div>
    </div>
```

很简单的布局, 其中`linex`、`liney`、`linez`是我画的坐标轴辅助线.  
红线为 X 轴, 绿线为 Y 轴, 蓝线为 Z 轴. 接着我们来看下正方体的主要 CSS 代码.

```
...
  .box-con{
      width: 50px;
      height: 50px;
      transform-style: preserve-3d;
      transform-origin: 50% 50%;
      transform: translateZ(25px) ;
      transition: all 2s cubic-bezier(0.075, 0.82, 0.165, 1);
  }
  .wall{
      width: 100%;
      height: 100%;
      border: 1px solid #fdd894;
      background-color: #fb7922;
    
  }
  .wall:nth-child(1) {
      transform: translateZ(25px);
  }
  .wall:nth-child(2) {
      transform: rotateX(180deg) translateZ(25px);
  }
  .wall:nth-child(3) {
      transform: rotateX(90deg) translateZ(25px);
  }
  .wall:nth-child(4) {
      transform: rotateX(-90deg) translateZ(25px);
  }
  .wall:nth-child(5) {
      transform: rotateY(90deg) translateZ(25px);
  }
  .wall:nth-child(6) {
      transform: rotateY(-90deg) translateZ(25px);
  }
```

粘贴一大堆 CSS 代码显得很蠢.  
其他 CSS 这里就不粘贴了, 有兴趣的同学可以直接下载源码查看. 界面搭建完成如图所示:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHpHuEytHHLgSon79xuAoHrqMDmanzEyZUzdT9kYsZYyfrpTktkmXmIoibibp9MSs2b9NIDfick3jWAibw/640?wx_fmt=png)

  

接下来就是重头戏了, 我们去写 js 代码来继续完成我们的游戏.

完成一个 3D 相机功能
============

相机在 3D 开发中必不可少, 使用相机功能不仅能查看 3D 世界模型, 同时也能实现很多实时的炫酷功能.

**一个 3d 相机需要哪些功能?**

最简单的, 上下左右能够 360 度无死角观察地图. 同时需要拉近拉远视距.

**通过鼠标交互**

鼠标左右移动可以旋转查看地图; 鼠标上下移动可以观察上下地图; 鼠标滚轮可以拉近拉远视距.

✅ _1. 监听鼠标事件_

首先, 我们需要通过监听鼠标事件来记录鼠标位置, 从而判断相机上下左右查看.

```
    /** 鼠标上次位置 */
    var lastX = 0, lastY = 0;
      /** 控制一次滑动 */
    var isDown = false;
      /** 监听鼠标按下 */
    document.addEventListener("mousedown", (e) => {
        lastX = e.clientX;
        lastY = e.clientY;
        isDown = true;
    });
        /** 监听鼠标移动 */
    document.addEventListener("mousemove", (e) => {
        if (!isDown) return;
        let _offsetX = e.clientX - lastX;
        let _offsetY = e.clientY - lastY;
        lastX = e.clientX;
        lastY = e.clientY;
        //判断方向
        var dirH = 1, dirV = 1;
        if (_offsetX < 0) {
            dirH = -1;
        }
        if (_offsetY > 0) {
            dirV = -1;
        }
    });
    document.addEventListener("mouseup", (e) => {
        isDown = false;
    });
```

✅ _2. 判断相机上下左右_

使用`perspective-origin`来设置相机的上下视线.  
使用`transform`来旋转 Z 轴查看左右方向上的 360 度.

```
/** 监听鼠标移动 */
    document.addEventListener("mousemove", (e) => {
        if (!isDown) return;
        let _offsetX = e.clientX - lastX;
        let _offsetY = e.clientY - lastY;
        lastX = e.clientX;
        lastY = e.clientY;
        var bg_style = document.defaultView.getComputedStyle(_ground, null).transform;
        var camera_style = document.defaultView.getComputedStyle(_camera, null).perspectiveOrigin;
        var matrix4 = new Matrix4();
        var _cy = +camera_style.split(' ')[1].split('px')[0];
        var str = bg_style.split("matrix3d(")[1].split(")")[0].split(",");
        var oldMartrix4 = str.map((item) => +item);
        var dirH = 1, dirV = 1;
        if (_offsetX < 0) {
            dirH = -1;
        }
        if (_offsetY > 0) {
            dirV = -1;
        }
        //每次移动旋转角度
        var angleZ = 2 * dirH;
        var newMartri4 = matrix4.set(Math.cos(angleZ * Math.PI / 180), -Math.sin(angleZ * Math.PI / 180), 0, 0, Math.sin(angleZ * Math.PI / 180), Math.cos(angleZ * Math.PI / 180), 0, 0, 0, 0, 1, 0, 0, 0, 0, 1);
        var new_mar = null;
        if (Math.abs(_offsetX) > Math.abs(_offsetY)) {
            new_mar = matrix4.multiplyMatrices(oldMartrix4, newMartri4);
        } else {
            _camera.style.perspectiveOrigin = `500px ${_cy + 10 * dirV}px`;
        }
        new_mar && (_ground.style.transform = `matrix3d(${new_mar.join(',')})`);
    });
```

这里使用了矩阵的方法来旋转 Z 轴, 矩阵类`Matrix4`是我临时写的一个方法类, 就俩方法, 一个设置二维数组`matrix4.set`, 一个矩阵相乘`matrix4.multiplyMatrices`.  
文末的源码地址中有, 这里就不再赘述了.

✅ _3. 监听滚轮拉近拉远距离_

这里就是根据`perspective`来设置视距.

```
//监听滚轮
document.addEventListener('mousewheel', (e) => {
    var per = document.defaultView.getComputedStyle(_camera, null).perspective;
    let newper = (+per.split("px")[0] + Math.floor(e.deltaY / 10)) + "px";
    _camera.style.perspective = newper
}, false);
```

> 注意:  
> perspective-origin 属性只有 X、Y 两个值, 做不到和 u3D 一样的相机.  
> 我这里取巧使用了对地平线的旋转, 从而达到一样的效果.  
> 滚轮拉近拉远视距有点别扭, 和 3D 引擎区别还是很大.

完成之后可以看到如下的场景, 已经可以随时观察我们的地图了.

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/H8M5QJDxMHpHuEytHHLgSon79xuAoHrqfLIhFtV0R0aUI3JCHspFbgORrQMEsvs9s0VBJqNtAXA3QHS9UIQLyA/640?wx_fmt=gif)

  

这样子, 一个 3D 相机就完成, 大家有兴趣的可以自己下去写一下, 还是很有意思的.

绘制迷宫棋盘
======

绘制格子地图最简单了, 我这里使用一个 15*15 的数组.  
「_0_」代表可以通过的路, 「_1_」代表障碍物.

```
var grid = [
    0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0,
    0, 1, 0, 1, 0, 0, 1, 0, 1, 0, 1, 1, 0, 1, 0,
    1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1,
    0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 1, 0, 1,
    0, 0, 1, 0, 1, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0,
    0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0,
    0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 1, 0,
    1, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0,
    1, 0, 1, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0, 1, 0,
    0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    1, 0, 1, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 0, 1,
    0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0,
    1, 1, 0, 0, 1, 0, 0, 1, 0, 1, 0, 1, 0, 0, 0,
    1, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0,
    0, 0, 1, 0, 1, 0, 1, 0, 0, 1, 1, 0, 0, 0, 0
];
```

然后我们去遍历这个数组, 得到地图.  
写一个方法去创建地图格子, 同时返回格子数组和节点数组.  
这里的`block`是在 html 中创建的一个预制体, 他是一个正方体.  
然后通过克隆节点的方式添加进棋盘中.

```
/** 棋盘 */
function pan() {
    const con = document.getElementsByClassName("pan")[0];
    const block = document.getElementsByClassName("block")[0];
    let elArr = [];
    grid.forEach((item, index) => {
        let r = Math.floor(index / 15);
        let c = index % 15;
        const gezi = document.createElement("div");
        gezi.classList = "pan-item"
        // gezi.innerHTML = `${r},${c}`
        con.appendChild(gezi);
        var newBlock = block.cloneNode(true);
        //障碍物
        if (item == 1) {
            gezi.appendChild(newBlock);
            blockArr.push(c + "-" + r);
        }
        elArr.push(gezi);
    });
    const panArr = arrTrans(15, grid);
    return { elArr, panArr };
}
const panData = pan();
```

可以看到, 我们的界面已经变成了这样.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHpHuEytHHLgSon79xuAoHrqCuUboJr4U0no0ic5ES1aZkZfoLyTQWFJ9XE7BUtJtSrjMS1FkQbWaqg/640?wx_fmt=png)

  

接下来, 我们需要去控制玩家移动了.

控制玩家移动
======

通过上下左右`w s a d`键来控制玩家移动.  
使用`transform`来移动和旋转玩家盒子.

✅ _监听键盘事件_

通过监听键盘事件`onkeydown`来判断`key`值的上下左右.

```
document.onkeydown = function (e) {
    /** 移动物体 */
    move(e.key);
}
```

✅ _进行位移_

在位移中, 使用`translate`来平移, Z 轴始终正对我们的相机, 所以我们只需要移动 X 轴和 Y 轴.  
声明一个变量记录当前位置.  
同时需要记录上次变换的`transform`的值, 这里我们就不继续矩阵变换了.

```
/** 当前位置 */
var position = { x: 0, y: 0 };
/** 记录上次变换值 */
var lastTransform = {
    translateX: '0px',
    translateY: '0px',
    translateZ: '25px',
    rotateX: '0deg',
    rotateY: '0deg',
    rotateZ: '0deg'
};
```

每一个格子都可以看成是二维数组的下标构成, 每次我们移动一个格子的距离.

```
 switch (key) {
    case 'w':
        position.y++;
        lastTransform.translateY = position.y * 50 + 'px';
        break;
    case 's':
        position.y--;
        lastTransform.translateY = position.y * 50 + 'px';
        break;
    case 'a':
        position.x++;
        lastTransform.translateX = position.x * 50 + 'px';
        break;
    case 'd':
        position.x--;
        lastTransform.translateX = position.x * 50 + 'px';
        break;
}
//赋值样式
for (let item in lastTransform) {
    strTransfrom += item + '(' + lastTransform[item] + ') ';
}
target.style.transform = strTransfrom;
```

到这里, 我们的玩家盒子已经可以移动了.

> 注意  
> 在 css3D 中的平移可以看成是世界坐标.  
> 所以我们只需要关心 X、Y 轴. 而不需要去移动 Z 轴. 即使我们进行了旋转.

✅ _在移动的过程中进行旋转_

在 CSS3D 中, 3D 旋转和其他 3D 引擎中不一样, 一般的诸如 u3D、threejs 中, 在每次旋转完成之后都会重新校对成世界坐标, 相对来说 就很好计算绕什么轴旋转多少度.

然而, 笔者也低估了 CSS3D 的旋转.  
我以为上下左右滚动一个正方体很简单. 事实并非如此.

CSS3D 的旋转涉及到四元数和万向锁.

比如我们旋转我们的玩家盒子. 如图所示:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHpHuEytHHLgSon79xuAoHrq5gw15erYTOP8PQ23YoS4fseA8YqjVRfFIBaC1ZibB1GZnHBkhCe8Atg/640?wx_fmt=png)首先, 第一个格子 (0,0) 向上绕 X 轴旋转 90 度, 就可以到达(1.0); 向左绕 Y 轴旋转 90 度, 可以到达(0,1); 那我们是不是就可以得到规律如下:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHpHuEytHHLgSon79xuAoHrq87fQ0YUYChag03XzrWQ5UDXDGbgRGE3fQxV0FSmSSa6FiaZXEnEDLaQ/640?wx_fmt=png)

  

如图中所示, 单纯的向上下, 向左右绕轴旋转没有问题, 但是要旋转到红色的格子, 两种不同走法, 到红色的格子之后旋转就会出现两种可能. 从而导致旋转出错.

同时这个规律虽然难寻, 但是可以写出来, 最重要的是, **按照这个规律来旋转 CSS3D 中的盒子, 是不对的**

**那有人就说了, 这不说的屁话吗?**

经过笔者实验, 倒是发现了一些规律. 我们继续按照这个规律往下走.

*   旋转 X 轴的时候, 同时看当前 Z 轴的度数, Z 轴为 90 度的奇数倍, 旋转 Y 轴, 否则旋转 X 轴.
    
*   旋转 Y 轴的时候, 同时看当前 Z 轴的度数, Z 轴为 90 度的奇数倍, 旋转 X 轴, 否则旋转 Z 轴.
    
*   旋转 Z 轴的时候, 继续旋转 Z 轴
    

这样子我们的旋转方向就搞定了.

```
if (nextRotateDir[0] == "X") {
    if (Math.floor(Math.abs(lastRotate.lastRotateZ) / 90) % 2 == 1) {
        lastTransform[`rotateY`] = (lastRotate[`lastRotateY`] + 90 * dir) + 'deg';
    } else {
        lastTransform[`rotateX`] = (lastRotate[`lastRotateX`] - 90 * dir) + 'deg';
    }
}
if (nextRotateDir[0] == "Y") {
    if (Math.floor(Math.abs(Math.abs(lastRotate.lastRotateZ)) / 90) % 2 == 1) {
        lastTransform[`rotateX`] = (lastRotate[`lastRotateX`] + 90 * dir) + 'deg';
    } else {
        lastTransform[`rotateZ`] = (lastRotate[`lastRotateZ`] + 90 * dir) + 'deg';
    }
}
if (nextRotateDir[0] == "Z") {
    lastTransform[`rotate${nextRotateDir[0]}`] = (lastRotate[`lastRotate${nextRotateDir[0]}`] - 90 * dir) + 'deg';
}
```

然而, 这还没有完, 这种方式的旋转还有个坑, 就是我不知道该旋转 90 度还是 - 90 度了.  
这里并不是简单的上下左右去加减.

**旋转方向对了, 旋转角度不知该如何计算了.**

具体代码可以查看源码.

**彩蛋时间**

⚠️⚠️⚠️ 同时这里会伴随着「_万向锁_」的出现, 即是 Z 轴与 X 轴重合了. 哈哈哈哈~  
⚠️⚠️⚠️ 这里笔者还没有解决, 也希望万能的网友能够出言帮忙~  
⚠️⚠️⚠️ 笔者后续解决了会更新的. 哈哈哈哈, 大坑.

好了, 这里问题不影响我们的项目. 我们继续讲如何找到最短路径并给出提示.

最短路径的计算
=======

在迷宫中, 从一个点到另一个点的最短路径怎么计算呢? 这里笔者使用的是广度优先遍历 (**BFS**) 算法来计算最短路径.

我们来思考:

1.  二维数组中找最短路径
    
2.  每一格的最短路径只有上下左右相邻的四格
    
3.  那么只要递归寻找每一格的最短距离直至找到终点
    

这里我们需要使用「_队列_」先进先出的特点.

我们先来看一张图:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHpHuEytHHLgSon79xuAoHrqwrnw6B9kX2DOicicbzD1UnfXxic3SMwjYjX2PyzsXI3xGqibVTyfcZBDFg/640?wx_fmt=png)

  

很清晰的可以得到最短路径.

> 注意  
> 使用两个长度为 4 的数组表示上下左右相邻的格子需要相加的下标偏移量.  
> 每次入队之前需要判断是否已经入队了.  
> 每次出队时需要判断是否是终点.  
> 需要记录当前入队的目标的父节点, 方便获取到最短路径.

我们来看下代码:

```
//春初路径
var stack = [];
/**
 * BFS 实现寻路
 * @param {*} grid 
 * @param {*} start {x: 0,y: 0}
 * @param {*} end {x: 3,y: 3}
 */
function getShortPath(grid, start, end, a) {
    let maxL_x = grid.length;
    let maxL_y = grid[0].length;
    let queue = new Queue();
    //最短步数
    let step = 0;
    //上左下右
    let dx = [1, 0, -1, 0];
    let dy = [0, 1, 0, -1];
    //加入第一个元素
    queue.enqueue(start);
    //存储一个一样的用来排查是否遍历过
    let mem = new Array(maxL_x);
    for (let n = 0; n < maxL_x; n++) {
        mem[n] = new Array(maxL_y);
        mem[n].fill(100);
    }
    while (!queue.isEmpty()) {
        let p = [];
        for (let i = queue.size(); i > 0; i--) {
            let preTraget = queue.dequeue();
            p.push(preTraget);
            //找到目标
            if (preTraget.x == end.x && preTraget.y == end.y) {
                stack.push(p);
                return step;
            }
            //遍历四个相邻格子
            for (let j = 0; j < 4; j++) {
                let nextX = preTraget.x + dx[j];
                let nextY = preTraget.y + dy[j];

                if (nextX < maxL_x && nextX >= 0 && nextY < maxL_y && nextY >= 0) {
                    let nextTraget = { x: nextX, y: nextY };
                    if (grid[nextX][nextY] == a && a < mem[nextX][nextY]) {
                        queue.enqueue({ ...nextTraget, f: { x: preTraget.x, y: preTraget.y } });
                        mem[nextX][nextY] = a;
                    }
                }
            }
        }
        stack.push(p);
        step++;
    }
}
/* 找出一条最短路径**/
function recall(end) {
    let path = [];
    let front = { x: end.x, y: end.y };
    while (stack.length) {
        let item = stack.pop();
        for (let i = 0; i < item.length; i++) {
            if (!item[i].f) break;
            if (item[i].x == front.x && item[i].y == front.y) {
                path.push({ x: item[i].x, y: item[i].y });
                front.x = item[i].f.x;
                front.y = item[i].f.y;
                break;
            }
        }
    }
    return path;
}
```

这样子我们就可以找到一条最短路径并得到最短的步数.  
然后我们继续去遍历我们的原数组 (即棋盘原数组).  
点击提示点亮路径.

```
var step = getShortPath(panArr, { x: 0, y: 0 }, { x: 14, y: 14 }, 0);
console.log("最短距离----", step);
_perstep.innerHTML = `请在<span>${step}</span>步内走到终点`;
var path = recall({ x: 14, y: 14 });
console.log("路径---", path);
/** 提示 */
var tipCount = 0;
_tip.addEventListener("click", () => {
    console.log("9999", tipCount)
    elArr.forEach((item, index) => {
        let r = Math.floor(index / 15);
        let c = index % 15;
        path.forEach((_item, i) => {
            if (_item.x == r && _item.y == c) {
                // console.log("ooo",_item)
                if (tipCount % 2 == 0)
                    item.classList = "pan-item pan-path";
                else
                    item.classList = "pan-item";
            }
        })
    });
    tipCount++;
});
```

这样子, 我们可以得到如图的提示:

![](https://mmbiz.qpic.cn/sz_mmbiz_png/H8M5QJDxMHpHuEytHHLgSon79xuAoHrqD9XFkqicj0sEocKo00WhFjuGHznXoaMfYawOnV275fVNSh14WqGO0NQ/640?wx_fmt=png)

  

大功告成. 嘿嘿, 是不是很惊艳的感觉~

尾声
==

当然, 我这里的这个小游戏还有可以完善的地方 比如:

*   可以增加道具, 拾取可以减少已走步数
    
*   可以增加配置关卡
    
*   还可以增加跳跃功能
    
*   ...
    

原来如此, CSS3D 能做的事还有很多, 怎么用全看自己的想象力有多丰富了.

哈哈哈, 真想用 CSS3D 写一个「_我的世界_」玩玩, 性能问题恐怕会有点大.

本文例子均在 PC 端体验较好.

试玩地址

https://xdq1553.github.io/-CSS3D-/

源码地址

https://github.com/xdq1553/-CSS3D-

> 作者：起小就些熊
> 
> https://juejin.cn/post/7000963575573381134

- EOF -