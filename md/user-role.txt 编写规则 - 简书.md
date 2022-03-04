> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/b2557296c402)

以下为 `user-role.txt` 文件的一个样本，编写规则直接看文件内的注释信息：

```
! https://www.cnblogs.com/edward2013/p/5560836.html
! See https://adblockplus.org/en/filter-cheatsheet
! 语法规则如下：
! 
! 1. 通配符支持。比如 *.example.com/* 实际书写时可省略 * ， 如.example.com/ ， 和 *.example.com/* 效果一样
! 2. 正则表达式支持。以 \ 开始和结束， 如 \[\w]+:\/\/example.com\
! 3. 例外规则 @@ ，如 @@*.example.com/* 满足 @@ 后规则的地址不使用代理
! 4. 匹配地址开始和结尾 | ，如 |http://example.com 、 example.com| 分别表示以 http://example.com 开始和以 example.com 结束的地址
! 5. || 标记，如 ||example.com 则 http://example.com 、https://example.com 、 ftp://example.com 等地址均满足条件
! 6. 注释 ! 。 如 !我是注释
!
!
!||static.npm.red
!||ip138.com
!||js.hs-analytics.net
!||js.hs-scripts.com
!||track.hubspot.com
!
@@https://spring.io/*
@@https://www.npmjs.com/*
@@https://github.com*
@@*.github.com/*
@@*.githubassets.com/*
*.githubusercontent.com/*
@@*.github.io/*
@@.imgur.com/


```