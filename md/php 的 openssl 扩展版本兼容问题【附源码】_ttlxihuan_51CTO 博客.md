> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/php2012web/2551373)

> php 的 openssl 扩展版本兼容问题【附源码】，因为服务器里在跑一些老项目兼容问题很多，一直运行 PHP7.0 版本没有升级。

因为服务器里在跑一些老项目兼容问题很多，一直运行 PHP7.0 版本没有升级。在对接微信分时需要使用到 AES-256-GCM 加密需要调整 PHP 环境，决定先升级到 PHP7.2，升级后发现 openssl_sign() 报错，排查后做下简单兼容调整说明。

做三方对接时时常会出现问题，尤其是运行环境发生变化时。对于 PHP 环境主要分为：php 版本、扩展库版本。如果出现兼容性问题则首先需要确认环境问题，避免走弯路。

openssl 在对接支付等功能时基本上都会使用到，一般三方有对应写好的 SDK，通常按指定的环境要求下运行问题不大。但有时 SDK 并没有太细说明，难免会出现兼容问题。

### 扩展库版本兼容

openssl 版本在 1.0.1 及以下时要求证书内容分段换行，否则 openssl_sign、openssl_verify 使用证书的函数会报错，比如：  
`openssl_sign(): supplied key param cannot be coerced into a private key in`

查看 php 安装的 openssl 库版本直接通过 openssl 扩展提供的常量 OPENSSL_VERSION_TEXT 获取：  
`var_dump(OPENSSL_VERSION_TEXT);`

排查时首先确认证书是否有错、参数是否配置错误，环境是否匹配。  
如果扩展库是 openssl-1.0.1 及以内则证书内容需要分段换行否则无法识别。

或者使用代码处理：

如果发现 openssl 库版本过低，快速处理办法就是通过函数去整理证书和使用 docker，最原始的办法是升级 openssl 再重新编译 PHP。

### PHP 版本兼容

openssl 扩展函数很多，迭代时会有些变化，比如 openssl_encrypt()、openssl_decrypt()，两个函数在 PHP7.1 时增加了新参数支持 AEAD(模式 GCM 和 CCM) 一种对称加密码算法，官方文档中说作了说明：  
 [https://www.php.net/manual/zh/function.openssl-encrypt.php](https://www.php.net/manual/zh/function.openssl-encrypt.php)  
 [https://www.php.net/manual/zh/function.openssl-decrypt.php](https://www.php.net/manual/zh/function.openssl-decrypt.php)

AEAD 加密在 PHP 中有个 sodium 扩展（依赖 libsodium 库），不过 php 在 openssl 中做了一点延升，升级 PHP7.1 及以上后就不需要再多安装扩展。如果不升级 PHP 可安装 sodium 扩展。