> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0_lJv1d0e00O2YOGboomgg)

假设你入职了一家小公司，既没有内部自研的 DevOps 平台，也没有专业的运维。需要你自己去搭建多环境 CI/CD 自动化部署，你会怎么做？

你可能需要综合考虑成本、服务器选择、CI/CD 易用度、可扩展等方面，对于不怎么熟悉 CI/CD 相关的小伙伴来说有点难以选择。

这里先给大家介绍两个可以免费使用的 CICD 平台：**阿里云效** 和 **Github Actions**。通过给一个 vitepress 静态站点 和一个 koa.js + mongodb 接口服务搭建多环境、多版本自动化部署，来熟悉具体使用方法。（本文暂不介绍 gitlab、Jenkins、docker、k8s 等）

<table><thead><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">项目类型</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">项目地址</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">DevOps 平台</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">部署位置</th><th data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">多环境</th></tr></thead><tbody><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">vitepress 静态站点</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Github</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">阿里云效免费版</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">阿里云 OSS 对象存储</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">测试 / 正式</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">koa.js + mongodb 接口服务</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Github</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">阿里云效免费版</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">腾讯云服务器</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">测试 / 正式</td></tr><tr data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">vitepress 静态站点</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Github</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">Github Actions</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">腾讯云服务器</td><td data-style="border-color: rgb(204, 204, 204); min-width: 85px;">测试 / 正式</td></tr></tbody></table>

1. 自己造轮子
--------

之前我写过一篇 Vue + Node.js 从 0 到 1 实现自动化部署工具 ，只需要点击一个按钮，就能实现自动化部署。但他的缺点在于只能在一个项目内运行，不支持多项目同时部署构建。如下图

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQicJ0icGCbe9ianHMHjNAtwPPeDqjOiccqQpx3iaqYw1yOicVZxVSmmA7SIJA/640?wx_fmt=other)

虽然后来我又将 zuo-deploy[1] 升级成了 Linux 操作面板，让他拥有了管理脚本、多项目同时部署、多 socket 消息通道、nginx 配置修改 / 重启、https 证书上传等功能，但由于是仅依赖 Node.js fs 文件操作实现的这些功能，没有数据库，无法很好的支持当前主流 DevOps 平台都支持的流水线、多环境、制品管理、版本回退、快速扩容等功能，难以满足企业级项目 CICD 环境要求。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQTKW9xfcfAiaDQ7HHqR2EzcwXnxRazXyU0BppKPiaiatfuvicU1fHvvB9Cw/640?wx_fmt=other)

于是我开始思考，怎么依赖现有的 github、gitlab、阿里云、腾讯云等平台提供的服务，来搭建一套企业级的 CI/CD 环境，下面是我的一些尝试记录。

2. 阿里云效一键部署静态站点到 OSS
--------------------

阿里云 - 云效，阿里云企业级一站式 DevOps，可以免费使用（会限制人数、流水线数量等，个人项目够用了）。

OSS 是对象存储的意思，一般一个项目对应一个 Bucket (存储桶)，可以通过一个地址来访问里面的文件，配置成静态站点后，将自己的域名通过 CNAME 解析到该地址，项目就能访问了。

我的站点 f.zuo11.com 就是部署在上面的。

我为什么要选择将静态站点部署到阿里云 OSS？

1.  **不需要你自己有服务器，且费用很低**。
    
2.  我之前 1 核 2G 1M 带宽的服务器，下载速度只有 128kb/s，静态资源稍微大点，加载时间就会变长，**站点放到 OSS 访问速度很快**，感觉带宽不低。
    
3.  前后端部署服务位置分离，防止服务器被 ddos 攻击后，静态站点、接口服务全挂掉。
    

### 2.1 流水线与底层逻辑

CI/CD 一般需要找到流水线设置，常规前端项目自动化部署分为 3 个阶段

1.  设置 git 源
    
2.  构建（比如前端项目执行 npm run build）、此时可以选择分支，是否执行 npm install
    
3.  部署（使用构建生成的文件部署）
    

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQhrmT0p0j3TL1EW3atL9WVGQpBsEdsFgfRFUtESPq4k9mAWJyQMqibYg/640?wx_fmt=other)

以自动化部署 github 仓库 vitepress 项目到阿里云 oss 为例，流水线运行逻辑

```
# 克隆代码
# 使用工作路径 /root/workspace/dev-zuo-blog
# [INFO] 清理文件夹 /root/workspace /root/workspace/dev-zuo-blog...
git clone https://github.com/dev-zuo/blog.git  --branch main /root/workspace/dev-zuo-blog
# 正克隆到 '/root/workspace/dev-zuo-blog'...
# [INFO]  cd /root/workspace/dev-zuo-blog
# [SUCCESS] 克隆成功
# [INFO] 获取到上一次运行commit ID 2268a51489cae3588d4b091734ba6f9cd8612379
# [18:55:16] ###commitId###:xx ###commitMsg###:chore: update git note
# 获取最近几次提交commit让你知道部署的是最新代码

# [18:55:26] [INFO] 执行用户命令
cnpm install
npm run build
# build log
# [18:55:46] build complete in 8.50s.

# [INFO] 使用工作路径/root/workspace/dev-zuo-blog
ossutil cp -r /root/workspace/dev-zuo-blog/.vitepress/dist oss://f-zuo11-com/ --update
# [18:55:52] [SUCCESS]上传成功


```

### 2.2 vitepress + oss 自动化部署

> 创建流水线后，运行该流水线，即可自动拉取 github 仓库最新代码，执行构建，并更新到阿里云 OSS

在 阿里云 - 云效 工作台，新建项目（DevOps），切换到流水线 Tab，新建流水线，选择空模板，进入后选择 Node.js 技术栈，选择 部署到 oss

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQLQEiaLzjc9icbxJJqt2xPn9b9iaAKamE0iaI7MqdGsZI5h3ePic4KuTWpkw/640?wx_fmt=other)

配置构建流程（github 源建议选择香港，不然可能 git clone 可能拉取不下来）

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQy2RM5wRJJhjDKotnDyrup4JsvOYqLdSH1K3kXa6NgZ9eAw3OWnicbbg/640?wx_fmt=other)

设置构建命令、构建生成文件目录，目标 oss

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQ3BUdakUezE7ia1yXmiaR7k6VEovz2qY9mNayxjAfjibRKhuUD0OAMzk9w/640?wx_fmt=other)

创建完成后，**每当有新提交的代码，点击运行流水线，即可完成新功能的部署**，很方便。整个构建部署过程可以查看对应的日志，体验非常好。

### 2.3 多环境配置方式

关于 oss 多环境，我的做法是

1、创建两条流水线，一条测试环境使用、一条生产环境

2、创建两个 OSS Bucket(f-zuo11-com、f-test-zuo11-com)，分别对象两个环境，主分支用于正式环境，版本分支 (比如 v1.0.0) 用于测试环境

3、分别使用 f.zuo11.com、f-test.zuo11.com 解析到不同的 oss bucket。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQKInFGNlg6DLXT0w5wnqABF5iaHuQYVoXk4q4vBSe5D7slVlCjSaufiaw/640?wx_fmt=other)

这样就实现了静态站点多版本、多环境自动化部署了，可以基本满足日常要求。

另外在多版本规范化的开发流程中，有如下建议

1、 **正式环境（线上）只能部署主分支**，测试环境部署对应的版本分支。

2、 测试验证通过后，以提交 Pull Request 的方式合并代码（github/gitlab 页面操作，不要直接用工具和命令合并），这样可以通过 merge 历史记录进行整个版本代码的 code review。

3、新版本代码 merge 到主分支后，再部署到预发、测试后即可正式发布，每次上线后记得打版本 tag，也可通过脚本自动打 tag。

### 2.4 注意点

创建新的 OSS Bucket 时，需要进行必要的 3 步操作才能正常访问：

1、在概览中找到该 oss bucket 的域名，添加 CNAME 域名解析到该地址

2、在 bucket 配置 - 域名管理中，绑定对应的域名

3、在数据管理 - 静态页面中配置默认首页为 index.html，勾选开通子目录首页 (index)，这样才能访问目录时，自动解析到 index.html

3. 阿里云效一键部署 Node 接口服务到云服务器
--------------------------

Koa.js + MongoDB 接口服务，怎么做自动化部署？这种需要开启一个 http 服务，需要有一台云服务器。

由于我早先写过一个 config.zuo11.com 配置中心玩具项目。已经部署到腾讯云服务器，于是这次正好将这个后端接口服务 zuo-config-server \- github[2] 拿来测试自动化部署。

### 3.1 自动化部署接口服务

由于 koa.js 接口服务不需要构建，因此仅需要 git pull 最新代码，然后运行 pm2 命令重启服务即可。

阿里云效 - 新建流水线 - 空的即可，删除构建、仅添加部署

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQD4dX69ia1mgpA2lHU3pn2mPxpBib1DHpHoePM4klazhQuA6XMOZcc9FA/640?wx_fmt=other)

部署 - 选择主机部署

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQegMWIlKMUCrfhK5ks2y6wJf4C2y8wueryEdtoKiaaBELCiawKsB1I6Aw/640?wx_fmt=other)

我们这里部署到腾讯云服务器，需要选择自有主机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQrjkZqiaafwrYvpXkXzC5gpbZibG5ickGwK4rIv9anR3CXOpMdKRBeibicjg/640?wx_fmt=other)

复制下面的命令，登录到对应的服务器去执行该命令，即可完成安装，完成后，可添加的主机列表就会出现对应的服务器。就可以选择该主机进行部署了。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQNvt1BVQibgicrNRUicgiaPm0Du0K8krqZWTszh9fDXCcjjMKFdGsBIksPQ/640?wx_fmt=other)

这里的部署方式采用修改部署脚本的方式。点击运行流水线即可在目标服务器执行对应的部署命令。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQFTUwkl8IyVR6e2XexiaNrI7D2gSRKzGsaqwEmFxh3JuapDvFItayyxg/640?wx_fmt=other)

对应部署脚本：1、进入目录 2、拉取更新代码 3、npm install 4、pm2 重启接口服务 5、部署完成

修改代码，新增一个接口 feat: add test-deploy api[3]

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQzSTiafoEpoiaxTzAC2Ugxhazu8IzUiaGVjbuLdQD6MK95a5afVXkyhctw/640?wx_fmt=other)

然后运行该流水线进行自动化部署，再测试该 API 是否能正常访问 `https://config.zuo11.com/share/test-deploy`，经过测试，运行良好。

### 3.2 多环境配置方式

由于接口服务端口是一致的，因此这里使用另一台服务器来当做测试环境。云效正好有免费 5 小时体验主机。新建一条测试环境流水线，建立方式和 3.1 一致，但增加一个首次部署，用于初次 clone 代码（也可以直接在服务器直接执行）。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQbdfd45O4LGmQDWibWzhbAclayoMBKibT1gsYWibvw1TQTcja9SoIA2uicQ/640?wx_fmt=other)

首次部署

```
# 首次部署
cd /root
git clone git@github.com:zuoxiaobai/zuo-config-server.git --branch v1.0.0


```

二次部署的脚本和 3.1 部署脚本一致，脚本运行日志如下图

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQD53ZM4jCRckkYmf7NvpAfBXqajFbT16Kd0RXASceBOBHDHlX2SiaKkg/640?wx_fmt=other)

### 3.3 全新空服务器环境安装

对于一台阿里云全新的主机想要部署上面的接口服务，需要进行如下几步

1、至少要安装 git、node、nginx

```
yum install nodejs npm git nginx


```

2、git clone 失败，需要先在 linux 服务器上配置 ssh，在将公钥添加到 github 设置中，参考 git ssh 配置 (http://www.zuo11.com/blog/2020/8/git_ssh.html)

3、接口服务需要安装 pm2，npm install -g pm2

**这里就体现出 docker 的好处了，环境都是预设好的，不用自己逐一安装配置**

部署完成后，可以使用 `http://47.102.120.17:5000/share/test-deploy` 访问接口，但是想通过 `http://config-test.zuo11.com/share/test-deploy` 直接访问即可，就需要先配置域名解析，再修改 nginx 配置了。nginx 默认安装目录 /etc/nginx/，修改 nginx.conf，增加如下配置

```
server {
        listen   80;
        server_name  config-test.zuo11.com;
        charset  utf-8;
        location / {
            # root   html;
            # index  index.html index.htm;
            proxy_pass http://127.0.0.1:5000;
        }
}


```

以上就完成了接口服务的多环境多版本自动化部署，对应流水线

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQVa8hgwjguJT7d3UlUvia2fjzF6nHE2Sf7iaiazEU2BMdHPEccyezHjiafw/640?wx_fmt=other)

4. Github Actions 自动将静态站点部署到云服务器
--------------------------------

先来看一张图，通过 Github Actions 将 vitepress 项目部署到腾讯云服务器，该图由 https://bddxg.top/ 提供。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQUqaWe5pfjGGkorcTDic7PoqFIx1GaqB7g7n3YuibqRUjMJviblZ0KEAlA/640?wx_fmt=other)

### 4.1 actions 指定某个版本有 push 时自动执行部署

这里尝试将 `github.com/dev-zuo/blog` 中的 vitepress 项目部署到腾讯云服务器。首先在项目目录中添加 github ci 配置文件 `.github/workflows/main.yml`，内容如下，当 v1.0.0 分支有代码 push 时，就会自动触发 github actions 执行下面的 jobs 任务，触发部署流程：1、打包 2、构建 3、通过 ssh 登录到腾讯云服务器进行发布 4、微信通知（非必须）

```
name: 更新博客到服务器

on:
  push:
    # push 代码的时候 哪个分支会受到影响 这里是 v1.0.0 分支
    branches:
      - v1.0.0 # 也可以设置为 main 分支

# 推送之后执行一系列的任务
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # 获取代码
      - name: 迁出代码
        # 使用action库 action/checkout获取代码
        uses: actions/checkout@main
      # 安装Node环境

      - name: 安装node.js
        # 使用action库  actions/setup-node安装node
        uses: actions/setup-node@main
        with:
          node-version: lts/*

      # 安装依赖
      - name: 安装依赖
        run: npm install

      # 打包
      - name: 打包
        run: npm run build
      # 上传到腾讯云
      - name: 发布到腾讯云
        uses: easingthemes/ssh-deploy@main
        env:
          # 私钥
          SSH_PRIVATE_KEY: ${{ secrets.PRIVATE_KEY }}
          # SCP参数
          ARGS: '-avzr --delete'
          # 源目录
          SOURCE: '.vitepress/dist/'
          # 服务器ip
          REMOTE_HOST: ${{ secrets.REMOTE_TXHOST }}
          # 用户
          REMOTE_USER: 'root'
          # 目标地址
          # TARGET: '/var/www/html'
          TARGET: '/root/demo.zuo11.com'
      # 推送信息到微信
      # - name: 推送信息到微信
      #   uses: easychen/github-action-server-chan@main
      #   with:
      #     sendkey: ${{ secrets.SERVER_J }}
      #     title: '网站更新完毕'


```

注意点：文件路径要检查是否正确，包括构建生成路径，服务器目标部署路径

里面用到的 ssh-deploy[4] 部署很方便，不需要在目标服务器装什么，有 ip、私钥就行。这个源码大致看了下，node.js 操作，代码量不大，非常适合学习。

掘金有篇写 ssh 部署的应该和这里 github action 部署用到的技术一致。参考 从零开始 Node 实现前端自动化部署 [5]

### 4.2 什么是 ssh？

阮一峰博客里解释的很好，SSH 原理与运用（一）：远程登录 [6]

> 简单说，SSH 是一种网络协议，用于计算机之间的加密登录。
> 
> 最早的时候，互联网通信都是明文通信，一旦被截获，内容就暴露无疑。1995 年，芬兰学者 Tatu Ylonen 设计了 SSH 协议，将登录信息全部加密，成为互联网安全的一个基本解决方案，迅速在全世界获得推广，目前已经成为 Linux 系统的标准配置。
> 
> SSH 主要用于远程登录。假定你要以用户名 user，登录远程主机 host，只要一条简单命令就可以了。`ssh user@host` => `ssh root@24.32.122.12` => 使用 root 账号登录到 ip 为 xx 的服务器

对于不了解 ssh 登录基础的同学，建议先看完这篇文章，再看后续内容。

另外也可以参考：

*   What is SSH\(Secure Shell \)\?[7]
    
*   The story of the SSH port is 22[8]
    

### 4.3 github secrets 变量配置

```
 # 私钥
 SSH_PRIVATE_KEY: ${{ secrets.PRIVATE_KEY }}
 # 服务器ip
 REMOTE_HOST: ${{ secrets.REMOTE_TXHOST }}


```

在 actions 配置中 secrets 变量配置位置在：当前项目仓库的 Settings - Secrets and variables - Actions 中，如下图

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQbatkMGInfbrPyuiarGGNPh6OueEjicXVia3dOYNVicz2oJ637XED7O05KQ/640?wx_fmt=other)

关于 ssh 私钥方式登录需要进行如下配置

1、使用任意电脑运行 `ssh-keygen` 生成 ssh 公钥、私钥对。

2、查看私钥 `cat ~/.ssh/id_rsa` 内容，设置到 github secrets 变量 PRIVATE_KEY 中，**这里有个注意点，私钥拷贝后，-----END RSA PRIVATE KEY----- 下面一定要空一行**，不然会提示 ssh 权限校验失败。

```
uh4+88TM83SIN3fIn1Kjx3yZWhJef9rCyQ+POPps7u7tM/fMbYpN
-----END RSA PRIVATE KEY-----



```

3、查看公钥 `cat ~/.ssh/id_rsa.pub` 内容，将内容添加到云服务器的 `~/.ssh/authorized_keys` 文件中，可以使用 `vi ~/.ssh/authorized_keys` 编辑，然后将公钥粘贴进去。还有一种方式是使用前面阮一峰那篇 ssh 文章中的 `ssh-copy-id root@服务器ip` 来自动添加到云服务器的可信列表，但 windows 默认不支持 ssh-copy-id 命令，且对服务器设置有要求，建议还是使用第一种方式写入内容。

### 4.4 触发 push，自动执行

以上设置成功后，提交代码后 push 会自动执行 actions 部署，如下图

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQwvrUdPviczhNDWyibs9oiaib6viaJIpC7xbe74ibUWfUGKbbbkTPV6bJyU8g/640?wx_fmt=other)

点进去后可以看到具体详情，如果发生错误可以查看日志，获取点击 Jobs 从新运行。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQAtNOPic88icZjibhcc7Som9vQ5YQh6Rvz7Thn8cFibBqkFgRm5yXGvNJ6g/640?wx_fmt=other)

### 4.5 多环境配置方式

.github\workflows 目录下可以有多个 .yml 文件，不同过的 .yml 监听不同分支的 push，进行不同的部署逻辑，即可形成两套环境部署。如下图![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/mshqAkialV7EkCT7ibH1D0SwWibUbAdDReQiahNWnj2RNOMYmImyCYdibHokl77e3WKcgib1B4g4kosZ9XmAxlfkGkaw/640?wx_fmt=other)

```
name: 生产环境部署
on: # 监听 main 分支上的 push 事件
  push:
    branches:
      - main
jobs:
# 省略


```

```
name: 更新博客到服务器

on:
  push:
    # push 代码的时候 哪个分支会受到影响 这里是 main 主分支1
    branches:
      - v1.0.0


```

5. 结束语
------

以上是自动化部署相关测试研究第一阶段的笔记内容，如果有不同看法、或者好的建议欢迎评论区留言

最后，如果觉得本篇文章对你有帮助，点个赞再走吧，ღ(´･ᴗ･`) 比心。