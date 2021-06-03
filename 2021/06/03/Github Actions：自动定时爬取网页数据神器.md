> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1622735325&ver=3108&signature=dqv36uFleBJ6v1EEAo5OK41IHqRSYiXhLqTLqhGj-2J3kmrW47alFwiXqUNR2h284Orslkcbxn*bRPuJpNQlJ6DuPUF8ghMvxVkYTo3rnOIvL4GsJPYPXfEK5CIxOut7&new=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/KTBIicBlcWJESh7lvPX1UtOCb7aJHRYHbrv7nfOzvCx85zFU1I3TrMqnmDelpUenRssFWCBd8ssiaeazYibm2Jd3g/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/KTBIicBlcWJGn2yW5WIZKsfucibS77GdwwevR5o5kxj6xoty4BvKO055V57jnJpbuzUlcDhzetnksgUedDeEmM4Q/640?wx_fmt=jpeg)

GitHub Actions 是 GitHub 的持续集成服务，于 2018 年 10 月推出。它具有抓取代码、运行测试、登录远程服务器，发布到第三方服务等等，GitHub 把这些操作（**push, pull, submit a pull request, or write an issue**）就称为 actions。

在日常的学习和工作中，爬取数据与信息提取的需求日益增多。当你看到身边同学 / 同事自己动手编写爬虫代码节省了大量时间时，难免心生羡慕。大数据时代下的数据信息往往是动态的，（**如按分钟、小时、天更新的****天气数据、股票日涨幅****数据以及遥感影像数据等等**），常规的爬虫难以满足需求，定时爬取显得尤为重要。定时爬虫一般方法是将写好的爬虫代码挂在服务器，设定代码定时运行时间即可实现定时爬取，但每年购买服务器是笔不小的开销，而 GitHub Actions 绝对是一个不错的选择（“**免费服务器**”）。

下文以使用 R 语言**定时爬取印度股票交易的日涨幅数据**为例，介绍 GitHub Actions 的应用，其详细配置指南可观看以下视频，视频中代码见 https://github.com/amrrs/scrape-automation。

GitHub Actions 的配置文件叫做 workflow 文件，存放在代码仓库的. github/workflows 目录。workflow 文件采用 YAML 格式，文件名可以任意取，但是后缀名统一为. yml，比如 main.yml。一个库可以有多个 workflow 文件。GitHub 只要发现. github/workflows 目录里面有. yml 文件，就会自动运行该文件。

**GitHub Actions 基本术语****：**

（1）**workflow** （工作流程）：持续集成一次运行的过程，就是一个 workflow。

（2）**job** （任务）：一个 workflow 由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。

（3）**step**（步骤）：每个 job 由多个 step 构成，一步步完成。

（4）**action** （动作）：每个 step 可以依次执行一个或多个命令（action）。

**1.** **name** **字段是 workflow 的名称。如果省略该字段，默认为当前 workflow 的文件名。**

```
# Hourly scraping
name: nifty50scrape
```

**2.** **on** **字段指定触发 workflow 的条件，通常是某些事件。**

**设置定时爬取的时间****（Runs at 13:00 UTC every day）**

```
# Controls when the action will run.
on:
  schedule:
    - cron:  '0 13 * * *'
# Runs at 13:00 UTC every day
```

**3. workflow 文件的主体是 jobs 字段，表示要执行的一项或多项任务。**  

**runs-on** **字段指定运行所需要的虚拟机环境**

```
jobs: 
  autoscrape:
    # The type of runner that the job will run on
    runs-on: macos-latest
```

**4. 安装 r 语言以及爬虫所需的 r 包**  

```
# Load repo and install R
    steps:
    - uses: actions/checkout@master
    - uses: r-lib/actions/setup-r@master
    # Set-up R
    - name: Install packages
      run: |
        R -e 'install.packages("tidyverse")'
        R -e 'install.packages("janitor")'
        R -e 'install.packages("rvest")'
```

**5. 运行爬虫代码。**  

```
# Run R script
    - name: Scrape
      run: Rscript nifty50_scraping.R
```

**6. 将爬取的数据定时推送至项目仓库 data 文件中。**  

```
# Add new files in data folder, commit along with other modified files, push
    - name: Commit files
      run: |
        git config --local user.name actions-user
        git config --local user.email "actions@github.com"
        git add data/* 
        git commit -am "GH ACTION Headlines $(date)"
        git push origin main
      env:
        REPO_KEY: ${{secrets.GITHUB_TOKEN}}
        username: github-actions
```

完整的 main.yml

```
# Hourly scraping
name: nifty50scrape
# Controls when the action will run.
on:
  schedule:
    - cron:  '0 13 * * *'
jobs: 
  autoscrape:
    # The type of runner that the job will run on
    runs-on: macos-latest
    # Load repo and install R
    steps:
    - uses: actions/checkout@master
    - uses: r-lib/actions/setup-r@master
    # Set-up R
    - name: Install packages
      run: |
        R -e 'install.packages("tidyverse")'
        R -e 'install.packages("janitor")'
        R -e 'install.packages("rvest")'
    # Run R script
    - name: Scrape
      run: Rscript nifty50_scraping.R
 # Add new files in data folder, commit along with other modified files, push
    - name: Commit files
      run: |
        git config --local user.name actions-user
        git config --local user.email "actions@github.com"
        git add data/*
        git commit -am "GH ACTION Headlines $(date)"
        git push origin main
      env:
        REPO_KEY: ${{secrets.GITHUB_TOKEN}}
        username: github-actions
```

**根据特定网页编写特定爬虫代码** **nifty50_scraping.R**

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/KTBIicBlcWJGn2yW5WIZKsfucibS77GdwwCFSmPZicwuOLDJrVicDG2Jf5hpMnjJzXgfvpH7GsmV0cXicvDvibPYqByQ/640?wx_fmt=jpeg)

https://www.moneycontrol.com/stocks/marketstats/nsegainer/index.php

![](https://mmbiz.qpic.cn/sz_mmbiz_png/KTBIicBlcWJGn2yW5WIZKsfucibS77GdwwRI3rl5o1AYGQW6BHrTYib82CAqpsDic9HKoiaJVd1NNXUOdEPukwxd5xA/640?wx_fmt=png)

https://github.com/amrrs/scrape-automation

+++  

**参考资料**

*   https://www.r-bloggers.com/2021/03/daily-stock-gainers-automated-web-scraping-in-r-with-github-actions/
    
*   https://github.com/amrrs/scrape-automation
    
*   https://ropenscilabs.github.io/actions_sandbox/
    
*   http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html
    

+++

  

  

  

  

**END**

**编译**｜丁刘勇  

**单位**｜云南大学国际河流与生态安全研究院

**欢迎转载**｜**欢迎投稿**

**合作事宜**：chzhding@ynu.edu.cn

**聚焦鱼类生态学与水生态领域**

**搭建一个科研信息分享、交流的平台**

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/KTBIicBlcWJHzd2yFNTBqjrgd96QiaG1CQ2zROqzUicZOm3L1ob4ZgcfL6rpl54xIRkVac86YhyBUsRqIGHj3orxg/640?wx_fmt=jpeg)