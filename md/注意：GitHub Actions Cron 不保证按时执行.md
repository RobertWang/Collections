> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/369780435)

GitHub Actions 支持定时执行，于是我用它定时拉取 Laravel 最新版，做成 Docker。结果发现没有按时执行，查资料发现：GitHub Actions 不保证按时执行，只是按时开始排队，根据平台资源的拥堵情况，可能等待几分钟或更久才被执行。

GitHub Actions 采用标准的 POSIX cron 语法：

[laravel-fans/laravel-docker](https://link.zhihu.com/?target=https%3A//github.com/laravel-fans/laravel-docker/blob/main/.github/workflows/laravel-6.yml)

```
name: Laravel 6
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  schedule:
    - cron: '*/5 * * * *'
```

解决办法都不太好：

1.  避开热门时间，比如：每小时 0 分、每天 0 点。
2.  Webhook 触发，因为 Webhook 相当于手动点击「立即执行」，非常可靠。但需要另一套 cron 来调用 Webhook，比较繁琐。

结论：不要用 GitHub Actions 做精确时间的任务，而用于不在乎晚几分钟的任务还是可以的。

结果效果挺好，Laravel 几天发布一个新版本，第二天就被做成了 [Docker](https://link.zhihu.com/?target=https%3A//hub.docker.com/r/laravelfans/laravel)。

![](https://pic1.zhimg.com/v2-9585605f99b0d6f38e159848b6ce4770_r.jpg)

参考资料：

GitHub 官方文档：[Events that trigger workflows](https://link.zhihu.com/?target=https%3A//docs.github.com/en/actions/reference/events-that-trigger-workflows%23scheduled-events)

> Note: The schedule event can be delayed during periods of high loads of GitHub Actions workflow runs. High load times include the start of every hour. To decrease the chance of delay, schedule your workflow to run at a different time of the hour.

Upptime 创始人的技术博客：

[GitHub Actions workflow not triggering at scheduled time | Upptime](https://link.zhihu.com/?target=https%3A//upptime.js.org/blog/2021/01/22/github-actions-schedule-not-working/)

GitHub 合作伙伴的回答：

[No assurance on scheduled jobs? - Code to Cloud / GitHub Actions - GitHub Support Community](https://link.zhihu.com/?target=https%3A//github.community/t/no-assurance-on-scheduled-jobs/133753)