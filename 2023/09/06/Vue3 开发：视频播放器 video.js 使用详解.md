> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xlsmbODT9xOlgBogv41rHg)

> Video.js 是一个通用的在网页上嵌入视频播放器的 JS 库，比原生 video 标签有更强大的功能、更好的兼容性、更美观等优点。是一个比较流行的视频播放器

**前言**

Video.js 是一个通用的在网页上嵌入视频播放器的 JS 库，比原生 video 标签有更强大的功能、更好的兼容性、更美观等优点。是一个比较流行的视频播放器，它的官网是 https://videojs.com/  

本篇文章就来看看在 vue3 项目中如何使用 video.js。

**安装使用**

首先安装 video.js：

```
pnpm install video.js --save

```

然后引入 css，在 mian.js 中：

```
import "video.js/dist/video-js.css";

```

注意：不要遗忘这一步，否则浏览器的样式会有问题。

在页面中加入 video 标签：

```
<video :id="playerId"></video>

```

class 必须是 video-js，然后需要设置一个 id。

最后初始化播放器：

```
import videojs from "video.js";
const player = videojs(playerId, {autoplay: true});
player.src(url);
player.on("ended", () => {//播放完成})

```

即可播放。

上面只是最简单的 demo，下面来说说 video.js 中比较常用的功能。

**配置**

在创建 videojs 的时候，第一个参数是对应的是播放器元素，可以是 id 也可以是 DOM Element；第二个参数是 options，即播放器的相关配置。  

这里我们说几个比较常用的，所有配置大家可以参考官网 https://videojs.com/guides/options/

**aotuplay**

自动播放，它有五个选项，可以是 boolean 也可以是字符串：

*   false：不自动播放
    
*   true：自动播放，但是如果浏览器不允许自动播放的话就会失效
    
*   “muted”：静音后自动播放。因为浏览器实际上是不允许自动播放声音，所以静音后自动播放基本不会失效，但是没有声音需要自己处理一下。
    
*   “play”：自动播放，与 true 效果一样。
    
*   “any”：自动播放，如果浏览器阻止的话会先静音再自动播放。
    

这里大家先了解一下，后面我会详细说一下自动播放的问题。

**src**

视频源

**width/height**

视频宽高，number 类型

**mute**

是否静音

**loop**

是否循环播放

**playsinline**

是否内联播放。用于移动端（尤其 iOS），在部分移动端浏览器上如果通过 video 标签进行视频播放，那么浏览器会进行劫持并通过一个最上层的播放组件来进行全屏播放。设置 playsinline 后会禁止这一行为，在原 video 标签内进行视频播放。

不过由于 Android 系统的碎片化，在部分厂商自带的浏览器上会没有效果。这个具体看我另外一篇专门讲解内联播放的文章。

**controls**

是否显示控制组件（包括控制栏和大播放按钮等）。如果是 true 即显示，同时支持一些用户操作，比如单击视频暂停 / 播放，双击全屏等。如果是 false 则不显示，同时也禁止了用户操作，这样我们需要自己实现控制。

**controlBar**

设置控制栏的内容，是一个 Object（ControlBarOptions）或者是布尔值。

如果是 true 则显示默认控制栏，否则不显示。

如果是 Object 则可以对控制栏中的按钮进行设置，这里就说说默认显示的几个属性：

*   playToggle：是否显示播放按钮
    
*   progressControl：是否显示进度条。除了 boolean，还可以设置一个 ProgressControlOptions 对象，更详细的配置进度条。
    
*   volumePanel：是否显示音量。除了 boolean，还可以设置一个 VolumePanelOptions 对象，更详细的配置音量组件。
    
*   pictureInPictureToggle：是否显示画中画按钮
    
*   remainingTimeDisplay：是否显示时长
    
*   fullscreenToggle：是否显示全屏按钮
    

controlBar 的前提是 controls 为 true，否则如何设置都不会显示。

**bigPlayButton**

在视频上显示大播放按钮。这样同样需要 controls 为 true，否则设置为 true 也不会显示。

**userActions**

用户操作，也是一个 Object（UserActions），有三个属性：

*   click：是否允许单击
    
*   doubleClick：是否允许双击
    
*   hotkeys：是否允许快捷键，也是一个 Object，包括全屏、静音和播放 / 暂停。
    

上面提到如果 controls 为 true 则同时支持用户操作，如果想显示控制栏又不允许这些用户操作，则可以设置 userActions 禁止这些操作即可，这样用户就只能通过点击控制栏上的按钮来控制。

**播放器操作**

上面通过 videojs 创建了一个 Player 对像，我们就可以通过这个对象的各种函数来操作播放器。  

同样这里只说说常用的函数，其他的大家参考官网 https://docs.videojs.com/player

*   `src(string | SourceObject | SourceObject[])`：设置视频源
    
*   `src():string`：获取当前视频源
    
*   `play()`：播放
    
*   `pause()`：暂停
    
*   `paused():boolean`：是否暂停
    
*   `currentTime(number)`：设置播放位置，就是 seek
    
*   `currentTime():number`：获取当前播放位置
    
*   `muted():boolean`：是否静音
    
*   `muted(boolean)`：设置静音
    
*   `duration():number`：获取时长
    
*   `controls(boolean)`：设置控制栏显示隐藏
    
*   `controls():boolean`：控制栏是否显示
    
*   `requestFullscreen()`：全屏播放
    
*   `exitFullscreen()`：退出全屏播放
    
*   `isFullscreen():boolean`：是否全屏播放
    
*   `dispose()`：销毁播放器
    
*   `error(MediaError)`：设置一个错误
    
*   `error():MediaError`：获取当前错误。配合 error 事件
    

**播放器事件**

通过`Player.on(string, EventListener)`函数可以设置播放器的监听事件，第一个参数是事件名称，第二个参数是回调。  

同样这里说说常用的事件，所有事件大家可以查阅官网 https://docs.videojs.com/player

*   canplay：视频可以播放
    
*   playing：播放
    
*   pause：暂停
    
*   timeupdate：播放进度更新
    
*   ended：播放完成
    
*   fullscreenchange：全屏状态改变
    
*   error：视频播放错误。可以配合 player 的 error 函数来获取处理错误。代码如下：
    

```
    player.on("error", () => {
      const error = player.error();
      console.log("video error:" + error.code + "-" + error.message);
    });

```

**自动播放**

因为这个问题比较重要，所以我单独详细说一说  

首先简单说一下浏览器的自动播放机制：

> 为了防止部分网站已打开就播放各种声音，尤其是广告影响用户体验，chrome 在 66 版本关闭了音频自动播放，其他浏览器也有各自类似的机制。
> 
> 不过 chrome 并不是完全禁止自动播放音频，而且要求在有用户交互行为前不允许自动播放音频，所以刚打开页面的时候（或刷新后）是不能自动播放音频的，但是如果用户有了交互，那么后续的音频都可以自动播放了。
> 
> 视频实际上是受音频影响，所以静音的话是可以自动播放的。目前一般有两种方式：一种就是视频不自动播放，由用户点击播放；一种就是静音自动播放，由用户自己打开声音。

但是我们可能有多条视频逐个播放，所以不能每个视频都静音或手动播放，那么你们就会说可以在第一条视频后设置自动播放，但是如果有其他页面来到播放页面，其实也可以自动播放，因为用户一定已经有过交互。

所以为了让用户有更流畅的体验，我们将 autoplay 设置为 "any"，这样一定会自动播放，但是有时候（比如刷新后）会没有声音。

我们可以在 playing 事件中判断一下当前是否静音，如果静音则提示用户打开声音即可，代码如下：

```
    VideoPlayer.player.on("playing", () => {
      if (VideoPlayer.player.muted()) {
        console.log("已静音啦");
        VideoPlayer.muteDialog?.destroy();
        VideoPlayer.muteDialog = modal.showInfo({
          titleTxt: "开启声音",
          contentTxt: "浏览器已自动静音，请手动开启声音",
          okTxt: "开启",
          onConfirm: () => {
            VideoPlayer.player.muted(false);
            VideoPlayer.muteDialog = null;
          }
        });
      }
    });


```

这样在大部分情况下用户都可以流畅的观看视频。

**微信**

在微信的浏览器中无法进行自动播放，如果使用上面的代码会发现视频没有自动播放，也没有任何弹窗。这就需要我们去手动播放，可以在 videojs 配置的时候将 bigPlayButton 打开（注意 controls 也要设置为 true），这样默认会显示一个大播放按钮，用户点击即可以播放。  

注意：controls 设置为 true 后控制栏默认也会显示，这样当点击大播放按钮播放后，控制栏会显示出来，如果不想显示控制栏则将 controlBar 设置为 false 即可。

**全屏播放**

再来说说全屏播放，一般情况下我们会隐藏默认的控制栏来自己实现一个，然后盖在 video 标签区域的底部，但是这样有一个问题：如果我们自己实现的功能有全屏播放，全屏播放的时候自己的控制栏就看不见了，这样就没法退出全屏了。  

当然 videojs 提供了 Components 来使我们可以自定义控制栏，甚至可以在播放器上直接添加 Button 等，具体可见 https://videojs.com/guides/components/ 。但是如果想完全实现一个不同样式的控制栏则会非常复杂，需要大量的代码。

所以我的解决方案是在非全屏的状态下显示自己的控制栏，在全屏的时候则显示默认的控制栏，这样全屏的时候也可以退出全屏了，这样就需要我们监听全屏状态，如下：

```
  VideoPlayer.player.on("fullscreenchange", () => {
    VideoPlayer.player.controls(VideoPlayer.player.isFullscreen());
  });

```

然后在我们自己的控制栏上加一个全屏按钮，点执行`VideoPlayer.player.requestFullscreen();`全屏即可。