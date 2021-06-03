> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwNjMxMTA5Mw==&mid=2651346664&idx=1&sn=9d4b81505a2078479164dd70a1bc1304&chksm=80f3ab32b7842224ccba739fc5f3765b132e805df56b26584a0c46a3a6a0bfe24ddd4ff92836&mpshare=1&scene=1&srcid=0603v1LVbknacr3hCEMFOG3d&sharer_sharetime=1622696169115&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

↓推荐关注↓

公众号

前言
--

在一次偶然查看 PHP 文档的时候，发现了一些有趣的内容，随着阅读的增加，越发觉得有趣的内容或者说时坑越来越多，所以我决定记录下来，分享出去，下文中一些内容摘录自一些优秀的博客、PHP 文档的用户笔记，或者文档原文。

尤其是文档原文，我发现很多人不会去读，很多东西也不会去注意（是的，我也是这样，所以借着这次机会，一起来学习一下。）

我忘了 PHP 函数的参数顺序，它们是随机的吗？
------------------------

PHP is a glue that brings together hundreds of external libraries, so sometimes this gets messy. However, a simple rule of thumb is as follows:

Array functionparameters are ordered as "needle, haystack" whereas String functionsare the opposite, so "haystack, needle".

**译：数组相关方法的参数顺序为，「needle, haystack」，字符串相关方法则是相反的 「haystack, needle」,**

来源: https://www.php.net/manual/zh/faq.using.php#faq.using.parameterorder

我应该如何保存 “盐”？
------------

当使用 password_hash() 或者 crypt() 函数时， “盐” 会被作为生成的散列值的一部分返回。你可以直接把完整的返回值存储到数据库中， 因为这个返回值中已经包含了足够的信息， 可以直接用在 password_verify() 或 crypt() 函数来进行密码验证。

下图展示了 crypt() 或 password_hash() 函数返回值的结构。如你所见，算法的信息以及 “盐” 都已经包含在返回值中， 在后续的密码验证中将会用到这些信息。

![](https://mmbiz.qpic.cn/mmbiz_png/GicBlLc5goGWAMqMicpibnIkibDNMVVrj0HpVnhrRRma3bjm55XRML6LibZOicod3jEa8MVRYO0PibYJr7PG2415aQyqQ/640?wx_fmt=png)

来源: https://www.php.net/manual/zh/faq.passwords.php#faq.password.storing-salts  

下面代码怎么没有分成两行显示？
---------------

在 PHP 中，一段代码的结束标记要么是 “?>” 要么是“?>\n”（\n 表示换行）。因此在上面的例子中，输出的句子将显示在同一行中，因为 PHP 忽略了代码结束标记后面的换行。这意味着如果要输出一个换行符，需要在每段 PHP 代码的结束标记后面多加一个换行。

PHP 为什么这么做呢？因为在格式化正常的 HTML 时，这样通常会更容易。假如输出了换行而你不需要这个换行时，就不得不用一个非常长的行来达到这样的效果，或者让产生的 HTML 页面的源文件的格式很难读。

来源: https://www.php.net/manual/zh/faq.using.php#faq.using.newlines

字符串连接操作符的优先级问题
--------------

如果你运行下面的代码，他将会输出一个`警告`和结果 `3` ，因为字符串连接操作符 `.` 和 数学运算符 `+` 、 `-` 的优先级时一样的，它们将从左往右执行。`Result:` 会被强转成数组 `0` 。

如果你在低版本的 PHP 中运行，会告诉你 中边不是一个数字，如果你在 7.4 中运行，会告诉你，在 PHP 8 中 `+` 、 `-` 的优先级将会被提高。如果你使用了 PHPSTORM 中的 EA 插件，将会提醒你这个问题。

```
<php
$var = 3;

echo "Result: " . $var + 3;


```

如果你不希望这样，那么最好使用括号把它包裹起来，就像下面那样。

```
<?php
$var = 3;

echo "Result: " . ($var + 3);


```

来源: https://www.php.net/manual/zh/language.operators.string.php#41950

字符串连接操作符与数字
-----------

运行下面代码，尤其是第三行，请注意，如果 `.` 左右存在空格，那么即使是一个数字，也将会作用成字符串连接。

```
<?php

echo "thr"."ee";           //prints the string "three"
echo "twe" . "lve";        //prints the string "twelve"
echo 1 . 2;                //prints the string "12"
echo 1.2;                  //prints the number 1.2
echo 1+2;                  //prints the number 3


```

来源: https://www.php.net/manual/zh/language.operators.string.php#41950

使用 http_build_query
-------------------

### NULL 的值将会被会略

```
<?php
$arr = array('test' => null, 'test2' => 1);

// test2=1
echo http_build_query($arr); 


```

来源: https://www.php.net/manual/zh/function.http-build-query.php#60523

### True 和 False 将会被转换成数字

```
<?php
$a = [teste1= true,teste2=false];
// teste1=1&teste2=0
echo http_build_query($a)


```

来源: https://www.php.net/manual/zh/function.http-build-query.php#122232

### 空的数组不会出现在结果中

```
<?php

$post_data = array('name'=>'miller', 'address'=>array('address_lines'=>array()), 'age'=>23);
// name=miller&age=23
echo http_build_query($post_data);


```

来源: https://www.php.net/manual/zh/function.http-build-query.php#109466

简述 OpCache 的原理
--------------

PHP 执行这段代码会经过如下 4 个步骤 (确切的来说，应该是 PHP 的语言引擎 Zend)

*   1. Scanning(Lexing) , 将 PHP 代码转换为语言片段 (Tokens)
    
*   2. Parsing, 将 Tokens 转换成简单而有意义的表达式
    
*   3. Compilation, 将表达式编译成 Opocdes
    
*   4. Execution, 顺次执行 Opcodes，每次一条，从而实现 PHP 脚本的功能。
    

现在有的 Cache 比如 APC, 可以使得 PHP 缓存住 Opcodes，这样，每次有请求来临的时候，就不需要重复执行前面 3 步，从而能大幅的提高 PHP 的执行速度。

来源: https://www.laruence.com/2008/06/18/221.html

var_dump(1...9) 输出什么?
---------------------

```
<?php

// 10.9
var_dump(1...9);


```

输出 10.9, 乍一看这个 var_dump 的输出很奇怪是不是？为什么呢？

这里教大家，如果看到一段 PHP 代码感觉输出很奇怪，第一反应是看下这段代码生成的 opcodes 是啥，虽然这个问题其实是词法分析阶段的问题，不过还是用 phpdbg 分析下吧 (一般为了防止 opcache 的影响，会传递 - n):

```
phpdbg -n -p /tmp/1.php
function name: (null)
L1-35 {main}() /tmp/1.php - 0x7f56d1a63460 + 4 ops
L2 #0 INIT_FCALL<1> 96 "var_dump"
L2 #1 SEND_VAL "10.9" 1
L2 #2 DO_ICALL
L35 #3 RETURN<-1> 1


```

所以这么看来，早在生成 opcode 之前，1...9 就变成了常量 10.9，考虑到这是字面量，我们现在去看看 zend_language_scanner.l, 找到这么一行:

```
DNUM ({LNUM}?"."{LNUM})|({LNUM}"."{LNUM}?)


```

这个是词法分析定义的浮点数的格式，到这里也就恍然大悟了：1...9 会被依次接受为: 1. (浮点数 1), 然后是 . (字符串连接符号） 然后是. 9(浮点数 0.9)

所以在编译阶段就会直接生成 “1” . “0.9” -> 字符串的字面量”10.9”

来源: https://www.laruence.com/2020/02/23/1990.html

HTTPOXY 漏洞
----------

这里有一个核心的背景是， 长久一来我们习惯了使用一个名为 "http_proxy" 的环境变量来设置我们的请求代理。

```
http_proxy=127.0.0.1:9999 wget http://www.laruence.com/


```

### 如何形成？

在 CGI(RFC 3875) 的模式的时候， 会把请求中的 Header， 加上 HTTP_ 前缀， 注册为环境变量, 所以如果你在 Header 中发送一个 Proxy:xxxxxx, 那么 PHP 就会把他注册为 HTTP_PROXY 环境变量， 于是 getenv("HTTP_PROXY") 就变成可被控制的了. 那么如果你的所有类似的请求， 都会被代理到攻击者想要的地址，之后攻击者就可以伪造，监听，篡改你的请求了

### 如何影响？

所以， 这个漏洞要影响你， 有几个核心前提是:

*   你的服务会对外请求资源
    
*   你的服务使用了 HTTP_PROXY(大写的) 环境变量来代理你的请求（可能是你自己写，或是使用一些有缺陷的类库）
    
*   你的服务跑在 PHP 的 CGI 模式下 (cgi, php-fpm)
    

### 如何处理？

以 Nginx 为例, 在配置中加入:

```
fastcgi_param HTTP_PROXY "";


```

所以建议, 即使你不受此次漏洞影响, 也应该加入这个配置.  
而如果你是一个类库的作者，或者你因为什么原因没有办法修改服务配置, 那么你就需要在代码中加入对 sapi 的判断， 除非是 cli 模式， 否则永远不要相信 http_proxy 环境变量，

```
<?php
if (php_sapi_name() == 'cli' && getenv('HTTP_PROXY')) {
  //只有CLI模式下, HTTP_PROXY环境变量才是可控的
}


```

补充: 从 PHP5.5.38 开始, getenv 增加了第二个参数, local_only = false, 如果这个参数为 true, 则只会从系统本地的环境变量表中获取, 从而修复这个问题, 并且默认的 PHP 将拦截 HTTP_PROXY:fix

HTTPOXY 漏洞说明 - 风雪之隅 https://www.laruence.com/2016/07/19/3101.html

运算符优先级
------

### && 和 and 在赋值运算中的问题

运行下面的代码，第一个 `$bool` 将打印为 `false` ，预期如此，但是第二个 `$bool` 将打印 `true` 。这是因为 `=` 的优先级高于 `and` 运算符，所以，第二个 `$bool` 将会被当成 `($bool = true) and false` 执行。

```
<?php

$bool = true && false;
// false
var_dump($bool);


$bool = true and false;
// true
var_dump($bool);


```

来源: https://www.php.net/manual/zh/language.operators.precedence.php#117390

### instanceof 运算符

你是否曾经写过下面这样的代码？

```
<?php

class A {

}

$A = new A();

var_dump((! $A instanceof A));

// 其实不用担心，因为 instanceof 的优先级要高于 ! ,你可以放心的使用，
// 不必添加括号，让他们看起来是一个表达式，但是在复杂的情况下例外。
var_dump(! $A instanceof A);


```

在你需要对 `instanceof` 运算的结果做取反运算时，因为取反运算符 `!` 的优先级低于 `instanceof` 所以，你不必再它们外面再加上一个圆括号来表明这是一组表达式，但是再复杂情况下例外。

array_map 的有趣用法
---------------

通常，我会使用 `array_map` 来处理一个数组，让他返回一个新的数组，当然，它的用处就是这样的，但是除了这种基础的用法，它其实还有一些有趣的用法，并且，这些用法都存在于 PHP 的手册中。

### 多个 array 用法

通常你会这样使用它。

```
<?php

$arr1 = ['apple', 'orange', 'mango', 'pear'];
$newArr1 = array_map('strtoupper',$arr1);


```

这只是一个简单的🌰，它会把所有的值转为大写的。那么看看下面的用法，猜猜会打印什么？

```
<?php

$arr1 = ['apple', 'orange', 'mango', 'pear'];
$arr2 = ['cauliflower', 'lettuce', 'kale', 'romaine'];

function map1($a, $b){
    var_dump($a, $b);
  // apple   cauliflower
  // orange  lettuce
  // mango   kale
  // pear    romaine
}

array_map('map1', $arr1, $arr2);


```

如上 `map1` 方法所示，将会顺序遍历 `$arr1` , `$arr2` 中的值，并且传递给 `map1` ，根据手册所定义：**如果多个数组的长度不一，即短的数组将会被填充空，至长的数组一样** 。

原生函数使用不当的话会比你想象的要慢
------------------

array_unique、array_merge 等，如果使用方法不正确，会比你想想的要慢，甚至是慢很多，远不如 foreach。

在下面这个回答中，列举了 PHP 中一些 array_* 方法的时间复杂度 performance - List of Big-O for PHP functions - Stack Overflow

小心代码中的比较
--------

下面的比较将会返回 `true`，是不是不敢相信？

因为两个 md5 值都有开始'0e'，所以 PHP 类型理解这些字符串是科学符号。根据定义，0 的任何次方都是 0，所以在这里会成立‎，所以当你确定一个变量的类型时，你最好使用 `===`(恒等于) 进行比较。

```
<?php

$a = md5('240610708');// 0e462097431906509019562988736854
$b = md5('QNKCDZO'); // 0e830400451993494058024219903391

var_dump($a == $b); // true


```

注意，当你在考虑使用 md5 存储密码时，你应该放弃这个想法，应该改用为 password_hash 系列方法。

来自：https://www.php.net/manual/zh/function.md5.php#123392

禁用 PHP 中不安全的 eval 方法
--------------------

众所周知， 在 php 中，eval 方法可以执行任意 PHP 代码，如果没有做好处理，被用户利用了， 就有可能会造成安全漏洞，所以最好想办法禁用它，谈到禁用 php 函数，你应该想到了 php.ini 中的 `disable_functions`参数，可以用来禁用 PHP 函数，一些集成环境中也会禁用一些高风险函数来降低风险。

但是，这个配置项，却禁用不了 eval 函数，因为根据官方文档的定义， eval 不是一个函数，他如同 echo 、这些特殊方法一样，他是一个语法结构，所以不能使用 `disable_functions`进行禁用，除此之外，还有 require、list、array、print、unset、include、isset、empty 、die、exit 等，这些都是语法结构，不是函数，如果你使用 `function_exists`判断，他们都会返回 `false`

如果你真的需要禁用 eval ，你得安装一些第三方扩展来实现，比如 mk-j/PHP_diseval_extension

参考：https://www.php.net/manual/zh/functions.variable-functions.php#functions.variable-functions

将任意类型转换为 null
-------------

听起来没什么用但是你确实可以这样做。

```
<?php

$a = 'Hi!';
// 在 PHP 7.2 以下，这行代码会返回 null，7.2 ~ 7.4 会返回 NULL，但是会提示被遗弃，
// 8.0 开始，将不再支持
var_dump((unset)$a);
var_dump($a);


```

除此之外，你还可以用 settype 函数 参考：https://www.php.net/settype

参考：https://www.php.net/manual/zh/function.unset.php#example-5601

isset 和 unset 同时支持多个参数
----------------------

unset 支持多个参数，想必大多数人是知道的，但是 isset 也支持哟。

```
<?php

var_dump(isset($a, $b, $c));

unset($a, $b, $c);


```

你不需要担心这几个变量没有被设置，他们在这里都是安全的，不会报错，在 isset 多个变量时，必须要所有变量都不为 `null`时，才会返回 `true`，当遇到一个不存在时，将会立即返回。

参考：https://www.php.net/isset

快速查询一个函数或者类或语法的参考
-----------------

当你要查询一个 php 方法或者对象或者语法时，你不需要打开 php 手册进行搜索，你只需要在 `https://php.net/<keyword>`后面跟上方法、语法、对象的名字即可，并且不需要关心大小 i 额，比如像下面这些链接。

*   https://php.net/curlfile
    
*   https://php.net/isset
    
*   https://php.net/if
    

使用反射调用 protected 或者 private 的类方法
--------------------------------

如果想避免一个方法被外部可见或者子类可见，可以采用 protected 或者 private 关键字来修改这些类，但是我们有时候又想在外部调用这些方法，应该怎么办呢？只能改成 public 吗？如果这是我们自己的代码，当然可以这样做，但是如果是引入的外部代码的话，可能就不太好直接修改了。

现在，我们可以在外部使用 **反射** 来调用这些方法，现在我们来定义一个 Lisa 类

```
<?php

class Lisa
{
    public function name()
    {
        return 'Lisa';
    }

    protected function age()
    {
        return 22;
    }

    private function weight()
    {
        return 95;
    }
  
    private static function eat(){
        return 1;
    }
}


```

通常情况下，我们是没有办法直接调用 age 和 weight 方法的，现在，我们使用反射来调用。

```
<?php
// ...
$reflectionClass = new ReflectionClass('Lisa');
$ageMethod = $reflectionClass->getMethod('age'); // 获取 age 方法
$ageMethod->setAccessible(true); // 设置可见性
// 调用这个方法，需要传入对象作为上下文
$age = $ageMethod->invoke($reflectionClass->newInstance());
var_dump($age);// 22


```

上面的代码看起来有些繁琐，还有一个更简单的办法。

```
<?php
// ...
$reflectionClass = new ReflectionClass('Lisa');
$weightMethod = $reflectionClass->getMethod('weight');// 获取 weight 方法
// 获取一个闭包，然后调用，同样需要传入对象作为上下文，后面调用的地方就可以传入参数
$weight = $weightMethod->getClosure($reflectionClass->newInstance())();
var_dump($weight);


```

调用静态方法

```
<?php
// ...
$reflectionClass = new ReflectionClass('Lisa');
$eatMethod = $reflectionClass->getMethod('eat');
$eatMethod->setAccessible(true);
$eat = $eatMethod->invoke(null); // 如果是一个静态方法，你可以传递一个 null
var_dump($eat);


```

同样，类成员也可以使用反射进行修改。参考：https://www.php.net/manual/zh/class.reflectionproperty.php

实例化一个类，但是绕过他的构造方法
-----------------

有没有这样想过？实例化一个类，但是却不想调用他的构造方法 (__construct)，这里也可以用反射做到。

```
<?php

class Dog
{
    private $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function getName()
    {
        return $this->name;
    }
}

$dogClass = new ReflectionClass('Dog');
// 创建一个新实例，但是不调用构造方法
$dogInstance = $dogClass->newInstanceWithoutConstructor();
var_dump($dogInstance->getName()); // null


```

如果你的环境不能使用反射，你还可以试试另一个很 cool 的方法，就是使用反序列化，可以参考包 doctrine/instantiator - Packagist

参考：https://www.php.net/manual/zh/reflectionclass.newinstancewithoutconstructor.php

获取类一个类的所有父类
-----------

使用 `class_parents`可以获取一个类的所有父类，并且支持自动加载。

```
<?php

class A{}
class B extends A{}
class C extends B{}
class D extends C{}

var_dump(class_parents('D'));
/*
array(3) {
  'C' =>
  string(1) "C"
  'B' =>
  string(1) "B"
  'A' =>
  string(1) "A"
}
*/


```

参考：https://www.php.net/manual/zh/function.class-parents.php

有趣的递增和递减
--------

### 递增递减不能作用域 bool 值

递增、递减不能使用在 false 上面，但是 `+=` 和 `-=` 可以

```
<?php

$a = false;

++$a;

var_dump($a);// false

$a++;

var_dump($a);// false

--$a;

var_dump($a);// false

$a--;

var_dump($a);// false

$a-= 1;

var_dump($a);// -1

$a+= 1;// 因为前面改变了，变成了 -1，所以下面是 0 ，请不要在这里疑惑

var_dump($a);// 0


```

### 递增可以作用域 NULL，但是递减不可以

```
<?php

$a = null;
++$a;
var_dump($a); //1

$a = null;
--$a;
var_dump($a); // null


```

### 递增可以作用于字母，但是递减不可以

a-y 递增时字母都将向后增加一个，但是当 z 的时候，就将会回到 aa ，循环如此，但是只能递增，不能递减

```
<?php

$a = 'a';
++$a;
var_dump($a); // b

$a = 'z';
++$a;
var_dump($a); // aa

$a = 'A';
++$a;
var_dump($a); // B

$a = 'Z';
++$a;
var_dump($a); // AA


```

### 混合递增数字和字母

现在你还可以把字母和数字混合起来，就像这样：

```
>>> $a = 'A1'
=> "A1"
>>> ++$a
=> "A2"
>>> ++$a
=> "A3"
>>> $a = '001A'
=> "001A"
>>> ++$a
=> "001B"
>>> ++$a
=> "001C"
>>> $a = 'A001'
=> "A001"
>>> ++$a
=> "A002"
>>> ++$a
=> "A003"


```

但是请注意一些意外情况，比如这样。

```
>>> $a = '9E0'
=> "9E0"
>>> ++$a
=> 10.0


```

这是因为 9E0 被当作成了浮点数的字符串表示，被 PHP 当成了 9*10^0 ，被评估成了 9 ，然后在执行的递增。

参考来源：https://www.php.net/manual/zh/language.operators.increment.php#109621

请注意你的嵌套强制类型转换，否则他会发生意外
----------------------

```
<?php

var_dump(TRUE === (boolean) (array) (int) FALSE);// true

var_dump((array) (int) FALSE);


```

因为当把 FALSE 转为数字是，他是 0，再转为数组后，就成了，`[0]`，所以再转为 boolean 时，将会返回 true，因为数组不为空，并且 `[0] != []`

参考：https://www.php.net/manual/zh/language.types.type-juggling.php#115373

高版本中的数字与字符串进行比较
---------------

自 PHP 8.0 开始。

> 数字与非数字形式的字符串之间的非严格比较现在将首先将数字转为字符串，然后比较这两个字符串。数字与数字形式的字符串之间的比较仍然像之前那样进行。请注意，这意味着 0 == "not-a-number" 现在将被认为是 false 。

<table data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)"><thead data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)"><tr data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><th data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">Comparison</th><th data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">Before</th><th data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)|rgb(240, 240, 240)" data-style="border-top-width: 1px; border-color: rgb(204, 204, 204); text-align: left; background-color: rgb(240, 240, 240); min-width: 85px;">After</th></tr></thead><tbody data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)"><tr data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">0 == "0"</td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)">true</strong></td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)">true</strong></td></tr><tr data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">0 == "0.0"</td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)">true</strong></td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)">true</strong></td></tr><tr data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">0 == "foo"</td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)">true</strong></td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)">false</strong></td></tr><tr data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">0 == ""</td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)">true</strong></td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)">false</strong></td></tr><tr data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: white;"><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">42 == "42"</td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)">true</strong></td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(25, 25, 25)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(255,255,255)">true</strong></td></tr><tr data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248);"><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;">42 == "42foo"</td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)">true</strong></td><td data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)" data-style="border-color: rgb(204, 204, 204); min-width: 85px;"><strong data-darkmode-color-16227007095287="rgb(163, 163, 163)" data-darkmode-original-color-16227007095287="#fff|rgb(0,0,0)" data-darkmode-bgcolor-16227007095287="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227007095287="#fff|rgb(248, 248, 248)">false</strong></td></tr></tbody></table>

参考：https://www.php.net/manual/zh/migration80.incompatible.php#migration80.incompatible.core

数组也可以直接比较
---------

你可以直接使用 `==` 比较两个数组有相同的键值对，如果这不是一个关联数组，那么就要保证值的顺序相对应，如果时一个关联数组，你就可以不用担心。

```
>>> $b = [1,2,3,4]
=> [
     1,
     2,
     3,
     4,
   ]
>>> $a = [1,2,3,4]
=> [
     1,
     2,
     3,
     4,
   ]
>>> $a == $b
=> true

// 注意，他不会比较类型。

>>> $a = [0,1,2,3,4]
=> [
     0,
     1,
     2,
     3,
     4,
   ]
>>> $b = [false,1,2,3,4]
=> [
     false,
     1,
     2,
     3,
     4,
   ]
>>> $a == $b
=> true

// 如果你要比较类型，你应该使用 ===

>>> $a === $b
=> false


```

无序的比较：下面的列表中，使用 == 将会返回 true ，因为他们的值是相等的，只是顺序不同，但是如果使用 === 将会返回类型，因为 === 的时候会考虑键值顺序和数据类型。

```
>>> $a = ['name'=>'Jack','sex'=>1,'age'=>18];
=> [
     "name" => "Jack",
     "sex" => 1,
     "age" => 18,
   ]
>>> $b = ['name'=>'Jack','age'=>18,'sex'=>1];
=> [
     "name" => "Jack",
     "age" => 18,
     "sex" => 1,
   ]
>>> $a == $b
=> true
>>> $a === $b
=> false
>>>


```

来源：PHP: 数组运算符 - Manual

合并数组
----

数组还可以相加 (+)，用来合并数组，使用 array_merge 可以合并数组可以把两个数组相加，想必是都知道的，但是其实 + 号也可以，虽然都是合并数组，这两个方法各有区别。`+` 更像是替换。

1、使用 array_merge 合并非关联数组时，不会过滤重复项目， + 会 (更像是替换)

```
>>> $a = [1,2,3]
=> [
     1,
     2,
     3,
   ]
>>> $b = [2,3,4]
=> [
     2,
     3,
     4,
   ]
>>> array_merge($a,$b)
=> [
     1,
     2,
     3,
     2,
     3,
     4,
   ]
>>> $a + $b
=> [
     1,
     2,
     3,
   ]


```

2、使用 array_merge 合并关联数组时，如果键重复，将会保留最后一个数组的值，而使用 + 将会保留第一个键下面的值。

```
>>> $a = ['name'=>'Jack','sex'=>1,'age'=>18];
=> [
     "name" => "Jack",
     "sex" => 1,
     "age" => 18,
   ]
>>> $b = ['name'=>'Jack','age'=>'18','sex'=>'1'];
=> [
     "name" => "Jack",
     "age" => "18",
     "sex" => "1",
   ]
>>> array_merge($a, $b)
=> [
     "name" => "Jack",
     "sex" => "1",
     "age" => "18",
   ]
>>> $a + $b
=> [
     "name" => "Jack",
     "sex" => 1,
     "age" => 18,
   ]


```

3、当关联数组中存在数字键时， array_merge 会重置数字键， + 则不会

```
>>> $a = ['name'=>'Jack','sex'=>1,'age'=>18];
=> [
     "name" => "Jack",
     "sex" => 1,
     "age" => 18,
   ]
>>> $b = ['name'=>'Jack','age'=>'18','sex'=>'1','10'=>'hi'];
=> [
     "name" => "Jack",
     "age" => "18",
     "sex" => "1",
     10 => "hi",
   ]
>>> array_merge($a,$b)
=> [
     "name" => "Jack",
     "sex" => "1",
     "age" => "18",
     //👇 注意这里
     0 => "hi",
   ]
>>> $a + $b
=> [
     "name" => "Jack",
     "sex" => 1,
     "age" => 18,
     //👇 注意这里
     10 => "hi",
   ]


```

下面用一张图来概括一下。

![](https://mmbiz.qpic.cn/mmbiz_png/GicBlLc5goGWAMqMicpibnIkibDNMVVrj0HpAV0nF8rt0RZbqicUQXXMOPt5jL8OvxNUG57NhXqibNGRiciawgS8bmmEww/640?wx_fmt=png)

图片来源：array_merge vs array_replace vs + (plus aka union) in PHP | SOFTonSOFA

参考
--

*   PHP: PHP 手册 - Manual   
    https://www.php.net/manual/zh/
    
*   风雪之隅 - Laruence 的博客   
    https://www.laruence.com/
    
*   performance - List of Big-O for PHP functions - Stack Overflow  
    https://stackoverflow.com/questions/2473989/list-of-big-o-for-php-functions/2484455#2484455  
    
*   array_merge vs array_replace vs + (plus aka union) in PHP | SOFTonSOFA  
    https://p.softonsofa.com/php-array_merge-vs-array_replace-vs-plus-aka-union/  
      
    

> 转自：思否  唯一丶
> 
> https://segmentfault.com/a/1190000040084826

- EOF -

推荐阅读  点击标题可跳转

1、[漫画 ：辞职前与老板的最后一次谈话有哪些禁忌？](http://mp.weixin.qq.com/s?__biz=MzAwNjMxMTA5Mw==&mid=2651346590&idx=1&sn=08f4ff286d7b50053a26589ca384f8b7&chksm=80f3ab44b78422521c3ec8f37458bbb4a02cee90c6807fe153da429a1037c9559ffbc514b989&scene=21#wechat_redirect)

2、[Unicode、UTF-8、UTF-16 终于懂了](http://mp.weixin.qq.com/s?__biz=MzAwNjMxMTA5Mw==&mid=2651346577&idx=1&sn=960cd8d8c4d03b4def324ea9ec01b311&chksm=80f3ab4bb784225daed9943f24691e0d1d71173ac9700f68c77f66f64b162c75c905212bd196&scene=21#wechat_redirect)

3、[他是世界上最杰出程序员之一，1 个月写了个操作系统，退休后去做飞行员！](http://mp.weixin.qq.com/s?__biz=MzAwNjMxMTA5Mw==&mid=2651346573&idx=1&sn=c0b7200c168bb0a89a394cb9eb8e25e6&chksm=80f3ab57b7842241ece527793d34c8fa7ee485eaad3803322ab66676b63438e5f7b4aa8e39e6&scene=21#wechat_redirect)