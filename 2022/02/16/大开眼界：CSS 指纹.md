> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650839435&idx=3&sn=587b97bc9d627817be3297f58d12b61d&chksm=bd010cc58a7685d302676c6f625eec734c107198ed45b31826511086c1971b371cb0f16b7172&mpshare=1&scene=1&srcid=0214MWdZdW5gMjQ5h4vdTqCR&sharer_sharetime=1644837583684&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来源 | 前端印象 （ID：zero2one01）

已获得原公众号的授权转载

大家好，我是零一。之前我发过一篇文章，是关于浏览器指纹的：浏览器隐身模式下的你，仍然没有任何隐私，里面介绍了各种各样的指纹生成方式，今天讨论另一个比较新奇的思路：CSS 指纹

什么是 CSS 指纹？
-----------

CSS 指纹是一种跟踪和收集用户信息的技术，这种方法主要是利用了 CSS 的一些特性来跟踪用户的浏览器和设备的各种特征，这些特征以后可以用来识别或跟踪用户

CSS 指纹如何生成
----------

原理比较简单，主要就是通过无数的媒体查询来给页面返回一套适用的 CSS 样式代码，这套 CSS 代码中会有很多的背景图片，背景图片的地址是一个特定的 URL，这个 URL 上携带了一些我们需要收集的参数，比如：

```
@media screen and (width: 300px) {
    body {
        background: url(https://zero2one/collect/info/width=300);
    }
}
```

这个媒体查询代码只会在用户设备宽度为 `300px` 时生效，所以我们上报的地址也可以带上 `width=300` 的信息，其它信息也类似这种方案去实现，这里就不一一列举了

为了避免信息的重复上报，服务端在接收到该信息上报后，最好将 HTTP 的状态码返回 410（Gone），这样该请求就会缓存下来，之后重复的请求都不会走到服务端，而是走的缓存。最终的效果就类似这样：

![](https://mmbiz.qpic.cn/mmbiz_png/lgHVurTfTcwzZS2Q3q3ldonarbs0tappKM7YdxU3siaziafaciaiawAsZjML6TO42BiarBD5E8mwl5oKhiaR0ib0dWbSg/640?wx_fmt=png)

Status 410(Gone)

同样的，用户本地安装了哪些字体也可以追踪到，不过实现起来有些麻烦，我们可以列举几百甚至几千种字体样式代码，让页面去本地加载对应字体，若本地没有该字体则发起网络请求到我们的服务端，最后对比一下哪几个字体没上报，就说明用户本地有哪些字体了~

举个例子🌰：

```
@font-face{
  font-family: abeezee;
  font-display: block;
  src: local(Abeezee),  /* 加载本地字体 Abeezee */
     url(/collect/info/font-name=Abeezee) /* 若加载失败，则上报信息 font-name=Abeezee */
}
@font-face{
  font-family: abel;
  font-display: block;
  src: local(Abel),   /* 加载本地字体 Abel */
      url(/some/url/font-name=Abel)  /* 若加载失败，则上报信息 font-name=Abel */
}
/* ...此处省略成百上千个类似字体样式代码 */
```

若最后我们服务端所有字体的信息上报都收到了，唯独没有收到 `/collect/info/font-name=Abeezee` 这条请求，说明该用户本地只安装了 `Abeezee` 这个字体

再多举几个例子🌰，判断用户当前是哪个浏览器：

```
@media screen and (-ms-high-contrast: active), (-ms-high-contrast: none){
  body {
    background: url(https://zero2one/collect/info/browser=IE10+)
  }
}

@media all and (min-width:0) {
  body {
    background: url(https://zero2one/collect/info/browser=IE9+)
  }
}

@-moz-document url-prefix() {
  body {
    background: url(https://zero2one/collect/info/browser=firefox)
  }
}

@media screen and (-webkit-min-device-pixel-ratio:0) {
 body {
    background: url(https://zero2one/collect/info/browser=chrome)
  }
}
```

更多的用户信息可以见媒体查询支持的功能

![](https://mmbiz.qpic.cn/mmbiz_png/lgHVurTfTcwzZS2Q3q3ldonarbs0tapp0KuL20Zt6zn5vYCH8TKKvMiaw36WlDLoyVWAq4X85wZiaZWkweuKLguw/640?wx_fmt=png)

媒体功能

为何要有 CSS 指纹
-----------

因为大多数的用户隐私追踪都是依赖于 JavaScript 、Cookies 的，尤其是 Cookies，你在大多数的网站都能见到一个弹窗向你请求 Cookies 的访问权限，例如 stack overflow 网站一进去就会出弹框：

![](https://mmbiz.qpic.cn/mmbiz_png/lgHVurTfTcwzZS2Q3q3ldonarbs0tappicW62KalvOtibndEFOkvrsOLUkSGDNG4Ozm6ddTIGb5UDkt7vOYiaiaJ8g/640?wx_fmt=png)

但无论是 JavaScript 还是 Cookies，都很容易反追踪，比如禁用页面的 JS 脚本、连接 VPN 或者是使用了某些反追踪的浏览器插件等等，而 CSS 指纹就可以不受这些的限制而收集到用户相关数据

CSS 指纹缺点
--------

上述所说的方案有一个最大的缺点就是：CSS 文件会特别大，如果带上所有字体的请求，甚至浏览器并发的请求都会达到数百个，这一定是会影响用户体验的

### 优化请求次数

要知道用户本地有哪些字体的代价比较高，或许可以通过本地有哪些字体来判断用户当前的操作系统，因为市面上大部分的操作系统自带的那些默认本地字体就只有那些，直接去请求这些字体就足够了~

我们现在每收集一条信息都要上报一次，如果不算字体请求上报，估计并发的上报请求也有几十条了，有一种思路就是请求合并，看看能否把所有要收集的参数拼接到一个 URL 上去，但目前为止仅在 CSS 里好像是不行的，因为 `url()` 是不能使用自定义 CSS 变量的，来看个例子：

```
::root {
  --request-url: 'https://zero2one/collect/info/width=300'
}


body {
  background: url(var(--request-url));   /* error */
}
```

这样用是不行的，为什么？大家都知道 `url()` 里的内容既可以加引号，也可以不加：

```
body {
  background: url(https://zero2one/collect/info/width=300);   /* right */
}

.root {
  background: url("https://zero2one/collect/info/width=300");   /* right */
}
```

这是历史遗留问题导致的，所以如果我们想在 `url()` 里使用自定义变量就会报错，CSS 在解析时会把 `url()` 中的所有内容当做 URL，而现在值中有非转义 `(` 会导致一个分析的错误，所以整个声明被作为无效抛出，那么想用自定义变量该怎么使用呢？

```
::root {
  --request-url: url('https://zero2one/collect/info/width=300')
}


body {
  background: var(--request-url);   /* right */
}
```

这样就没有问题了！

好了言归正传，正是因为这样的问题，我们似乎没法对用户数据做一个拼接合并上报，所以 CSS Values and Units Module Level 4（你们也可以叫它 CSS4）提出了这样一个草案

![](https://mmbiz.qpic.cn/mmbiz_png/lgHVurTfTcwzZS2Q3q3ldonarbs0tappIxA7txSicCmE8525zmia7kByz4yvVeArcibclV3iaPa5d0ib3uksNb3gt2w/640?wx_fmt=png)

url 草案

简单来说的话，就是 `url()` 里面可以填入`字符串` + `0或多个修饰符`，去看了一下修饰符的含义，似乎修饰符可以写 CSS 函数（类似`calc()`、`var()`、`attr()` ...），这样不就满足我们的需求了吗？来模拟写一下：

```
::root {
  --screen-width: 'width=300';
  --screen-height: '&height=500';
}

body {
  background: url('https://zero2one/collect/info?' var(--screen-width) var(--screen-height));  
}
```

这样就模拟了我们 JS 中的字符串拼接~ 岂不是美哉，然后再加上其它媒体查询的变量修改，就实现了一次请求上报所有信息的需求

```
::root {
  --screen-width: '';
  --screen-height: '';
}

@media screen and (width: 300px) {
  ::root {
    --screen-width: 'width=300';
  }
}

@media screen and (height: 500px) {
  ::root {
    --screen-height: '&height=500';
  }
}

body {
  background: url('https://zero2one/collect/info?' var(--screen-width) var(--screen-height));  
}
```

希望这个草案能顺利通过~ 这样 CSS 指纹的方案就又更加完善了！

END
---

本文对于 CSS 指纹 的探讨就到这里了，如果有什么问题或者想法，可以在评论区留言，我们互相交流讨论！