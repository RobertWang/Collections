> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=3×tamp=1622546283&ver=1&signature=n3*Uy7*7QR39jA2iDtmrAphX96B5tdsSIF*tv98dWSnKAYxHxZ2DWyQa38uIN4SXetxugZFJMkAx-HzQXWxbK2uaSJNvwclldhID6OLAcG0v1IGfAJDNhzPf3rsuNfLvrFVLQSpibMTQUEA606egl0h9usCBGsFCo3CaOe5ToYw=)

经过十几天的忙碌，张小五手上的项目终于如期上线，虽然很累，但内心无比的充实与喜悦。

喝了杯热咖啡，小五在椅子上慵懒地躺着，享受着这份静谧的时光。

"周末好好休息下吧，我先跟你讨论个事儿啊。"

" 咱们线上运行的代码，出于各种各样的情况，可能会有好多 **Fatal Error**、**Exception**。有没有办法，在出现 Fatal Error、Exception 的时候，咱们能自动捕获，并写到 Log 文件里？"

"嗯... 这个嘛，出现 Fatal Error 的时候，脚本就终止了，不好捕获啊。"

"对，是不好捕获。但是对于出现的 Fatal Error、Exception 我们不知道的话，不能提前发现问题，就像身边有个隐形的刺客一样，让人内心特别虚啊..."

"这样啊，Z 哥，那我这几天试一下吧！"

"好的，小五，这个挺重要的，相信你！"

"哈哈，Z 哥你还是不要抱太大希望，我努力试一下就是了。"

2

从 Google 到 SO

对于码农来说，从 Google 到 StackOverflow 是解决问题的通途，当然张小五也不例外。  

哈！不搜不知道，一搜吓一跳，PHP 还真有捕获 Error 和 Exception 的函数呢。

1.  `//设置一个用户的函数来处理脚本中出现的错误。`
    
2.  `set_error_handler($callback)`
    
3.  `//设置一个用户的函数来处理脚本中出现的异常。`
    
4.  `set_exception_handler($callback)`
    

张小五不自觉的笑了笑：“哈哈，不愧是世界上最好的语言！”

说干就干，看看这两个函数的威力怎样, 不一会, 小五就写出了测试代码。

1.  `<?php`
    
2.  `//设置异常捕获函数`
    
3.  `set_exception_handler("my_exception");`
    
4.  `function my_exception($exception){`
    
5.   `echo 'Exception Catched:'.$exception->getMessage();`
    
6.  `}`
    
7.  `//抛出异常`
    
8.  `throw new Exception("I am Exception");`
    

![](http://mmbiz.qpic.cn/mmbiz_png/8XvYFFcdpY2kJAgsQ6DsmwH7I7waicdWiaEA3pN63gdEJBNCZrATAMd2CibNGp5uvYbAr9wy7g3GPtUAX4contIicA/0?wx_fmt=png)

Yes，抛出的一个 Exception 真的被捕获了！

"接下来再测下 set_error_handler()，你可不能让我失望啊！" 小五心想。

1.  `<?php`
    
2.  `set_error_handler("error_handler");`
    
3.  `function error_handler($errno,$errstr,$errfile,$errline){`
    
4.   `$str=<<<EOF`
    
5.   `"errno":$errno`
    
6.   `"errstr":$errstr`
    
7.   `"errfile":$errfile`
    
8.   `"errline":$errline`
    
9.  `EOF;`
    
10.  `//获取到错误可以自己处理，比如记Log、报警等等`
    
11.   `echo $str;`
    
12.  `}`
    
13.  `echo $test;//$test未定义，会报一个notice级别的错误`
    

![](http://mmbiz.qpic.cn/mmbiz_png/8XvYFFcdpY2kJAgsQ6DsmwH7I7waicdWiaNfJb6V9VN0Z7hUhFIMqS7kK3K3GvJxPrYJUolL8lPBeZ6OoNURLjibQ/0?wx_fmt=png)

不错，Notice 级别的错误也捕获到了！

接下来再测一下 Fatal Error，如果 Fatal Error 也能捕获到，这个需求就实现了!

抑制住激动的心情，小五很快写完了测试代码。

1.  `<?php`
    
2.  `set_error_handler("error_handler");`
    
3.  `function error_handler($errno,$errstr,$errfile,$errline){`
    
4.   `$str=<<<EOF`
    
5.   `"errno":$errno`
    
6.   `"errstr":$errstr`
    
7.   `"errfile":$errfile`
    
8.   `"errline":$errline`
    
9.  `EOF;`
    
10.  `//获取到错误可以自己处理，比如记Log、报警等等`
    
11.   `echo $str;`
    
12.  `}`
    
13.  `//调用一个不存在的函数，会出现Fatal Error`
    
14.  `test();`
    

小五屏住呼吸，等待着奇迹的出现。![](http://mmbiz.qpic.cn/mmbiz_jpg/US10Gcd0tQEBNhn1uHVbVL9604SySChCqOgnyqh4WCzKKLfumnrG9AODdva4xicbUxj7xU6icGyuUIwvcXI5oxiag/0)

"咣当"，手起指落，几行报错跃然屏上...

![](http://mmbiz.qpic.cn/mmbiz_png/8XvYFFcdpY2kJAgsQ6DsmwH7I7waicdWia4eLJZic1eJCLiaHee2icWyEN5vEAaPwGQibSoRPAkOKx39V5CWsJA3ODtA/0?wx_fmt=png)

神马？Fatal Error 竟然没捕获到？怎么可能？

正在小五陷入沉思的时候，不经意间，小五瞥见了函数的说明：

> 以下级别的错误不能由用户定义的函数来处理：
> 
>  E_ERROR、 E_PARSE、 E_CORE_ERROR、 E_CORE_WARNING、 E_COMPILE_ERROR、 E_COMPILE_WARNING，和在 调用 set_error_handler() 函数所在文件中产生的大多数 E_STRICT。

也就是：set_error_handler($callback) 只能捕获系统产生的一些 Warning、Notice 级别的 Error。

呜呼悲催，好不容易找到了解决办法，没想到这函数竟然还是个半吊子，很多级别的错误捕获不到...![](http://mmbiz.qpic.cn/mmbiz_png/US10Gcd0tQEBNhn1uHVbVL9604SySChC4KqVibfEK2ak3wV2N3KLRwbrKRy4AHmaeh5UrDl89SHsKWc1G1rTc7w/0)

3

众里寻他千百度

王小五从不是轻言放弃的人，他又继续搜索，寻找着解决办法...

"嗯？哈哈，SO 上还真有人遇到这问题!"

小五专注地看着答案，边看边敲了起来：

要实现这个需求，需要用到两个函数:

`register_shutdown_function()`

`error_get_last()`。

**register_shutdown_function()**

```
register_shutdown_function($callback)

```

register_shutdown_function()，就把你要注册进去的 function 放进【假装是队列吧】，等到脚本正常退出或显式调用 exit() 时，再把注册进去的 function 拉出来执行.

register_shutdown_function() 调用的 3 种情况：

- 脚本正常退出时；

- 在脚本运行 (run-time not parse-time) 出错退出时；

- 用户调用 exit 方法退出时。

**error_get_last()**

```
error_get_last();//函数获取最后发生的错误。

```

该函数以数组的形式返回最后发生的错误。

返回的数组包含 4 个键和值：

[type] - 错误类型

[line] - 发生错误所在的行

> 在 parse-time 出错的时候，是不会调用 register_shutdown_function() 函数的。
> 
> 只有在 run-time 出错的时候，才会调用 register_shutdown_function()。

为了更好的理解，下面我们举例说明：

NO.1

error_handler.php

1.  `<?php`
    
2.  `register_shutdown_function("error_handler");`
    
3.  `function error_handler(){`
    
4.   `echo "Yeah,it's worked!";`
    
5.  `}`
    
6.  `function test(){}`
    
7.  `function test(){}`
    

执行结果如下：

![](http://mmbiz.qpic.cn/mmbiz_png/8XvYFFcdpY2kJAgsQ6DsmwH7I7waicdWiaPGfPRno1Ot1LcdkibkcGfCslTGOBp8kIribOVPia5LbCJDiak75stPiaf9g/0?wx_fmt=png)

原因分析:  

在执行 error_handler.php 的时候，由于重复定义了两个函数 test(), 在 php 的 parse-time 就出错了（不是 run-time），所以不能回调 register_shutdown_function() 中注册的函数。

NO.2

error_handler.php

1.  `<?php`
    
2.  `register_shutdown_function("error_handler");`
    
3.  `function error_handler(){`
    
4.   `echo "Yeah,it's worked!";`
    
5.  `}`
    
6.  `if(true){`
    
7.   `function test(){}`
    
8.  `}`
    
9.  `function test(){}`
    

执行结果如下：

![](http://mmbiz.qpic.cn/mmbiz_png/8XvYFFcdpY2kJAgsQ6DsmwH7I7waicdWiaanS0FBNH4L2CUlDRsaZYBu2tIicbmuWydbpDiaehe7BFALn42EDt0zHg/0?wx_fmt=png)

原因分析:  

我们看到，上面回调了 register_shutdown_function() 中注册的函数。

因为我们加了一个 if() 判断，if() 里面的 test() 方法，相当于一个闭包，与外面的 test() 名称不冲突。

1.  `<?php`
    
2.  `register_shutdown_function("error_handler");`
    
3.  `function error_handler(){`
    
4.   `echo "Yeah,it's worked!";`
    
5.  `}`
    

1.  `<?php`
    
2.  `include './error_handler.php';`
    
3.  `function test(){}`
    
4.  `function test(){}`
    

![](http://mmbiz.qpic.cn/mmbiz_png/8XvYFFcdpY2kJAgsQ6DsmwH7I7waicdWiaqHUBBEawvELcmkbvhG9VcKAEHORBpnciaiciaE1zn10T73d6XwR5oEFdg/0?wx_fmt=png)

原因分析:

当我们在运行 test.php 的时候, 因为 redeclare 了两个 test() 方法，所以 php 的语法解析器在 parse-time 的时候就出错了。 所以不能回调 register_shutdown_function() 中的方法，不能 catch 住这个 fatal error。

NO.4 

error_handler.php

1.  `<?php`
    
2.  `register_shutdown_function("error_handler");`
    
3.  `function error_handler(){`
    
4.   `echo "Yeah,it's worked!";`
    
5.  `}`
    

test.php

1.  `<?php`
    
2.  `function test(){}`
    
3.  `function test(){}`
    

include_all.php

1.  `<?php`
    
2.  `require './error_handler.php';`
    
3.  `require './test.php';`
    

执行 include_all.php 的结果如下：

![](http://mmbiz.qpic.cn/mmbiz_png/8XvYFFcdpY2kJAgsQ6DsmwH7I7waicdWiaLRqIAfGiaWp4VU16t1vXNe1jllnhiaibvOzd3xh2CbytpnmYPQlNU83JQ/0?wx_fmt=png)

结果分析：  

上面我们捕获了 fatal_error。

因为在运行 include_all.php 的时候，include_all.php 本身语法并没有出错，也就是在 parse-time 的时候并没有出错，而是 include 的文件出错了，也就是在 run-time 的时候出错了，这个时候是能回调 register_shutdown_function() 中的函数的。

> 强烈建议：如果我们要使用 register_shutdown_function 进行错误捕捉，使用 NO.4，最后一种方法，可以确保错误都能捕捉到。

4

蓦然回首解需求

"哇塞，原来可以这样啊!"  

王小五按答案中举的例子认真的敲完代码，瞬间明白了解决的办法。

真可谓 "众里寻他千百度，蓦然回首，那人却在灯火阑珊处。" 小二不自觉的感叹道！

"好了，我自己就写一个 error_handler 脚本吧，确保每次都能获取到想要的 Fatal Error。"

1.  `<?php`
    
2.  `register_shutdown_function( "fatal_handler" );`
    
3.  `set_error_handler("error_handler");`
    

5.  `define('E_FATAL', E_ERROR | E_USER_ERROR | E_CORE_ERROR |`
    
6.   `E_COMPILE_ERROR | E_RECOVERABLE_ERROR| E_PARSE );`
    

8.  `//获取fatal error`
    
9.  `function fatal_handler() {`
    
10.   `$error = error_get_last();`
    
11.   `if($error && ($error["type"]===($error["type"] & E_FATAL))) {`
    
12.   `$errno   = $error["type"];`
    
13.   `$errfile = $error["file"];`
    
14.   `$errline = $error["line"];`
    
15.   `$errstr  = $error["message"];`
    
16.   `error_handler($errno,$errstr,$errfile,$errline);`
    
17.   `}`
    
18.  `}`
    
19.  `//获取所有的error`
    
20.  `function error_handler($errno,$errstr,$errfile,$errline){`
    
21.   `$str=<<<EOF`
    
22.   `"errno":$errno`
    
23.   `"errstr":$errstr`
    
24.   `"errfile":$errfile`
    
25.   `"errline":$errline`
    
26.  `EOF;`
    
27.  `//获取到错误可以自己处理，比如记Log、报警等等`
    
28.   `echo $str;`
    
29.  `}`
    

有了这个脚本，我再按 SO 上说的第四种方法去执行，那这个需求就实现了！

5

不负众望

王小五兴冲冲的找到 Z 哥，详细的说明了自己的研究成果。

第二天，小五按照公司现有的框架规则，结合上面的解决办法，不一会就实现了需求。

"不错啊，小五，我就说你可以吧！" Z 哥高兴的说到。

"哈哈，Z 哥，这下所有的错误都在掌握之中了!"

END

![](http://mmbiz.qpic.cn/mmbiz_jpg/8XvYFFcdpY2kJAgsQ6DsmwH7I7waicdWialjpdjpv73usZgpqib1JibGEkKou8GuLVy5HQjfvCerJ53uQOGsCaYm7w/0?wx_fmt=jpeg)