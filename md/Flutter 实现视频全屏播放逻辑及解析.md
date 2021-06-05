> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247506638&idx=2&sn=d6bbe3b8e8218b2f5757f1b9fa7bd06b&chksm=e8797fb4df0ef6a23b879dad00cfd44cfeedd58f1b3e231c1b5feca46d1c22793be65603ebf5&mpshare=1&scene=1&srcid=0605VRuzxbAc5xHSE1YBADgW&sharer_sharetime=1622826516552&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

一、前言
----

相信做过移动端视频开发的同学应该了解，想要实现视频从普通播放到全屏播放的逻辑并不是很简单，比如在 GSYVideoPlayer 中的动态全屏切换效果，就使用了创建全新的 `Surface` 来替换实现:

*   创建全新的 `Surface` ，并将对于的 `View` 添加到应用顶层的 `DecorView` 中；
    
*   在全屏时将新创建的 `Surface` 并设置到 Player Core ;
    
*   同步两个 `View` 的播放状态参数和旋转系统界面;
    
*   退出全屏时移除 `DecorView` 中的 `Surface`，切换 List Item 中的 `Surface` 给 Player Core ，同步状态。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j9CXw3c38v4l7pZtGNicClVBThNeyMXyhtxydGna3ycflAwcz59ZXwDB6QvMBrlNDBH6G1ApPhlhPAQZBgsdI4w/640?wx_fmt=png)

> 事实上 Flutter 中实现全屏切换效果很简单，后面会一并介绍为什么在 Flutter 上实现会如此简单。

二、实现效果
------

如下图所示是 Flutter 中实现后的全屏效果，而实现这个效果的关键就是**跳堆栈就可以了！是的，Flutter 中简单地跳页面就能够实现无缝的全屏切换。**

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j9CXw3c38v4l7pZtGNicClVBThNeyMXyhVcDAsGiaibr6OkRlyX2zxLuSTJffDf9sPFI6l4RPZVUrqc7qic9vkTl3A/640?wx_fmt=gif)

如下代码所示，首先在正常播放页面下加入官方 `video_player` 插件的 `VideoPlayer` 控件，并且初始化 `VideoPlayerController` 用于加载需要播放的视频并初始化，另外此处还用了 `Hero` 控件用于实现页面跳转过渡的动画效果。

```
@override
  void initState() {
    super.initState();
    _controller = VideoPlayerController.network(
        'https://res.exexm.com/cw_145225549855002')
      ..initialize().then((_) {
        // Ensure the first frame is shown after the video is initialized, even before the play button has been pressed.
        setState(() {});
      });
  }
 Container(
   height: 200,
   margin: EdgeInsets.only(
       top: MediaQueryData.fromWindow(
               WidgetsBinding.instance.window)
           .padding
           .top),
   color: Colors.black,
   child: _controller.value.initialized
       ? Hero(
           tag: "player",
           child: AspectRatio(
             aspectRatio: _controller.value.aspectRatio,
             child: VideoPlayer(_controller),
           ),
         )
       : Container(
           alignment: Alignment.center,
           child: CircularProgressIndicator(),
         ),
 ))
```

如下代码所示，之后在全屏的页面中同样使用 `Hero` 控件和 `VideoPlayer` 控件实现过渡动画和视频渲染。

这里的 `VideoPlayerController` 可以通过构造方法传递进来，也可以通过 `InheritedWidget`实现共享传递，只要是和前面普通播放界面的 `controller` 是同一个即可。

```
Container(
     color: Colors.black,
     child: Stack(
       children: <Widget>[
         Center(
           child: Hero(
             tag: "player",
             child: AspectRatio(
               aspectRatio: widget.controller.value.aspectRatio,
               child: VideoPlayer(widget.controller),
             ),
           ),
         ),
         Padding(
           padding: EdgeInsets.only(top: 25, right: 20),
           child: IconButton(
             icon: const BackButtonIcon(),
             color: Colors.white,
             onPressed: () {
               Navigator.pop(context);
             },
           ),
         )
       ],
     ),
    )
```

另外在 Flutter 中，只需要通过 `SystemChrome.setPreferredOrientations` 方法就可以快速实现应用的横竖屏切换。

最后如下代码所示，只需要通过 `Navigator` 调用页面跳转就可以实现全屏和非全屏的无缝切换了。

```
Navigator.of(context)
                      .push(MaterialPageRoute(builder: (context) {
                    return VideoFullPage(_controller);
                  }));
```

是不是很简单，只需要 `VideoPlayer` 、`Hero` 和 `Navigator` 就可以快速实现全屏切换播放的效果，**那为什么在 Flutter 上可以这样简单的实现呢？**

三、实现逻辑
------

之所以可以如此简单地实现动态化全屏效果，其实主要涉及到  `video_player` 插件在 Flutter 上的实现：**外接纹理 `Texture`** 。

因为 Flutter 中的控件基本上是平台无关的，而其控件主要是由 Flutter Engine 直接绘制，简单地说就是：**原生平台仅仅提供了一个 `Activity` / `ViewController` 容器, 之后由容器内提供一个 `Surface` 给 Flutter Engine 用户绘制。**

所以 Flutter 中控件的渲染堆栈是独立的，没办法和原生平台直接混合使用，这时候为了能够在 Flutter 中插入原生平台的部分功能，**Flutter 除了提供了 `PlatformView` 这样的实现逻辑之外，还提供了 `Texture`作为 外接纹理的支持。**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j9CXw3c38v4l7pZtGNicClVBThNeyMXyh8cLPGoic2RljiaOSickric1NlwsYqdmkmrwWLhEgeT1pMfGiaH2NV3d2VgA/640?wx_fmt=png)

如上图所示，在《Flutter 完整实战详解》 中介绍过，**Flutter 的界面渲染是需要经历 `Widget`-> `RenderObject` -> `Layer` 的过程**，而在 `Layer` 的渲染过程中，当出现一个 `TextureLayer` 节点时，说明这个节点使用了 Flutter 中的 `Texture` 控件，那么这个控件的内容就会由原生平台提供，而**管理 `Texture` 主要是通过 `textureId` 进行识别的**。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j9CXw3c38v4l7pZtGNicClVBThNeyMXyhEpodeYtDUNBJicicz9gHWEvhZJUfCtWSpx0HPBYbwTq6T4qOXZKL2clw/640?wx_fmt=png)

举个例子，在 Android 原生层中 `video_player` 使用的是 `exoplayer` 播放内核，那么如上图所示，**`VideoPlayerController` 会在初始化的时候通过 `MethodChannel` 和原生端通信，之后准备好播放内核和 `Surface`，最后将对应的 `textureId` 返回到 Dart 中**。

**所以在前面的代码中，需要在全屏和非全屏页面使用同一个 `VideoPlayerController`，这样它们就具备了同一个 `textureId`**。

具备同一个 `textureId` 后，那么只要原生层不停止播放， `textureId` 对应的原生数据就一直处于更新状态，而这时候虽然跳转路由页面，但不同的 `VideoPlayer` 内部的 `Texture` 控件用的是同一个 `VideoPlayerController`，也就是同一个 `textureId` ，所以它们会呈现出通用的画面。

如下图所示，这个过程简单总结就是： **Flutter 和原生平台通过 `PixelBuffer` 为介质进行交互，原生层将数据写入 `PixelBuffer` ，Flutter 通过注册好的 `textureId` 获取到 `PixelBuffer` 之后由 Flutter Engine 绘制**。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j9CXw3c38v4l7pZtGNicClVBThNeyMXyh1a5E1KvTyf8yNYpOsgQk9ibyKZ342Bs4n9OrJnWcfxqCPkE5Uia9jDSw/640?wx_fmt=png)

**最后需要注意的是，在 iOS 上在实现页面旋转时， `SystemChrome.setPreferredOrientations` 方法可能会出现无效**，这个问题在 issue #23913 和 #13238 中有提及，这里可能需要自己多实现一个原生接口进行兼容，当然在 auto_orientation 或者 orientation 等第三方库也进行了这方面的兼容。

**另外 iOS 的页面旋转还确定是否打开了旋转配置的开关**。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j9CXw3c38v4l7pZtGNicClVBThNeyMXyhQ6MVlia8QmRAgRCZg43Cw3nO4YTecOfXvibHXulAt0dYxwKSibhZrkmJg/640?wx_fmt=png)