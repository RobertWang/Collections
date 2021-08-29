> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LUfeTzUoA5IrrbOckONPIA)

如果我们用 Canvas 实现了一些动画效果，需要将它回放出来，很多人通常就是用录屏工具将屏幕内容录下来播放，很少有人知道，Canvas 可以直接通过现代浏览器支持的 Media Streams API 来转成视频。

Canvas 对象支持 captureStream 方法，这个方法会返回一个 MediaStream 对象。然后，我们可以通过这个对象创建一个 MediaRecorder 来录屏。

我们看一个简单的例子。

录制视频
----

首先我们写一个非常简单的 Canvas 动画效果，代码如下：

```
const canvas = document.querySelector('canvas');
const ctx = canvas.getContext('2d');
const {width, height} = canvas;

ctx.fillStyle = 'red';

function draw(rotation = 0) {
  ctx.clearRect(0, 0, 1000, 1000);
  ctx.save();
  ctx.translate(width / 2, height / 2);
  ctx.rotate(rotation);
  ctx.translate(-width / 2, -height / 2);
  ctx.beginPath();
  ctx.rect(200, 200, 200, 200);
  ctx.fill();
  ctx.restore();
}

function update(t) {
  draw(t / 500);
  requestAnimationFrame(update);
}
update(0);
```

这个效果实现一个 200 宽高的矩形在画布中心旋转。

![](https://mmbiz.qpic.cn/mmbiz_gif/lP9iauFI73z9GACPvlGjR8NXiavCq3ic8XnUd3GB8Amf8PyiaBhyrIA2N2SejBNufTldg2tosyYrGsqdibYTHTxNialQ/640?wx_fmt=gif)

接下来，我们将这个效果录制下来，假设我们录制 6 秒的视频，首先我们要获取 MediaStream 对象：

```
const stream = canvas.captureStream();
```

然后，我们创建一个 MediaRecorder：

```
const recorder = new MediaRecorder(stream, { mimeType: 'video/webm' });
```

接着我们可以注册 ondataavailable 事件，将数据记录下来：

```
const data = [];
recorder.ondataavailable = function (event) {
  if (event.data && event.data.size) {
    data.push(event.data);
  }
};
```

在 onstop 事件里，我们通过 Blob 对象，将数据写入到页面上的 video 标签中。

```
recorder.onstop = () => {
  const url = URL.createObjectURL(new Blob(data, { type: 'video/webm' }));
  document.querySelector("#videoContainer").style.display = "block";
  document.querySelector("video").src = url;
};
```

如果你不了解 Blob 对象，可在文章底部查看文档。

最后，我们开始录制视频，并设定在 6 秒钟之后停止录制：

```
recorder.start();

setTimeout(() => {
  recorder.stop();
}, 6000);
```

这样，就可以达到我们想要的录屏效果了。

![](https://mmbiz.qpic.cn/mmbiz_gif/lP9iauFI73z9GACPvlGjR8NXiavCq3ic8XniaYpY21Bibn1Ac9oRvmMlrHDXNffPVCuZicmhiaZvEnibibatWcCqNBYFL3w/640?wx_fmt=gif)

完整的代码在 **https://codepen.io/akira-cn/pen/LYNVewy**。

与音频结合
-----

Canvas 录制好的视频，我们还可以将它和音频结合，方法是通过 ffmpeg 的 Web 端来合成。

浏览器可以通过 WebAssembly 来集成 ffmpeg，具体的项目在 **https://github.com/ffmpegwasm/ffmpeg.wasm**，有兴趣的同学可以研究下。

ffmpeg 的 Web API 用起来还比较复杂，奇舞团的同学开发了一个非常好用的封装，项目地址 **https://github.com/welefen/canvas2video**，可以在这里 **https://codepen.io/akira-cn/pen/XWdbZbd** 看到具体的例子。

总结
--

在一些需要动态回放需求的场景里，我们可以用 Canvas 来实时创建视频然后回放，从而不必使用 JS 重新绘制，这是一种比较节省资源的方式。另外，Canvas 的录制功能对于一些在线教学的场景也很有价值，录屏软件消耗资源较多，而在 Web 上直接录制 Canvas，这可能是一种更简单便捷的方式。

资料推荐
----

*   **Media Streams API：**https://developer.mozilla.org/zh-CN/docs/Web/API/Media_Streams_API
    
*   **Blob 对象：**https://github.com/akira-cn/FE_You_dont_know/issues/12
    

* * *