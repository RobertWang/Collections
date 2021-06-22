> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwNzc2NTk0NQ==&mid=2247516602&idx=2&sn=af4b3552ed68938d2d3e11108ca64409&chksm=970fa96ca078207a3c1cdbb01dba68c38b0f6214fa8af0a02546b63b5b9d4bf8f6f45c382299&mpshare=1&scene=1&srcid=0622wtbFUo4FIyCXnYBXx8bb&sharer_sharetime=1624360780499&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

Git 版本对比相关操作
------------

*   [1] 输出工作区和暂存区的不同。
    

```
git diff


```

*   [2] 展示暂存区和最近版本的不同
    

```
git diff --cached


```

*   [3] 展示暂存区、工作区和最近版本的不同
    

```
git diff HEAD


```

*   [4] 展示本地仓库中任意两个 commit 之间的文件变动
    

```
git diff <commit-id> <commit-id>


```

Git 分支管理相关操作
------------

*   [1] 展示本地分支关联远程仓库
    

```
git branch -vv


```

*   [2] 列出所有远程分支
    

```
git branch -r


```

*   [3] 列出本地和远程分支
    

```
git branch -a


```

*   [4] 查看远程分支和本地分支的对应关系
    

```
git remote show origin


```

*   [5] 删除本地分支
    

```
# 创建并切换到本地分支git checkout -b <branch-name># 删除本地分支git branch -d <local-branchname># 重命名本地分支git branch -m <new-branch-name># 快速切换到上一个分支git checkout -# 跨分支提交git checkout <branch-name> && git cherry-pick <commit-id>


```

*   [6] 删除远程分支
    

```
git push origin --delete <remote-branchname>


```

*   [7] 远程删除了分支本地也想删除
    

```
git remote prune origin


```

*   [8] 关联远程分支
    

```
# 关联之后 git branch -vv 就可以展示关联的远程分支名了# 同时推送到远程仓库直接 git push 命令且不需要指定远程仓库了git branch -u origin/mybranch


```

Git 文件处理相关操作
------------

*   [1] 展示所有 tracked 的文件
    

bash

```
git ls-files -t


```

*   [2] 展示所有 untracked 的文件
    

bash

```
git ls-files --others


```

*   [3] 展示所有忽略的文件
    

bash

```
git status --ignoredgit ls-files --others -i --exclude-standard


```

*   [4] 强制删除 untracked 的文件
    

bash

```
# 使用clean命令后，删除的文件无法找回# 不会影响tracked的文件的改动，只会删除untracked的文件# 如果不指定文件文件名，则清空所有工作的untracked文件git clean <file-name> -f


```

*   [5] 强制删除 untracked 的目录
    

bash

```
# 如果不指定目录名称，则清空所有工作的untracked目录git clean <directory-name> -df


```

*   [6] 清除 gitignore 文件中记录的文件
    

bash

```
git clean -X -f


```

*   [7] 恢复删除的文件
    

bash

```
# 得到deleting_commit信息git rev-list -n 1 HEAD -- <file_path># 回到删除文件deleting_commit之前的状态git checkout <deleting_commit>^ -- <file_path>


```

Git 远程仓库相关操作
------------

*   [1] 列出所有远程仓库
    

bash

```
git remote


```

*   [2] 修改远程仓库的 url 地址
    

bash

```
git remote set-url origin <URL>


```

*   [3] 增加远程仓库地址
    

bash

```
git remote add origin <remote-url>


```

Git 存储状态相关操作
------------

*   [1] 存储当前的修改但不用提交 commit
    

bash

```
git stash


```

*   [2] 保存当前状态包括 untracked 的文件
    

bash

```
git stash -u


```

*   [3] 展示所有 stashes 信息
    

bash

```
git stash list


```

*   [4] 回到某个 stash 状态
    

bash

```
git stash apply <stash@{n}>


```

*   [5] 回到最后一个 stash 的状态并删除这个 stash 信息
    

bash

```
git stash pop


```

*   [6] 删除所有的 stash 信息
    

bash

```
git stash clear


```

*   [7] 从 stash 中拿出某个文件的修改
    

bash

```
git checkout <stash@{n}> -- <file-path>


```

Git 配置代码相关操作
------------

*   [1] 配置 ssh 代理
    

bash

```
# 直接使用shadowsocks提供的socks5代理端口$ cat ~/.ssh/configHost gitlab.comProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %pHost github.comProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p


```

*   [2] 配置 http 和 socks 代理
    

bash

```
# 适用于 privoxy 将 socks 协议转为 http 协议的 http 端口git config --global socks.proxy '127.0.0.1:1080'git config --global http.proxy 'http://127.0.0.1:8001'git config --global https.proxy 'http://127.0.0.1:8001'


```

Git 其他高级相关操作
------------

*   [1] 把某一个分支到导出成一个文件
    

bash

```
git bundle create <file> <branch-name>


```

*   [2] 把某一个文件导入成一个分支
    

bash

```
# 新建一个分支，分支内容就是上面 git bundle create 命令导出的内容git clone repo.bundle <repo-dir> -b <branch-name>


```

*   [3] 修改上一个 commit 的描述
    

bash

```
# 如果暂存区有改动同时也会将暂存区的改动提交到上一个commit中去git commit --amend


```

*   [4] 查看某段代码是谁写的
    

bash

```
git blame <file-name>


```

*   [5] 回到某个 commit 状态并删除后面的 commit 提交
    

bash

```
# 和revert命令不同，reset 命令会抹去某个commit_id之后的所有commit提交# 默认就是-mixed参数git reset <commit-id># 回退至上个版本将重置HEAD到另外一个commit# 并且重置暂存区以便和HEAD相匹配，但是也到此为止，工作区不会被更改git reset -- mixed HEAD^# 回退至三个版本之前，只回退了commit的信息，暂存区和工作区与回退之前保持一致# 如果还要提交，直接commit即可git reset -- soft HEAD~3# 彻底回退到指定commit-id的状态，暂存区和工作区也会变为指定commit-id版本的内容git reset -- hard <commit-id>


```

*   [6] 回到远程仓库的状态
    

bash

```
# 抛弃本地所有的修改，回到远程仓库的状态git fetch --all && git reset --hard origin/master


```

*   [7] 重设第一个 commit 信息
    

bash

```
# 也就是把所有的改动都重新放回工作区并清空所有的commit信息，这样就可以重新提交第一个commit了git update-ref -d HEAD


```

*   [8] 查找已经删除的文件提交
    

bash

```
# 模糊查找git log --all --full-history -- "**/thefile.*"# 精确查找git log --all --full-history -- <path-to-file>git log --diff-filter=D --summary | grep <file_name> | awk '{print $4; exit}' | xargs git log --all --# 查看所有删除文件git log --diff-filter=D --summary | grep delete# 查看产出文件是谁提交的git log --diff-filter=D --summary | grep -C 10 <file_name>


```

Git 给 Github 配置 RSS
-------------------

```
/* Repo releases */https://github.com/:owner/:repo/releases.atom/* Repo commits */https://github.com/:owner/:repo/commits.atom/* Private feed (You can find Subscribe to your news feed in dashboard page after login) */https://github.com/:user.private.atom?token=:secret/* Repo tags */https://github.com/:user/:repo/tags.atom/* User activity */https://github.com/:user.atom


```

寻求报道、约稿、文案投放：  
添加微信 xixiaoyao-1，备注 “商务合作”