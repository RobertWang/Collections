> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7066277167163768839)

公众号「蝉沐风」，欢迎大家关注交流

这种方法已经 out 了，请各位移步 [VSCode 官方配置同步方案](https://juejin.cn/post/7066622158184644621 "https://juejin.cn/post/7066622158184644621")

1. 关于 Settings Sync 插件
----------------------

`Setings Sync`插件可以同步你的 VSCode 配置到`Github Gist`，当你更换电脑重新搭建 VSCode 环境的时候，直接使用该插件拉取你之前同步的配置即可，不至于让你一切重新开始

> Gist 可以简单理解为是保存代码片段的小仓库

2. 新手教程
-------

### 2.1 新手下载安装

点击扩展按钮，搜索`Setings Sync`并安装，会自动弹出以下界面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e38724f3660044fdaf849459789e525d~tplv-k3u1fbpfcp-watermark.awebp)

点击`LOGIN WITH GITHUB`按钮，输入你的用户名和密码进行授权

如果你从未创建过任何 Gist，顺利的话你会看到如下界面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1aeeb65a8dfb4881934a41354cb35f69~tplv-k3u1fbpfcp-watermark.awebp)

这是在告诉你

> 在你的账号里没找到任何的 Gist，点击 SKIP 这个按钮，会为你自动创建一个 Gist 用来同步 VSCode 配置

如果你曾经创建过 Gist，会列出所有的 Gist，你选择其中一个进行同步配置即可。

如果你是一个新手，从来没有进行过`Settings Sync`插件的配置以及`Token`的折腾，到这一步大概率就可以进行配置上传了。

得益于 VSCode 和 Github 同属于一个东家（微软），系统会为你自动配置好`Gist ID`和`令牌Token`，点击`EDIT CONFIGURATION`，看一下自动给你生成的配置信息

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6974edf4e05e4dddaaec893bd7206985~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef5d89fdcaba46eea71f4974a5469239~tplv-k3u1fbpfcp-watermark.awebp)

如果你的「获取令牌」处的令牌为空，不着急，先试一下上传配置功能是不是能用，能用的话就不用管了

上传配置的快捷键，上传一下试试吧

> Windows：Shift + Alt + U
> 
> MacOS：Shift + Option + U

### 2.2 验证配置成功

如果 VSCode 已经告诉你同步成功了，那就是成功了。你要偏不信，那就登录你的 Github，点击你的头像，再点击`Your gists`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c41a568e23244e39b2ad776a553a73c~tplv-k3u1fbpfcp-watermark.awebp)

配置已经成功上传到 Gist 了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bb8472ec5b84b34b77e3ea51f72165c~tplv-k3u1fbpfcp-watermark.awebp)

点击进入这个 Gist，看一下地址栏后边的一串数字，这个就是`Gist ID`（就是上文提到的自动给你配置的信息），插件只有知道这个信息才知道将配置同步到哪个 Gist

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/edaccfc62cd7483fb310f50a1b88d98d~tplv-k3u1fbpfcp-watermark.awebp)

到此为止，如果你配置成功了，那跳过第 3、4 部分，看看如何拉取配置就可以了。

3. 如果上述过程你不顺利
-------------

如果你有特殊体质，安装的过程中总是出点幺蛾子，那就接着往下看吧

这个插件需要的就是两个信息而已

*   `Gist ID`
*   `Token`

如果系统没有自动给你生成，那就自己动手

### 3.1 创建 Gist

在 Gist 列表页点击右上方 + 按钮，创建一个 Gist

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/967cf3a9cbb2404181f10b60335994da~tplv-k3u1fbpfcp-watermark.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfd6758fbd624df7bf598f9a73a0b895~tplv-k3u1fbpfcp-watermark.awebp)

创建成功之后找到你的`Gist ID`

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bce73cdfafa406d8d882c0feccda974~tplv-k3u1fbpfcp-watermark.awebp?)

Gist 到此为止，接下来获取`Token`

### 3.2 创建 Token

到 Github 上点击你的头像，点击`Settings`，然后左侧栏找到`Developer settings`，然后继续点击`Personal access tokens`

点击`Generate new token`按钮，写上你的 token 说明（Note），选择过期时间 Expiration（我一般选永久，因为嫌麻烦），然后**勾选`gist`选项**（这一步很重要，不要漏！！！）。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36ea0613e5304a52b354c5809c383c50~tplv-k3u1fbpfcp-watermark.awebp)

点击`Generate token`按钮生成`Token`，生成之后千万记得保存一下，因为你只会看到这一次！

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19faf8cfd67d4ecfb7686057459d0cfe~tplv-k3u1fbpfcp-watermark.awebp)

### 3.3 配置插件

把这两个信息填写到插件的配置文件中就可以了。

大功告成！

4. 本人还有更加不顺利的体验
---------------

我在原来的电脑上明明是第一次安装这个插件，我以为一切都会给我自动配置，然而当我同步配置的时候给我弹出这个错误提示

> Sync: GitHub 令牌无效或已过期。请重新生成。

我都忘记什么时候搞过 Token 了，所以我完整走了一遍第 3 步的流程，得到了`Gist ID`和`Token`。

然而！！！我的 VSCode 压根不会出现以下这个界面了，不出现我就没法配置插件啊。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23566584a1d74fe693c67707867440dc~tplv-k3u1fbpfcp-watermark.awebp)

我想过重装，然后抽了自己一个耳光，本来就是要同步这台电脑上的 VSCode 配置，却要我重装？？？

下面是解决步骤

### 4.1 找到 Settings Sync 插件的配置文件所在位置

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b6164030726430ba3f969b255d8b911~tplv-k3u1fbpfcp-watermark.awebp)

1.  点击插件按钮，找到`Settings Sync`这个插件，点击右下方的齿轮
    
2.  点击`Extension Settings`按钮，右侧出现所有配置项
    
3.  选择`Extensions——Code Settings Sync`，出现该插件的所有可视化配置 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7970c09521924f3a8c7f7b80d4e423df~tplv-k3u1fbpfcp-watermark.awebp)
    
4.  点击右上角的`Open Settings（JSON）`按钮，看图中我圈出来的部分，就是 VSCode 的配置文件所在目录了 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f3acb6c4a614532998febdc0562838d~tplv-k3u1fbpfcp-watermark.awebp)
    

### 4.2 修改 syncLocalSettings.json

进入`Code/User`目录，和`settings.json`文件同级的有个`syncLocalSettings.json`文件，修改其中的`token` ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/15057c4a443845b9a011d3d131a89f86~tplv-k3u1fbpfcp-watermark.awebp) 然后就可以开心地同步配置了呀！

5. 拉取配置
-------

在其他电脑上拉取同步配置的快捷键如下

> Windows: Shift + Alt + D
> 
> MacOS: Shift + Option + D

完！