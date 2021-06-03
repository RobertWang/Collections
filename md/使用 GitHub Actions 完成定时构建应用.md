> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844904046776565767)

> 写这篇文章是无意之间看到阮一峰老师发布的 GitHub Actions 入门教程里面介绍了 GitHub Actions 的一些概念，碰巧我之前用爬虫 + vuePress 构件了一个 typescript......

写这篇文章是无意之间看到阮一峰老师发布的 [GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)里面介绍了 GitHub Actions 的一些概念，碰巧我之前用爬虫 + vuePress 构件了一个 [typescript 的中文手册](https://bosens-china.github.io/Typescript-manual/)，下面就以每天定时构建这个应用为背景介绍如何使用 GitHub Actions 完成下面工作。

下面内容主要以实践为主。

思路
--

1.  编写 bash 上传脚本
2.  执行爬虫命令构建应用；
3.  定时执行脚本，完成上传
4.  出现错误发送邮件通知我；

### deploy.sh

```
#!/usr/bin/env sh

# 确保脚本抛出遇到的错误
set -e
# 下面是脚本命令，根据你的需求来选取。
npm run xxxx
# 进入生成的文件夹
cd docs/.vuepress/dist
git init
git add -A
git commit -m 'deploy'
# 强制推送到指定分支
git push -f git@github.com:bosens-China/Typescript-manual.git master:gh-pages
cd -
复制代码
```

脚本执行的功能很简单，执行构建命令，之后进入构建的文件夹创建 git 仓库之后推送，这里需要注意一点的就是我们会将任务放置在`GitHub Actions`上，**而 git 推送是需要权限的**，下面就来解决和这个问题。

### ssh-keygen

运行

```
ssh-keygen
复制代码
```

创建`id_rsa`和`id_rsa.pub`两个文件，分别对应私钥文件和公钥文件。

之后将`id_rsa.pub`中的公钥添加到 Github 对应仓库的 Deploye keys 中

![](https://user-gold-cdn.xitu.io/2020/1/13/16f9deb5aa175f0f?imageView2/0/w/1280/h/960/ignore-error/1)

再将`id_rsa`的私钥文件添加到 Github 对应仓库的 Secrets 中，这里将名称定义为`ACTION_DEPLOY_KEY`，你可以自行定义，目的就是为了让虚拟机拥有 git 仓库的权限

![](https://user-gold-cdn.xitu.io/2020/1/13/16f9deb52f7e5467?imageView2/0/w/1280/h/960/ignore-error/1)

OK，到这一步我们得准备工作就基本上完成了，下面就是对 yml 配置文件的编写了。

### deploy.yml

我们在根目录新建一个`.github`的文件夹在文件夹内新建一个文件`deploy.yml`

```
# 触发的事件
on:
  push:
    branches:
      - master
# 定时任务在utc的9点执行，换算北京时间需要 + 8也就是凌晨五点
  schedule:
    - cron: "0 21 * * *"

jobs:
  build:
    runs-on: ubuntu-18.04

    strategy:
      matrix:
        node-version: [12.x]

    steps:
      - uses: actions/checkout@v1
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: git Actions
        uses: srt32/git-actions@v0.0.3
      - name: Setup
        env:
          ACTION_DEPLOY_KEY: ${{ secrets.ACTION_DEPLOY_KEY }}
        run: |
          mkdir -p ~/.ssh/
          echo "$ACTION_DEPLOY_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.email "你的邮箱"
          git config --global user.name "你的名字"
          npm install
          npm run updata
复制代码
```

简单说下上面脚本干了什么事情。

1.  拉取源码（actions/checkout@v1）
2.  安装 node 和 git（actions/setup-node@v1 和 srt32/git-actions@v0.0.3）
3.  将我们的私匙放置在. ssh 目录下（Setup）
4.  执行`deploy.sh`脚本，上述的最后指定的 npm run updata 值就是`bash deploy.sh`

在上面我指定了 on 它是触发的事件，我在这里指定了每天定时和`master`分支提交的时候执行这段脚本；

`jobs`则是动作，动作可以有很多 build 就是一个动作，需要特别注意一点 **runs-on: ubuntu-18.04** 的值是指定虚拟机这个字段是必填的，不建议使用`windows`虚拟机。

至于上面的`uses`字段，这个是官方提供的模块，你可以这样理解，它可以方便我们的操作，你也可以自己编写脚本之后上传到 github 的市场上，更多的内容我推荐到[官方文档上查阅](https://github.com/features/actions)

### 执行错误邮件通知

因为上面说这个是爬虫的应用，为了知道错误我需要邮件及时通知我，这里就使用第三方`nodemailer`的模块为例。

```
function sendMail(e) {
  const mailTransport = nodemailer.createTransport({
    host: "smtp.qq.com",
    secureConnection: true,
    auth: {
      user: "xxx@qq.com",
      pass: "vbwjzecplhehibed"
    }
  });
  const options = {
    from: "xxxx@qq.com",
    to: "xxx@qq.com",
    subject: "typescript-book出现错误",
    text: e instanceof Error ? e.message : e
  };
  mailTransport.sendMail(options);
}
复制代码
```

稍微注意一下`auth`字段，有一个 pass 这个本来是密码不过如果直接把密码暴露出来可能会造成不安全，所以也支持授权码，如果你想了解更多可以[什么是授权码，它又是如何设置？](https://link.jianshu.com/?t=http%3A%2F%2Fservice.mail.qq.com%2Fcgi-bin%2Fhelp%3Fsubtype%3D1%26%26no%3D1001256%26%26id%3D28)点击了解。

最后你需要为邮箱开通`SMTP服务`，qq 邮箱直接点开设置查找就行了，其他邮箱也类似。 之后再调用这个函数就 OK 了。

### 查看效果

上面写完后我们的定时发布脚本就完成了，剩下的就是提交代码触发钩子来执行我们的任务，这里我 push`master`分支代码，打开 actions

![](https://user-gold-cdn.xitu.io/2020/1/13/16f9ded560ff8454?imageView2/0/w/1280/h/960/ignore-error/1)

右侧就是我们脚本的执行次数和状态了，√ 表示成功，你可以详细点开查看更多的细节，例如下面的错误，你可以展开查看报错的原因是什么，方便对错误定位和调试，这里不做展开了。

最后
--

这篇文章写完发现更多是分享我怎么完成的，希望对你有所帮助，如果喜欢请给一下 Star