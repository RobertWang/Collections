> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [learnku.com](https://learnku.com/php/t/62188)

> 你有没有想过在 PHP 中有 yield 的好处是什么？我有一些相关的要点想介绍给你，以节省 googling 的时间: 什么是 yield.yield 与 return 之间的差异. yield 的相关使用设置.......

[英文原文](https://medium.com/tech-tajawal/use-memory-gently-with-yield-in-php-7e62e2480b8d) / [翻译](https://learnku.com/php/c/translations) / 786 / 1 / 创建于 1 年前 / [3 个改进](https://learnku.com/php/t/62188/patches)

你有没有想过在 PHP 中有 **yield** 的好处是什么？我有一些相关的要点想介绍给你，以节省 googling 的时间:

1.  什么是 **_yield_**.
2.  **_yield_** 与 **_return_** 之间的差异.
3.  **_yield_** 的相关使用设置.
4.  结论.
5.  参考.

1. 什么是 **_yield_**[#](#61e61a)
------------------------------

> 生成器函数看起来和普通函数一样，只是生成器不返回值，而是生成所需的值。

看看以下示例：

```
function getValues() {
    yield 'value';
}// 打印字符串 "value"
echo getValues();

```

当然，这不是它的工作方式，前面的例子会给你一个致命的错误: `Object Of class Generator could not be convert to string`，让我们弄清楚:

2. **yield** 与 **return** 的不同[#](#02e8b5)
-----------------------------------------

前面的错误意味着 `getValues()` 函数没有按预期返回字符串，让我们检查它的类型：

```
function getValues() {
    return 'value';
}
var_dump(getValues()); // string(5) "value"
function getValues() {
    yield 'value';
}
var_dump(getValues()); // class Generator#1 (0) {}

```

[**Generator**](https://php.net/manual/en/class.generator.php) 类实现了 [**Iterator**](https://php.net/manual/en/class.iterator.php) 接口，这意味着您必须循环使用 `getValue()` 函数才能获取值：

```
foreach (getValues() as $value) {
   echo $value;
}
// using variable is also alright
$values = getValues();
foreach ($values as $value) {
   echo $value;
}

```

但这不是唯一的区别！

> 生成器允许您编写使用 `foreach` 的代码对一组数据进行迭代的代码，而无需在内存中构建数组，那样可能会导致超出内存限制。

在下面的例子里我们创建一个有 800,000 元素的数字同时从 `getValues()` 方法中返回他，同时在此期间，我们将使用函数 [`memory_get_usage()`](https://php.net/manual/en/function.memory-get-usage.php) 来获取分配给次脚本的内存， 我们将会每增加 200,000 个元素来获取一下内存使用量，这意味着我们将会提出四个检查点：

```
<?php
function getValues() {
   $valuesArray = [];
   // 获取初始内存使用量
   echo round(memory_get_usage() / 1024 / 1024, 2) . ' MB' . PHP_EOL;
   for ($i = 1; $i < 800000; $i++) {
      $valuesArray[] = $i;
      // 为了让我们能进行分析，所以我们测量一下内存使用量
      if (($i % 200000) == 0) {
         // 来 MB 为单位获取内存使用量
         echo round(memory_get_usage() / 1024 / 1024, 2) . ' MB'. PHP_EOL;
      }
   }
   return $valuesArray;
}
$myValues = getValues(); // 一旦我们调用函数将会在这里创建数组
foreach ($myValues as $value) {}

```

在前面的示例中 输出结果为内存消耗，脚本的输出结果如下

```
0.34 MB
8.35 MB
16.35 MB
32.35 MB

```

这意味着我们这几行脚本消耗了超过 30M 的内存，每次我们向 `$valuesArray` 数组添加元素，都会增加它在内存中的大小:

举个使用 **_yield_** 的例子：

```
<?php
function getValues() {
   // 获取初始内存使用
   echo round(memory_get_usage() / 1024 / 1024, 2) . ' MB' . PHP_EOL;
   for ($i = 1; $i < 800000; $i++) {
      yield $i;

      // 开始进行分析 测量内存占用
      if (($i % 200000) == 0) {
         // get memory usage in megabytes
         echo round(memory_get_usage() / 1024 / 1024, 2) . ' MB'. PHP_EOL;
      }
   }
}

$myValues = getValues(); // 遍历完毕`values`之前不进行任何操作
foreach ($myValues as $value) {} // 此处开始获取`values`

```

这个脚本的输出令人惊讶：

```
0.34 MB
0.34 MB
0.34 MB
0.34 MB

```

这不意味着你从 **_return_** 表达式迁移到 **_yield_**, 但如果你在应用中创建会导致服务器上内存出问题的巨大数组，则 **_yield_** 更加适合你的情况。

3. 什么是 “**_yield”_** 选项[#](#dce736)
-----------------------------------

这里有很多 **_yield_** 的选项，我需要强调其中的几个:

a. 使用 **_yield_**, 你也可以使用 **_return_**:

```
function getValues() {
   yield 'value';
   return 'returnValue';
}
$values = getValues();
foreach ($values as $value) {}
echo $values->getReturn(); // 'returnValue'

```

b. 返回键值对:

```
function getValues() {
   yield 'key' => 'value';
}
$values = getValues();
foreach ($values as $key => $value) {
   echo $key . ' => ' . $value;
}

```

点击 [这里](https://php.net/manual/en/language.generators.syntax.php) 查看更多.

4. 结论[#](#ee12ff)
-----------------

这个主题的主要原因是为了明确 **_yield_** 和 **_return_** 特别是在内存使用方面的区别，使用一些例子是因为我发现他对任何开发人员思考真的很重要。

5. 参考[#](#01238d)
-----------------

1.  [php.net/manual/en/language.generat...](https://php.net/manual/en/language.generators.syntax.php)
2.  [php.net/manual/en/class.generator....](https://php.net/manual/en/class.generator.php)
3.  [php.net/manual/en/language.generat...](https://php.net/manual/en/language.generators.php)
4.  [php.net/manual/en/function.memory-...](https://php.net/manual/en/function.memory-get-usage.php)

> 本文中的所有译文仅用于学习和交流目的，转载请务必注明文章译者、出处、和本文链接  
> 我们的翻译工作遵照 [CC 协议](https://learnku.com/docs/guide/cc4.0/6589)，如果我们的工作有侵犯到您的权益，请及时联系我们。

* * *

> 原文地址：[https://medium.com/tech-tajawal/use-memo...](https://medium.com/tech-tajawal/use-memory-gently-with-yield-in-php-7e62e2480b8d)
> 
> 译文地址：[https://learnku.com/php/t/62188](https://learnku.com/php/t/62188)