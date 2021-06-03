> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [segmentfault.com](https://segmentfault.com/q/1010000010105968/a-1020000010119519)

> 我是一个前端菜鸟，和后端（JAVA）配合做一个后台管理系统的项目，之前没做过 cookie 相关或者说跨域相关的项目，遇到这个问题困扰好多天了，后端小哥跟我都很难受，请大家不吝赐教。

我是一个前端菜鸟，和后端（JAVA）配合做一个后台管理系统的项目，之前没做过 cookie 相关或者说跨域相关的项目，遇到这个问题困扰好多天了，后端小哥跟我都很难受，请大家不吝赐教。

业务逻辑是这样的：由于前后端分开部署，我需要在 www.a.com/home/login.html 页面上通过 ajax 调用 10.0.2.101 的一个用户登录接口，调用成功后接口会返回一个 cookie（Set-Cookie: JSESSIONID=xxxxxxxxxxxxxx; Domain=www.a.com; Path=/），JAVA 要求我后续调用其他任何接口时必须将 cookie 随 ajax 一同发给他，他根据 jsessionId 做身份验证。

但前端页面 www.a.com 和后台接口 10.0.2.101 肯定属于跨域了，浏览器不会响应后台发来的 cookie，于是我们就把后端的接口挪到了 a.com 上，应该从跨域变成同源了吧，因为 a.com 是 www.a.com 的父域，结果问题依旧，浏览器不理会 a.com 发来的 set-cookie 指令，也不会在 ajax 到 a.com 的过程中携带任何手工创建的 cookie！  
（完全相同域名的情况我测试了，比如 a.com 调用 a.com 下的一个 php，是可以正常 set-cookie 以及 ajax 携带 cookie 的，但我们正式环境不会允许前、后端在完全相同的域中）

为了便于测试，JAVA 部分的代码重写成了 PHP，结果浏览器（chrome，FF，IE...）虽然收到 response 里的 cookie，却依然不存储 cookie。

PHP 代码如下：

```
<?php
$origin = isset($_SERVER['HTTP_ORIGIN'])? $_SERVER['HTTP_ORIGIN'] : '';
// 指定允许其他域名访问
header('Access-Control-Allow-Origin:'.$origin);
// 响应类型
header('Access-Control-Allow-Methods:GET');
// 响应头设置
header('Access-Control-Allow-Headers:x-requested-with,content-type');

header('P3P: CP="CAO DSP COR CUR ADM DEV TAI PSA PSD IVAi IVDi CONi TELo OTPi OUR DELi SAMi OTRi UNRi PUBi IND PHY ONL UNI PUR FIN COM NAV INT DEM CNT STA POL HEA PRE GOV"');

// 发送一个简单的 cookie
setcookie("TestCookie1", "my cookie value1", time() + 3600, "/", "a.com");
// 发送一个简单的 cookie
setcookie("TestCookie2", "my cookie value2",-1,"/",".a.com");
?>

```

JS 代码如下：

```
$.ajax({
    type: "get",
    async: true,
    data: null,
    url: "http://a.com/test/test.php",
    dataType: "json",
    // crossDomain: true, // 这行代码加或不加都试过了
    beforeSend: function(req){
        req.withCredencials=true;
    },
    success: function(data){
        
    },
    error: function(err){
        
    }
})

```

部分截图如下：

![](https://segmentfault.com/img/bVQzb2?w=1115&h=589)

![](https://segmentfault.com/img/bVQzb5?w=1208&h=366)

![](https://segmentfault.com/img/bVQzb7?w=605&h=469)