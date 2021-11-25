> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7014504173022478344)

项目打包后，代码都是经过压缩加密的，如果运行时报错，输出的错误信息无法准确得知是哪里的代码报错。  有了 map 就可以像未加密的代码一样，准确的输出是哪一行哪一列有错。 所以该文件如果项目不需要是可以去除掉。

去除. map 文件
----------

通过设置 `vue.config.js`文件 中的 `productionSourceMap`属性为`false`，`dist`包将小很多。

vue.config.js

```
module.exports = {
    productionSourceMap:false
}
复制代码
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d82a6f43098544f89282f68fecab8288~tplv-k3u1fbpfcp-watermark.awebp?)

至此完成了 `15Mb`到 `5Mb`的蜕变。

优化静态资源图片
--------

在打包完成后我发现静态图片被打包到了一个文件夹中，且这个文件夹占用的大小也将近`1Mb`。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76a749d6740445b092f7c4377da12d4c~tplv-k3u1fbpfcp-watermark.awebp?)

查看内容也都是一些普通的图片。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/424d6baded1a4480b7fb6d12e29eab09~tplv-k3u1fbpfcp-watermark.awebp?)

在项目初期就这么大的资源，项目变大了还得了。且这么做在后期维护上存在诸多问题。

### 静态资源图片的劣势

1. 在后期项目功能新增时候。`同事A`和`同事B`都需要展示同一张`示例图`。若他们的功能不属于一个模块下，那么他们很有可能在各自的文件夹下上传`相同的图片`。那么打包就会产生 2 张相同的图片，占用 2 份`内存资源`。

2. 若后期功能`修改`或`删除`，我们是否会想到将它老的图片删除（大多数都应该没考虑到吧）。当然我们也可以通过定期的检查是否有没用到的图片将它删除，但这往往需要很多的`人力物力`。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb9728e9dec9404fb26b7ac7c61ec506~tplv-k3u1fbpfcp-watermark.awebp?)

### OSS 替换图片静态资源方案

我们可以通过将`静态资源`上传`OSS`的方式实现将`静态资源`从项目打包中分离。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/509239c5913e447196ee928435d0f21b~tplv-k3u1fbpfcp-watermark.awebp?)

就是将静态资源压缩后放到`OSS`上去管理，这样我们就不用管它在文件的哪个位置了 (因为直接访问的外链地址)。

我分析了下这样的做法有以下几点优势：

1.  去除静态资源在打包中的占用，极大的减少打包出来的体积。(可增加打包速度和减少包上传到服务器上的时间)。
    
2.  资源是`CDN加速`的，在一般情况下会比直接放在服务器上快一点。(`CDN`会根据访问者地址选取最近的服务器进行资源传输)
    
3.  方便维护，若某个功能去除了我们不必担心还有静态文件残留。因为引用已经被删除了，项目不会去请求不用到的静态资源。
    
4.  统一化管理，项目的参与者很多，文件放的位置也不统一 (目前有`static`下、`assets`下和`就近的文件夹`)。统一化管理后我们很轻松地就能在`OSS`上分找到`我们的项目`、`模块`和`其他资源`。
    

### 配置 OSS 资源到开发环境

第一步：我们需要上传我们的资源到`OSS对象存储`。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/747cec59f0104f07a6f8def51787d764~tplv-k3u1fbpfcp-watermark.awebp?)

复制所得到的图片`URL路径`。 关于如何上传`OSS`大家可以参考这篇文章：[快速开始 OSS](https://link.juejin.cn?target=https%3A%2F%2Fhelp.aliyun.com%2Fdocument_detail%2F31883.html "https://help.aliyun.com/document_detail/31883.html")

第二步: 配置全局`OSS` 的 `BaseUrl`。这一步主要是为了支持后续拓展，若我们后期需要迁移或修改我们`OSS`的库那么只要修改`OSS` 的`BaseUrl`就可以了。

配置关于`img标签`的地址替换：

main.js

```
import Vue from 'vue';
// 全局变量
Vue.prototype.$OSS = 'https://你的bucket名称.oss-cn-hangzhou.aliyuncs.com/你的bucket下的文件夹'

复制代码
```

使用

index.vue

```
<div class="login-logo">
     <img :src="$OSS + '/images/login/logo.png'" alt="logo" />
 </div>
复制代码
```

配置关于`样式`中的图片引用地址: base.less

```
/* OSS地址（Less变量） */
@oss: 'https://你的bucket名称.oss-cn-hangzhou.aliyuncs.com/你的bucket下的文件夹';
复制代码
```

使用

index.vue

```
<template>
  <!-- 页面-->
  <div id="index-box">
  </div>
</template>

<style lang="less" scoped>
/*引入OSS的BaseUrl */
@import '@/assets/styles/base.less';
#index-box {
  background: url('@{oss}/images/admin/login-box.png');
 }
</style>
复制代码
```

至此我们本地就可以正常访问到上传的图片了

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77c5874f1ee04815b5ef7d3820d015a5~tplv-k3u1fbpfcp-watermark.awebp?)

运行打包命令后发现比之前快了好多，打出的包也只有`4Mb`不到了。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/118249a476cd4ca6a81527b591fd17cb~tplv-k3u1fbpfcp-watermark.awebp?)

放到`nginx`下运行发现也是正常的：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/088938a9788a4eb084d052f414472b12~tplv-k3u1fbpfcp-watermark.awebp?)

至此我们对`静态资源`的优化就结束了。

拓展
--

当然本篇文章只是讲述了关于优化打包大小的相关操作，在`网站性能`还需大家进一步优化。

以下推荐一篇文章，在项目优化上讲的非常好。

[网站性能优化实战——从 12.67s 到 1.06s 的故事](https://juejin.cn/post/6844903655330562062 "https://juejin.cn/post/6844903655330562062")

另外推荐给大家一个图片压缩网站，可以优化我们的图片大小且是免费的。

[tinypng.com/](https://link.juejin.cn?target=https%3A%2F%2Ftinypng.com%2F "https://tinypng.com/")