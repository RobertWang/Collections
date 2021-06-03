> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xugaoyi.com](https://xugaoyi.com/pages/f44d2f9ad04ab8d3/#baidupush-sh%E6%89%A7%E8%A1%8C%E7%99%BE%E5%BA%A6%E6%8E%A8%E9%80%81%E5%91%BD%E4%BB%A4)

> web 前端技术博客, 简洁至上, 专注 web 前端学习与总结。JavaScript,js,ES6,TypeScript,vue,python,css3,html5,Node,git,github 等技术文章。

博客上线已经有些日子了，却发现百度迟迟没有收录我的博客页面，在百度推送工具当中除了有自动推送的代码嵌入网站之外，还有一个实时的主动推送更高效。

最近刚好了解到 GitHub Actions 的定时运行代码功能，可以用它来每天自动运行命令生成所有博客链接并进行一次性推送给百度。

GitHub Actions 是一个 CI/CD（持续集成 / 持续部署）工具，但也可用作代码运行环境。**功能非常强大，能够玩出许多花样。**

[#](#百度主动链接推送) 百度主动链接推送
-----------------------

链接主动推送在百度站长中有介绍，如图。

![](https://cdn.jsdelivr.net/gh/xugaoyi/image_store/blog/20200103124306.png)

具体使用方法就是创建一个文件`urls.txt`，文件内每行一条链接的格式写入提交的多个链接，如图。

![](https://cdn.jsdelivr.net/gh/xugaoyi/image_store/blog/20200103124305.png)

运行命令

```
curl -H 'Content-Type:text/plain' --data-binary @urls.txt "http://data.zz.baidu.com/urls?site=xugaoyi.com&token=T5PEAzhG*****"
```

1  

上面命令的地址和参数由百度站长提供。运行完命令会返回推送结果，不出意外的话就会把`urls.txt`内的所有链接一次性推送给百度。

这个方法虽然比嵌入网站头部的自动推送更高效，但是也有它的麻烦之处，就是得自己填入链接到`urls.txt`文件，然后手动运行命令。

[#](#自动生成urls-txt) 自动生成 urls.txt
--------------------------------

没关系，技术的本质就是让人 "偷懒" 的。于是，我写了一个 nodejs 工具，用于把所有的博客页面链接生成到`urls.txt`

```
// baiduPush.js

/**
 * 生成百度链接推送文件
 */
const fs = require('fs');
const path = require('path');
const logger = require('tracer').colorConsole();
const matter = require('gray-matter'); // FrontMatter解析器 https://github.com/jonschlinkert/gray-matter
const readFileList = require('./modules/readFileList');
const urlsRoot = path.join(__dirname, '..', 'urls.txt'); // 百度链接推送文件
const DOMAIN = process.argv.splice(2)[0]; // 获取命令行传入的参数

if (!DOMAIN) {
  logger.error('请在运行此文件时指定一个你要进行百度推送的域名参数，例：node utils/baiduPush.js https://xugaoyi.com')
  return
}

main();
function main() {
  fs.writeFileSync(urlsRoot, DOMAIN)
  const files = readFileList(); // 读取所有md文件数据

  files.forEach( file => {
    const { data } = matter(fs.readFileSync(file.filePath, 'utf8')); 

    if (data.permalink) {
      const link = `\r\n${DOMAIN}${data.permalink}/`;
      console.log(link)
      fs.appendFileSync(urlsRoot, link);
    }
  })
}
```

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  
23  
24  
25  
26  
27  
28  
29  
30  
31  
32  
33  

上面代码仅是针对我个人的博客生成链接到`urls.txt`文件。更多代码在 [这里 (opens new window)](https://github.com/xugaoyi/vuepress-theme-vdoing/blob/master/utils/baiduPush.js)。

运行如下命令就可以生产一个包含博客所有链接的`urls.txt`文件：

```
node utils/baiduPush.js https://xugaoyi.com
```

1  

哈哈，第一个麻烦解决了😏，接下来是解决第二个需要手动运行推送命令的问题。

> **如果你没办法自动生成，你也可以自己手动创建一个`urls.txt`文件，放到 github 仓库。**

[#](#github-actions-定时运行代码) GitHub Actions 定时运行代码
-------------------------------------------------

今天的主角 GitHub Actions 要登场了。（相关：[GitHub Actions 入门教程 (opens new window)](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html?20191227113947#comment-last)、[GitHub Actions 实现自动部署静态博客 (opens new window)](https://xugaoyi.com/pages/6b9d359ec5aa5019/)）

GitHub Actions 是一个 CI/CD（持续集成 / 持续部署）工具，但也可用作代码运行环境。**功能非常强大，能够玩出许多花样。**

### [#](#配置-github-actions) 配置 GitHub Actions

触发 GitHub Actions 需要在项目仓库新建一个`.github/workflows`子目录，里面是 [YAML 格式 (opens new window)](https://xugaoyi.com/pages/4e8444e2d534d14f/) 配置文件，文件名可以随便取。GitHub 只要发现配置文件，就会运行 Actions。

配置文件的第一部分是触发条件。

```
## baiduPush.yml
name: 'baiduPush'
 
on:
  push:
  schedule:
    - cron: '0 23 * * *'
```

1  
2  
3  
4  
5  
6  
7  

上面代码中，`name`字段是配置文件的描述，`on`字段是触发条件。我们指定两种情况下触发，第一种是代码 Push 进仓库，第二种是[定时任务 (opens new window)](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/events-that-trigger-workflows#scheduled-events-schedule)，每天在国际标准时间 23 点（北京时间 + 8，即早上 7 点）运行。

> 定时设置看[这里 (opens new window)](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/events-that-trigger-workflows#scheduled-events-schedule)

接着，就是运行流程。

```
jobs:
  bot:
    runs-on: ubuntu-latest # 运行环境为最新版的Ubuntu
    steps:
      - name: 'Checkout codes' # 步骤一，获取仓库代码
        uses: actions/checkout@v1
      - name: 'Run baiduPush.sh' # 步骤二，执行sh命令文件
        run: npm install && npm run baiduPush # 运行命令。（注意，运行目录是仓库根目录）
```

1  
2  
3  
4  
5  
6  
7  
8  

上面代码中，指定运行环境是最新的 ubuntu，流程的第一步是从代码仓库获取代码，第二步运行两个命令，先安装项目依赖，再运行写在`package.json`的`baiduPush`命令。完整代码看 [这里 (opens new window)](https://github.com/xugaoyi/vuepress-theme-vdoing/blob/master/.github/workflows/baiduPush.yml)

### [#](#baidupush命令在package-json配置) `baiduPush`命令在`package.json`配置

```
// package.json
"scripts": {
	"baiduPush": "node utils/baiduPush.js https://xugaoyi.com && bash baiduPush.sh"
}
```

1  
2  
3  
4  

上面脚本中在`node utils/baiduPush.js`的后面加入你的域名参数。运行此命令生成`urls.txt`文件，然后执行`baiduPush.sh`文件。

注意，在使用 window 系统时，请使用 git bash 命令窗运行上面的脚本。

> `baiduPush`命令之所以没有放在`baiduPush.yml`的 run 里面是因为我想在本地也可以执行`npm run baiduPush`命令。

### [#](#baidupush-sh执行百度推送命令) `baiduPush.sh`执行百度推送命令

`baiduPush.sh`文件：

```
#!/usr/bin/env sh

set -e

# 百度链接推送
curl -H 'Content-Type:text/plain' --data-binary @urls.txt "http://data.zz.baidu.com/urls?site=https://xugaoyi.com&token=T5PEAzhGa*****"

rm -rf urls.txt # 灭迹
```

1  
2  
3  
4  
5  
6  
7  
8  

上面代码中，把`urls.txt`文件中的所有链接一次性推送。

> baiduPush.sh 内的命令之所以没有写在`package.json`是因为我觉得命令太长了，不方便阅读。

写好配置，推送到仓库，就会在每天的早上 7 点钟，自动运行命令生成一个包含博客所有页面链接的`urls.txt`文件，并把所有链接一次性推送到百度。麻麻再也不用担心我的网站不被收录~~😘 😘 😘

在这个基础上可以扩展，使用 GitHub Actions 满足你自己的各种定时需求。

[#](#相关文章) 相关文章
---------------

[《 GitHub Actions 实现自动部署静态博客》 (opens new window)](https://xugaoyi.com/pages/6b9d359ec5aa5019/)

[《解决百度无法收录搭建在 GitHub 上的静态博客的问题》 (opens new window)](https://xugaoyi.com/pages/41f87d890d0a02af/)