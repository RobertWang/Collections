> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3MDA0NTMzMA==&mid=2247493769&idx=6&sn=28e5bcfab7dec131b5a2a540f1e4b5e5&chksm=fcf7ce74cb804762ad480c5bc6112d2b245eef9018b1184ebdd9a41427db8ea00de553f23012&mpshare=1&scene=1&srcid=0605niTMObJHnlIx9ameQF3O&sharer_sharetime=1622826626212&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

作者：开源最前线（ID：OpenSourceTop） 猿妹编译 

编译自：https://github.com/github/semantic

首先呢，运行 semantic --help 获取最新的完整选项列表:

**解析**

```
Usage: semantic parse ([--sexpression] | [--json] | [--json-graph] | [--symbols]
                      | [--dot] | [--show] | [--quiet]) [FILES...]
  Generate parse trees for path(s)

Available options:
  --sexpression            Output s-expression parse trees (default)
  --json                   Output JSON parse trees
  --json-graph             Output JSON adjacency list
  --symbols                Output JSON symbol list
  --dot                    Output DOT graph parse trees
  --show                   Output using the Show instance (debug only, format
                           subject to change without notice)
  --quiet                  Don't produce output, but show timing stats
```

Semantic 使用树形图来生成解析树，现在我们拿一个简单的程序来解析你会看的更明了，打开 test.A.py 文件，粘贴如下：

```
def Foo(x):
    return x
print Foo("hi")
```

现在，让我们生成一个抽象语法树（AST）

```
$ semantic parse test.A.py
(Statements
  (Annotation
    (Function
      (Identifier)
      (Identifier)
      (Return
        (Identifier)))
    (Empty))
  (Call
    (Identifier)
    (Call
      (Identifier)
      (TextElement)
      (Empty))
    (Empty)))
```

默认的 s-expression 输出是一种很好的格式，可以快速可视化代码结构。我们可以看到有一个声明的函数，然后有一个调用表达式，嵌套在另一个调用表达式中，它与函数调用 print 和 Foo。你还可以使用其他的输出格式。

**DIFF（比较）**

```
Usage: semantic diff ([--sexpression] | [--json] | [--json-graph] | [--toc] |
                     [--dot] | [--show]) [FILE_A] [FILE_B]
  Compute changes between paths

Available options:
  --sexpression            Output s-expression diff tree (default)
  --json                   Output JSON diff trees
  --json-graph             Output JSON diff trees
  --toc                    Output JSON table of contents diff summary
  --dot                    Output the diff as a DOT graph
  --show                   Output using the Show instance (debug only, format
                           subject to change without notice)
```

**Graph（图）**

```
Usage: semantic graph ([--imports] | [--calls]) [--packages] ([--dot] | [--json]
                      | [--show]) ([--root DIR] [--exclude-dir DIR]
                      DIR:LANGUAGE | FILE | --language ARG (FILES... | --stdin))
  Compute a graph for a directory or from a top-level entry point module

Available options:
  --imports                Compute an import graph (default)
  --calls                  Compute a call graph
  --packages               Include a vertex for the package, with edges from it
                           to each module
  --dot                    Output in DOT graph format (default)
  --json                   Output JSON graph
  --show                   Output using the Show instance (debug only, format
                           subject to change without notice)
  --root DIR               Root directory of project. Optional, defaults to
                           entry file/directory.
  --exclude-dir DIR        Exclude a directory (e.g. vendor)
  --language ARG           The language for the analysis.
  --stdin                  Read a list of newline-separated paths to analyze
                           from stdin.
```

**语言支持**

多语言支持是 Semantic 的一大优势，目前支持 Ruby、JavaScript、typescript、Python、Go、PHP、Java 等主流编程语言都支持

![](https://mmbiz.qpic.cn/mmbiz_png/kOTNkic5gVBF6HLiayzg2b0mPeABZU4xpiclqDdK641WvS672G5DEz1yibNF55MVunMIX3Y3RyrFjFuJHL2Y8SAk1A/640?wx_fmt=jpeg)

Semantic 最低要求 GHC 8.6.4 和 Cabal 2.4，建议使用 ghcup 沙箱 GHC 版本，为通过操作系统的软件包管理器安装的 GHC 软件包可能无法安装静态链接版本的 GHC 启动库。

```
git clone git@github.com:github/semantic.git
cd semantic
script/bootstrap
cabal new-build
cabal new-test
cabal new-run semantic -- --help
```

创建者使用 cabal 的 Nix 风格的本地版本进行开发。要快速入门，可以按照以上操作。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/kOTNkic5gVBF5PoShib8SDibtQuia5H6MujRF3E74cek7edmS40amPia6nLV9VgbiaRGyFje3aVYaxBdib4oQdqwGiaibpw/640?wx_fmt=png)

目前，semantic 已经在 GitHub 上获得 **8000** 个 Star，**450** 个 Fork，感兴趣的可以到 GitHub 上查阅更多详情。

GitHub 地址：

https://github.com/github/semantic