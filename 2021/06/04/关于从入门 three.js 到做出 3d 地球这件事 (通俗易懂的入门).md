> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651574123&idx=2&sn=52a2852d0105d69e69b3027f90f34b42&chksm=802518aab75291bc237a2b0ea3dfa3e2d36def01fc200ceedc65f79c63f50622b6bced485f74&scene=21#wechat_redirect)

↓推荐关注↓

公众号

### 开篇介绍

如果你没接触过 3d 可视化技术, 你也许会认为可视化非常难, 光是一个物体的阴影要如何计算就相当复杂, 但是告诉你个好消息, 阴影的计算都是集成好的, 而我们只要设置好光源的位置, 绘制好物体就可以了, 真的没有想象中那么复杂, 本文面向有前端基础, 但零可视化基础的同学, 我会从最基础的入门知识说起。

学习可视化方面的技术会让我们对计算机, 对前端技术有更深的理解, 还可以做出更多有趣味的东西来, 本文是我踩了好多坑后总结出来的, 我更清楚一个初入门的小白哪里不懂。

`three.js`是 `webgl`的第三方库, 它更适合不太复杂的可视化项目, 而我们要做的 3d 地球项目使用它来做会更简单, 所以选择了它, 放心后面也会说`webgl`相关知识 。

当前效果如下:

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTy1rtCm2CCmQYb06Yic0yNcmek4dibKKl72HEZLsb3h22NqEUNVOPRLiccA/640?wx_fmt=png)

### 一. 关于此系列文章

1.  `自食其力:`不管是在公司还是网上都有类似的库, 但是当遇到 bug 或是缺少功能的情况时就会很麻烦, 例如我们公司的 FGL 库 (一个内网绘制 3d 景象的技术), 它官网上的例子很多都是错的, 使用起来也是一堆问题, 比如无法精准选择某个国家, 点击事件消融等 bug。还比如说`Echarts`的地球, 它太注重真实感并且用起来有点卡, 以及交互做的不太好。
    
2.  `直指核心:` 去年我通过看书、看文章、看视频认真的学习`three.js`, 并做出了 3d 地球这个项目, 而这个系列文章将会直指做出 3d 地图的核心知识, 尽量不随意扩散知识面。
    
3.  `更好入门:` 网上的教学文章千篇一律, 点进去阅读完感觉其对于一个`three.js`零基础的同学来说都不太好懂, 教学视频里的知识点太广泛, 事无巨细的罗列, 而这个系列文章将更突出绘制 3d 地球这个重点。
    
4.  `同道中人:` 我学习`three.js`就是为了做出 3d 地球, 期间走了不少弯路, 被某些问题卡了很久, 所以我更懂一个刚入门的人困惑的点在哪里。
    
5.  `专注vue:` 市面上较少专门针对`vue`做到开箱即用的 3d 地球插件, 而我们就要编写这样一款产品。
    
6.  `不断学习:` 编写文章也是我提高自己能力的一种方法, 死磕每个知识点让自己的理解更上一层楼。
    

### 二. 任务目标

1.  入门`three.js`技术。
    
2.  绘制出 3d 地球。
    
3.  做成专门`vue`使用的库。
    
4.  后期也会介绍`着色器`的概念与基本的使用技巧。
    
5.  会介绍少量`webgl`的相关用法, 并且会有部分数学知识。
    

### 三. 文章主线剧情与支线任务

*   主线剧情: 围绕着如何做出 3d 地球, 这部分在 vue 工程里面进行。
    
*   支线任务: 每个分散的知识点, 可能与 3d 地球没关系, 但是它能帮助我们更好的理解 3d 技术, 而这些知识点我就不在 vue 项目里面演示了, 会单独创建一个 html 文件来演示说明。
    

### 四. 理解坐标系: 别着急写代码先有基本模型

像绘制图形这类技术, 最基本的概念就坐标系, 下图是`二维坐标系`, 我们的故事就从这个家伙开始。

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTy51gD1p6wVVVqBlUnSqIkFxpxkUj6Xn0DibZJRic2Q6hoPDtZ6WECdtlw/640?wx_fmt=png)

我们用`(0, 0)`表示坐标的中心点, 绘制一条起点为中心点长度为 1 的线段可以使用 `(0, 0) (1, 0)`这两个点相连表示。

###### 关于向量的概念后面需要用数学知识的时候再介绍, 前几篇文章就越通俗越好。

在`three.js`中我们要打交道的就是下面这位`三维坐标系`

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTy9O9CgoHX1vlk3hK1XZtB6LyYrXM1HBde8dAfjMd9wPhFt7LjwLxib6w/640?wx_fmt=png)

他的坐标原点就是`(0, 0, 0)`, 绘制一条起点为中心点的长度为 1 的线段可以是 `(0, 0, 0) (1, 0, 0)`。

这里要记住, `three.js`里面设置的默认坐标系就是这种形式`x向右, y向上, z向前`, 之所以说是默是因为它可以修改。

上图中, 观看这个三维坐标系的目光其实是在斜上方, 正常情况下在我们开发的时候`z轴`是正对着我们的眼睛的, 所以你只能看到`z轴`是一个点,

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTyLCfWDfIpQT904soL1z6YpBEbdwwmHtUJseFVMbLkBvGgvO77wuibYbA/640?wx_fmt=png)

###### 在开发与学习的时候, 最好先把坐标系绘制到页面上, 方便我们更好的绘制。

### 五. 相机的概念

假设现在我们的正前方有一个`三维坐标系`的全息投影, 那么此时你的眼睛就相当于一架相机, 你看到的 `坐标系`景象取决于你站的位置。

在`three.js`中就有这样一个对象, 他就是负责从哪个角度观察我们绘制的 3d 世界, 也就是`相机`这个概念的由来。

相机分为两种, 正投影相机和透视投影相机, 正投影相机就是你站的多远你看到的物体的大小都不变, 透视投影相机就是物体会`近大远小`, 下面是张引用图 (图片来自网络)。

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTyRALVJcs5DOSP4BraZLyFFno6Gicl1b7OVGWCSr6YBvJvayNfRTDNN9w/640?wx_fmt=png)

正投影相机可以用在`工程制图`上, 或者可以做一些视觉欺骗小游戏。

###### 本文主要目的是绘制 3d 地球所以主要使用透视投影相机

### 六. 绘制坐标系, 安放摄像机 (代码安排上)

引入`three.js`, 可以把包下载到本地, 也可以直接获取在 cdn 上的资源, 引入之后全局会出现`THREE`对象, 我们就可以开始编程之旅了。

一个普普通通的 html 空文件的 script 标签里面, 发生着这样的故事: 让我们逐句解析

###### 第一步: 创建场景, 也就是虚拟的空间

我们之后绘制的`3d物体`都要放入这个空间里面, 你可以把它当做一个鸿蒙空间神器, 里面有一个小世界, 而我们是掌控者 (很中二)。

```
const scene = new THREE.Scene();


```

###### 第二步: 创建相机

相机的概念上面讲述过了, `PerspectiveCamera`这个类就是`透视投影相机`, 我们来逐个攻破他参数的意思。

1.  `35`: `视角`也就是我们左眼与右眼可以看到的横向角度, 其越小物体则越大, 因为目光变狭窄会突出物体, 你可以做一个实验, 聚精会神的盯着看一个物体, 你就会发现此时你左右两边本来靠余光可以看到的物体你现在看不清, 这个就是你的视角变小了, 变小视角还可以使目标物体比例变大, 我们知道这些就够理解这个数字了, 后期可以利用这个原理做一些令人惊讶的动画特效。
    
2.  `window.innerWidth / window.innerHeight`: 纵横比`宽/高`, 这里宽高不会去写`px`这种单位, 坐标系里面是一种抽象的长度单位, 所以要告诉浏览器咱们当前显示图像的区域的宽高比例 (可以当它是百分比布局, 就像我们写 css 布局时使用`vh` `vw`为单位)。
    
3.  `1`: `近平面`, 简单理解就是当一个`图像`距离`相机`的距离小于 1 的时候, 就不显示这个图像了。
    
4.  `1000`: `远平面`, 简单理解就是当一个`图像`距离`相机`的距离大于 1000 的时候, 就不显示这个图像了。
    
5.  `camera.position.z = 10;` 相机的坐标不设置的话, 默认就是 (0, 0, 0) 坐标原点, 这样类似脑袋在坐标轴原点上看坐标轴, 所以这里要设置距离坐标中心有一定距离, 也就是远距离观察这个坐标系。
    

```
const camera = new THREE.PerspectiveCamera(35, window.innerWidth / window.innerHeight, 1, 1000);
camera.position.z = 10;


```

*   无聊的知识: 我们在玩`3d游戏`的时候, 是不是有时候与另一个游戏人物距离太近了就会出现`人物中空`的效果, 这些很可能就是他的某些部分距离你相机的距离, 小于了`近平面`的距离导致的。
    
*   物体距离眼睛越近越大, 越远越小, 因为一个物品无限大与无限远没有意义, 显示起来浪费性能, 所以才会设置近平面与远平面。
    

###### 第三步: 生成渲染实例

1.  `WebGLRenderer`生成一个渲染实例, 用来渲染我们所有的 3d 效果。
    
2.  `setSize`设置场景的宽高。
    
3.  `setClearColor`设置背景色, 这个背景色不是平面的, 是全方位的, 你可以想想成你在一个屋子里, 这个颜色就是屋子墙壁、地板、天花板的颜色 (.5 是透明度)。
    
4.  `renderer.domElement`生成的渲染的实例, 这个要放到对应的 dom 容器里面 (是个 canvas 标签)。
    

```
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setClearColor(0x00FFFF, .5)
document.body.appendChild(renderer.domElement);


```

*   知识点: `setClearColor`不写就是黑色
    
*   知识点: `setClearColor`可以直接写 "red" 这种, 不用必须 16 进制。
    

###### 第四步: 插入坐标系实例

1.  `AxisHelper`: 用于生成辅助坐标实例, `2`代表这个坐标系的长度, 因为我们不一定需要多长的辅助线。
    
2.  `scene`: 老朋友`场景`, 它的`add`方法就是把某某某加入到场景中来。
    

```
const axisHelper = new THREE.AxisHelper(2)
scene.add(axisHelper)


```

###### 第五步: 渲染出来

1.  第一个参数是`场景`, 第二个参数是`相机`。
    

```
renderer.render(scene, camera);


```

下面是效果图, z 轴正对着我们所以看不到:

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTymosqqvmxH4TetKqlmwBiaDjoWu7dCMs1UcgjC8h1Gy8w9NGGNDpDaoA/640?wx_fmt=png)

在斜上方看到是如下的效果, 之后的章节会说如何调整相机的位置与角度

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTymH6ecqKnl70C3ube22Vic2EAWDcKYfKydbst1ZlLPq9ZsJKb49n6rog/640?wx_fmt=png)

完整的代码如下

```
<html>
<body>
    <script src="https://cdn.bootcdn.net/ajax/libs/three.js/r122/three.min.js"></script>
    <script>
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(35, window.innerWidth / window.innerHeight, 1, 1000);
        camera.position.z = 10;
        const renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setClearColor(0x00FFFF, .5)
        document.body.appendChild(renderer.domElement);
        const axisHelper = new THREE.AxisHelper(2)
        scene.add(axisHelper)
        renderer.render(scene, camera);
    </script>
</body>
</html>


```

### 七. 第一个立方体

不画一个立方体感觉对不起 `第一篇`这个题目, 要注意了在`three.js`中你可以理解为绘制一个几何体需要两部分, 一个是`几何体`本身, 比如这个几何体的长宽高, 另一个就是`材质`可以简单理解为表面的颜色样式。     `geometry`这个单词我们会经常打交道的, 来一起记下它吧。

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTyA9gicFb2kOWycv9jty3YAPvbg5icGo4UT9E5fcUXAWSQDHvjibuEbJ1og/640?wx_fmt=png)

###### `BoxGeometry` 长方体

```
const geometry = new THREE.BoxGeometry(1, 2, 3);


```

1.  `1:` '长', 也可以理解为在不设置坐标的时候在 x 轴上的长度。
    
2.  `2:` '高', 也可以理解为在不设置坐标的时候在 y 轴上的长度。
    
3.  `3:` '宽', 也可以理解为在不设置坐标的时候在 z 轴上的长度。
    

new 出来的实例上面会有这个几何体的点的信息, 面的信息等等, 这个后面再详细说这次主要入门。

###### `MeshBasicMaterial` 材质

颜色与上面设置`setClearColor`一样, 什么写法都行的, 下面是我设置了一个红色的材质。`const material = new THREE.MeshBasicMaterial({ color: 'red' });`

###### 生成'网格' `Mesh`

`const cube = new THREE.Mesh(geometry, material);`网格上含有位置信息、旋转信息、缩放信息等等, 他需要用`几何体`与`材质`两个参数, 但其实并不像网上说的必须要有材质, 不传材质也能显示。

###### 放入场景

也就是场景对象`scene`本身有个`add`方法。`scene.add(cube);`

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTybVhM5NvXfiaFdAkNqQUVA2IjbYXS1VnRxZGDkonrdwwHQhicy3HVnBvQ/640?wx_fmt=png)

右上方视角

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTy9MZdiaicicU6sHAev7qwOaicB0RnS4CYtCPljySxdwxP8DEsYrBWpgwfyA/640?wx_fmt=png)

###### 放入场景的几种方式

1: 我直接放入`geometry``scene.add(geometry);` 会报错了, 可以理解为不是网格对象所以报错了。以后遇到这类报错一定要考虑类型问题。

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTyCLf7nLFfQH9Ymy6yqzEsBYTllGIVUr9dmicg55EFPKOtWJMk0lMEic2w/640?wx_fmt=png)

2: 未设置`材质`

```
const cube = new THREE.Mesh(geometry);

scene.add(cube);


```

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTyGXMZUlgLiaT1gvIq8IkicQ8yXmINgHhwWozJl47gsia4BB5w4Z1reUYLQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/zPh0erYjkib0PJ0npnc8eKwRtF7kFRRTyticVkpw5vBscUscrJ1jKPvcJOo3TJYCZYpEAIk36CxUJMsYGEcOoaVQ/640?wx_fmt=png)

白白的一片, 并且控制台没有报错。

### 八. 全部代码

```
<html>
<body>
    <script src="https://cdn.bootcdn.net/ajax/libs/three.js/r122/three.min.js"></script>
    <script src="./utils/OrbitControls.js"></script>
    <script>
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(35, window.innerWidth / window.innerHeight, 1, 1000);
        camera.position.z = 10;
        const renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setClearColor(0x00FFFF, .5)
        document.body.appendChild(renderer.domElement);
        const axisHelper = new THREE.AxisHelper(2)
        scene.add(axisHelper)

        const geometry = new THREE.BoxGeometry(1, 2, 3);
        const material = new THREE.MeshBasicMaterial({ color: 'red' });
        const cube = new THREE.Mesh(geometry, material);
        scene.add(cube);

        renderer.render(scene, camera);
    </script>
</body>

</html>


```

### end

第一篇写的内容并不多, 等基本知识储备够了就可以开始编写`3d地球`了, 那里将会很有意思。**希望与你一起进步。**

> 转自：lulu_up
> 
> https://segmentfault.com/a/1190000039647481

- EOF -

推荐阅读  点击标题可跳转

1、[three.js 实现露珠滴落动画](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651572586&idx=2&sn=e78859314bd92c8c08f3d09729587677&chksm=80251eabb75297bdbb3c761e61220dd00a03ab0d90846663092456aa56a9ccd9a3f640482af1&scene=21#wechat_redirect)

2、[Threejs 开发 3D 地图实践总结](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651552364&idx=2&sn=abcf94176ee991dc93e348537d14d484&chksm=8025adadb75224bb6c3b352dc8bf613757024a6da167bee5aaa7755ffc6ca154fc364fcd61a5&scene=21#wechat_redirect)

3、[图解 WebGL & Three.js 工作原理](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651553370&idx=1&sn=43b9187fa5b0bd51f2399d573cf66221&chksm=8025a99bb752208dcfb4240c1761c592bb379afb7a8600b8f70654a46ceaacfb4b115927e6a0&scene=21#wechat_redirect)

公众号

点赞和在看就是最大的支持❤️