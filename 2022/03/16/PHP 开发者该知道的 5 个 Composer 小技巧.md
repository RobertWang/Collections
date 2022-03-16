> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.phpcomposer.com](https://www.phpcomposer.com/5-features-to-know-about-composer-php/)

> Composer 中文网致力于推广 PHP 世界的包管理工具 Composer 在国内的普及以及独立开发并维护 Packagist 中国全量镜像系统。

PHP 开发者该知道的 5 个 Composer 小技巧
----------------------------

• 2014-09-10

Composer 是新一代的 PHP 依赖管理工具。其介绍和基本用法可以看这篇《[Composer PHP 依赖管理的新时代](https://www.phpcomposer.com/5-features-to-know-about-composer-php/composer-the-new-age-of-dependency-manager-for-php)》。本文介绍使用 Composer 的五个小技巧，希望能给你的 PHP 开发带来方便。

1. 仅更新单个库
---------

只想更新某个特定的库，不想更新它的所有依赖，很简单：

```
composer update foo/bar


```

此外，这个技巧还可以用来解决 “警告信息问题”。你一定见过这样的警告信息：

```
Warning: The lock file is not up to date with the latest changes in composer.json, you may be getting outdated dependencies, run update to update them.


```

擦，哪里出问题了？别惊慌！如果你编辑了`composer.json`，你应该会看到这样的信息。比如，如果你增加或更新了细节信息，比如库的描述、作者、更多参数，甚至仅仅增加了一个空格，都会改变文件的 md5sum。然后 Composer 就会警告你哈希值和`composer.lock`中记载的不同。

那么我们该怎么办呢？`update`命令可以更新 lock 文件，但是如果仅仅增加了一些描述，应该是不打算更新任何库。这种情况下，只需`update nothing`：

```
$ composer update nothing
Loading composer repositories with package information
Updating dependencies
Nothing to install or update
Writing lock file
Generating autoload files


```

这样一来，Composer 不会更新库，但是会更新`composer.lock`。注意`nothing`并不是`update`命令的关键字。只是没有`nothing` 这个包导致的结果。如果你输入`foobar`，结果也一样。

如果你用的 Composer 版本足够新，那么你可以直接使用`--lock`选项：

```
composer update --lock


```

2. 不编辑`composer.json`的情况下安装库
----------------------------

你可能会觉得每安装一个库都需要修改`composer.json`太麻烦，那么你可以直接使用`require`命令。

```
composer require "foo/bar:1.0.0"


```

这个方法也可以用来快速地新开一个项目。`init`命令有`--require`选项，可以自动编写`composer.json`：（注意我们使用`-n`，这样就不用回答问题）

```
$ composer init --require=foo/bar:1.0.0 -n
$ cat composer.json
{
    "require": {
        "foo/bar": "1.0.0"
    }
}


```

3. 派生很容易
--------

初始化的时候，你试过`create-project`命令么？

```
composer create-project doctrine/orm path 2.2.0


```

这会自动克隆仓库，并检出指定的版本。克隆库的时候用这个命令很方便，不需要搜寻原始的 URI 了。

4. 考虑缓存，`dist`包优先
-----------------

最近一年以来的 Composer 会自动存档你下载的`dist`包。默认设置下，`dist`包用于加了 tag 的版本，例如`"symfony/symfony": "v2.1.4"`，或者是通配符或版本区间，`"2.1.*"`或`">=2.2,<2.3-dev"`（如果你使用`stable`作为你的`minimum-stability`）。

dist 包也可以用于诸如`dev-master`之类的分支，Github 允许你下载某个 git 引用的压缩包。为了强制使用压缩包，而不是克隆源代码，你可以使用`install`和`update`的`--prefer-dist`选项。

下面是一个例子（我使用了`--profile`选项来显示执行时间）：

```
$ composer init --require="twig/twig:1.*" -n --profile
Memory usage: 3.94MB (peak: 4.08MB), time: 0s

$ composer install --profile
Loading composer repositories with package information
Installing dependencies
  - Installing twig/twig (v1.12.2)
    Downloading: 100%

Writing lock file
Generating autoload files
Memory usage: 10.13MB (peak: 12.65MB), time: 4.71s

$ rm -rf vendor

$ composer install --profile
Loading composer repositories with package information
Installing dependencies from lock file
  - Installing twig/twig (v1.12.2)
    Loading from cache

Generating autoload files
Memory usage: 4.96MB (peak: 5.57MB), time: 0.45s


```

这里，`twig/twig:1.12.2`的压缩包被保存在`~/.composer/cache/files/twig/twig/1.12.2.0-v1.12.2.zip`。重新安装包时直接使用。

5. 若要修改，源代码优先
-------------

当你需要修改库的时候，克隆源代码就比下载包方便了。你可以使用`--prefer-source`来强制选择克隆源代码。

```
composer update symfony/yaml --prefer-source


```

接下来你可以修改文件：

```
composer status -v
You have changes in the following dependencies:
/path/to/app/vendor/symfony/yaml/Symfony/Component/Yaml:
    M Dumper.php


```

当你试图更新一个修改过的库的时候，Composer 会提醒你，询问是否放弃修改：

```
$ composer update
Loading composer repositories with package information
Updating dependencies
  - Updating symfony/symfony v2.2.0 (v2.2.0- => v2.2.0)
    The package has modified files:
    M Dumper.php
    Discard changes [y,n,v,s,?]?


```

为生产环境作准备
--------

最后提醒一下，在部署代码到生产环境的时候，别忘了优化一下自动加载：

```
composer dump-autoload --optimize


```

安装包的时候可以同样使用`--optimize-autoloader`。不加这一选项，你可能会发现 [20% 到 25% 的性能损失](http://www.ricardclau.com/2013/03/apc-vs-zend-optimizer-benchmarks-with-symfony2/)。

如果你需要帮助，或者想要了解某个命令的细节，你可以阅读[官方文档](http://getcomposer.org/)或者[中文文档](http://docs.phpcomposer.com/)，也可以查看 JoliCode 做的这个[交互式备忘单](http://composer.json.jolicode.com/)。

* * *

原文地址：[5 features to know about Composer PHP](http://moquet.net/blog/5-features-about-composer-php/)  
译文地址：[PHP 开发者该知道的 5 个 Composer 小技巧](http://segmentfault.com/a/1190000000355928)