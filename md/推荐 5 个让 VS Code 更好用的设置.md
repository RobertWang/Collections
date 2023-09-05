> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TtIuTMg8vpoWcjoLtIs9CA)

> 作者：LeonFE  
> https://juejin.cn/post/7269407929592643643

1. 学习并自定义快捷键
------------

在 VS Code 中，快捷键可以说是最重要的提效手段。合理利用快捷键，你的编码效率可以至少提高一倍。首先，你需要学习一些 VS Code 中最常用和最重要的快捷键组合:

这些快捷键可以极大提高你的编码效率，需要花时间熟练掌握。

之后，你可以根据自己的使用习惯，自定义一些快捷键组合。例如：

*   文件目录快捷键，我会自定义为 `Shift + Ctrl/Command + E`
    
*   Git 操作面板我会自定义为 `Shift + Ctrl/Command + G`
    

自定义快捷键可以让你更好地记忆和利用这些提效功能。

2. 安装优化界面主题
-----------

VS Code 提供了丰富的主题扩展，可以让你对编辑器的颜色、字体等界面样式进行自定义。对我来说，一个好的 VS Code 主题，最重要的是不同功能区域之间要有清晰的视觉区分。具体来说，通过为不同区域添加边框、背景等元素，可以让导航栏、侧边栏、编辑器等区域明确分开，不会出现视觉混乱的情况。

这样可以帮助我更快定位到需要的功能区域，减少视觉搜索时间，从而提高工作效率。将功能区分明确划分开来，是我选择一款 VS Code 主题最首要的考虑点。

我比较喜欢的一个主题是 **Moegi Theme**，这款主题拥有色调柔和、功能区块明显的样式。它同时支持日间和夜间两种模式，可以自动切换到系统设置的模式。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/ddV6trCtibeSgJ9TUDLSzMG0q125vC9HZhBhibbB88uda4PMUoxLZOaFN2ibpOC96zPfscZG9z6VoPAaEIdGda5mA/640?wx_fmt=png)

3. 安装提升编码效率的插件
--------------

VS Code 最大的优势之一就是拥有非常强大的扩展插件生态。针对各种语言和工作流程，都有插件可以安装来增强功能和效率。

这里我推荐几款通用的、可以提效的插件:

*   **Smart Clicks**: 通过双击来快速选择或扩展选择范围，可以避免长时间拖动鼠标进行选择
    
*   **Error Lens**: 可以在代码末尾（内联）实时显示错误和警告标记，让你更快定位到代码问题
    
*   **GitLens**: 显示代码行修改记录和历史，辅助 Git 开发
    
*   **Pretty TypeScript Errors**: 帮助开发者更好的阅读 TypeScript 错误
    

根据自己的需求，安装 1-2 款常用的插件，就可以获得显著的提效体验。

4. 个性化设置
--------

例如，我会设置工作台默认采用暗黑模式，并跟随系统切换:

```
{
  "workbench.preferredDarkColorTheme": "Moegi Dark"，
  "workbench.preferredLightColorTheme": "Moegi Light"，
}


```

将侧边栏放在编辑器的右侧，这样可以减少因代码过长而滑动横轴的机会：

```
{
  "workbench.sideBar.location": "right"
}


```

另外，我习惯 Git Diff 对比采用上下样式，所以会进行如下设置:

```
{
  "diffEditor.renderSideBySide": false
}


```

5. 用快捷键提示学习快捷键
--------------