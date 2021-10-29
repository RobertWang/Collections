> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [cloud.tencent.com](https://cloud.tencent.com/developer/article/1069064?from=14588)

> glog 简介 glog 是著名的 google 开源 C++ 日志库 glog 的 golang 版本，glog 是一个轻量级的日志库，上手简单不需要配置文件并且稳定高效，但是可以...

glog 简介
-------

glog 是著名的 google 开源 C++ 日志库 glog 的 golang 版本，glog 是一个轻量级的日志库，上手简单不需要配置文件并且稳定高效，但是可以自定义控制的内容就少了。 glog 主要有以下几个特点： 1. glog 有四种日志等级 INFO < WARING < ERROR < FATAL，不同等级的日志是打印到不同文件的，低等级的日志文件中（INFO）会包含高等级的日志信息（ERROR） 2. 通过命令行传递参数 –log_dir 指定日志文件的存放目录，默认为 os.TempDir() 3. 可以根据文件大小切割日志文件，但是不能根据日期切割日志文件 4. 日志输出格式是固定的 (Lmmdd hh:mm:ss.uuuuuu threadid file:line] msg…) 不可以自定义 5. 在程序开始时需要调用 flag.Parse()解析命令行参数，在程序退出时需要调用 glog.Flush() 确保将缓存区中的内容输出到文件中。

使用事例
----

![](https://ask.qcloudimg.com/http-save/yehe-1408376/xu16jd9fkq.jpeg?imageView2/2/w/1620)

源码分析
----

我们顺着事例代码中的 glog.Error(“error glog”) 这行代码来看下，来看下日志内容是如何输出到文件中去的。

![](https://ask.qcloudimg.com/http-save/yehe-1408376/xu16jd9fkq.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1408376/vxpsgg8gag.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1408376/oj2ysczxeu.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1408376/tpdrnx3mbw.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1408376/fy9cnkdcgu.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1408376/ahuuoee3e5.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1408376/0pewag5s7m.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1408376/zvyalkhgi7.jpeg)

vlog 简介
-------

一般的日志库会提供日志输出级别，当日志信息的级别低于输出级别时则不会输出该日志信息。我们使用其他日志库时会使用 log.Debug()打印出调试信息，在测试环境下将日志库的输出级别设置为 DEBUG，调试信息就会输出便于我们查看程序的具体运行情况，而在线上程序中将日志的输出级别设置为 INFO 调试信息就不会输出。 glog 则采用另外一种方式实现这种功能，glog 提供让用户自定义分级信息的功能，用户自定义分级与 glog 自带的日志等级 (INFO ERROR) 是完全分离的，在命令行参数设置中独立设置 “v” 或“vmodule”参数来控制。

![](https://ask.qcloudimg.com/http-save/yehe-1408376/lwcmi0b7zq.jpeg)

在测试环境下我们运行程序时指定用户自定义级别为 1 (–v=1)，上面的日志信息就会输出。 而在线上环境中指定自定义级别为 0(–v=0)，上面的日志信息则不会输出。

![](https://ask.qcloudimg.com/http-save/yehe-1408376/p6gfpraty9.jpeg)

修改 glog 源码
----------

glog 有些功能与我们常用的日志库不太一样或者没有我们期望的功能，可以修改 glog 的源码来实现我们的需求。比如我们之前使用的日志库是有 DEBUG INFO ERROR FATAL 级别的，我们可以修改 glog 源码增加 DEBUG 级别，删除 WARN 级别，已于我们的原有系统保持一致。 具体修改内容查看 github 源码

**设置等级控制日志的输出** 实现原理：定义一个输出等级变量，提供接口给用户可以设置该变量的值，默认为 INFO，在输出日志时检查日志信息的等级是否大于输出等级，如果大于则输出日志信息否则不输出

![](https://ask.qcloudimg.com/http-save/yehe-1408376/gjzoik11i7.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1408376/wk98sua53b.jpeg)

**每天自动切割日志文件** 实现原理：在创建日志文件时记录下创建文件的日期 (MMDD)，输出每条日志信息时判断当前日期与日志文件的创建日期是否一致，如果不一致则创建新的日志文件。

![](https://ask.qcloudimg.com/http-save/yehe-1408376/b2od4v2hd8.jpeg)

![](https://ask.qcloudimg.com/http-save/yehe-1408376/g0pvykfhad.jpeg)

转自：http://shanks.leanote.com/post/Untitled-55ca439338f41148cd000759-18

本文分享自微信公众号 - Golang 语言社区（Golangweb）

原文出处及转载信息见文内详细说明，如有侵权，请联系 yunjia_community@tencent.com 删除。

原始发表时间：2016-06-07

本文参与[腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan)，欢迎正在阅读的你也加入，一起分享。