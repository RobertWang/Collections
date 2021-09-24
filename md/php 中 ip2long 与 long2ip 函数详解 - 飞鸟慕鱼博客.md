> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.feiniaomy.com](https://www.feiniaomy.com/post/630.html)

> 在网站的优化中如果将 IP 地址转成整形来存储的话，可以大大的降低数据库的索引时间，而且还节省了很多的服务器资源。

在网站的优化中如果将 IP 地址转成整形来存储的话，可以大大的降低数据库的索引时间，而且还节省了很多的服务器资源。在 php 中，可以借助内置的预设函数 ip2long() 和 long2ip() 来实现 IP 地地数据类型的转换，但需要要注意的是  ip2long() 和 long2ip() 只能实现 IPV4 地址的处理！

php ip2long() 函数详解
------------------

ip2long：将 IPV4 地址转换成长整型数字！

语法：

```
ip2long($ip_address)

```

复制

参数：$ip_address 为 IPV4 地址

示例：

```
<?php
$ip = '127.168.0.125';


$ip2 = '192.0.34.166';
echo ip2long($ip2);


?>

```

复制

php long2ip() 函数详解
------------------

long2ip：将长整型数字转化为标准的 IPV4 地址！

语法：

```
long2ip($proper_address)

```

复制

参数：$proper_address 要转换的长整型数字

示例：

```
<?php
$ip = '127.168.0.125';
$ipstr = ip2long($ip);
echo long2ip($ipstr);

$ip2 = '192.0.34.166';
$ipstr2 = ip2long($ip2);
echo long2ip($ipstr2);

?>

```

复制

ip2long() 转换后的负数问题
------------------

当 ip 地址比较大时，转换后的整数会有超过最大整形范围 (也就是溢出) 的情况。而超过的部份就会成为负数，为避免此情况的出现我们需要对 ip2long(0 的结果进行处理一下！

处理方法 1：

示例代码：

```
<?php
$ip = "192.0.34.166";
$strip = bindec(decbin(ip2long($ip))); 
echo $strip;

echo long2ip($strip);
?>

```

复制

注：decbin 函数将十进制转换为二进制，bindec 函数将二进制转换为整形。

处理方法 2：

使用类 C 的 sprintf 方法来避免 ip2long 出现负数的问题

示例代码：

```
<?php
echo sprintf("%u\n", ip2long("157.23.56.90"));

?>

```

复制