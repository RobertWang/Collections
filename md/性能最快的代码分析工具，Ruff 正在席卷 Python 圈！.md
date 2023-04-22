> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TV4PuXwxkmsLgEVvt5DZqg)

![](https://mmbiz.qpic.cn/mmbiz_png/LLRiaS9YfFTMaRu30ej8Lez7W6H5GCTIlsIqGhQFGKicic0adIKsjlRMIB8OkpXLvjfsonFvqmsWDyoXWkdKOqFYw/640?wx_fmt=png)  

几天前，Python 开源社区又出了一个不小的新闻：HTTPX 和 Starlette 在同一天将在用的代码分析工具（flake8、autoflake 和 isort）统一替换成了 Ruff。

HTTPX 是一个支持异步的 HTTP 客户端，Starlette 是一个轻量级的 ASGI 框架，它们都是 Python 社区里的明星项目，目前加起来有近 20K star。它们都选择了使用 Ruff，再次扩大了 Ruff 的应用版图。

Ruff 是个诞生仅仅 8 个月的新兴项目，但已呈现出一种席卷 Python 社区的趋势！很多知名的开源项目已采纳 Ruff，比如 Transformers、Pandas、FastAPI、Airflow、SciPy、Bokeh、Jupyter、LangChain、PaddlePaddle、Sphinx、Pydantic、LlamaIndex……

Ruff 是什么？为什么它能吸引大量的开源项目使用？相比于其它代码分析工具，它有哪些突出之处，是否还有一些局限性？现在是否值得将项目在用的工具都替换成它呢？

带着这些问题，本文将带你全方位了解这个火爆的项目。

Ruff 加速 Rust 与 Python 的融合
-------------------------

Ruff 诞生于 2022 年 8 月，它是一个用 Rust 语言编写的高性能的 Python 静态代码分析工具，比其它分析工具快几个数量级（10-100 倍），而且功能也很全面。

![](https://mmbiz.qpic.cn/mmbiz_png/LLRiaS9YfFTMaRu30ej8Lez7W6H5GCTIlNs4FkQugAkicFSiaX9IvVxdcN94F7Xh82kwLPshoiaavbyicibgrI4JkHdg/640?wx_fmt=png)从头检测 CPython 代码库的结果对比

`代码分析工具` 即 Linter，用于检查代码中的语法错误、编码规范问题、潜在的逻辑问题和代码质量问题等，可以提供实时反馈和自动修复建议。

在 Ruff 出现之前，社区里的代码分析工具呈现出百花齐放之势，比如有 Pylint、Flake8、Autoflake、Pyflakes、Pycodestyle 等等，它们的共同点是都使用 Python 编写而成。

Ruff 异军突起，在性能方面立于不败之地，主要得益于 Rust 天然的速度优势。Ruff 的出现，就像基于大语言模型的 ChatGPT 横空出世，所有竞争对手瞬间就黯淡失色了。

在 Ruff 项目成熟后，他将用 Rust 开发高性能的 Python 类型检查工具，到时候，目前流行的 Mypy、Pytype、Pyright 和 Pyre 等工具将迎来一大劲敌。（题外话：Python 社区纷乱繁多的虚拟环境管理工具和依赖包管理工具，也有望迎来变革了吧！）

![](https://mmbiz.qpic.cn/mmbiz_png/LLRiaS9YfFTMaRu30ej8Lez7W6H5GCTIlcbwxfG5YWC4lDO9mfpkiaS1yjoiamnSaicPeL0j4I5gjZmgUwEiawSOTQQ/640?wx_fmt=png)他的目标是让 Python 生态更加高效

这里还必须介绍两个 Rust 项目，因为 Ruff 的成功离不开它们：

*   RustPython：用 Rust 写成的 Python 解释器。Ruff 利用了它高性能的 AST 解析器，以此实现了自己的 AST 遍历、访问器抽象和代码质量检测逻辑
    
*   Maturin：用 Rust 写成的打包工具，可以将 Rust 项目打包成 Python 可用的包，从而可以被我们 “pip install” 后使用，且不需要配置 Rust 环境
    

Ruff 的优点与局限性
------------

介绍完最关键的特性后（速度极快、支持 pip），我们接下来看看 Ruff 的其它方面。

总体而言，它具有这些特点：

*   支持 `pyproject.toml`
    
*   兼容 Python 3.11
    
*   超过 500 条内置规则，与 Flake8 内置的规则集近乎对等
    
*   重新实现了数十个 Flake8 插件，如 flake8-bugbear、flake8-comprehensions 等
    
*   支持自动修复，可自动纠正错误（例如，删除未使用的导入）
    
*   内置缓存，可避免重复分析未更改的文件
    
*   支持 VS Code、Pycharm、Neovim、Sublime Text、Emacs 等编辑器
    
*   对 monorepo 友好，具有分层和级联配置
    

首先最值得介绍的是它支持的规则。Ruff 借鉴了流行的工具如 Flake8、autoflake、isort、pyupgrade、yesqa 等等，然后用 Rust 重新实现了超过 500 条规则。它本身不支持插件，但是吸收了数十个常用的 Flake8 插件的设计，使得已囊括的规则范围比其它任何工具都大。

![](https://mmbiz.qpic.cn/mmbiz_png/LLRiaS9YfFTMaRu30ej8Lez7W6H5GCTIlgwic5s3AvkNU2KyrCdGXd34DyoGXunc2TcfdGYwnmKAAS2iaicC3rjznA/640?wx_fmt=png)实现了的部分 flake8 插件

Ruff 的作者还非常熟悉其它语言的分析工具，比如 Rust 的 Clippy 和 JavaScript 的 ESLint，并从这些项目上得到了设计上的启发。

Ruff 站在了多个工具 / 插件的肩膀上，重新实现了它们验证过的规则，也借鉴了它们的 API 和实现细节，这使得它扮演了一种 “集大成” 的角色，很方便使用者们作工具的顺滑迁移。

Ruff 第二个值得介绍的特点是，它没有局限于 Linter 的定位，而是借鉴 Rome、Prettier 和 Black 这些代码格式化工具（Formatter），也实现了代码格式化的功能。借鉴了 Autoflake、ESLint、Fixit 等工具，实现了代码自动纠错的功能。另外，它还借鉴了使用很广泛的 isort，支持对 import 作快速排序。

这些表明作者的目标并不只是开发一款优秀的代码分析工具，而是在静态代码分析的核心功能外，要创造出更多的可能性。此举是开发者的福音啊，以后一个工具就能满足多种诉求，再也不必纠结于不同工具的选型、协作与维护了！

Ruff 还有其它的优点，例如支持 `pyproject.toml` 、支持 Python 3.11、支持只分析变更的文件，等等。另外，它也有着一些局限性：

*   支持的 lint 规则还有不够
    
*   不支持使用插件，扩展性不强
    
*   用 Rust 开发的，因此不便于在出错时 debug，也不便于 Python 开发者给它贡献代码
    

关于第一点，毕竟 Ruff 只是 8 个月大的新生项目，支持更多的规则，只是时间问题。至于插件带来的扩展性和编程语言的开发者生态，原因也是 Rust，属于 “有得必有失” 了。

Ruff 的使用
--------

介绍完 Ruff 的整体情况后，我们接着看看该如何使用它吧。

首先是安装，可以用 Conda 和其它包管理工具，也可以直接用 pip：

```
pip install ruff


```

可以通过以下命令运行：

```
ruff check .                        # 分析当前及子目录内的所有文件
ruff check path/to/code/            # 分析指定目录及子目录内的所有文件
ruff check path/to/code/*.py        # 分析指定目录内的所有py文件
ruff check path/to/code/to/file.py  # 分析 file.py


```

可以用作预提交的钩子：

```
- repo: https://github.com/charliermarsh/ruff-pre-commit
  # Ruff version.
  rev: 'v0.0.261'
  hooks:
    - id: ruff


```

可以通过 `pyproject.toml` ，`ruff.toml` 或 `.ruff.toml` 文件进行配置，默认配置已能满足基本使用，详细配置可以参见文档的 Configuration 。

Ruff 提供了官方的 VS Code 插件，可以快速上手：

![](https://mmbiz.qpic.cn/mmbiz_gif/LLRiaS9YfFTMaRu30ej8Lez7W6H5GCTIlrNbKIBnLrS1QdM69ibFspica6d2UIuoo9UeAE7saJhbfWmicnomw8l5wA/640?wx_fmt=gif)Ruff 的 VS Code 插件

Ruff 官方没有提供 Pycharm 的插件，社区中有人发布了一个 Ruff 插件。

另外，它还提供了`ruff-lsp` ，可以被集成到任何支持 Language Server Protocol 的编辑器中，例如 Neovim、Sublime Text、Emacs 等等。

小结
--

本文从 HTTPX 和 Starlette 采纳 Ruff 的新闻开始，向读者介绍了这个仅诞生 8 个月却俘获了一大批知名开源项目。它最突出的特点是使用 Rust 开发，因此在性能方面远远超越同类工具，此外，它借鉴了众多工具和插件的设计，不仅静态代码分析的规则全面，而且还具备代码格式化、代码自动纠错和 import 排序等非其它 linter 所拥有的功能。

Ruff 的成功为 Python 社区提供了一个鲜活的榜样，可以预见，我们将迎来一波用 Rust 开发的高性能工具。Ruff 的成功，与最近火爆的 ChatGPT 一样，它们传递出了一个 “**这事儿能成**” 的信号，从而会引爆一场使用新技术的变革！（非常巧合的是：Rust 1.0 在 2015 年 5 月发布，而 OpenAI 在 2015 年 12 月成立。）

总体而言，Ruff 非常强大，凭实力而风靡 Python 社区，绝对推荐使用！它的使用文档很友好，如果你想了解更多细节，可以去翻查。