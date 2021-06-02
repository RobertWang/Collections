> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jb51.net](https://www.jb51.net/article/50932.htm)

> 这篇文章主要介绍了 PHP 捕获 Fatal error 错误的方法, 使用 register_shutdown_function 来捕获 Fatal error 错误, 需要的朋友可以参考下

 更新时间：2014 年 06 月 11 日 11:02:27   作者：  

这篇文章主要介绍了 PHP 捕获 Fatal error 错误的方法, 使用 register_shutdown_function 来捕获 Fatal error 错误, 需要的朋友可以参考下

Fatal error 一般是不需要捕获的, 但是在一个复杂的程序中, 如果偶然出现内存不足导致 fatal error 就难以处理了.

比如. fatal error 出在 MySQL 类中 fetch 的时候. 这个时候就很难定位到真正问题所在了.

PHP 异常处理中 可以通过 set_error_handler 来捕获. 但是却只能捕获 NOTICE/WARNING 级别的错误, 对于 E_ERROR 是无能为力的.

register_shutdown_function 能解决 set_error_handler 的不足.

通过此函数注册好程序结束回调函数, 就可以捕获平时捕获不到的错误了. 再通过 error_get_last 对错误进行判断. 就容易找出难以定位的问题了.

function shutdown_function()   
{   
    $e = error_get_last();     
    print_r($e);   
}  
register_shutdown_function('shutdown_function');