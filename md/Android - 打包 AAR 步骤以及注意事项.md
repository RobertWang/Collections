> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIyNTY4NjU0OQ==&mid=2247507738&idx=2&sn=d1444a5c8f797394041f3ed3fa921bd3&chksm=e8797a60df0ef3769027836cf54e5499c95358e64c7e34f9456cda525c81e260172382340bdf&mpshare=1&scene=1&srcid=0714uTgJcHqlRgVjT0bQpxIc&sharer_sharetime=1626229168076&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

### 简介

![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTB5KpoLnqFbPD0PhI9CXQwrTzhKiasiaKt9h4jzSibxwvxz6EmGs33oeaA/640?wx_fmt=png)

*   *.jar：只包含了 class 文件与清单文件 ，不包含资源文件，如图片等所有 res 中的文件。
    
*   *.aar：包含所有资源 ，class 以及 res 资源文件全部包含
    

### 新工程 (无依赖) 打包 AAR 的步骤

![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTeGz92WL55CxsP3q5XELC2DN5N92KUiad2swLWCiarveE5P2JyyRqVOsQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTKLQcVnTLkic3XYwMoX6aXOgPIOxb9fbpxfDBLJ8p29q6q4XW28wl39w/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTkra9GPuBbEyCHlDZmltRWGGibgeibWSNSjGyPI3SicxWUNMxjMrepBCaQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTIlia81nYVbFFgUaD2MHxZf7HyoyBOFwdD4apn1IhhhZTe1iaMmXibY9FA/640?wx_fmt=png)

### 成型的项目 (有依赖) 如何快速打包 AAR

![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTiaibTJ6hiaiaZWicVjBkjQKiaHvdFABI5n7V2dwDRDklxdhVDO3cjNNYXl2Q/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTkgHMKQOHx6L9ONLNPxzya8iab0ibYKYtMcLTkz1r727cK2sWhlM3Xl7g/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTSZPgLicVpT4ZYQ23UZuXWLZH0PONiaLSKDHX2Sv9rUyQFVOe5Qe1d6lQ/640?wx_fmt=png)

```
<intent-filter>
               <action android: />

               <category android: />
           </intent-filter>

```

![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTOo8hnt6Syibt6SNLjVHiaAUSzqMhrYYQcMe1JTibAxemdcOnXQynp2EMQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/82jN7o40p6m4ZXUv63kw9ZTznp2ZIibQTaKJZCA2mpjtERhXYptwphnhA1EHEQcSAMVhYRmfdZrvgVh8xgMABCA/640?wx_fmt=png)

### 注意事项

**1.** 主项目需要依赖 AAR 中所依赖的远程库，否则会出现 ClassNotFound 异常  
这里也许某个依赖库你们的版本会发生冲突，这就需要你们协调了

**3.** 如果该 aar 包里面有微信支付，分享等第三方库，你要在主工程中使用，要记得在 gradle 里面替换 applicationId，或者用你主工程的包名和 key 去获取第三方操作的 key 和 id    
以分享为例，如果你清单文件中的分享 KEY 与主项目中的 build gradle 文件中的分享 KEY 不同的话，就会包清单文件异常的。

今天就分析到这里，下次继续新内容！