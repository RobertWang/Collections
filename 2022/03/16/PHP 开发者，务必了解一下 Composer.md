> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1647401967&ver=3679&signature=4u0igkJFQ4YwgmdSCJ2Z79Ler4fyfihmN0p9WS9vweh8bLDQ7SGFheP8wxGt7cNtjvxDchuavPhADfgixgTOcEf6G3Of2Jrces0tqZ6E2RfQD4s9smihMa3urkF1y9HU&new=1)

> Composer 是一个非常流行的 PHP 包依赖管理工具

Composer 是一个非常流行的 PHP 包依赖管理工具, 已经取代 PEAR 包管理器, 对于 PHP 开发者来说掌握 Composer 是必须的.  

对于使用者来说 Composer 非常的简单, 通过简单的一条命令将需要的代码包下载到 vendor 目录下, 然后开发者就可以引入包并使用了.

其中的关键在于你项目定义的 composer.json, 可以定义项目需要依赖的包 (可能有多个), 而依赖的包可能又依赖其他的包 (这就是组件的好处), 这些都不用你烦心, Composer 会自动下载你需要的一切, 一切在于 composer.json 的定义.

Composer 对于使用者来说是很透明, 但是其背后的理念还是需要了解一下的, 其的诞生也不是偶然的, 得益于 Github 的快速发展, PHP 语言也越来越现代化, 显得更高大上了.

要了解这个，得先从历史开始说起…

#### PHP 最早加载类的方法

初学 PHP 时，最早会面对的问题之一就是 require 与 include 差别何在？require_once 与 include_once 又是什么？

弄懂这些问题之后，如果不使用 framework，直接开发，便常出现类似这样的 code：

```
// whatever.php
// 这档案需要用到几个类别
require  'xxx_class.php';
require  'yyy_class.php';
require  'zzz_class.php';
// ...


```

然后在其他档案会出现：

```
// another.php
// 这档案需要用到几个类别
require  'yyy_class.php';
require  'zzz_class.php';
// ...


```

这样的结果，会产生至少两个问题：

许多档案用到同样几个 class，于是在不同地方都需要载入一次。当类别多了起来，会显得很乱、忘记载入时还会出现 error。那么，不如试试一种懒惰的作法？

写一个 php，负责载入所有类别：

```
// load_everything.php
require  'xxx_class.php';
require  'yyy_class.php';
require  'zzz_class.php';
require  'aaa_class.php';
require  'bbb_class.php';
require  'ccc_class.php';


```

然后在其他档案都载入这支档案即可：

```
require  'load_everything.php'


```

结果新问题又来了：当类别很多的时候，随便一个 web page 都会载入一堆 code，占用大量内存，怎么办呢？

#### __autoload

为解决这个问题，PHP 5 开始提供__autoload 这种俗称 “magic method” 的函式。

当你要使用的类别 PHP 找不到时，它会将类别名称当成字串丢进这个函式，在 PHP 喷 error 投降之前，做最后的尝试：

```
// autoload.php
function  __autoload($classname)  {
    if  ($classname  ===  'xxx.php'){
        $filename  =  "./".  $classname  .".php";
        include_once($filename);
    }  else  if  ($classname  ===  'yyy.php'){
        $filename  =  "./other_library/".  $classname  .".php";
        include_once($filename);
    }  else  if  ($classname  ===  'zzz.php'){
        $filename  =  "./my_library/".  $classname  .".php";
        include_once($filename);
    }
    // blah
}


```

也因为 PHP 这种 “投降前最后一次尝试” 的行为，有时会让没注意到的人困惑“奇怪我的 code 怎么跑得动？我根本没有`require`啊..”，所以被称为 “magic method”。

如此一来，问题似乎解决了？

可惜还是有小缺点..，就是这个__autoload 函式内容会变得很巨大。以上面的例子来说，一下会去根目录找、一下会去`other_library`资料夹、一下会去`my_library`资料夹寻找。在整理档案的时候，显得有些混乱。

#### spl_autoload_register

于是 PHP 从 5.1.2 开始，多提供了一个函式。可以多写几个 autoload 函式，然后注册起来，效果跟直接使用__autoload 相同。现在可以针对不同用途的类别，分批`autoload`了。

```
spl_autoload_register('my_library_loader');
spl_autoload_register('other_library_loader');
spl_autoload_register('basic_loader');
function  my_library_loader($classname)  {
    $filename  =  "./my_library/".  $classname  .".php";
    include_once($filename);
}
function  other_library_loader($classname)  {
    $filename  =  "./other_library/".  $classname  .".php";
    include_once($filename);
}
function  basic_loader($classname)  {
    $filename  =  "./".  $classname  .".php";
    include_once($filename);
}

```

每个 loader 内容可以做很多变化。可以多写判断式让它更智慧、可以进行字串处理…。自动载入类别的问题终于解决了… 但是一大串一大串的 autoload，手动去写这些，很麻烦，这个时候就会想到能不能用一种工具直接去生成呢？这个时候就有了`composer`了。

#### Composer

建立一个 composer.json 档，里面输入这些：

```
{
    "autoload":  {
        "classmap":  [
            "my_library",
            "other_library"
        ]
    }
}


```

再来，在 terminal 输入 `composer install`

执行成功之后，你会看到一个 vendor 资料夹，内含一个`autoload.php`。没错，跟你梦想的一样。你只要载入这个档案：

```
require  'vendor/autoload.php';


```

你需要的所有类别，都会在适当的时候、以适当的方式自动载入。php 再也不会 error 说你 “类别尚未定义” 了！

这 vendor 资料夹里面的一切，都只是 php code 而已，并没有特别神奇的地方。只要去看`autoload.php`的原始码，就能知道`composer`到底写了哪些`php code`给你。

#### 最后提供一个扩展包下载，直接用 composer 就好了。

查询一下那几个套件在 “https://packagist.org/” 的名称、还有你需要的版本号。

把刚刚的`composer.json`改成这样：

```
{
    "require":  {
        "google/apiclient":  "1.0.*@beta",
        "guzzlehttp/guzzle":  "~4.0",
        "doctrine/dbal":  "~2.4"
    },
    "autoload":  {
        "classmap":  [
            "my_library"
        ]
    }
}

```

然后`composer install`指令除了自动载入你的类别之外、还会自动下载你需要的类别、然后自动载入它们。

一样`require vendor/autoload.php`就可以了。

composer 还有很多基本用法，请大家移步到 composer 文档学习！

链接：http://blog.startphp.cn/thread-165-1-1.html