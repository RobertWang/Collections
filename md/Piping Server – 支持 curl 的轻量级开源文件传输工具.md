> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.appinn.com](https://www.appinn.com/piping-server/)

**Piping** 是一个轻量级的开源文件传输工具，可自托管，支持使用 curl、wget 下载，可更广泛的在无浏览器的设备上使用。传输方式基于 HTTP/HTTPS，使用 Stream 流式传输，可传输任何数据，比如屏幕共享、远程桌面、共享绘画、文字聊天等内容，也无大小限制。开发者曾测试不间断用 64 天传输了 1PB 文件。@[Appinn](https://www.appinn.com/piping-server/)

![](https://img3.appinn.net/images/202111/piping-server.jpg!o)

curl 是广泛存在于现代设备的开源工具，小到路由器、光猫，大到火星无人直升机机智号 (Ingenuity)，都使用了 curl。于是 Piping 成为了适应更广，比需要浏览器传输文件，还要再进一步的工具。

Piping Server 搭建
----------------

使用 Docker 都很容易：

```
docker run -p 8080:8080 nwtgck/piping-server
```

使用 Rust 重写的版本：

```
docker run -p 8181:8080 nwtgck/piping-server-rust
```

然后，就能打开浏览器使用了：

![](https://img3.appinn.net/images/202111/screen-appinn2021-11-10_15_45_57.jpg!o)

在 Step 2 中的 **Secret Path** 就是将来下载文件时的路径，比如输入了 appinn，那么下载地址就是 ip: 端口 / appinn

注意 Piping 服务器并不保存任何数据，它需要你开着浏览器，实时传输。

公共服务器
-----

既然不保存数据，所以也节省资源，会有不少的公共服务器，可以直接拿来用：

*   [https://ppng.io](https://ppng.io/?utm_source=appinn.com)
*   [https://piping.glitch.me](https://piping.glitch.me/?utm_source=appinn.com)

并且还有一个漂亮一些的 UI 界面，即开即用：

*   [https://piping-ui.org/](https://piping-ui.org/?utm_source=appinn.com)

![](https://img3.appinn.net/images/202111/screen-appinn2021-11-10_15_55_28.jpg!o)

Piping Server 更多是面向开发者的工具，多数情况下使用命令行操作，不过好在网页版本让门槛降低了不少，当你碰到一台连浏览器都没有的设备时，说不定 Piping 就派上用场了。

*   [Github 地址](https://github.com/nwtgck/piping-server)

先别急，最后，开发者自己做过一个实验：

> 在我的实验中，Piping Server 在单个 HTTP 请求中传输了 1,110TB（≈1PB）至少 64 天 2 小时。 这意味着它可以传输大量数据并将请求保留大约 2 个月。

先不要惊讶这个大小和传输时长，没错这两项都挺厉害的，但青小蛙算了一下，1PB 数据用时 64 天，按 1PB=1024TB 来算，那么一天就传输了 16TB 数据，一个小时 600GB，~这是不是开发者的网速有点慢？ 🐶~ （搞错了单位，过了两天也没看出来，数学翻车了🙈）

原始链接：https://www.appinn.com/piping-server/

感谢 @**路过**、@zxc 同学在 [croc](https://www.appinn.com/croc/) 一文中的推荐。