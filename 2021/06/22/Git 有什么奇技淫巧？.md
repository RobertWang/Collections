> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zhihu.com](https://www.zhihu.com/question/27462267/answer/96992683) ![](https://pic4.zhimg.com/2151b92f0_xs.jpg?source=1940ef5c)王奥​

title: Git 常用命令和 Git 团队使用规范指南  
date: 2016-04-22 16:22:32  
categories: 学习 | Study  
description: Git 是目前世界上最先进的分布式版本控制系统  
---

![](https://pic1.zhimg.com/50/0df9740f9b41522ab12c14dbbf6bbad4_hd.jpg?source=1940ef5c)

## 前言

在 2005 年的某一天，Linux 之父 Linus Torvalds 发布了他的又一个里程碑作品——Git。它的出现改变了软件开发流程，大大地提高了开发流畅度，直到现在仍十分流行，完全没有衰退的迹象。其实一般情况下，只需要掌握 git 的几个常用命令即可，但是在使用的过程中难免会遇到各种复杂的需求，这时候经常需要搜索，非常麻烦，故总结了一下自己平常会用到的 git 操作。本文根据团队实践记录 Git 入门指南和 Git 常用命令，文章中不仅记录了 Git 的搭建和使用教程，还参考了大量 Git 团队使用规范上的经验，希望大家可以结合自己团队的实际应用场景让 Git 协作优雅的落地。

> Git 是目前世界上最先进的分布式版本控制系统

## 更新记录

2016 年 04 月 22 日 - 初稿

阅读原文 - [Git 常用命令和 Git 团队使用规范指南](https://link.zhihu.com/?target=http%3A//wsgzao.github.io/post/git/)

** 扩展阅读 **

Git Book - [Git - Book](https://link.zhihu.com/?target=https%3A//git-scm.com/book/zh/)  
git 简明指南 - [git - the simple guide](https://link.zhihu.com/?target=http%3A//rogerdudler.github.io/git-guide/index.zh.html)  
常用 Git 命令清单 - [http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html](https://link.zhihu.com/?target=http%3A//www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)  
猴子都能懂的 GIT 入门 - [http://backlogtool.com/git-guide/cn/](https://link.zhihu.com/?target=http%3A//backlogtool.com/git-guide/cn/)  
Git 教程 - [Git 教程 - 廖雪峰的官方网站](https://link.zhihu.com/?target=http%3A//www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

## SVN 与 Git 的最主要的区别

SVN 是集中式版本控制系统，版本库是集中放在中央服务器的，而干活的时候，用的都是自己的电脑，所以首先要从中央服务器哪里得到最新的版本，然后干活，干完后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，如果在局域网还可以，带宽够大，速度够快，如果在互联网下，如果网速慢的话，就纳闷了。

Git 是分布式版本控制系统，那么它就没有中央服务器的，每个人的电脑就是一个完整的版本库，这样，工作的时候就不需要联网了，因为版本都是在自己的电脑上。既然每个人的电脑都有一个完整的版本库，那多个人如何协作呢？比如说自己在电脑上改了文件 A，其他人也在电脑上改了文件 A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

## Git 搭建和使用

> Git 上手并不难，深入学习还是建议多实践，可以参考扩展阅读中廖雪峰的 Git 教程

### Git 服务端

> 服务端搭建 Git 很简单，有更多需求不妨试试 Gogs 和 Gitlab

使用 Gogs 轻松搭建可能比 GitLab 更好用的 Git 服务平台 - [使用 Gogs 轻松搭建可能比 GitLab 更好用的 Git 服务平台](https://link.zhihu.com/?target=http%3A//wsgzao.github.io/post/gogs/)

``` bash  
#安装 git  
sudo apt-get install git  
yum install git

#创建一个 git 用户，用来运行 git 服务  
sudo adduser git

#创建证书使用公钥免密码登录 (可选)  
ssh-keygen -t rsa  
vi ~/.ssh/authorized_keys

#初始化 Git 仓库  
sudo git init --bare sample.git  
sudo chown -R git:git sample.git

#禁用 shell 登录  
vi /etc/passwd  
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell

#在客户端上克隆远程仓库  
git clone git@server:/srv/sample.git

```

管理公钥推荐使用 Gitosis  
Gitosis - [GitHub - res0nat0r/gitosis: Manage git repositories, provide access to them over SSH, with tight access control and not needing shell accounts.](https://link.zhihu.com/?target=https%3A//github.com/res0nat0r/gitosis)  
Gitosis 配置手记 - [Gitosis 配置手记](https://link.zhihu.com/?target=http%3A//debugo.com/gitosis/)

管理权限推荐使用 Gitolite  
Gitolite - [GitHub - sitaramc/gitolite: Hosting git repositories -- Gitolite allows you to setup git hosting on a central server, with very fine-grained access control and many (many!) more powerful features.](https://link.zhihu.com/?target=https%3A//github.com/sitaramc/gitolite)

### Git 客户端

> Git 客户端可以按个人习惯来选择，遵守团队协作中的 Git 规范标准才是更重要的

Git - [Git](https://link.zhihu.com/?target=https%3A//git-scm.com/)  
TortoiseGit - [TortoiseGit â€“ Windows Shell Interface to Git](https://link.zhihu.com/?target=https%3A//tortoisegit.org/)  
SourceTree - [Free Mercurial and Git Client for Windows and Mac](https://link.zhihu.com/?target=https%3A//www.sourcetreeapp.com/)

``` bash  
#以最基本的 Git 命令行为例，先下载 Git  
[Git - Downloads](https://link.zhihu.com/?target=https%3A//git-scm.com/download/)

#配置 git 提交用户名和邮箱，定义别名方便区分  
git config --global user.name "你的姓名"  
git config --global user.email "you@example.com"

#克隆仓库  
git clone cap@172.28.70.243:/cap/cap.git

$ git clone cap@172.28.70.243:/cap/cap.git  
Cloning into 'cap'...  
warning: You appear to have cloned an empty repository.  
Checking connectivity... done.

#测试推送  
touch README  
git add README  
git commit -m "add readme"  
git push origin master

Counting objects: 3, done.  
Writing objects: 100% (3/3), 199 bytes | 0 bytes/s, done.  
Total 3 (delta 0), reused 0 (delta 0)  
To cap@172.28.70.243:/cap/cap.git  
* [new branch] master -> master

```

## Git 常用命令

** 符号约定 **

- `<xxx>` 自定义内容  
- `[xxx]` 可选内容  
- `[<xxx>]` 自定义可选内容

``` bash  
#初始设置  
git config --global user.name "<用户名>" #设置用户名  
git config --global user.email "<电子邮件>" #设置电子邮件

#本地操作  
git add [-i] #保存更新，-i 为逐个确认。  
git status #检查更新。  
git commit [-a] -m "< 更新说明 >" #提交更新，-a 为包含内容修改和增删，-m 为说明信息，也可以使用 -am。

#远端操作  
git clone <git 地址> #克隆到本地。  
git fetch #远端抓取。  
git merge #与本地当前分支合并。  
git pull [<远端别名>] [< 远端 branch>] #抓取并合并, 相当于第 2、3 步  
git push [-f] [< 远端别名 >] [< 远端 branch>] #推送到远端，-f 为强制覆盖  
git remote add <别名> <git 地址 > #设置远端别名  
git remote [-v] #列出远端，-v 为详细信息  
git remote show <远端别名> #查看远端信息  
git remote rename <远端别名> < 新远端别名 > #重命名远端  
git remote rm <远端别名> #删除远端  
git remote update [<远端别名>] #更新分支列表

#分支相关  
git branch [-r] [-a] #列出分支，-r 远端 ,-a 全部  
git branch <分支名> #新建分支  
git branch -b <分支名> #新建并切换分支  
git branch -d <分支名> #删除分支  
git checkout <分支名> #切换到分支  
git checkout -b <本地 branch> [-t < 远端别名 >/< 远端分支 >] #-b 新建本地分支并切换到分支, -t 绑定远端分支  
git merge <分支名> #合并某分支到当前分支

```

Git 常用命令 - [Git 常用命令](https://link.zhihu.com/?target=http%3A//gityuan.com/2015/06/27/git-notes/)

![](https://pic4.zhimg.com/50/7e27e33239eeee59001da23a23483ccb_hd.jpg?source=1940ef5c)

- workspace: 本地的工作目录。（记作 A）  
- index：缓存区域，临时保存本地改动。（记作 B）  
- local repository: 本地仓库，只想最后一次提交 HEAD。（记作 C）  
- remote repository：远程仓库。（记作 D）

> 以下所有的命令的功能说明，都采用上述的标记的 A、B、C、D 的方式来阐述。

``` bash  
#初始化  
git init // 创建  
git clone /path/to/repository // 检出  
git config --global user.email "you@example.com" // 配置 email  
git config --global user.name "Name" // 配置用户名

#操作  
git add <file> // 文件添加，A → B  
git add . // 所有文件添加，A → B

git commit -m "代码提交信息" // 文件提交，B → C  
git commit --amend // 与上次 commit 合并, *B → C

git push origin master // 推送至 master 分支, C → D  
git pull // 更新本地仓库至最新改动， D → A  
git fetch // 抓取远程仓库更新， D → C

git log // 查看提交记录  
git status // 查看修改状态  
git diff// 查看详细修改内容  
git show// 显示某次提交的内容

#撤销操作  
git reset <file>// 某个文件索引会回滚到最后一次提交， C → B  
git reset// 索引会回滚到最后一次提交， C → B  
git reset --hard // 索引会回滚到最后一次提交， C → B → A

git checkout // 从 index 复制到 workspace， B → A  
git checkout -- files // 文件从 index 复制到 workspace， B → A  
git checkout HEAD -- files // 文件从 local repository 复制到 workspace， C → A

#分支相关  
git checkout -b branch_name // 创建名叫 “branch_name” 的分支，并切换过去  
git checkout master // 切换回主分支  
git branch -d branch_name // 删除名叫 “branch_name” 的分支  
git push origin branch_name // 推送分支到远端仓库  
git merge branch_name // 合并分支 branch_name 到当前分支 (如 master)  
git rebase // 衍合，线性化的自动， D → A

#冲突处理  
git diff // 对比 workspace 与 index  
git diff HEAD // 对于 workspace 与最后一次 commit  
git diff <source_branch> <target_branch> // 对比差异  
git add <filename> // 修改完冲突，需要 add 以标记合并成功

#其他  
gitk // 开灯图形化 git  
git config color.ui true // 彩色的 git 输出  
git config format.pretty oneline // 显示历史记录时，每个提交的信息只显示一行  
git add -i // 交互式添加文件到暂存区  
```

## Git 使用规范

Git 使用规范流程 - [http://www.ruanyifeng.com/blog/2015/08/git-use-process.html](https://link.zhihu.com/?target=http%3A//www.ruanyifeng.com/blog/2015/08/git-use-process.html)  
团队中的 Git 实践 - [团队中的 Git 实践](https://link.zhihu.com/?target=https%3A//ourai.ws/posts/working-with-git-in-team/)  
构家网 git 团队协作使用规范 v2 - [构家网 git 团队协作范 使用规范 v2_百度文库](https://link.zhihu.com/?target=http%3A//wenku.baidu.com/view/e1430d1b7f1922791788e81e)

> Git 使用规范提醒

- 使用 Git 过程中，必须通过创建分支进行开发，坚决禁止在主干分支上直接开发。review 的同事有责任检查其他同事是否遵循分支规范。  
- 在 Git 中，默认是不会提交空目录的，如果想提交某个空目录到版本库中，需要在该目录下新建一个 .gitignore 的空白文件，就可以提交了  
- 把外部文件纳入到自己的 Git 分支来的时候一定要记得是先比对，确认所有修改都是自己修改的，然后再纳入。不然，容易出现代码回溯  
- 多人协作时，不要各自在自己的 Git 分支开发，然后发文件合并。正确的方法应该是开一个远程分支，然后一起在远程分支里协作。不然，容易出现代码回溯（即别人的代码被覆盖的情况）  
- 每个人提交代码是一定要 git diff 看提交的东西是不是都是自己修改的。如果有不是自己修改的内容，很可能就是代码回溯  
- review 代码的时候如果看到有被删除掉的代码，一定要确实是否是写代码的同事自己删除的。如果不是，很可能就是代码回溯![](https://pic1.zhimg.com/79825af73_xs.jpg?source=1940ef5c)杨森

1. 在尝试过所有命令都不能把你从深渊里挽救出来的时候，git reflog 也许能救你一命。

比如撤销一次 rebase（rebase 可是会直接修改历史的，一定要了解原理后再使用） [Undoing a git rebase](https://link.zhihu.com/?target=http%3A//stackoverflow.com/questions/134882/undoing-a-git-rebase)

2. 每次 merge 完总是出现很多 .orig 文件，使用 git clean -f 干掉所有 untracked files

3. rebase 一个 diverged 分支一直要解决冲突很痛苦，可以尝试在自己的分支先 squash 一下，git rebase -i，然后再 rebase 主干，解决一次冲突就 ok 了

4. 本地有很多其实早就被删除的远程分支，可以用 git remote prune origin 全部清除掉，这样再 checkout 别的分支时就清晰多了![](https://pic4.zhimg.com/9bfc5cc62_xs.jpg?source=1940ef5c)大 Kk

```
git bisect


```

有没有过写了一天的代码，checkin 无数，结果突然发现之前没注意的地方 break 的时候？

这个时候要在茫茫 commits 里寻找那个错误的 commit 是多么的痛苦啊。`git-bisect` 就是大救星！

git-bisect 本质上就是一个二分法，用起来也很简单：  

```
git bisect start        #start
git bisect bad          #current branch is bad
git bisect good <SHA-1> #some old commit that is good


```

然后只要不停的告诉 git 当前 commit 是不是好的，  

```
git bisect good


```

或者  

```
git bisect bad


```

就能找到罪魁祸首了！