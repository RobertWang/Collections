> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u010324331/article/details/85328872)

##### 错误原因

有时候在本地使用 RSA 秘钥没有问题，在服务器上面会报错。这种情况可能是 RSA 的秘钥格式问题导致

##### 解决办法

> **使用到的函数**  
> **wordwrap**  
> 定义和用法  
> wordwrap() 函数按照指定长度对字符串进行折行处理。  
> 注释：该函数可能会在行的开头留下空格。

> **语法**  
> wordwrap(string,width,break,cut)  
> string 必需。规定要进行折行的字符串。  
> width 可选。规定最大行宽度。默认是 75。  
> break 可选。规定作为分隔符使用的字符（字串断开字符）。默认是 “\n”。  
> cut 可选。规定是否对大于指定宽度的单词进行折行：FALSE - 默认。不折行 TRUE - 折行

> **技术细节**  
> 返回值： 如果成功，则返回折行后的字符串。如果失败，则返回 FALSE。  
> PHP 版本： 4.0.2+  
> 更新日志： 在 PHP 4.0.3 中，新增了 cut 参数。

##### 示例

```
// 私钥
$privateKey = "-----BEGIN RSA PRIVATE KEY-----\n" . wordwrap($this->privateKey, 64, "\n",true) . "\n-----END RSA PRIVATE KEY-----\n";
// 公钥
$publicKey = "-----BEGIN PUBLIC KEY-----\n" . wordwrap($this->publicKey, 64, "\n",true) . "\n-----END PUBLIC KEY-----\n";

```