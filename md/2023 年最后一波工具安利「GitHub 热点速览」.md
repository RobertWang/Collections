> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/euvGlda8oSK_Z5j5Y6m3mg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/xBgIbW1vdNMkfC4hr6ctBZLaOSyOX4Z2iagmS0CwkOibv1uHgMDsYicGQbuTcjjQC4rWyibA4cjvmVwH6IRiccY0trw/640?wx_fmt=jpeg&from=appmsg)

作者：HelloGitHub - 小鱼干

2023 年还有两周就要接近尾声了，2023 年的热点速览还有一波工具好安利：比如上周推荐之后上了热榜的远程调试工具 page-spy-web，让调试像呼吸一般自然方便；还有轻量级的搜索引擎 orama，可以让你下载 B 站视频好好过个元旦的下载姬 downkyi，JS 格式化工具 biome，Meta 开源的 JS 库 stylex 也是颇受欢迎。

以下内容摘录自微博 @HelloGitHub 的 GitHub Trending 及 Hacker News 热帖（简称 HN 热帖），选项标准：`新发布` | `实用` | `有趣`，根据项目 release 时间分类，发布时间不超过 14 day 的项目会标注 `New`，无该标志则说明项目 release 超过半月。由于本文篇幅有限，还有部分项目未能在本文展示，望周知 🌝

![](https://mmbiz.qpic.cn/mmbiz_png/xBgIbW1vdNMkfC4hr6ctBZLaOSyOX4Z278Y39VBSGfG9Zsjvic5qqgw1Vic93nCgWQc93x6agxiayPVhicU3ib47FWA/640?wx_fmt=png&from=appmsg)

1. 本周特推
-------

### 1.1 远程调试：page-spy-web

**主语言：TypeScript**

PageSpy 是货拉拉开源的、来调试远程 Web 项目的工具。基于对原生 API 的封装，它将调用原生方法时的参数进行过滤、转化，整理成格式规范的消息供调试端消费；调试端收到消息数据，提供类控制台可交互式的功能界面将数据呈现出来。

> GitHub 地址→https://github.com/HuolalaTech/page-spy-web

![](https://mmbiz.qpic.cn/mmbiz_png/xBgIbW1vdNMkfC4hr6ctBZLaOSyOX4Z2TM9gOYbicScoK2Wwiaiar4SYStX74Wxv8pbXd0RrR9YP82aTYtgzEovXg/640?wx_fmt=png&from=appmsg)

### 1.2 搜索引擎：orama

**主语言：TypeScript**

Orama 是一个快速的、包含所有功能的全文和向量搜索引擎，完全用 TypeScript 编写，且没有任何依赖。Orama 具有快速、存内、容错别字、轻巧（小于 2kb）等特性，它可在浏览器、服务器和边缘计算环境中运行全文、向量和混合搜索查询。

> GitHub 地址→https://github.com/oramasearch/orama

![](https://mmbiz.qpic.cn/mmbiz_png/xBgIbW1vdNMkfC4hr6ctBZLaOSyOX4Z24eAMiagpKPHib8YxfLdUa7r3XfICg0ic52ic0apFTObP9qzkfIbA7HjFwg/640?wx_fmt=png&from=appmsg)

2. GitHub Trending 周榜
---------------------

### 2.1 语言学习：project-based-learning

**本周 star 增长数：1,300+**

实践出真知，在实践中了解、掌握编程技巧一直是大家默认的学习语言法。project-based-learning 收录了一系列编程项目，当中涉及了多种技术，在你做项目的过程中便掌握各种技能。

> GitHub 地址→https://github.com/practical-tutorials/project-based-learning

### 2.2 自定义用户界面：stylex

**本周 star 增长数：1,850+**，**主语言：JavaScript**

`New` StyleX 是 Meta 开源、用于为优化的用户界面定义样式的 JavaScript 库。它结合了内联样式和静态 CSS 的优点，作为一个 CSS-in-JS，所有样式在编译时会捆绑在静态 CSS 文件。

示例：

```
import stylex from '@stylexjs/stylex';

const styles = stylex.create({
  root: {
    padding: 10,
  },
  element: {
    backgroundColor: 'red',
  },
});

const styleProps = stylex.apply(styles.root, styles.element);


```

> GitHub 地址→https://github.com/facebook/stylex

![](https://mmbiz.qpic.cn/mmbiz_png/xBgIbW1vdNMkfC4hr6ctBZLaOSyOX4Z2NkZ2xG8VhOnqxUiarQT0mibdWSibcRtOouXG84n13HgB67ARt7NzIh0ow/640?wx_fmt=png&from=appmsg)

### 2.3 Web 开发套件：biome

**本周 star 增长数：850+**，**主语言：Rust**

Biome 是一个速度极快的格式化工具，适用于 JavaScript、TypeScript、JSX 和 JSON，与 Prettier 的兼容性达到 96%。此外，它还是个代码检查器，适用于 JavaScript、TypeScript 和 JSX，它包含了来自 ESLint、TypeScript ESLint 以及其他来源的超过 170 条规则。它会提供详细且具有上下文的诊断信息，帮你改进 Web 代码！

> GitHub 地址→https://github.com/biomejs/biome

### 2.4 3D 模型生成：threestudio

**本周 star 增长数：650+**，**主语言：Python**

threestudio 是一个统一的框架，用于从文本提示、单个图像和少量图像创建 3D 内容，它通过提升 2D 文本到图像生成模型来实现 3D 成像。

> GitHub 地址→https://github.com/threestudio-project/threestudio

![](https://mmbiz.qpic.cn/mmbiz_gif/xBgIbW1vdNMkfC4hr6ctBZLaOSyOX4Z2aJWFMib4Ie8xZCZvZTiceZySYlEQFMnwCOMcIGn5nhpsazlq8acBO4mQ/640?wx_fmt=gif&from=appmsg)

### 2.5 哔哩下载姬：downkyi

**本周 star 增长数：350+**，**主语言：C#**

哔哩下载姬 downkyi，哔哩哔哩网站视频下载工具，支持批量下载，支持 8K、HDR、杜比视界，提供工具箱（音视频提取、去水印等）。

> GitHub 地址→https://github.com/leiurayer/downkyi

![](https://mmbiz.qpic.cn/mmbiz_png/xBgIbW1vdNMkfC4hr6ctBZLaOSyOX4Z2BhvXlYzCRIBUDFmicPRHtllTygtUXGCjexNWQ0Z4CS69bcEvf3icSZfg/640?wx_fmt=png&from=appmsg)

3. HelloGitHub 热项
-----------------

在这个章节，我们将会分享下本周 HelloGitHub 网站上的热门项目，HG 开源项目评价体系刚上线不久，期待你的评价。

### 3.1 沙盒环境：devbox

**主语言：Go**

该项目可以创建一个可移植、隔离、用于开发的独立 shell，无需 Docker 和虚拟机。比如你的项目使用 Python 和 Go 语言，用这个工具仅需一条命令就能初始化一个独立的开发环境。

```
# 安装
curl -fsSL https://get.jetpack.io/devbox | bash
# 初始化
devbox init
# 安装 Python 和 Go
devbox add python2 go_1_18
# 激活
devbox shell


```

> HG 评价地址→https://hellogithub.com/repository/0d6c899756c94a10bd31b030cfb940f8

### 3.2 数字手写笔：dpoint

**主语言：Python**

该项目通过摄像头跟踪和惯性测量，实现了 6DoF 输入。触控笔可用于任何平面，仅需消费级的摄像头配合使用。

> HG 评价地址→https://hellogithub.com/repository/69a05c4a9e0640f993d71858c27efc8d

![](https://mmbiz.qpic.cn/mmbiz_png/xBgIbW1vdNMkfC4hr6ctBZLaOSyOX4Z2gaESGktluia86ibr6Yt9ExicD3CdUib01DLkGpGvOpFve80ShgICIayvzw/640?wx_fmt=png&from=appmsg)

4. 往期回顾
-------

往期回顾：

*   [又有新框架上线了，测试、AI 通通有「GitHub 热点速览」](https://mp.weixin.qq.com/s?__biz=MzA5MzYyNzQ0MQ==&mid=2247517204&idx=1&sn=794322a8a06f745529c0031aa82e37ca&scene=21#wechat_redirect)
    
*   [叮咚，你的微信年度聊天报告请查收「GitHub 热点速览」](https://mp.weixin.qq.com/s?__biz=MzA5MzYyNzQ0MQ==&mid=2247517175&idx=1&sn=24ddf7fcd27fa09a3fe09add71258ce1&scene=21#wechat_redirect)
    

以上为 2023 年第 51 个工作周的 GitHub Trending 🎉如果你 Pick 其他好玩、实用的 GitHub 项目，来 HelloGitHub 和大家一起分享下哟 🌝

- END -