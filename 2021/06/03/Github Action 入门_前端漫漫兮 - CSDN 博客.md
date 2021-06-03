> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/hcz804933522/article/details/103998855)

github action
-------------

### 概念介绍

#### 能力介绍

*   支持分支 build, test, package, release, or deploy
*   支持 end-to-end continuous integration (CI) and continuous deployment [(CD)](https://help.github.com/en/articles/about-continuous-integration "(CD)")
*   支持在第三方云平台、github 平台、以及 [开发者自己的服务器] 构建(https://help.github.com/en/actions/automating-your-workflow-with-github-actions/about-self-hosted-runners)
*   支持的 action 有三类： 同 repository 下 action、public action repository 以及 github [action](https://help.github.com/en/articles/building-actions "action") market
*   支持[事件](https://help.github.com/en/articles/configuring-a-workflow "事件")触发[特定构建](https://help.github.com/en/articles/workflow-syntax-for-github-actions "特定构建")
*   支持邮件发送[运行状态通知](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/managing-a-workflow-run "运行状态通知")
*   仅 github 平台服务器存在限制

#### [术语介绍](https://help.github.com/en/github/automating-your-workflow-with-github-actions/core-concepts-for-github-actions "术语介绍")

*   action ： workflow 最小执行单元
*   Artifact ： workflow 运行，产生的中间文件，包括日志、测试结果等
*   Continuous integration (CI)：自动构建、测试
*   Continuous deployment (CD)： 自动部署
*   Event: 触发 workflow
*   GitHub-hosted runner：githut 平台的虚拟机
*   Job: workflow 的分解，可串行存在依赖；可并行
*   Runner： 运行环境
*   Self-hosted runner： 开发者自己的运新环境
*   Step: job 的分解

#### github 平台服务器限制

*   一个 repo，可最多同时开 20 个 workflows；超过 20，则排队等待
*   一个 workflow 下的每个 job，最多运行 6 小时；超过，直接结束
*   所有分支下的 job，根据 github 级别不同，限制不同；超过，进入队列等待。
*   1 小时内，最多 1000 次 请求，也就是 1.5api/1m
*   乱用 action，可能会被 github 限制账户哦

<table><tbody><tr><th>GitHub plan<br></th><th>Total concurrent jobs<br></th><th>Maximum concurrent macOS jobs</th></tr><tr><td>Free</td><td>20</td><td>5</td></tr><tr><td>Pro<br></td><td>40<br></td><td>5</td></tr><tr><td>Team</td><td>60</td><td>5</td></tr><tr><td>Enterprise<br></td><td>180<br></td><td>15</td></tr></tbody></table>

*   github 会生成相关使用[统计情况](https://github.com/settings/billing "统计情况")

### action yml 文件

#### 语法结构

```
  name: Greet Everyone
  # This workflow is triggered on pushes to the repository.
  on: [push]

  jobs:
    build:
      # Job name is Greeting
      name: Greeting
      # This job runs on Linux
      runs-on: ubuntu-latest
      steps:
        # This step uses GitHub's hello-world-javascript-action: https://github.com/actions/hello-world-javascript-action
        - name: Hello world
          uses: actions/hello-world-javascript-action@v1
          with:
            who-to-greet: 'Mona the Octocat'
          id: hello
        # This step prints an output (time) from the previous step's action.
        - name: Echo the greeting's time
          run: echo 'The time was ${{ steps.hello.outputs.time }}.'
```

#### 如何创建 yml 文件

*   .github/workflows 创建. yml or .yaml 后缀，名称随意
*   GITHUB_ 前缀的变量名，为 github 使用；否则会抛出错误
*   环境变量区分大小写
*   action secrets 必须放在环境变量或者作为输入，

#### step secret

```
  steps:
    - name: Hello world
      run: echo Hello world $FIRST_NAME $middle_name $Last_Name!
      env:
        FIRST_NAME: Mona
        middle_name: The
        Last_Name: Octocat
```

```
  steps:
  - name: My first action
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      FIRST_NAME: Mona
      LAST_NAME: Octocat
```

```
  steps:
  - name: Hello world action
    with: # Set the secret as an input
      super_secret: ${{ secrets.SuperSecret }}
    env: # Or as an environment variable
      super_secret: ${{ secrets.SuperSecret }}
```

jobsjob_idstepsenv)

#### [express 语法](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/contexts-and-expression-syntax-for-github-actions "express 语法")

### 使用技巧

#### step

```
   steps:
    - uses: actions/setup-node@74bc508 # Reference a specific commit
    - uses: actions/setup-node@v1      # Reference the major version of a release
    - uses: actions/setup-node@v1.2    # Reference a minor version of a release
    - uses: actions/setup-node@master  # Reference a branch
```

#### [触发时机](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions "触发时机")

```
  # push 
  name: descriptive-workflow-name
  on: push
```

```
  # 每小时
  on:
  schedule:
    - cron:  '0 * * * *'
```

```
  #特定分支、tag、路径
  on:
  push:
    branches:
      - master
    tags:
      - v1
    # file paths to consider in the event. Optional; defaults to all.
    paths:
      - 'test/*'
```

#### [github host 运行环境](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners "github host运行环境")

```
   runs-on: ubuntu-latest 

```

github 服务器采用如下统一配置:

```
    * 2-core CPU
    * 7 GB of RAM memory
    * 14 GB of SSD disk space
```

推荐使用 linux, 资源消耗存在不同比率。

```
    * linux：1，
    * window:2 
    * mac:10.
```

#### [self-host](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/using-self-hosted-runners-in-a-workflow "self-host")

```
   runs-on: [self-hosted, linux, ARM32]

```

##### 引用 action

*   public repo

```
   {owner}/{repo}@{ref} or {owner}/{repo}/{path}@{ref}. 

```

*   same repo

```
   {owner}/{repo}@{ref} or ./path/to/dir

   |-- hello-world (repository)
   |   |__ .github
   |       └── workflows
   |           └── my-first-workflow.yml
   |       └── actions
   |           |__ hello-world-action
   |               └── action.yml

   jobs:
   build:
    runs-on: ubuntu-latest
    steps:
      # This step checks out a copy of your repository.
      - uses: actions/checkout@v1
      # This step references the directory that contains the action.
      - uses: ../github/actions/hello-world-action
```

*   docker container

```
   docker://{image}:{tag}

```

#### [显示 workflow status](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/configuring-a-workflow "显示workflow status")

```
  ![](https://github.com/actions/hello-world/workflows/Greet Everyone/badge.svg)

```

#### [使用命令行](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions "使用命令行")

```
  jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: docker://gcr.io/cloud-builders/gradle
      - name: Install Dependencies
        run: npm install
        shell: bash
```

#### with 传参

```
  jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/hello_world@master
        with:
          first_name: Mona
          middle_name: The
          last_name: Octocat   
```

first_name , 会被转化为 INPUT_FIRST_NAME 使用

GITHUB_TOKEN 不满足，可以使用 [personal accesss_token](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/authenticating-with-the-github_token)

### [action 管理](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/managing-a-workflow-run "action 管理")

#### 读权限可以看日志

#### 写权限才可以停止 action

#### step debug

```
   ACTIONS_STEP_DEBUG = true

```

#### runner debug

```
  ACTIONS_RUNNER_DEBUG = true

```

### 参考文献

#### [trying-github-actions](https://glebbahmutov.com/blog/trying-github-actions/ "trying-github-actions")

#### [environment-variables](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/using-environment-variables "environment-variables")

欢迎关注我们【前端漫漫】，了解最新文章动态![](https://img-blog.csdnimg.cn/20200116090200439.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hjejgwNDkzMzUyMg==,size_16,color_FFFFFF,t_70)联系邮箱：simple2012hcz@126.com