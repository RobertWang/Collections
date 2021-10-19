> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/xiangweiqiang/article/details/99738631)

PHP RSA 报错  
openssl_sign(): supplied key param cannot be coerced into a private key

一般是私钥格式不正确，转换一下就好了。

![](https://img-blog.csdnimg.cn/20190819170054419.png)  
主要函数：

```
	chunk_split();
	
	"-----BEGIN RSA PRIVATE KEY-----\n$str-----END RSA PRIVATE KEY-----\n";

```

然后再重试

openssl_sign($data, $sign, $privateKey,‘sha256’)

还是不行的话再重新找解决方法。