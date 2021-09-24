> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.shiqidu.com](https://www.shiqidu.com/d/934)

> 在开发中，经常需要将 IP 地址转成整型进行保存，这样不仅有利于做索引，并且原本需要 15 个字节的存储空间，转换后只需 4 个字节就能存储了。

在开发中，经常需要将 IP 地址转成整型进行保存，这样不仅有利于做索引，并且原本需要 15 个字节的存储空间，转换后只需 4 个字节就能存储了。但是很多人对于 ip2long 的结果有时候是负数并不理解，本文将详细解释这一点。因为 ip2long 只支持 IPv4，所以本文也是基于 IPv4 来描述和编码的。

右移
--

### 逻辑右移

右移多少位，则在高位补多少位 0。

### 算术右移

对无符号数做算术右移和逻辑右移的结果是相同的。但是对一个有符号数做算术右移，则右移多少位，即在高位补多少位 1。

### 注意事项

对于 C 来说，只提供了 >> 右移运算符，究竟是逻辑右移还是算术右移这取决于编译器的行为，因此一般只提倡对无符号数进行位操作。

IPv4 地址是如何表示的
-------------

IPv4 使用无符号 32 位地址，因此最多有 2 的 32 次方减 1(4294967295) 个地址。一般的书写法为用 4 个小数点分开的十进制数，记为：A.B.C.D，比如：157.23.56.90。

IPv4 地址转换成无符号整型
---------------

![](https://www.shiqidu.com//upload/article/20200921/a1d9686a39878653785a9d9174cbe374_origin.png)

IPv4 地址的每一个十进制数都为无符号的字节，因此范围在 0~255，将 IPv4 地址转成无符号整型其实就是将每个十进制数放在对应的 8 位上组成一个 4 字节的无符号整型。依上图表示：157 在高 8 位，90 在低 8 位，23 和 56 在中间对应的 8 位上。来看一个 C 实现的例子：

```
#include <stdio.h>

int main(int argc, char** argv)
{
    unsigned int ip_long = (157 << 24) | (23 << 16) | (56 << 8) | 90;
    printf("%u\n", ip_long);
    printf("%d\n", ip_long);

    return 0;
}


```

```
$ gcc -o ip2long main.c
$ ./ip2long
2635544666
-1659422630

```

可以看到，即使 ip_long 声明为无符号整型，在输出时也需要指明 %u 来格式化输出为无符号整型。这是因为 157 大于 127(二进制为 01111111)，也就是说如果 157(8 位) 用二进制来表示，最高位必然是 1。当将 157 放在一个 4 字节整型的高 8 位时，导致这个 4 字节整型的最高位为 1。虽然 ip_long 定义为无符号整型，但 printf 函数并不知道，因此需要指明无符号格式化字符。如果最高位为 0，则使用 %d 就可以了，来看另一个例子：

```
#include <stdio.h>

int main(int argc, char** argv)
{
    unsigned int ip_long = (120 << 24) | (23 << 16) | (56 << 8) | 90;
    printf("%u\n", ip_long);
    printf("%d\n", ip_long);

    return 0;
}

```

```
$ gcc -o ip2long main.c
$ ./ip2long
2014787674
2014787674

```

PHP 是如何做的
---------

现在已经知道了为什么会出现负数。对于动态类型语言来说，数据类型一般是有符号的，所以需要我们自己转成无符号整型。PHP 有内置函数 ip2long 来将 IPv4 地址转换成整型，也提供了类 C 的 sprintf 方法，因此很容易解决出现负数的问题：

```
<?php
echo sprintf("%u\n", ip2long("157.23.56.90"));

```

```
$ php -f test.php
2635544666

```

JavaScript 是如何做的
----------------

JavaScript 既没有提供 ip2long 方法，也没有提供类 C 的格式化函数。但 JavaScript 却同时提供了逻辑右移 (>>>) 和算术右移 (>>) 运算符，所以解决的方法也很简单，对结果再跟 0 做逻辑右移即可：

```
<script type="text/javascript">
console.log(((157 << 24) | (23 << 16) | (56 << 8) | 90) >>> 0);
</script>

```

```
2635544666

```

PHP 和 JavaScript 的 long2ip 的实现
------------------------------

有了前面的知识，long2ip 的实现就很简单了。只须从 ip2long 的结果中取出每 8 位形成的十进制数，再用点 (.) 连接就可以了。之前的例子都是用 IP(157.23.56.90)来举例的，它的 ip2long 的结果是：2635544666。

PHP 的 long2ip 的实现
-----------------

```
<?php
$ip_long = 2635544666;
echo long2ip($ip_long) . "\n";

```

```
php -f test.php
157.23.56.90

```

以上代码是由 PHP 的内置函数 long2ip 来实现的。但是对于想通过移位来自己实现的童鞋来说，可能没有那么简单。因为 PHP 的 >> 运算符是算术右移运算符，所以如果最高位是 1 的话，右移的结果是在高位补 1，这跟结果不符。但是我们可以用另一种思路去解决：保存最高位 (符号位)，然后将最高位置 0，之后再将高 8 位的最高位置 1(这取决于之前保存的符号位)。代码实现如下：

```
<?php
$ip_long = 2635544666;

$msb = 0;
if ($ip_long & 0x80000000)
{
    $msb = 1;
}


$uip_long = $ip_long & 0x7fffffff;
$ip1 = $uip_long >> 24;
if ($msb == 1)
{
    $ip1 |= 0x80;
}

$ip2 = ($uip_long >> 16) & 0xff; 
$ip3 = ($uip_long >> 8) & 0xff;
$ip4 = $uip_long & 0xff;
echo $ip1 . '.' . $ip2 . '.' . $ip3 . '.' . $ip4 . "\n";

```

```
$ php -f test.php
157.23.56.90

```

虽然以上代码能得到正确的结果，但是并不推荐这样做。因为以上代码是假设 PHP 中的数据类型是 32 位的。这样将会有移植性问题。我们可以跟 0xff 做与运算来取得低 8 位，这样做的好处是兼容性好。代码如下：

```
<?php
$ip_long = 2635544666;
$ip1 = ($ip_long >> 24) & 0xff; 
$ip2 = ($ip_long >> 16) & 0xff;
$ip3 = ($ip_long >> 8) & 0xff;
$ip4 = $ip_long & 0xff;
echo $ip1 . '.' . $ip2 . '.' . $ip3 . '.' . $ip4 . "\n";

```

```
$ php -f test.php
157.23.56.90

```

另外还可以通过 pack 和 unpack 方法来实现，但要注意的是 IPv4 应使用大端序。

JavaScript 的 long2ip 的实现
------------------------

```
<script type="text/javascript">
var ip_long = 2635544666;
var ip1 = (ip_long >> 24) & 0xff;
var ip2 = (ip_long >> 16) & 0xff;
var ip3 = (ip_long >> 8) & 0xff;
var ip4 = ip_long & 0xff;
console.log(ip1 + "." + ip2 + "." + ip3 + "." + ip4);
</script>

```

```
157.23.56.90

```

知其然，而知其所以然。这样以后就不会为 ip2long 的结果是负数而感到惊讶了。

> 转载地址 [https://my.oschina.net/goal/blog/198049](https://www.shiqidu.com/url/https://my.oschina.net/goal/blog/198049)