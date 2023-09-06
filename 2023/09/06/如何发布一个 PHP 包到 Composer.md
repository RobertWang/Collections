> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KLRHCBE-9I9kyJ1WTph65Q)

> 假设您已经以某种方式设法制作了一个 PHP 包或 PHP 框架。那么如何把他共享到 composer 上呢？

假设您已经以某种方式设法制作了一个 PHP 包或 PHP 框架。那么如何把他共享到 composer 上呢？，本篇文件教您如何操作。

Composer 工作原理
=============

Composer 下面有一个名为 Packagist[1] 的 PHP 包存储库。当我们执行 composer require 时实际上就是从 Packagist 拉取代码到您的项目中，为了让用户 composer require 到您的代码，我们只需将代码上传到 Packagist 即可

创建一个 Packagist 帐户
=================

打开 Packgist 网址：https://packagist.org/  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfDfWtkd5rT0at65t3S63zdCziaMguOP3iazKu8DsIoEib1dQdycjzibcveIvm6H4eXlgI3r0KRcQaSBQ/640?wx_fmt=png)image.png

  
点击右上角的 “创建帐户”。这样您将被重定向到以下页面：  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfDfWtkd5rT0at65t3S63zdNDeehQXcEHp0joMRMYSc9PhkDC4MBICS8xFVfekPOVXr1YKqzARQiaQ/640?wx_fmt=png)image.png

  
填写好信息后单击 “**注册**”。检查您的邮箱是否收到注册验证的邮件，点击里面的验证链接执行验证。  
接下来输入您之前设置的凭据登录您的帐户。  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfDfWtkd5rT0at65t3S63zdkOkTHibWNMpxE69gL3n1KlgDrBK2icXBLzhMth7pYrLPp8bDUztMIKJA/640?wx_fmt=png)image.png

  
登录后，要将您的包提交给 Packagist 或 Composer，请单击右上角的 “提交”。进入提交页面  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfDfWtkd5rT0at65t3S63zdspxkib9B4tA5d8BcZvKl732KAz3v3VTyUOKdV8sEdiaKeoXGHYb7IEVw/640?wx_fmt=png)image.png

  
可以看到，提交之前我们需要先提交到 Git 或者 Svn 上

编写 Composer.json
================

在我们将包提交到 Github 之前，我们要先确保包的文档目录结构如下

```
your-package-name/
├── src/
│   ├── ... (Framework source files)
├── composer.json
└── README.md

```

**Composer.json** 的内容如下

```
{
  "name": "your-username/your-package-name",
  "description": "Description of your package or framework",
  "type": "library",
  "license": "MIT",
  "authors": [
    {
      "name": "Your Name",
      "email": "your@email.com"
    }
  ],
  "autoload": {
    "psr-4": {
      "YourNamespace\\Framework\\": "src/"
    }
  },
  "require": {},
  "require-dev": {},
  "minimum-stability": "dev"
}

```

现在将 项目文件推送到 github 并获取 URL。假设您的 github 仓库地址是：https://packagist.org/packages/submit  
现在我们跳转到 packagist 的的 submit 页面，先地址填入  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfDfWtkd5rT0at65t3S63zdsBEajuhRXIyib3OHo4Nf5iakp635Eo2bXSSchsRbHL2ICUKKPtwFy11w/640?wx_fmt=png)image.png

  
并输入您的源代码 URL。并单击 “Check”。  

![](https://mmbiz.qpic.cn/mmbiz_png/x0Iuy6awYQfDfWtkd5rT0at65t3S63zdyjhj8sbBDCQibQCLbEC39clzdyWicvh0c6ZUJfdrOicfnVcLWnnMDHAcg/640?wx_fmt=png)image.png

  
如果检测一切成功，它将变成 “Submit”。  
只需点击它即可！

测试 Composer require
===================

当我们的 PHP 包发布成功了之后，要获取框架，请打开终端或 CMD 并输入以下内容：

```
composer require your-username/your-package-name

```

如果成功拉下来代码，表示我们的包已经提交成功！

#### 引用链接

`[1]` Packagist: _https://packagist.org/_