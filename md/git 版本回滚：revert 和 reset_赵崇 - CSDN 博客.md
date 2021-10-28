> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/zc474235918/article/details/60136724)

       在使用 git 的时候，如果错误 push 之后，经常会回滚版本。  
git 的回滚有两种方式：

一：revert 命令
-----------

git revert 版本 id：  
       这个命令可以用一个相反的提交来回滚指定版本所做的修改。然后在 git push 即可吧线上的代码更新。  
       在使用 git revert 的时候，遇到一个问题。对于 merge 提交的代码，会出现下面的错误：

```
Commit XXX is a merge but no -m option was given.
```

       对于 merge 的分值，如果要 revert 的话，需要制定回滚到哪个版本，因为 merge 设计到了两个版本：  
如：

```
Merge branch 'rms-develop' into master
```

回滚这种情况的，可以使用

```
git revert 版本id -m 1
```

       这个 1 指的是 master，2 指的是 develop。现在是在 master 分支上回滚 develop 合并到 master 上的部分代码。  
       这种方式，是用一种反向的 push 来重新提交一次。git 中可以看到操作记录。

二 reset
-------

       这个命令，是一种重置，及错误提交了。我要删除这个提交记录。也可以实现回滚。

两个命令的对比：
--------

       看似达到的效果是一样的, 其实完全不同.  
第一:  
       上面我们说的如果你已经 push 到线上代码库, reset 删除指定 commit 以后, 你 git push 可能导致一大堆冲突. 但是 revert 并不会.  
第二:  
       如果在日后现有分支和历史分支需要合并的时候, reset 恢复部分的代码依然会出现在历史分支里. 但是 revert 方向提交的 commit 并不会出现在历史分支里.  
第三:  
       reset 是在正常的 commit 历史中, 删除了指定的 commit, 这时 HEAD 是向后移动了, 而 revert 是在正常的 commit 历史中再 commit 一次, 只不过是反向提交, 他的 HEAD 是一直向前的.