> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7009119994415939614)

如果应用是使用`React`开发的，并使用了`BrowserRouter`，那么就会出现这个问题（反正我是遇到了），刷新页面后出现`404`，`Not Found`。

这里我找到了两种解决方案：

（1）使用`HashRouter`

不使用`BrowserRouter`，改为使用`HashRouter`。

```
<HashRouter>
	<App />
</HashRouter>
复制代码
```

（2）云开发添加「路由配置」

继续使用`BrowserRouter`，但腾讯云 CloudBase 需要做一些设置。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6235b0bcebe942a9ac2c265e9f61aa3e~tplv-k3u1fbpfcp-watermark.awebp)

点击「静态网站托管」中的「基础配置」，在「路由配置」中添加一条重定向规则如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf1ec7a6fc704a76a9bf9826b2159783~tplv-k3u1fbpfcp-watermark.awebp) 错误码`404`时重定向到 `/index.html`即可解决问题。

2. 部署后应用页面还是原来的版本
-----------------

点击「静态网站托管」中的「基础配置」，在「节点缓存配置」中设置缓存时间为`0`：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92218d747d6146929ad54a5b755e073d~tplv-k3u1fbpfcp-watermark.awebp)

在「浏览器缓存配置」中设置缓存时间为`不缓存`（时间为`0`）：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1cc69631beda4248abd79b22c2c4b343~tplv-k3u1fbpfcp-watermark.awebp)

3. React 应用，如果有警告会部署失败
----------------------

在本地预览 React 应用时，就算有警告也是可以运行的，但是自动部署的时候就会报错，从而导致部署失败。

建议在本地预览时，如果出现警告，就修改代码将警告去除。

还有一个方案，在 React 应用的根目录下，打开`package.json`文件：

找到`scripts`下的`build`，在其值之前加上`CI=false&&`。

比如，原来的是：

```
"build": "react-app-rewired build"
复制代码
```

将其改为：

```
"build": "CI=false&&react-app-rewired build"
复制代码
```