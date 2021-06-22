> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lmonkey.com](https://www.lmonkey.com/t/MXyGwWeyp)

> 展示帮助信息 githelp-g 回到远程仓库的状态 抛弃本地所有的修改，回到远程仓库的状态。

```
git help -g


```

抛弃本地所有的修改，回到远程仓库的状态。

```
git fetch --all && git reset --hard origin/master


```

也就是把所有的改动都重新放回工作区，并清空所有的 commit，这样就可以重新提交第一个 commit 了

```
git update-ref -d HEAD


```

输出工作区和暂存区的 different (不同)。

```
git diff


```

还可以展示本地仓库中任意两个 commit 之间的文件变动：

```
git diff <commit-id> <commit-id>


```

展示暂存区和最近版本的不同

输出暂存区和本地最近的版本 (commit) 的 different (不同)。

```
git diff --cached


```

输出工作区、暂存区 和本地最近的版本 (commit) 的 different (不同)。

```
git diff HEAD


```

```
git checkout -


```

```
git branch --merged master | grep -v '^\*\|  master' | xargs -n 1 git branch -d


```

```
git branch -vv


```

关联之后，git branch -vv 就可以展示关联的远程分支名了，同时推送到远程仓库直接：git push，不需要指定远程仓库了。

```
git branch -u origin/mybranch


```

或者在 push 时加上 - u 参数

```
git push origin/mybranch -u


```

-r 参数相当于：remote

```
git branch -r


```

-a 参数相当于：all

```
git branch -a


```

```
git checkout -b <branch-name>


```

```
git checkout -b <branch-name> origin/<branch-name>


```

```
git branch -d <local-branchname>


```

```
git push origin --delete <remote-branchname>


```

或者

```
git push origin :<remote-branchname>


```

```
git branch -m <new-branch-name>


```

```
git tag


```

展示当前分支的最近的 tag

```
git describe --tags --abbrev=0


```

```
git tag -ln


```

```
git tag <version-number>


```

默认 tag 是打在最近的一次 commit 上，如果需要指定 commit 打 tag：

```
$ git tag -a <version-number> -m "v1.0 发布(描述)" <commit-id>


```

首先要保证本地创建好了标签才可以推送标签到远程仓库：

```
git push origin <local-version-number>


```

一次性推送所有标签，同步到远程仓库：

```
git push origin --tags


```

```
git tag -d <tag-name>


```

删除远程标签需要先删除本地标签，再执行下面的命令：

```
git push origin :refs/tags/<tag-name>


```

一般上线之前都会打 tag，就是为了防止上线后出现问题，方便快速回退到上一版本。下面的命令是回到某一标签下的状态：

```
git checkout -b branch_name tag_name


```

```
git checkout <file-name>


```

放弃所有修改：

```
git checkout .


```

```
git rev-list -n 1 HEAD -- <file_path> #得到 deleting_commit

git checkout <deleting_commit>^ -- <file_path> #回到删除文件 deleting_commit 之前的状态


```

```
git revert <commit-id>


```

和 revert 的区别：reset 命令会抹去某个 commit id 之后的所有 commit

```
git reset <commit-id>  #默认就是-mixed参数。


```

git reset –mixed HEAD^ #回退至上个版本，它将重置 HEAD 到另外一个 commit, 并且重置暂存区以便和 HEAD 相匹配，但是也到此为止。工作区不会被更改。

```
git reset –soft HEAD~3  #回退至三个版本之前，只回退了commit的信息，暂存区和工作区与回退之前保持一致。如果还要提交，直接commit即可   

git reset –hard <commit-id>  #彻底回退到指定commit-id的状态，暂存区和工作区也会变为指定commit-id版本的内容


```

```
git commit --amend


```

```
git log


```

blame 的意思为‘责怪’，你懂的。

```
git blame <file-name>


```

每次更新了 HEAD 的 git 命令比如 commint、amend、cherry-pick、reset、revert 等都会被记录下来（不限分支），就像 shell 的 history 一样。 这样你可以 reset 到任何一次更新了 HEAD 的操作之后，而不仅仅是回到当前分支下的某个 commit 之后的状态。

```
git reflog


```

```
git commit --amend --author='Author Name <email@address.com>'


```

```
git remote set-url origin <URL>


```

```
git remote add origin <remote-url>


```

```
git remote


```

```
git whatchanged --since='2 weeks ago'


```

这个过程需要 cherry-pick 命令，参考

```
git checkout <branch-name> && git cherry-pick <commit-id>


```

简化命令

```
git config --global alias.<handle> <command>


```

比如：git status 改成 git st，这样可以简化命令

```
git config --global alias.st status


```

详解可以参考廖雪峰老师的 git 教程

```
git stash


```

untracked 文件：新建的文件

```
git stash -u


```

```
git stash list


```

```
git stash apply <stash@{n}>


```

回到最后一个 stash 的状态，并删除这个 stash

```
git stash pop


```

```
git stash clear


```

```
git checkout <stash@{n}> -- <file-path>


```

```
git ls-files -t


```

```
git ls-files --others


```

```
git ls-files --others -i --exclude-standard


```

可以用来删除新建的文件。如果不指定文件文件名，则清空所有工作的 untracked 文件。clean 命令，注意两点：  
clean 后，删除的文件无法找回  
不会影响 tracked 的文件的改动，只会删除 untracked 的文件

```
git clean <file-name> -f


```

可以用来删除新建的目录，注意：这个命令也可以用来删除 untracked 的文件。详情见上一条

```
git clean <directory-name> -df


```

```
git log --pretty=oneline --graph --decorate --all


```

把某一个分支到导出成一个文件

```
git bundle create <file> <branch-name>


```

新建一个分支，分支内容就是上面 git bundle create 命令导出的内容

```
git clone repo.bundle <repo-dir> -b <branch-name>


```

```
git rebase --autostash


```

```
git fetch origin pull/<id>/head:<branch-name>


```

```
git diff --word-diff


```

```
git clean -X -f


```

注意： config 分为：当前目录（local）和全局（golbal）的 config，默认为当前目录的 config

```
git config --local --list (当前目录)
git config --global --list (全局)


```

```
git status --ignored

commit历史中显示Branch1有的，但是Branch2没有commit

git log Branch1 ^Branch2


```

```
git log --show-signature


```

```
git config --global --unset <entry-name>


```

相当于保存修改，但是重写 commit 历史

```
git checkout --orphan <branch-name>


```

```
git show <branch-name>:<file-name>


```

```
git clone -b <branch-name> --single-branch https://github.com/user/repo.git


```

关闭 track 指定文件的改动，也就是 Git 将不会在记录这个文件的改动

```
git update-index --assume-unchanged path/to/file


```

恢复 track 指定文件的改动

```
git update-index --no-assume-unchanged path/to/file


```

不再将文件的权限变化视作改动

```
git config core.fileMode false


```

最新的放在最上面

```
git for-each-ref --sort=-committerdate --format='%(refname:short)' refs/heads/


```

通过 grep 查找，given-text：所需要查找的字段

```
git log --all --grep='<given-text>'


```

不添加参数，默认是 - mixed

```
git reset <file-name>


```

```
git push -f <remote-name> <branch-name>


```

```
git grep 'admin'


```

```
git shortlog -sn


```