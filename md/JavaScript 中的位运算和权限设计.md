> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651557434&idx=1&sn=70a1a192b4c4dc78550e084a464bc6e4&chksm=802559fbb752d0edc90bf53342a94220f482fdfc1c3a826ff36ddc08b3cc671eee38b3c25667&scene=21#wechat_redirect)

（给前端大全加星标，提升前端技能）

> 作者：云音乐前端团队
> 
> https://musicfe.dev/javascript-bitwise-operators/

1. 内容概要  

----------

本文主要讨论以下两个问题：

*   JavaScript 的位运算：先简单回顾下位运算，平时用的少，相信不少人和我一样忘的差不多了
    
*   权限设计：根据位运算的特点，设计一个权限系统（添加、删除、判断等）
    

2. JavaScript 位运算
-----------------

### 2.1. Number

![](https://mmbiz.qpic.cn/mmbiz_png/aDoYvepE5x1ibekJ2zx7jCmicRBzYwyh25JowVdUgxsW6YBMj5eDQr8T70tw50N6SN2qfF84ibTsv0zNMpUKBBo1A/640?wx_fmt=png)

*   sign bit（符号）: 用来表示正负号
    
*   exponent（指数）: 用来表示次方数
    
*   mantissa（尾数）: 用来表示精确度
    

也就是说一个数字的范围只能在 -(2^53 -1) 至 2^53 -1 之间。

> 既然讲到这里，就多说一句：0.1 + 0.2 算不准的原因也在于此。浮点数用二进制表达时是无穷的，且最多 53 位，必须截断，进而产生误差。最简单的解决办法就是放大一定倍数变成整数，计算完成后再缩小。不过更稳妥的办法是使用下文将会提到的 math.js 等工具库。

```
// 十进制
123456789
0

// 二进制：前缀 0b，0B
0b10000000000000000000000000000000 // 2147483648
0b01111111100000000000000000000000 // 2139095040
0B00000000011111111111111111111111 // 8388607

// 八进制：前缀 0o，0O（以前支持前缀 0）
0o755 // 493
0o644 // 420

// 十六进制：前缀 0x，0X
0xFFFFFFFFFFFFFFFFF // 295147905179352830000
0x123456789ABCDEF   // 81985529216486900
0XA                 // 10


```

好了，Number 就说这么多，接下来看 JavaScript 中的位运算。

### 2.2. 位运算

按位操作符将其操作数当作 32 位的比特序列（由 0 和 1 组成）操作，返回值依然是标准的 JavaScript 数值。JavaScript 中的按位操作符有：

<table width="657"><thead><tr data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><th data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; min-width: 70px; text-align: left; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)">运算符</span></th><th data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; min-width: 70px; text-align: left; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)">用法</span></th><th data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; min-width: 70px; text-align: left; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)">描述</span></th></tr></thead><tbody><tr data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">按位与（AND）</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><code>a &amp; b</code></td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">对于每一个比特位，只有两个操作数相应的比特位都是 1 时，结果才为 1，否则为 0。</td></tr><tr data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">按位或（OR）</span></td><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><code data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">a \| b</span></code></td><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">对于每一个比特位，当两个操作数相应的比特位至少有一个 1 时，结果为 1，否则为 0。</span></td></tr><tr data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">按位异或（XOR）</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><code>a ^ b</code></td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">对于每一个比特位，当两个操作数相应的比特位有且只有一个 1 时，结果为 1，否则为 0。</td></tr><tr data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">按位非（NOT）</span></td><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><code data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">~a</span></code></td><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">反转操作数的比特位，即 0 变成 1，1 变成 0。</span></td></tr><tr data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">左移（Left shift）</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><code>a &lt;&lt; b</code></td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">将 a 的二进制形式向左移 b (&lt;32) 比特位，右边用 0 填充。</td></tr><tr data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">有符号右移</span></td><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><code data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">a &gt;&gt; b</span></code></td><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">将 a 的二进制表示向右移 b (&lt;32) 位，丢弃被移出的位。</span></td></tr><tr data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">无符号右移</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><code>a &gt;&gt;&gt; b</code></td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">将 a 的二进制表示向右移 b (&lt;32) 位，丢弃被移出的位，并使用 0 在左侧填充。</td></tr></tbody></table>

下面举几个例子，主要看下 `AND` 和 `OR`：

```
# 例子1
    A = 10001001
    B = 10010000
A | B = 10011001

# 例子2
    A = 10001001
    C = 10001000
A | C = 10001001


```

```
# 例子1
    A = 10001001
    B = 10010000
A & B = 10000000

# 例子2
    A = 10001001
    C = 10001000
A & C = 10001000


```

3. 位运算在权限系统中的使用
---------------

传统的权限系统里，存在很多关联关系，如用户和权限的关联，用户和角色的关联。系统越大，关联关系越多，越难以维护。而引入位运算，可以巧妙的解决该问题。

在讲 “位运算在权限系统中的使用” 之前，我们先假定两个前提，**下文所有的讨论都是基于这两个前提的**：

1.  每种权限码都是唯一的（这是显然的）
    
2.  所有权限码的二进制数形式，有且只有一位值为 1，其余全部为 0（`2^n`）
    

如果用户权限和权限码，全部使用二级制数字表示，再结合上面 `AND` 和 `OR`的例子，分析位运算的特点，不难发现：

*   `|` 可以用来赋予权限
    
*   `&` 可以用来校验权限
    

为了讲的更明白，这里用 Linux 中的实例分析下，Linux 的文件权限分为读、写和执行，有字母和数字等多种表现形式：

<table width="657"><thead><tr data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><th data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; min-width: 70px; text-align: left; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)">权限</span></th><th data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; min-width: 70px; text-align: left; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)">字母表示</span></th><th data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; min-width: 70px; text-align: left; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)">数字表示</span></th><th data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-top-width: 1px; border-color: rgb(204, 204, 204); background-color: rgb(240, 240, 240); max-width: 100%; min-width: 70px; text-align: left; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(40, 40, 40)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(240, 240, 240)">二进制</span></th></tr></thead><tbody><tr data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">读</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">r</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">4</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">0b100</td></tr><tr data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); background-color: rgb(248, 248, 248); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">写</span></td><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">w</span></td><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">2</span></td><td data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)" data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;"><span data-darkmode-bgcolor-16227762313175="rgb(32, 32, 32)" data-darkmode-original-bgcolor-16227762313175="#fff|rgb(248, 248, 248)">0b010</span></td></tr><tr data-style="max-width: 100%; border-width: 1px 0px 0px; border-right-style: initial; border-bottom-style: initial; border-left-style: initial; border-right-color: initial; border-bottom-color: initial; border-left-color: initial; border-top-style: solid; border-top-color: rgb(204, 204, 204); box-sizing: border-box !important; overflow-wrap: break-word !important;"><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">执行</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">x</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">1</td><td data-style="padding-right: 15px; padding-left: 15px; word-break: break-all; border-color: rgb(204, 204, 204); max-width: 100%; min-width: 70px; overflow-wrap: break-word !important; box-sizing: border-box !important;">0b001</td></tr></tbody></table>

可以看到，权限用 1、2、4（也就是 `2^n`）表示，转换为二进制后，都是只有一位是 1，其余为 0。我们通过几个例子看下，如何利用二进制的特点执行权限的添加，校验和删除。

### 3.1. 添加权限

```
let r = 0b100
let w = 0b010
let x = 0b001

// 给用户赋全部权限（使用前面讲的 | 操作）
let user = r | w | x

console.log(user)
// 7

console.log(user.toString(2))
// 111

//     r = 0b100
//     w = 0b010
//     r = 0b001
// r|w|x = 0b111


```

可以看到，执行 `r | w | x` 后，`user` 的三位都是 1，表明拥有了全部三个权限。

> Linux 下出现权限问题时，最粗暴的解决方案就是 `chmod 777 xxx`，这里的 `7` 就代表了：可读，可写，可执行。而三个 `7` 分别代表：文件所有者，文件所有者所在组，所有其他用户。

### 3.2. 校验权限

刚才演示了权限的添加，下面演示权限校验：

```
let r = 0b100
let w = 0b010
let x = 0b001

// 给用户赋 r w 两个权限
let user = r | w
// user = 6
// user = 0b110 (二进制)

console.log((user & r) === r) // true  有 r 权限
console.log((user & w) === w) // true  有 w 权限
console.log((user & x) === x) // false 没有 x 权限


```

如前所料，通过 `用户权限 & 权限 code === 权限 code` 就可以判断出用户是否拥有该权限。

### 3.3. 删除权限

我们讲了用 `|` 赋予权限，使用 `&` 判断权限，那么删除权限呢？删除权限的本质其实是**将指定位置上的 1 重置为 0**。上个例子里用户权限是 `0b110`，拥有读和写两个权限，现在想删除读的权限，本质上就是将第三位的 1 重置为 0，变为 `0b010`：

```
let r = 0b100
let w = 0b010
let x = 0b001

let user = 0b010;

console.log((user & r) === r) // false 没有 r 权限
console.log((user & w) === w) // true  有 w 权限
console.log((user & x) === x) // false 没有 x 权限


```

那么具体怎么操作呢？其实有两种方案，最简单的就是异或 `^`，按照上文的介绍 “当两个操作数相应的比特位有且只有一个 1 时，结果为 1，否则为 0”，所以异或其实是 toggle 操作，无则增，有则减：

```
let r    = 0b100
let w    = 0b010
let x    = 0b001
let user = 0b110 // 有 r w 两个权限

// 执行异或操作，删除 r 权限
user = user ^ r

console.log((user & r) === r) // false 没有 r 权限
console.log((user & w) === w) // true  有 w 权限
console.log((user & x) === x) // false 没有 x 权限

console.log(user.toString(2)) // 现在 user 是 0b010

// 再执行一次异或操作
user = user ^ r

console.log((user & r) === r) // true  有 r 权限
console.log((user & w) === w) // true  有 w 权限
console.log((user & x) === x) // false 没有 x 权限

console.log(user.toString(2)) // 现在 user 又变回 0b110


```

那么如果单纯的想删除权限（而不是无则增，有则减）怎么办呢？答案是执行 `&(~code)`，先取反，再执行与操作：

```
let r    = 0b100
let w    = 0b010
let x    = 0b001
let user = 0b110 // 有 r w 两个权限

// 删除 r 权限
user = user & (~r)

console.log((user & r) === r) // false 没有 r 权限
console.log((user & w) === w) // true  有 w 权限
console.log((user & x) === x) // false 没有 x 权限

console.log(user.toString(2)) // 现在 user 是 0b010

// 再执行一次
user = user & (~r)

console.log((user & r) === r) // false 没有 r 权限
console.log((user & w) === w) // true  有 w 权限
console.log((user & x) === x) // false 没有 x 权限

console.log(user.toString(2)) // 现在 user 还是 0b010，并不会新增


```

4. 局限性和解决办法
-----------

前面我们回顾了 JavaScript 中的 Number 和位运算，并且了解了基于位运算的权限系统原理和 Linux 文件系统权限的实例。

上述的所有都有前提条件：1、**每种权限码都是唯一的**；2、**每个权限码的二进制数形式，有且只有一位值为 1（****`2^n`）**。也就是说，权限码只能是 1, 2, 4, 8,...,1024,... 而上文提到，一个数字的范围只能在 -(2^53 -1) 和 2^53 -1 之间，JavaScript 的按位操作符又是将其操作数当作 **32 位**比特序列的。那么同一个应用下可用的权限数就非常有限了。这也是该方案的局限性。

为了突破这个限制，这里提出一个叫 “权限空间” 的概念，既然权限数有限，那么不妨就多开辟几个空间来存放。

基于权限空间，我们定义两个格式：

1.  **权限 code**，字符串，形如 `index,pos`。其中 `pos` 表示 32 位二进制数中 1 的位置（其余全是 0）； `index` 表示**权限空间**，用于突破 JavaScript 数字位数的限制，是从 0 开始的正整数，每个权限 code 都要归属于一个权限空间。`index` 和 `pos` 使用英文逗号隔开。
    
2.  **用户权限**，字符串，形如 `1,16,16`。英文逗号分隔每一个**权限空间**的权限值。例如 `1,16,16` 的意思就是，权限空间 0 的权限值是 1，权限空间 1 的权限值是 16，权限空间 2 的权限是 16。
    

干说可能不好懂，直接上代码：

```
// 用户的权限 code
let userCode = ""

// 假设系统里有这些权限
// 纯模拟，正常情况下是按顺序的，如 0,0 0,1 0,2 ...，尽可能占满一个权限空间，再使用下一个
const permissions = {
  SYS_SETTING: {
    value: "0,0",   // index = 0, pos = 0
    info: "系统权限"
  },
  DATA_ADMIN: {
    value: "0,8",
    info: "数据库权限"
  },
  USER_ADD: {
    value: "0,22",
    info: "用户新增权限"
  },
  USER_EDIT: {
    value: "0,30",
    info: "用户编辑权限"
  },
  USER_VIEW: {
    value: "1,2",   // index = 1, pos = 2
    info: "用户查看权限"
  },
  USER_DELETE: {
    value: "1,17",
    info: "用户删除权限"
  },
  POST_ADD: {
    value: "1,28",
    info: "文章新增权限"
  },
  POST_EDIT: {
    value: "2,4",
    info: "文章编辑权限"
  },
  POST_VIEW: {
    value: "2,19",
    info: "文章查看权限"
  },
  POST_DELETE: {
    value: "2,26",
    info: "文章删除权限"
  }
}

// 添加权限
const addPermission = (userCode, permission) => {
  const userPermission = userCode ? userCode.split(",") : []
  const [index, pos] = permission.value.split(",")

  userPermission[index] = (userPermission[index] || 0) | Math.pow(2, pos)

  return userPermission.join(",")
}

// 删除权限
const delPermission = (userCode, permission) => {
  const userPermission = userCode ? userCode.split(",") : []
  const [index, pos] = permission.value.split(",")

  userPermission[index] = (userPermission[index] || 0) & (~Math.pow(2, pos))

  return userPermission.join(",")
}

// 判断是否有权限
const hasPermission = (userCode, permission) => {
  const userPermission = userCode ? userCode.split(",") : []
  const [index, pos] = permission.value.split(",")
  const permissionValue = Math.pow(2, pos)

  return (userPermission[index] & permissionValue) === permissionValue
}

// 列出用户拥有的全部权限
const listPermission = userCode => {
  const results = []

  if (!userCode) {
    return results
  }

  Object.values(permissions).forEach(permission => {
    if (hasPermission(userCode, permission)) {
      results.push(permission.info)
    }
  })

  return results
}

const log = () => {
  console.log(`userCode: ${JSON.stringify(userCode, null, " ")}`)
  console.log(`权限列表: ${listPermission(userCode).join("; ")}`)
  console.log("")
}

userCode = addPermission(userCode, permissions.SYS_SETTING)
log()
// userCode: "1"
// 权限列表: 系统权限

userCode = addPermission(userCode, permissions.POST_EDIT)
log()
// userCode: "1,,16"
// 权限列表: 系统权限; 文章编辑权限

userCode = addPermission(userCode, permissions.USER_EDIT)
log()
// userCode: "1073741825,,16"
// 权限列表: 系统权限; 用户编辑权限; 文章编辑权限

userCode = addPermission(userCode, permissions.USER_DELETE)
log()
// userCode: "1073741825,131072,16"
// 权限列表: 系统权限; 用户编辑权限; 用户删除权限; 文章编辑权限

userCode = delPermission(userCode, permissions.USER_EDIT)
log()
// userCode: "1,131072,16"
// 权限列表: 系统权限; 用户删除权限; 文章编辑权限

userCode = delPermission(userCode, permissions.USER_EDIT)
log()
// userCode: "1,131072,16"
// 权限列表: 系统权限; 用户删除权限; 文章编辑权限

userCode = delPermission(userCode, permissions.USER_DELETE)
userCode = delPermission(userCode, permissions.SYS_SETTING)
userCode = delPermission(userCode, permissions.POST_EDIT)
log()
// userCode: "0,0,0"
// 权限列表:

userCode = addPermission(userCode, permissions.SYS_SETTING)
log()
// userCode: "1,0,0"
// 权限列表: 系统权限


```

除了通过引入**权限空间**的概念突破二进制运算的位数限制，还可以使用 math.js 的 `bignumber`，直接运算超过 32 位的二进制数，具体可以看它的文档，这里就不细说了。

5. 适用场景和问题
----------

如果按照当前使用最广泛的 RBAC 模型设计权限系统，那么一般会有这么几个实体：应用，权限，角色，用户。用户权限可以直接来自权限，也可以来自角色：

*   一个应用下有多个权限
    
*   权限和角色是多对多的关系
    
*   用户和角色是多对多的关系
    
*   用户和权限是多对多的关系
    

在此种模型下，一般会有用户与权限，用户与角色，角色与权限的对应关系表。想象一个商城后台权限管理系统，可能会有上万，甚至十几万店铺（应用），每个店铺可能会有数十个用户，角色，权限。随着业务的不断发展，刚才提到的那三张对应关系表会越来越大，越来越难以维护。

而进制转换的方法则可以省略对应关系表，减少查询，节省空间。当然，省略掉对应关系不是没有坏处的，例如下面几个问题：

*   如何高效的查找我的权限？
    
*   如何高效的查找拥有某权限的所有用户？
    
*   如何控制权限的有效期？
    

所以进制转换的方案比较适合刚才提到的应用极其多，而每个应用中用户，权限，角色数量较少的场景。

6. 其他方案
-------

除了二进制方案，当然还有其他方案可以达到类似的效果，例如直接使用一个 1 和 0 组成的字符串，权限点对应 index，1 表示拥有权限，0 表示没有权限。举个例子：添加 0、删除 1、编辑 2，用户 A 拥有添加和编辑的权限，则 userCode 为 101；用户 B 拥有全部权限，userCode 为 111。这种方案比二进制转换简单，但是浪费空间。

还有利用质数的方案，权限点全部为质数，用户权限为他所拥有的全部权限点的乘积。如：权限点是 2、3、5、7、11，用户权限是 5 * 7 * 11 = 385。这种方案麻烦的地方在于获取质数（新增权限点）和质因数分解（判断权限），权限点特别多的时候就快成 RSA 了，如果只有增删改查个别几个权限，倒是可以考虑。

7. 参考
-----

*   MDN：JavaScript 数字和日期
    
*   双精度浮点类型
    
*   MDN：按位操作符
    
*   【小知识大道理】被忽视的位运算
    
*   为什么不要在 JavaScript 中使用位操作符？
    
*   角色权限设计的 100 种解法
    
*   权限系统与 RBAC 模型概述
    
*   权限设计及算法
    
*   基于角色的访问控制
    

觉得本文对你有帮助？请分享给更多人