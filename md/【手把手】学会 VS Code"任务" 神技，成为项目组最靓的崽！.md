> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7035448197883363359) **1. 明白`VS Code 任务系统`是什么？**  
**2. 按步骤学会一步步配置一些简易而实用的`VS Code任务`**

> 在不知道 VSCode 任务系统的人看来，它就像是**魔法**一样！

利用任务，可以有多便捷？
------------

**背景：** 我司的代码合入采用的是 `从主仓库fork` => `从个人仓库提Merge Request`这种`github`经典模式。  
**日常：** 因此我司员工经常需要依次执行以下`4条指令`或者在`VSCode源代码管理中依次执行以下四个操作`：

1.  `$ git stash push -u -m xxx` (将当前未提交的内容存储)
2.  `$ git pull base --rebase` (从主仓库变基拉取代码)
3.  `$ git push origin --force` (向个人仓库推送)
4.  `$ git stash pop` (弹出之前存储的内容)

**魔法：** 熟悉 VS Code 任务系统的我，在执行以上内容时，只需要两步：

1.  按下`一个快捷键`。
2.  连点`两下回车键`。

如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1cb510f515cf4c84a2e06f8cb627e9de~tplv-k3u1fbpfcp-watermark.awebp?)  
接下来 VS Code 竟自行完成了以上四个步骤！

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/932580e828284fe8a26083711509779e~tplv-k3u1fbpfcp-watermark.awebp?)

这不仅能让我把上面这种耗时的日常操作浓缩到**不到两秒**的操作中，还让能不经意间在同事面前展示一下那神秘的`极客范`。

_**那么？VS Code 任务系统到底是什么？它能做什么？我们要怎么使用它呢？**_

什么是 VS Code 任务系统？
-----------------

> VS Code 任务系统支持用户通过`可视化界面、热键`来触发`运行脚本或启动程序`的效果。它的行为是`通过配置`来定义的。

官方地址：[# VS Code 任务](https://link.juejin.cn?target=https%3A%2F%2Fcode.visualstudio.com%2FDocs%2Feditor%2Ftasks "https://code.visualstudio.com/Docs/editor/tasks")  
关键词解读：

1.  目标：`运行脚本、启动程序`；  
    任务系统的终极目标，是去执行一些你期望执行的`脚本`或`执行程序`。以本文开始时的例子为例，执行`git 命令`便是执行脚本了。
2.  触发方式：`可视化界面、热键`；  
    你可以通过快捷键唤出`任务列表`进行选择，或者直接执行你设置了热键的任务。
3.  定义方式：`配置`；  
    JSON 格式。

动手一：配置一个最简单的`git-fetch`任务
-------------------------

1.  在项目根目录下创建一个`.vscode`文件夹，并创建一个`.vscode/tasks.json`文件。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0991187f86894b5695b936613485f801~tplv-k3u1fbpfcp-watermark.awebp?)

2.  在 tasks.json 内输入如下内容：

```
{
  "version": "2.0.0",
  "tasks": [
    {
      // 任务的名称
      "label": "git-fetch",
      // 任务类别，shell代表脚本
      "type": "shell",
      // 任务脚本，可以是yarn/npm/git 等
      "command": "git",
      // 命令参数
      "args": [
        "fetch"
      ],
      // 声明无需扫描脚本输出
      "problemMatcher": []
    }
  ]
}
复制代码

```

3.  设置**热键**

在 VS Code 中打开： `文件`-`首选项`-`键盘快捷方式`，或者同时按下：`Ctrl K S`三个按键。 此时你的 VS Code 会进入`热键设置页面`，在搜索栏搜索`workbench.action.tasks.runTask`或者`任务: 运行任务`, 选中, 并设置一个你习惯的`组合式快捷键`。

> 比如我，设置的快捷键是：`Ctrl + Alt + R`

4.  调用**任务**

使用你刚才定义的快捷键，如：`Ctrl + Alt + R`，你可以看到所有的任务列表，就包含你刚定义的内容，输入`git-fetch`，就能显示你刚才定义的任务。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a6102633a484a06b44e001118a511e7~tplv-k3u1fbpfcp-watermark.awebp?)

> 放心，只有第一次你需要输入任务名搜索，后续它都会推荐你最近使用过的任务。

选中任务，按下回车。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96cb66a36a834a45be0dad229355f563~tplv-k3u1fbpfcp-watermark.awebp?)  
VS Code 终端栏里显示以上内容，则表示你的任务被`成功执行`了。

动手二：配置`带参数`选择或输入的任务
-------------------

### 1. 在任务执行时选择`分支`

上面，我们已经成功设置了最简单的一任务，可以用来执行一些冗余的命令行，比如：  
`git pull base dev --rebase`  
但缺乏动态参数，也主动了它的使用场景不够灵活。  
以上面这条`git pull base dev --rebase`为例，如果你的项目有多个分支，而你需要用命令在多个分支之间切换的话，" 分别给`dev`和`release`分支创建任务 " 可实在是个太笨的办法了。  
此时，如果有一个下拉框，让我们选择分支名，该多好啊...  
嘿！  
**VS Code 任务**刚好有这方面的能力。  
把你刚才的 tasks.json 做一下调整，如下：

```
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "git-pull",
      "type": "shell",
      "command": "git",
      "args": [
        "pull",
        "base",
        "${input:branch}", // 变量，会在下面的inputs中搜寻名叫branch的id
        "--rebase"
      ],
      "group": "build",
      "problemMatcher": []
    },
  ],
  "inputs": [
    {
      "id": "branch", // 输入参数的id，与上面变量${input:branch}这个branch保持一致
      "type": "pickString",
      "options": [
        "dev",
        "release"
      ],
      "description": "请输入分支"
    }
  ],
}
复制代码

```

执行'git-pull'任务，你会发现 VS Code 顶部弹出如下对话框：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b887ca5c9c564671b41272905c2b0b37~tplv-k3u1fbpfcp-watermark.awebp?)  
选中你需要的分支，如:`release`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b48bacb2df5041428363115cf6f81961~tplv-k3u1fbpfcp-watermark.awebp?)  
**成功了！！**  
任务系统成功执行了你期望它拉取的分支。（虽然这个分支并不存在）

### 2. 在任务执行时输入参数

在上面 2.1 的例子里的 inputs 中加入一项：

```
    {
      "type": "promptString",
      "id": "branchName",
      "description": "input your branch name",
      "default": "release"
    }
复制代码

```

并且，将 tasks 里，代表`变量`的那一行`"${input:branch}"`，改成`"${input:branchName}"`。  
运行任务：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e29a5a3f6c545239ba4d2bdf9cbef45~tplv-k3u1fbpfcp-watermark.awebp?) 可以看到，输入框可以带默认值，并且可以`手动输入进行修改`。  
按下回车，**继续任务**。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b763220684e4bdea87bed23f11a6e1b~tplv-k3u1fbpfcp-watermark.awebp?)

动手三：复合式任务，完成任务的排列组合
-------------------

虽然上面完成了一些简单的任务配置，但当我们需要去完成**一系列**小任务时，就会显得非常**不方便**。

> 正如文章开头的例子，依次完成了 `stash push => pull => stash pop => push`四个操作。

在之前步骤的基础上，在配置文件的 tasks 中增加两项任务：

```
tasks: [
...，
    {
      "label": "git-push",
      "type": "shell",
      "command": "git",
      "args": [
        "pull",
        "origin",
        "${input:branchName}", // 变量
      ],
      "problemMatcher": []
    }，
    {
      "label": "git-pull-push",
      "type": "shell",
      "dependsOn": [ // 依赖的任务
          "git-pull",
          "git-push"
      ],
      "dependsOrder": "sequence", // 代表是依次执行，不设置会并行执行
      "problemMatcher": []
    }
    
复制代码

```

配置完成后，在任务中选中：`git-pull-push`并执行。  
在收入分支名时直接按下回车，使用默认的`release`作为分支名。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e432dd0b6f7455ba20b9d8e2006014a~tplv-k3u1fbpfcp-watermark.awebp?)

控制台内，已经依次执行了`git-pull`和`git-push`两个任务。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2332a879bf804c6c891d9aac60c0b71e~tplv-k3u1fbpfcp-watermark.awebp?)

更多能力
----

更多细节，还请参考官方网站：  
[官网地址](https://link.juejin.cn?target=https%3A%2F%2Fcode.visualstudio.com%2FDocs%2Feditor%2Ftasks "https://code.visualstudio.com/Docs/editor/tasks")

总结
--

通过以上例子，我们可以一窥 **VS Code 任务系统**的一角，感受到 VS Code 在**私人订制**上的巨大潜力。  
可以进行一些畅享，通过**任务系统**配合**代码生成脚本**完成半自动的开发等等~~  
_**快去配置你的 VS Code 任务配置吧！**_