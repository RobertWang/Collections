> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/h2zZhou/p/6136948.html)

简介
--

svn 和 git 都是常用的版本管理软件，但是 git 无论在理念或是功能上都比 svn 更为先进。但是有的公司是以 svn 作为中央仓库，这时 git 与 svn 代码的同步就可以通过 git-svn 这个软件进行，从而用 git 管理 svn 代码。最后的效果相当于把 svn 仓库当作 git 的一个 remote（远程仓库），而你本地的代码都是通过 git 来管理，只有 push 到 svn 时才会把你本地的 commit 同步到 svn。

从 svn 克隆
--------

首先看一看用于测试的 svn 项目结构，svn 的仓库路径是`file:///d/Projects/svn_repo`，可以用`svnadmin create svn_repo`命令新建。该仓库有 2 个分支，1 个 tag，属于 svn 标准布局。

SVN 项目结构：

```
/d/proj1
├── branches
│   ├── a
│   │   └── readme.txt
│   └── b
│       ├── 11.txt
│       └── readme.txt
├── tags
│   └── v1.0
│       ├── 11.txt
│       └── readme.txt
└── trunk
    └── readme.txt
```

命令格式：`git svn clone <svn仓库路径> [本地文件夹名] [其他参数]` 相当于`git clone`  
示例： `git svn clone file:///d/Projects/svn_repo proj1_git -s --prefix=svn/`  
参数说明：

*   `-s` 告诉 Git 该 Subversion 仓库遵循了基本的分支和标签命名法则，也就是标准布局。  
    如果你的主干 (trunk，相当于非分布式版本控制里的 master 分支，代表开发的主线），分支(branches) 或者标签 (tags) 以不同的方式命名，则应做出相应改变。  
    `-s`参数其实是`-T trunk -b branches -t tags`的缩写，这些参数告诉 git 这些文件夹与 git 分支、tag、master 的对应关系。
*   `--prefix=svn/` 给 svn 的所有 remote 名称增加了一个前缀 svn，这样比较统一，而且可以防止`warning: refname 'xxx' is ambiguous.`

现在，看下用 git-svn 克隆的项目情况（运行 git branch -a），此处 git 的分支情况是与 svn 文件夹对应的。

```
* master
  remotes/svn/a
  remotes/svn/b
  remotes/svn/tags/v1.0
  remotes/svn/trunk
```

### 只下载指定版本之后的历史

如果 svn 上的 commit 次数非常多, git svn clone 就会非常慢，一般超过几百个版本就要大概十分钟。此时可以在 clone 的时候只下载部分版本，  
命令：`git svn clone -r<开始版本号>:<结束版本号> <svn项目地址> [其他参数]`  
示例：`git svn clone -r2:HEAD file:///d/Projects/svn_repo proj1_git -s`  
说明：其中 2 为 svn 版本号，HEAD 代表最新版本号，就是只下载 svn 服务器上版本 2 到最新的版本的代码.

工作流程
----

简单来说就是，首次新建分支会记录和 svn 远程对应分支的追踪关系，之后你的所有 commit 都是在本地的；并且和纯 git 管理的项目没有区别，只是在`git svn rebase`和`git svn dcommit`的时候才会和 svn 仓库发生关系

### 一般工作流程（推荐）

1.  新建分支`git checkout -b <本地分支名称> <远程分支名称>`  
    示例：`git checkout -b a svn/a`  
    说明：此处新建了一个本地分支 a，与 svn 的 a 分支对应。
2.  在本地工作，commit 到对应分支上
3.  `git svn rebase` 从 svn 上更新代码, 相当于 svn 的 update。
4.  `git svn dcommit` 提交你的 commit 到 svn 远程仓库，建议提交前都先运行下 git svn rebase。

### 在 git 本地其他分支工作的情况

1.  `git chechout -b a svn/a` 此处新建了一个本地分支 a，与 svn 的 a 分支对应。
2.  `git checkout -b feature1` 在 a 分支的基础上，开一个本地 feture1 分支
3.  在 feture1 分支进行开发，有了多次 commit
4.  在 feture1 分支上进行`git svn rebase` 和 `git svn dcommit`，这样 feature1 的 commit 也会提交到 svn 的 a 分支上。  
    需要注意的是要记住 feture1 是从哪个分支 checkout 的，它的 svn 远程分支就与哪个相同。比如此处是 a 分支，那么 svn 分支就是 svn/a，commit 就会提交到 svn 的 a 分支。

SVN 分支管理
--------

### 新建分支到 svn

命令：`git svn branch <分支名称>`  
示例：`git svn branch c_by_git`  
说明：在 svn 仓库上建了了一个 c_by_git 分支  
分支情况

```
a
* master
  remotes/svn/a
  remotes/svn/b
  remotes/svn/c_by_git
  remotes/svn/tags/v1.0
  remotes/svn/trunk
```

### 删除 svn 分支

*   删除 svn 分支目录`svn rm <svn分支路径> -m <commit信息>`  
    示例：`svn rm file:///d/Projects/svn_repo/branches/c_by_git -m 'rm branch'`
*   删除远程跟踪分支`git branch -D -r <远程分支名称>`  
    示例：`git branch -D -r svn/c_by_git`

SVN 上 tag 管理
------------

### 新建 tag

命令：`git svn tag <tag名称>`  
示例：`git svn tag v1.1`  
说明：在 svn 仓库上建了一个 v1.1tag

### 删除 tag

1.  删除 svn 目录`svn rm <svntag路径> -m <commit信息>`  
    示例：`svn rm file:///d/Projects/svn_repo/tags/v1.1 -m 'rm tag'`
    
2.  删除远程跟踪分支`git branch -D -r <远程分支名称>`  
    示例：`git branch -D -r svn/tags/v1.1`  
    说明：svn 的 tag 和分支在 git 看来是一样的，所以此处还是用的 git branch
    

冲突解决
----

如果本地和 svn 都进行了修改，则不能快速前进，git svn rebase 会出现错误。  
这时应该按以下步骤操作：

1.  手动修改冲突文件，修改完成后`git add`
    
2.  `git rebase --continue`
    
3.  `git svn dcommit`
    

svn 不遵循规范的情况
------------

**以上讲的都是 svn 仓库是标准的情况，如果不标准，则以下几个地方都会有所不同。**主要就是每个步骤基本都要添加 svn 的具体路径。  
先看看，示例项目的结构，仓库路径是`file:///d/Projects/svn_repo2`。这个项目主分支是 dev 文件夹，branch1 和 tag1 文件夹分别代表的是一个分支和 tag。

svn 项目结构：

```
/d/proj2
├── branch1
│   └── file1.txt
├── dev
│   └── file1.txt
└── tag1
    └── file1.txt
```

### 从 svn 克隆

命令：`git svn clone <svn项目地址，要包含具体分支路径> [本地文件夹名]`  
示例：`git svn clone file:///d/Projects/svn_repo2/dev proj2_svn`

### 添加远程分支信息

命令：

1.  `git config --add svn-remote.<远程分支名称>.url <svn地址，要包含具体分支路径>`
2.  `git config --add svn-remote.<远程分支名称>.fetch :refs/remotes/<远程分支名称>`

示例：

1.  `git config --add svn-remote.svn/branch1.url file:///d/Projects/svn_repo2/branch1`
2.  `git config --add svn-remote.svn/branch1.fetch :refs/remotes/svn/branch1`

说明：此处的 “远程分支名称” 可以随意填写，只要这三个保持一致即可。建议都给他们增加`svn/`前缀，这样 svn 的所有分支显示起来会比较一致，与上面 clone 时的`--prefix=svn/`类似。

### 新建本地分支，与 svn 对应

命令：

1.  `git svn fetch <远程分支名称>` 获取 svn 仓库该分支的代码
2.  `git checkout -b <本地分支名> <远程分支名称>`

示例：

1.  `git svn fetch svn/branch1`
2.  `git checkout -b branch1 svn/branch1`

分支情况：

```
* branch1
  master
  remotes/git-svn
  remotes/svn/branch1
```