> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650806752&idx=5&sn=3e9bd5943ad9ef475a476742d36d814e&chksm=bd018cae8a7605b8490ce1289c6be5e958860bfcd22c51a1fea538fe78f089e0ed1dbb1b3ba5&mpshare=1&scene=1&srcid=0619dw63xhJTO6ecPat8Ctjb&sharer_sharetime=1624113353722&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> **开源最前线（ID：OpenSourceTop） 猿妹编译**
> 
> 链接：https://github.com/helix-editor/helix

但是市面上的主流编辑器就那些，已经很久没看到新面孔了，最近，GitHub 上出现了一个很火的项目 —— 后现代文本编辑器 Helix。这个编辑器被称为是后现代编辑器。

helix 受 kakoune/neovim 启发的编辑器，用 Rust 编写，编辑模型基于 kakoune。主要具有以下特性：

*   类似 Vim 的模态编辑
    
*   多项选择（Multiple selections）
    
*   内置语言服务器支持
    
*   通过 tree-sitter 实现语法高亮和代码编辑
    

Helix 打包了各种发行版，你也可以选择从源代码快速构建的方法：

`git clone --recurse-submodules --shallow-submodules -j8 https://github.com/helix-editor/helix  
cd helix  
cargo install --path helix-term  
`

这会将 hx 二进制文件安装到 $HOME/.cargo/bin，现在将 runtime/ 目录复制到某处。默认情况下，Helix 将在 config 目录或与可执行文件相同的目录中查找运行时，但这可以通过 HELIX_RUNTIME 环境变量覆盖。

如果要将 runtime / 目录嵌入到 Helix 二进制文件中，可以使用以下命令构建它：

`cargo install --path helix-term --features "embed_runtime"  
`

**Arch Linux**

AUR 提供两个软件包：

*   helix-bin：包含来自 GitHub 版本的预构建二进制文件
    
*   helix-git: 构建此存储库的主分支
    

**MacOS 系统**

Helix 可以通过自制软件安装在 MacOS 上：

`brew tap helix-editor/helix  
brew install helix  
`

目前，helix 已经在 Github 上标星 **2.7K**，累计分支 **81**（Github 地址：https://github.com/helix-editor/helix）

 推荐阅读：

[学会这 21 条，你离 Vim 大神就不远了！](http://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650795668&idx=4&sn=3748c3305758e2f7ab80ec3ad20c1dbe&chksm=befe7bda8989f2cc30d86568f1d9739df43e52dbd8b2a5227816d596143f33511a5139d852c2&scene=21#wechat_redirect)  

[提高 VS Code 编辑器性能的 5 个技](http://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650806290&idx=5&sn=a63ab520f1d90884039697b261ffab68&chksm=bd018d5c8a76044a5662f5413a31e19237f1465b9fb5196487d05ba5f0bf5ab94b9f35d07624&scene=21#wechat_redirect)  

[如何在 Vim 中更改颜色和主题](http://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650756319&idx=3&sn=020967b0a0ada78a385687330df60eb2&chksm=befec19189894887b91ac3c73ad757fb1d7173a32f4fcd6aae45882d8e4901b75196b1e12aa6&scene=21#wechat_redirect)  

[Github 爆火！21 岁理工男开源的十六进制编辑器爆赞](http://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650782454&idx=5&sn=e20e68d67f3ac9190a2df6ad853e971d&chksm=befe2fb88989a6ae555363e2be127cd79a51ece0e562d978277a6da15f9d5f029fd11be5d227&scene=21#wechat_redirect)  

[](http://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650806290&idx=3&sn=74f426734a73868beb7eb18b397e7126&chksm=bd018d5c8a76044a3d3654f4df4f4cfbe40fd211eec269e4b553b740f13ad75368e8365a41d7&scene=21#wechat_redirect)[Web 开发中使用了 Vim 作为主编辑器之后......](http://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650748217&idx=4&sn=bf988a9778e94315e5fc4e228a5ac49c&chksm=befea07789892961dc74906b1f4e6b7a7d1cf851805ef1b098e0178b96b73a5baf0f11e43c42&scene=21#wechat_redirect)