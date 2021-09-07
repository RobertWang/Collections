> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6gwAkAoIRwii2wDm5Zc8iQ)

作者介绍

背景
--

base64 的编码原理网上讲解较多，但解码原理讲解较少，并且没有对其中的内部实现原理进行剖析。想要彻底了解 base64 的编码与解码原理，请耐心看完此文，你一定会有所收获。

涉及算法与逻辑运算概念
-----------

在探究 base64 编码原理和解码原理的过程中，我们首先需要了解下面会用到的算法和逻辑运算的概念，这样才能真正的吃透 base64 的编码原理和解码原理，体会到其中算法的精妙，甚至是在思考的过程中得到意想不到的收获。不清楚 base64 编码表和 ascII 编码表的同学可直接前往文末查看。

### 短除法

短除法运算方法是先用一个除数除以能被它除尽的一个质数，以此类推，除到商是质数为止。  
通过短除法，十进制数可以不断除以 2 得到多个余数。最后，将余数从下到上进行排列组合，得到二进制数，我们以字符 n 对应的 ascII 编码 110 为例。

```
    110 / 2  = 55...0
    55  / 2  = 27...1
    27  / 2  = 13...1
    13  / 2  = 6...1
    6   / 2  = 3...0
    3   / 2  = 1...1
    1   / 2  = 0...1
```

将余数从下到上进行排列组合，得到字符 n 对应的 ascII 编码 110 转二进制为 1101110，因为一字节对应 8 位 (bit), 所以需要向前补 0 补足 8 位，得到 01101110。其余字符同理可得。

### 按权展开求和

按权展开求和, 8 位二进制数从右到左，次数是 0 到 7 依次递增, 基数 * 底数次数，从左到右依次累加，相加结果为对应十进制数。我们以二进制数 01101110 转 10 进制为例：

> (01101110)2 = 0 * 20 + 1 * 21 + 1 * 22 + 1 * 23 + 0 * 24 + 1 * 25 + 1 * 26 + 0 * 27

### 位概念

二进制数系统中，每个 0 或 1 就是一个位 (bit，比特)，也叫存储单元，位是数据存储的最小单位。其中 8bit 就称为一个字节（Byte）。

### 移位运算符

移位运算符在程序设计中，是位操作运算符的一种。移位运算符可以在二进制的基础上对数字进行平移。按照平移的方向和填充数字的规则分为三种：<<(左移)、>>(带符号右移)和>>>(无符号右移)。我们在 base64 的编码和解码过程中操作的又是正数，所以仅使用 <<(左移)、>>(带符号右移) 两种运算符。

> 1.  左移运算：是将一个二进制位的操作数按指定移动的位数向左移动，移出位被丢弃，右边移出的空位一律补 0。
>     
> 2.  右移运算：是将一个二进制位的操作数按指定移动的位数向右移动，移出位被丢弃，左边移出的空位一律补 0，或者补符号位，这由不同的机器而定。在使用补码作为机器数的机器中，正数的符号位为 0，负数的符号位为 1。
>     

我们用大白话来描述左移位，一共有 8 个座位，坐了 8 个人，在 8 个座位不动的情况下，现在我让这 8 个人往左挪 2 个座位，于是最左边的两个人站了起来，没有座位坐，而最右边空出来了两个座位。移位操作就相当于站起来的人出局，留出来的空位补 0.

```
    // 左移
    01101000 << 2 -> 101000(左侧移出位被丢弃) -> 10100000(右侧空位一律补0)
    // 右移
    01101000 >> 2 -> 011010(右侧移出位被丢弃) -> 00011010(左侧空位一律补0)
```

### 与运算、或运算

与运算、或运算都是计算机中一种基本的逻辑运算方式。

> 1.  与运算：符号表示为 &。运算规则：两位同时为 “1”，结果才为 “1”，否则为 0
>     
> 2.  或运算：符号表示为｜。运算规则：两位只要有一位为 “1”，结果就为 “1”，否则为 0
>     

什么是 base64 编码
-------------

Base64 编码是将字符串以每 3 个 8 比特 (bit) 的字节子序列拆分成 4 个 6 比特 (bit) 的字节 (6 比特有效字节，最左边两个永远为 0，其实也是 8 比特的字节) 子序列，再将得到的子序列查找 Base64 的编码索引表，得到对应的字符拼接成新的字符串的一种编码方式。

每 3 个 8 比特 (bit) 的字节子序列拆分成 4 个 6 比特 (bit) 的字节的拆分过程如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNN9juUqCmrNAVb6X90dxbe7uLfBvO8cnBSEKKJCBa5pFlnACoaTLLVb0hY9UdGWrib6n3k6fPPc6nWQ/640?wx_fmt=png)

base64

### 为什么 base64 编码后的大小是原来的 4/3 倍

因为 6 和 8 的最大公倍数是 24，所以 3 个 8 比特的字节刚好可以拆分成 4 个 6 比特的字节，3_8 = 6_4。计算机中，因为一个字节需要 8 个存储单元存储，所以我们要把 6 个比特往前面补两位 0，补足 8 个比特。如下图所示：![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNN9juUqCmrNAVb6X90dxbe7ug387DMNZRlxUINBBhkPnagAYnXdxYQdvVE4u5lWD6bGmdUrF2icBj7w/640?wx_fmt=png)很明显，补足后所需的存储单元为 32 个，是原来所需的 24 个的 4/3 倍。现在大家明白为什么 base64 编码后的大小是原来的 4/3 倍了吧。

### 为什么命名为 base64 呢？

> 因为 6 位 (bit) 的二进制数有 2 的 6 次方个, 也就是二进制数 (00000000-00111111) 之间的代表 0-63 的 64 个二进制数。

### 不是说一个字节是用 8 位二进制表示的吗，为什么不是 2 的 8 次方？

> 因为我们得到的 8 位二进制数的前两位永远是 0，真正的有效位只有 6 位，所以我们所能够得到的二进制数只有 2 的 6 次方个。

### Base64 字符是哪 64 个？

Base64 的编码索引表，字符选用了 "A-Z、a-z、0-9、+、/" 64 个可打印字符来代表 (00000000-00111111) 这 64 个二进制数。即

```
    let base64EncodeChars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
```

编码原理
----

我们不妨自己先思考一下，要把 3 个字节拆分成 4 个字节可以怎么做？你的实现思路和我的实现思路有哪个不同，我们之间又会碰出怎样的火花？

### 流程图

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNN9juUqCmrNAVb6X90dxbe7ugjB166iaUQkicgibp50WG98b4yJ6Vib2AB91XLJyEViaPJUb4LSlXa8FcrQ/640?wx_fmt=png)

流程图

### 思路

分析映射关系：abc -> xyzi。我们从高位到低位添加索引来分析这个过程

*   x: (前面补两个 0)a 的前六位 => 00a7a6a5a4a3a2
    
*   y: (前面补两个 0)a 的后两位 + b 的前四位 => 00a1a0b7b6b5b4
    
*   z: (前面补两个 0)b 的后四位 + c 的前两位 => 00b3b2b1b0c7c6
    
*   i: (前面补两个 0)c 的后六位 => 00c5c4c3c2c1c0 通过上述的映射关系，我们很容易得到下面的实现思路：
    

1.  将字符对应的 ascII 编码转为 8 位二进制数
    
2.  将每三个 8 位二进制数进行以下操作
    

*   将第一个数右移位 2 位，得到第一个 6 位有效位二进制数
    
*   将第一个数 & 0x3 之后左移位 4 位，得到第二个 6 位有效位二进制数的第一个和第二个有效位，将第二个数 & 0xf0 之后右移位 4 位，得到第二个 6 位有效位二进制数的后四位有效位，两者取且得到第二个 6 位有效位二进制
    
*   将第二个数 & 0xf 之后左移位 2 位，得到第三个 6 位有效位二进制数的前四位有效位，将第三个数 & 0xC0 之后右移位 6 位，得到第三个 6 位有效位二进制数的后两位有效位，两者取且得到第三个 6 位有效位二进制
    
*   将第三个数 & 0x3f，得到第四个 6 位有效位二进制数
    

4.  将获得的 6 位有效位二进制数转十进制，查找对应 base64 字符
    

我们以 hao 字符串为例，观察 base64 编码的过程，我们将上面转换通过代码逻辑分析实现吧。

### 代码实现

```
// 输入字符串
let str = 'hao'
// base64字符串
let base64EncodeChars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
// 定义输入、输出字节的二进制数
let char1, char2, char3, out1, out2, out3, out4, out
// 将字符对应的ascII编码转为8位二进制数
char1 = str.charCodeAt(0) & 0xff // 104  01101000
char2 = str.charCodeAt(1) & 0xff // 97  01100001
char3 = str.charCodeAt(2) & 0xff // 111  01101111
// 输出6位有效字节二进制数
6out1 = char1 >> 2 // 26  011010
out2 = (char1 & 0x3) << 4 | (char2 & 0xf0) >> 4 // 6  000110
out3 = (char2 & 0xf) << 2 | (char3 & 0xc0) >> 6 // 5  000101
out4 = char3 & 0x3f // 47 101111

out = base64EncodeChars[out1] + base64EncodeChars[out2] + base64EncodeChars[out3] + base64EncodeChars[out4] // aGFv
```

算法剖析

1.  out1: char1 >> 2
    
    ```
    01101000 -> 00011010
    ```
    
2.  out2 = (char1 & 0x3) << 4 | (char2 & 0xf0) >> 4
    
    ```
    // 且运算
    01101000        01100001
    00000011        11110000
    --------        --------
    00000000        01100000
    
    // 移位运算后得
    00000000        00000110
    
    // 或运算
    --------
    ```
    

第三个字符第四个字符同理

整理上述代码，扩展至多字符字符串

```
// 输入字符串
let str = 'haohaohao'
// base64字符串
let base64EncodeChars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'

// 获取字符串长度
let len = str.length
// 当前字符索引
let index = 0
// 输出字符串
let out = ''
while(index < len) {
    // 定义输入、输出字节的二进制数
    let char1, char2, char3, out1, out2, out3, out4
    // 将字符对应的ascII编码转为8位二进制数
    char1 = str.charCodeAt(index++) & 0xff // 104  01101000
    char2 = str.charCodeAt(index++) & 0xff // 97  01100001
    char3 = str.charCodeAt(index++) & 0xff // 111  01101111
    // 输出6位有效字节二进制数
    out1 = char1 >> 2 // 26  011010
    out2 = (char1 & 0x3) << 4 | (char2 & 0xf0) >> 4 // 6  000110
    out3 = (char2 & 0xf) << 2 | (char3 & 0xc0) >> 6 // 5  000101
    out4 = char3 & 0x3f // 47 101111

    out = out + base64EncodeChars[out1] + base64EncodeChars[out2] + base64EncodeChars[out3] + base64EncodeChars[out4] // aGFv
}
```

原字符串长度不是 3 的整倍数的情况，需要特殊处理

```
    ...
    char1 = str.charCodeAt(index++) & 0xff // 104  01101000
    if (index == len) {
        out2 = (char1 & 0x3) << 4
        out = out + base64EncodeChars[out1] + base64EncodeChars[out2] + '=='
        return out
    }
    char2 = str.charCodeAt(index++) & 0xff // 97  01100001
    if (index == len) {
        out1 = char1 >> 2 // 26  011010
        out2 = (char1 & 0x3) << 4 | (char2 & 0xf0) >> 4 // 6  000110
        out3 = (char2 & 0xf) << 2
        out = out + base64EncodeChars[out1] + base64EncodeChars[out2] + base64EncodeChars[out3] + '='
        return out
    }
    ...
```

全部代码

```
function base64Encode(str) {
    // base64字符串
    let base64EncodeChars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'

    // 获取字符串长度
    let len = str.length
    // 当前字符索引
    let index = 0
    // 输出字符串
    let out = ''
    while(index < len) {
        // 定义输入、输出字节的二进制数
        let char1, char2, char3, out1, out2, out3, out4
        // 将字符对应的ascII编码转为8位二进制数
        char1 = str.charCodeAt(index++) & 0xff
        out1 = char1 >> 2
        if (index == len) {
            out2 = (char1 & 0x3) << 4
            out = out + base64EncodeChars[out1] + base64EncodeChars[out2] + '=='
            return out
        }
        char2 = str.charCodeAt(index++) & 0xff
        out2 = (char1 & 0x3) << 4 | (char2 & 0xf0) >> 4 
        if (index == len) {
            out3 = (char2 & 0xf) << 2
            out = out + base64EncodeChars[out1] + base64EncodeChars[out2] + base64EncodeChars[out3] + '='
            return out
        }
        char3 = str.charCodeAt(index++) & 0xff
        // 输出6位有效字节二进制数
        out3 = (char2 & 0xf) << 2 | (char3 & 0xc0) >> 6
        out4 = char3 & 0x3f

        out = out + base64EncodeChars[out1] + base64EncodeChars[out2] + base64EncodeChars[out3] + base64EncodeChars[out4]
    }
    return out
}
base64Encode('haohao') // aGFvaGFv
base64Encode('haoha') // aGFvaGE=
base64Encode('haoh') // aGFvaA==
```

解码原理
----

逆向推导，由每 4 个 6 位有效位的二进制数合并成 3 个 8 位二进制数，根据 ascII 编码映射到对应字符后拼接字符串

### 思路

分析映射关系 xyzi -> abc

*   a: x 后六位 + y 第三、四位 => x5x4x3x2x1x0y5y4
    
*   b: y 后四位 + z 第三、四、五、六位 => y3y2y1y0z5z4z3z2
    
*   c: z 后两位 + i 后六位 => z1z0i5i4i3i2i1i0
    

1.  将字符对应的 base64 字符集的索引转为 6 位有效位二进制数
    
2.  将每四个 6 位有效位二进制数进行以下操作
    

1.  第一个二进制数左移位 2 位，得到新二进制数的前 6 位，第二个二进制数 & 0x30 之后右移位 4 位，或运算后得到第一个新二进制数
    
2.  第二个二进制数 & 0xf 之后左移位 4 位，第三个二进制数 & 0x3c 之后右移位 2 位，或运算后得到第二个新二进制数
    
3.  第二个二进制数 & 0x3 之后左移位 6 位，与第四个二进制数或运算后得到第二个新二进制数
    

4.  根据 ascII 编码映射到对应字符后拼接字符串
    

### 代码实现

```
// base64字符串
let str = 'aGFv'
// base64字符集
let base64CharsArr = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'.split('')
// 获取索引值
let char1 = base64CharsArr.findIndex(char => char==str[0]) & 0xff // 26  011010
let char2 = base64CharsArr.findIndex(char => char==str[1]) & 0xff // 6  000110
let char3 = base64CharsArr.findIndex(char => char==str[2]) & 0xff // 5  000101
let char4 = base64CharsArr.findIndex(char => char==str[3]) & 0xff // 47  101111
let out1, out2, out3, out
// 位运算
out1 = char1 << 2 | (char2 & 0x30) >> 4
out2 = (char2 & 0xf) << 4 | (char3 & 0x3c) >> 2
out3 = (char3 & 0x3) << 6 | char4
console.log(out1, out2, out3)
out = String.fromCharCode(out1) + String.fromCharCode(out2) + String.fromCharCode(out3)
```

遇到有用'='补过位的情况时

```
function base64decode(str) {
    // base64字符集
    let base64CharsArr = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'.split('')
    let char1 = base64CharsArr.findIndex(char => char==str[0])
    let char2 = base64CharsArr.findIndex(char => char==str[1])
    let out1, out2, out3, out
    if (char1 == -1 || char2 == -1) return out
    char1 = char1 & 0xff
    char2 = char2 & 0xff
    let char3 = base64CharsArr.findIndex(char => char==str[2])
    // 第三位不在base64对照表中时，只拼接第一个字符串
    if (char3 == -1) {
        out1 = char1 << 2 | (char2 & 0x30) >> 4
        out = String.fromCharCode(out1)
        return out
    }
    let char4 = base64CharsArr.findIndex(char => char==str[3])
    // 第三位不在base64对照表中时，只拼接第一个和第二个字符串
    if (char4 == -1) {
        out1 = char1 << 2 | (char2 & 0x30) >> 4
        out2 = (char2 & 0xf) << 4 | (char3 & 0x3c) >> 2
        out = String.fromCharCode(out1) + String.fromCharCode(out2)
        return out
    }
    // 位运算
    out1 = char1 << 2 | (char2 & 0x30) >> 4
    out2 = (char2 & 0xf) << 4 | (char3 & 0x3c) >> 2
    out3 = (char3 & 0x3) << 6 | char4
    console.log(out1, out2, out3)
    out = String.fromCharCode(out1) + String.fromCharCode(out2) + String.fromCharCode(out3)
    return out
}
```

解码整个字符串，整理代码后

```
function base64decode(str) {
    // base64字符集
    let base64CharsArr = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'.split('')
    let i = 0
    let len = str.length
    let out = ''
    while(i < len) {
        let char1 = base64CharsArr.findIndex(char => char==str[i])
        i++
        let char2 = base64CharsArr.findIndex(char => char==str[i])
        i++
        let out1, out2, out3
        if (char1 == -1 || char2 == -1) return out
        char1 = char1 & 0xff
        char2 = char2 & 0xff
        let char3 = base64CharsArr.findIndex(char => char==str[i])
        i++
        // 第三位不在base64对照表中时，只拼接第一个字符串
        out1 = char1 << 2 | (char2 & 0x30) >> 4
        if (char3 == -1) {
            out = out + String.fromCharCode(out1)
            return out
        }
        let char4 = base64CharsArr.findIndex(char => char==str[i])
        i++
        // 第三位不在base64对照表中时，只拼接第一个和第二个字符串
        out2 = (char2 & 0xf) << 4 | (char3 & 0x3c) >> 2
        if (char4 == -1) {
            out = out + String.fromCharCode(out1) + String.fromCharCode(out2)
            return out
        }
        // 位运算
        out3 = (char3 & 0x3) << 6 | char4
        console.log(out1, out2, out3)
        out = out + String.fromCharCode(out1) + String.fromCharCode(out2) + String.fromCharCode(out3)
    }
    return out
}
base64decode('aGFvaGFv') // haohao
base64decode('aGFvaGE=') // haoha
base64decode('aGFvaA==') // haoh
```

上述解码核心是字符与 base64 字符集索引的映射，网上看到过使用 AccII 编码索引映射 base64 字符索引的方法

```
let base64DecodeChars = [-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 62, -1, -1, -1, 63, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, -1, -1, -1, -1, -1, -1, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, -1, -1, -1, -1, -1, -1, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, -1, -1, -1, -1, -1]
// 
let char1 = 'hao'.charCodeAt(0) // h -> 104
base64DecodeChars[char1] // 33 -> base64编码表中的h
```

由此可见，base64DecodeChars 对照 accII 编码表的索引存放的是 base64 编码表的对应字符的索引。

总结
--

说起 Base64 编码可能有些奇怪，因为大多数的编码都是由字符转化成二进制的过程，而从二进制转成字符的过程称为解码。而 Base64 的概念就恰好反了，由二进制转到字符称为编码，由字符到二进制称为解码。Base64 是一种数据编码方式，可做简单加密使用，我们可以改变 base64 编码映射顺序来形成自己独特的加密算法进行加密解密。

编码表
---

![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNN9juUqCmrNAVb6X90dxbe7umxQwGfTzsYiakKjxR9BtSUFt1nydUVWvrGVmRXCloicOKsKYVtqM0GUQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/T81bAV0NNN9juUqCmrNAVb6X90dxbe7uapibLBMKQcLYk3X6pPV6szmuqdbdooQWPwshB9TRtjlAb4JhbIvfQjw/640?wx_fmt=png)