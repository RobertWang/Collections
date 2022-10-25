> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzM0MjcyMQ==&mid=2650163149&idx=1&sn=8480e232507b2e03e0ce764e2b572e46&chksm=bed99ca389ae15b5db30631307a2865be62c31497f89fc003ff374b100624b33835f8966e350&scene=21#wechat_redirect)

开发者 Wesley Smits 以自身经验为例罗列了 11 个他认为已经没用的扩展。“由于扩展可能会导致性能问题、增加 CPU 使用率，并且可能与其他扩展或本地功能发生冲突，因此最好将扩展限制为你所需要的扩展”。

Wesley Smits 指出，有些扩展的下载页面顶部有甚至有明确的弃用通知，但在 Medium、dev.to、Reddit 等一些平台上却仍有推荐贴。值得一提的是，这些扩展中有许多是原生存在于 Visual Studio Code 中，所以可以通过设置菜单启用 / 禁用或进行控制。

这些设置可以通过 UI 或 JSON 配置来控制。Wesley Smits 在文中以 JSON 版本为例建议：可以通过命令面板（⇧ ⌘P）打开全局 Visual Studio 代码设置的 settings.json。输入 settings，然后选择 "Preferences: Open User Settings (JSON)"。

这 11 个扩展具体包括：

**1、Auto Rename Tag — 1050 万次下载**

截至文章撰写时（10 月 14 日），这个扩展的下载量已超过 1050 万次。Smits 称，这个扩展频繁地出现在每个推荐扩展的文章中。然而事实上，VS Code 已经通过内部设置提供了同样的功能。通过启用以下设置，你的 tags 将自动重命名，而无需第三方扩展。

```
"editor.linkedEditing": true

```

链接：https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag

**2、Auto Close Tag — 800 万次下载**

这个非常受欢迎的扩展，与前一个扩展的作者是同一个人。目前，这个扩展的功能也已被添加到了 VS Code 中。

```
"html.autoClosingTags": true,
"javascript.autoClosingTags": true,
"typescript.autoClosingTags": true

```

链接：https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag

**3、Auto Import — 250 万次下载**

此扩展可自动查找、解析并为 Typescript 和 TSX 中的所有可用导入提供 code actions 和 code completion。

但实际上 VS Code 中有很多设置可以帮助动安排导入。可以为 JavaScript/TypeScript 使用 auto-import suggestions，在文件移动时更新导入，并在 top 使用绝对路径组织导入。

```
"javascript.suggest.autoImports": true,
"typescript.suggest.autoImports": true,
"javascript.updateImportsOnFileMove.enabled": "always",
"typescript.updateImportsOnFileMove.enabled": "always",
"source.organizeImports": true

```

链接：https://marketplace.visualstudio.com/items?itemName=steoates.autoimport

**4、Settings Sync — 340 万次下载**

这个扩展可以让 VS Code 设置在不同的设备之间保持同步。不过 VS Code 官方已经在大约两年前原生地添加了这个特性。

有关如何设置的更多信息，可阅读官方 VS Code 文档：https://code.visualstudio.com/docs/editor/settings-sync

链接：https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync

**5、Trailing Spaces— 110 万次下载**

此扩展 highlights training spaces ，并允许你使用命令将它们全部删除。VS Code 有一个设置，启用后会在你保存文件时删除所有 trailing spaces。启用设置：

```
"files.trimTrailingWhitespace": true,

```

链接：https://marketplace.visualstudio.com/items?itemName=shardulm94.trailing-spaces

**6、Path Intellisense — 800 万次下载**

此扩展自动完成路径和文件名，下载量超过 800 万次。同一开发人员针对 NPM 模块自动补全提供了此扩展的对应版本，下载量接近 500 万次。然而，与许多特性一样，VS Code 不久前添加了对这些特性的原生支持。这些功能默认开启，无需启用任何设置。

链接：https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense&ssr=false#overview

**7、NPM — 560 万次下载**

此扩展允许你用一个命令运行 NPM 脚本。但是 VS Code 也已经提供了提供了一种原生的处理方式。VS Code 有一个称为 Task Auto Detection 的功能，任务可以被自动检测到，也可以自定义为自动运行。

链接：https://marketplace.visualstudio.com/items?itemName=eg2.vscode-npm-script

**8、HTML Snippets — 840 万次下载**

此扩展允许你使用 shorthand 的方式快速放置 HTML snippets。以下两个扩展的概念也是如此：

*   CSS Snippets
    
*   HTML Boilerplate
    

这些扩展都提供 shorthand 版本，VS Code 已经原生支持这些版本。VS Code 内置了 Emmet，一个可以用 easy-to-remember shorthands 快速写出 HTML 和 CSS 的工具。此外，Emmet 提供了一个开箱即用的 HTML 模板，并允许你定义自己的自定义片段。

链接：https://marketplace.visualstudio.com/items?itemName=abusaidm.html-snippets

**9、HTMLTagWrap— 415K 次下载**

此扩展程序和其他类似扩展程序允许你选择一些 HTML 代码并将其 wrap 在 tag 中。

VVS Code 通过 Emmet 的一个命令也提供了这个功能。只要用 CTRL/CMD + Shift + P 打开命令面板，并查找 `Emmet: Wrap with abbreviation`。然后你可以输入任何你想要的 emmet 缩写，这可以是单个 tag、多个嵌套 tag、带有类或 ID 的 tag。

链接：https://marketplace.visualstudio.com/items?itemName=bradgashler.htmltagwrap

**10、Lorem Ipsum — 473K 次下载**

此扩展可让你快速将 Lorem Ipsum 文本块插入到你的 markup 中。Emmet 同样也支持此功能。你可以键入 `lorem` 并按 Tab 键以生成 30 字的段落。或者，如果你需要多个块，可以编写类似的内容以满足需求：

```
ul>li*3>p>lorem

```

链接：https://marketplace.visualstudio.com/items?itemName=Tyriar.lorem-ipsum

**11、Bracket Pair Colorizer (1 & 2) — 520 万次下载**

Bracket Pair Colorizer 及其继任者的安装量已超过 1100 万。鉴于此需求量，开发团队已在 VS Code core 中实现了 bracket pair colorizer，并声称这使其速度提高了 10.000 倍。可以通过启用以下设置来启用 Bracket pair coloring：

```
"editor.bracketPairColorization.enabled": true

```

以下是上述设置的完整清单：

```
{ 
    "editor.linkedEditing": true,
    "html.autoClosingTags": true,
    "javascript.autoClosingTags": true,
    "typescript.autoClosingTags": true,
    "javascript.suggest.autoImports": true,
    "typescript.suggest.autoImports": true,
    "javascript.updateImportsOnFileMove.enabled": "always",
    "typescript.updateImportsOnFileMove.enabled": "always",
    "source.organizeImports": true,
    "files.trimTrailingWhitespace": true,
    "editor.bracketPairColorization.enabled": true
}

```

链接：

https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer-2

Wesley Smits 最后作出结论称，Visual Studio Code 有一个广泛的扩展市场，可以增加你的便利度。但在安装其中一个之前，最好先看看它是否还没有原生支持。随着时间的推移，包含改进和功能的每月发布更新，越来越多的 Visual Studio Code 扩展将不再需要。

更多详情可查看全文：https://javascript.plainenglish.io/10-visual-studio-code-extensions-you-dont-need-6f7904132a57

对此，Reddit 上也有网友分享称，“有一堆扩展是 bulitin 的，你可以禁用所有你不需要的。进入扩展面板，搜索 @builtin”。

相关链接：https://javascript.plainenglish.io/10-visual-studio-code-extensions-you-dont-need-6f7904132a57

https://www.reddit.com/r/programming/comments/y3wud2/10_visual_studio_code_extensions_you_dont_need/