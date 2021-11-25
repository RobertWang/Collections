> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qwl-aWZNRaSTBINH0dshTw)

基本原理
----

```
<canvas id="canvas" />
```

```
const canvas = document.getElementById('canvas');

function draw(){
  drawPeopleAction(canvas);
  drawBackground(canvas);
  requestAnimationFrame(draw);
}
```

Canvas 画布自身并不支持分层，为了实现分层需要我们引入多个 `<canvas />` 元素。每个 `<canvas />` 元素就是一个分层，这些 `<canvas />` 元素的长宽以及位置都是相同的，最终会完全是重叠在一起的。通过 `z-index` 可以对图层重叠的顺序进行调整。

![](https://mmbiz.qpic.cn/mmbiz_png/RqbfwXSnN35q581nthv1uFLaHHGvbetozrq4icXCHickP0QZIVp8aGchKFx6uakIwCULWAibm5BCKibASK3yesQEHw/640?wx_fmt=png)

可以将上面的代码进行分层优化：

```
<canvas id="backgroundCanvas" />
<canvas id="peopleActionCanvas" />
```

```
const peopleActionCanvas = document.getElementById('peopleActionCanvas');
const backgroundCanvas = document.getElementById('backgroundCanvas');

function draw(){
  drawPeopleAction(peopleActionCanvas);
  if (needDrawBackground) {
    drawBackground(backgroundCanvas);
  }
  requestAnimationFrame(draw);
}
```

相关应用
----

Fabric.js 支持用户在画布上进行选择、拖拽等交互操作，这些交互功能的实现用到了分层技术。Fabric.js 会创建两个 <canvas /> 元素，所有交互操作和事件处理相关的逻辑会放在了一个图层中（`upper-canvas`）；而具体展示的内容会在另一个图层中 （`lower-canvas`）完成渲染。如果画布中的内容不需要与用户交互，对于这种单纯的静态展示的场景，Fabric.js 不做分层，只用一个 <canvas /> 进行绘制，具体细节可以参考`fabric.StaticCanvas` 方法。

```
<div>
  <canvas class="lower-canvas"></canvas>
  <canvas class="upper-canvas"></canvas>
</div>
```

Konva.js 支持开发人员添加和组织图层 (Konva.Layer)，相对来说可以更加方便、灵活地使用图层。值得注意的是，Konva.js 给出了一条建议：要控制图层的数量，最多不超过 3 ～ 5 个。