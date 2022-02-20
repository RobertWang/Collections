> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3NDgxMzI0Mw==&mid=2247502786&idx=2&sn=78dac980788a3b105a4c34ed550623c2&chksm=fd2e2e96ca59a780b3b297123ab4640427237d8ec4cae09852ff64a3df373484c2a65ec2e200&mpshare=1&scene=1&srcid=0220G6gLXyQEOhLfJrPlJz5G&sharer_sharetime=1645322502562&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

> InnoDB 是一个将表中的数据存储到磁盘上的存储引擎。

### 1. InnoDB 是干嘛的？

InnoDB 是一个将表中的数据存储到磁盘上的存储引擎。

### 2. InnoDB 是如何读写数据的？

InnoDB 处理数据的过程是发生在内存中的，需要把磁盘中的数据加载到内存中，如果是处理写入或修改请求的话，还需要把内存中的内容刷新到磁盘上。

读写磁盘的速度非常慢，和内存读写差了几个数量级，所以当我们想从表中获取某些记录时，InnoDB 存储引擎将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，InnoDB 中页的大小默认为 16 KB。也就是在一般情况下，一次最少从磁盘中读取 16KB 的内容到内存中，或者一次最少把内存中的 16KB 内容刷新到磁盘中。

所以当你用 postman 测试一个分页查询接口时，发现第一次打印耗时 300 ~ 400ms，往后不停的查找下一页都是 30 ~ 40ms，原因就是第一次请求接口时，读数据库的时候需要读磁盘，从磁盘加载 16KB 的数据到内存，往后下一页的数据都是从内存中获取，没有再读磁盘，除非在内存中的 16KB 的数据中找不到，才会再次读磁盘获取下一个 16KB 的数据到内存中。

我们不讨论 mysql 8.0 舍弃的查询缓存特性，我测试过 mysql 5.7 中关闭了查询缓存，也仍然是第一次慢，后续查询很快，查询时间相差大概 10 倍的样子

> 温馨提示：分页查询和数据库的一页 16KB 中的 "页" 是两个概念。

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uIsuHEycJN1P86UXyO7IHABy47T2xgJAXicGDDkBW0459lYjzH1W9QPA/640?wx_fmt=png)

注意：`innodb_page_size`变量在服务器运行过程中不可以更改，只能在第一次初始化 MySQL 数据目录时指定。所以页在运行时的大小不可更改。

### 3. varchar 疑问千千万——InnoDB 行格式

看到这里，你一定有着和我相同的疑问，比如 varchar(255) 后面这个最大长度应该怎么选择呢？为什么不能`varchar(65535)`而最大只能`varchar(16383)`呢？我来带你看！

我们平时是以记录为单位来向表中插入数据的，这些记录在磁盘上的存放方式也被称为行格式或者记录格式。行格式有 4 种，分别是 Dynamic、Compact、Redundant 和 Compressed

MySQL 5 + 默认行格式都是 Dynamic， 在 MySQL 5 和 MySQL 8 经过验证确实是的。

```
SHOW VARIABLES LIKE "innodb_default_row_format"
```

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uvoL5ScUn3RZdKEUcFnAg1ic368OCBtm0PqTNUto2GXD5xBBCrkLIz9g/640?wx_fmt=png)

大家在业务中和平时使用中都几乎没有修改过或者注意过 InnoDB 行格式，那么我就只重点讲默认行格式 dynamic，让大家更深层次理解平时开发中的 varchar。

请记住这个表结构，后面会围绕这个来讲

```
CREATE TABLE test ( 
c1 VARCHAR(10), 
c2 VARCHAR(10) NOT NULL, 
c3 CHAR(10), 
c4 VARCHAR(10)) CHARSET = utf8mb4;
```

现在业务数据库字符集都是 utf8mb4，我就以这个来讲，把理解难度降到最低。

```
INSERT INTO test ( c1, c2, c3, c4 )
VALUES('aaaa', '你好啊', 'cc', 'd'),('eeee', 'fff', NULL, NULL);
```

现在，表中的记录就是这样

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uZWxVZV9C5l0EF1q0KaezglmZCs0ABrr1axj75W0Uf5nyFbQxhsBHNg/640?wx_fmt=png)

#### 3.1 dynamic——innodb 默认行格式

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8ubuLhPPsbcBficQDsZSFOAsRmum3iaKlvhNOONdDMD8wHsXXA1FoDrHQQ/640?wx_fmt=png)

关于记录的额外信息这部分，是服务器为了描述这条记录而不得不额外添加的一些信息，这些额外信息分为 3 类，分别是变长字段长度列表、NULL 值列表和记录头信息。

在这里我只讲变长字段长度列表、NULL 值列表。因为记录头信息非常的绕和本篇没多大关系。

#### 3.2 innodb 怎么知道 varchar 真正有多长？——变长字段长度列表

一些变长的数据类型，比如 VARCHAR(M)、各种 TEXT 类型，各种 BLOB 类型，变长数据类型的字段中存储多少字节的数据是不固定的，在存储真实数据的时候需要把这些数据占用的字节数也存起来。

就像设计 String 类型，不仅仅是存放真实数据的 char 数组，还有 length 变量去记录字符串长度。又比如 input 输入框最大限制 500 字，但是你还得有一个变量去统计真实在输入框内有多少字符。同理，varchar 也有记录真实数据长度的变量（假设为 L，后文沿用方便描述），L 表示 varchar 真实占用的字节数，innodb 最多分配 2 个字节去表示这个 L，就像`unsigned short`类型，2 个字节，寄存器最多只有 16 位来让你存这个长度，所以 L 记录范围是`2^16 - 1 = 65535`。

这些变长字段 (比如 varchar) 占用的存储空间分为两部分：

*   真正的数据内容部分，放在对应的列
    
*   真实占用的字节数，放在变长字段列表部分
    

我们拿 test 表中的第一条记录来举个例子。因为 test 表的 c1、c2、c4 列都是 VARCHAR(10) 类型的，说明最大 10 个字符，所以这三个列的值的长度都需要保存在记录开头处，因为 test 表中的各个列都使用的是 utf8mb4 字符集，每个字符最大需要 4 个字节来进行编码（不使用 utf8 而是 utf8mb4 是因为可能存储 emoji 表情，如果只是文字，utf8 就足够），来看一下第一条记录各变长字段内容的长度：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uVIvRLagn9N0q0XyxiatFfIXX3ibkCMT9WGkJObDzUiamfGCerniapWWNFQ/640?wx_fmt=png)

怎么确定这些字段有多少字节？

比如这里 c2 的 "你好啊"，使用如下 sql 可以确定

```
SELECT LENGTH(c2) from test where c1='aaaa';
```

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8u6vl9eonFNs3kQLqmfmxqMIU18rq0icNic9ZjOvJE37FH2vJV6VnLegEQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uhd0wRiciaGibPibRnicLdTmhqC9Wkxy7VWYq7ww7PeRUHVpEefto8vk1DNw/640?wx_fmt=png)

各变长字段数据占用的字节数按照列的顺序逆序存放！！

由于第一行记录中 c1、c2、c4 列中的字符串都比较短，也就是说 varchar 真实占用的字节数比较小，L 用 1 个字节 (8 个 bit 位) 就可以表示，但是如果 varchar 真实占用的字节数比较多，L 可能就需要用 2 个字节 (16 个 bit 位) 来表示。到底 varchar 能存多少字节呢？继续往下看。

#### 3.3 varchar(M) 能存多少个字符，为什么提示最大 16383？

首先要理解 varchar(M) 的 M 是说字符个数，而不是字节。

为什么不能 varchar(20000) 之类的，是 20000 个字符放不下吗？

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uGM851iampDR8coGLLGsLMMaicWFKVnjxibdG5ic9LT38pKT0TfImibSI8ibQ/640?wx_fmt=png)

为什么提示只能最大 16383 个字符呢？这个数字是怎么算出来的？

**这个我就得和你好好唠嗑了！**

varchar 是变长的，varchar(64) 能存放 0~64 个字符不等，并不一定是存了最大 64 个字符，谁知道这个类型到底存了几个字符呢？

innodb 设计的时候，就已经考虑到了，不过是用字节作为单位，后续我们可以根据对应字符集转变为字符来理解，innodb 必须记录变长字段 varchar 真实占用的字节数 L。前面说过了，innodb 最多分配 2 个字节 (16 个 bit 位) 的空间去记录这个 L。

InnoDB 有它的一套规则，我们引入 W、M 和 L 这几个符号：

1.  假设某个字符集中最多需要 W 字节来表示一个字符
    

*   utf8mb4 字符集中的 W 就是 4
    
*   utf8 字符集中 W 就是 3
    
*   gbk 字符集中的 W 就是 2
    
*   ascii 字符集中的 W 就是 1。
    

3.  对于变长类型 VARCHAR(M) 来说，这种类型表示能存储最多 M 个字符（注意是字符不是字节）
    

*   所以这个类型能表示的字符串最多占用的字节数就是 M × W。
    

5.  假设它实际存储的字符串占用的字节数是 L。
    

来看极限边界情况，innodb 为了记录一下 varchar 真实存储多少个字节，最多分配 2 个字节的空间去记录，2 个字节 16 个比特位，全部为 1，最大能记录的数字是`2^16-1`是 65535 个，innodb 最大能记录 varchar 占用的字节数就是 65535 个，utf8mb4 字符集一个字符是最大是 4 个字节，`65535 / 4 = 16383.75`，只要 varchar 字符数不超过 16383 个，innodb 就可以记录真实占用的长度 L，再多就记录不了了！所以就能解释刚刚的图了，`varchar(20000)`不行，最大也就 16383 个字符

但是！这里强调是有但是的！

行最大长度是 65535 字节，行里面有很多东西，包括变长字段列表、NULL 值列表、记录头信息。你得考虑该字段如果允许为 NULL，NULL 值列表会占用一个字节 (只要没超过 8 个字段)，每一列字段的变长字段实际长度会花费 1~2 个字节，如果该字段的数据太大，会变成溢出列，该字段的数据会分成很多行存储（后面会讲，你可以看完 NULL 值列表和溢出列后再回来看这个例子）。所以即便提示 16383 个字符，你也绝对不可能存到 16383。

我做了个测试

```
create table t2 ( name varchar(16383))charset=utf8mb4;
```

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8u6YJ4F8ibrhRiaLEcGtgaOCrvPwhvOJ6iaCmjfRRBqw64HlmzZlysBcK2A/640?wx_fmt=png)

不断往这个字段添加字符保存测试，最后发现，这些字符总长度到极限也就是 48545 字节。

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uACvINoY9HjqMm0D7lEicgNiaIu5GMfkic2NzP9RLpJ030eMyRWqNUibyQw/640?wx_fmt=png)

如果超过就会报错

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8u2hcGF23iccz8ETicjVkwcicibYz8vJsy0iam68jiboc4amxOic5luA6ia3xSKQ/640?wx_fmt=png)

这里 48545 个字节，再多一个字符就会报错，远不到 65535 字节，差了 1W 多字节。主要是因为溢出列的原因，数据分散在不同的行中，所以，很长的数据，建议往 text 类型考虑。这个现象可以看出，varchar(M) 的 M 很大，实际是达不到 M 这个边界值的。

下面说明一下规则 (讲解中字符集用 utf8mb4，W=4)

**规则一：如果允许存储的最大字节数 M × W <= 255，varchar 占用的真实字节数 L 只分配 1 个字节来表示。**

有人说，允许存储的最大字节数`M × W <= 255`，即允许存储的最大字符数 <= `⌊255 / 4⌋ = 63`个时，varchar 占用的真实字节数 L 仅分配 1 个字节就能表示。这个结论正确吗？

显然错误，因为这里`255 / 4`，你怎么知道每个存储的一个字符是 4 个字节呢？难道全部存的 emoji 表情？不存字母汉字啥的？

InnoDB 在读记录的变长字段长度列表时先查看表结构，如果某个变长字段允许存储的最大字节数不大于 255 时，只用 1 个字节来表示真实数据占用的字节。

**规则二：如果允许存储的最大字节数 M × W > 255，则分为两种情况：**

如果实际存储字节 L <= 127，varchar 占用的真实字节数 L 仅分配 1 个字节就能表示。(⌊ … ⌋表示向下取整)

有人说，实际存储字节 L <= 127，即实际存储字符 <= ⌊127 / 4⌋ = 31 个时，varchar 占用的真实字节数 L 仅分配 1 个字节就能表示。这个结论正确吗？

显然错误，因为这里 127 / 4，你怎么知道实际存储的一个字符是 4 个字节呢？难道全部存的 emoji 表情？不存字母汉字啥的？

如果实际存储字节 L > 127，varchar 占用的真实字节数 L 需要分配 2 个字节才能表示。

> 另外需要注意的是，变长字段列表只存储非 NULL 的列的长度。

表记录是这样的

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uZWxVZV9C5l0EF1q0KaezglmZCs0ABrr1axj75W0Uf5nyFbQxhsBHNg/640?wx_fmt=png)

对于第二条记录，c4 列值为 NULL，所以只存储 c1 和 c2 列即可。

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uDHJeE2RsVBGVic7q1D8CoFBkywf7WNuzP3aQXsNXhicUVB9Iic5XRGV6Q/640?wx_fmt=png)

第一条记录的变长字段长度列表部分占用 3 字节空间，因为有 c1、c2、c4 列，且内容都很少，每列真实占用字节数用 1 个字节可以表示，加起来就是 3 个字节，第二条记录变长字段长度列表部分占用 2 字节。

当然，并不是所有记录都有这个变长字段长度列表部分，比方说表中所有的列都不是变长的数据类型或者 所有列的值都是 NULL 的话，这一部分就不需要有。实际业务开发中，几乎没有不使用 varchar 的，所以实际开发中的记录都会有变长字段长度列表部分

#### 3.4 记录为 NULL，innodb 如何处理？——NULL 值列表

能仔细看到这里，你肯定是个高手了。如果你和我一样开发规范中不推荐 NULL，一般都写 NOT NULL，其实记录中就不存在 NULL 值列表了，也节省了空间。

如果表中的某些列可能存储 NULL 值，把这些 NULL 值都放到记录的真实数据中存储会很占地方，所以 dynamic 行格式把这些值为 NULL 的列统一管理起来，存储到 NULL 值列表中，它的处理过程是这样的：

1. 统计表中允许存储 NULL 的列有哪些。

主键列、被 NOT NULL 修饰的列都是不可以存储 NULL 值的，所以在统计的时候不会把这些列算进去。比方说表 test 的 3 个列 c1、c3、c4 都是允许存储 NULL 值的，而 c2 列是被 NOT NULL 修饰，不允许存储 NULL 值。

2. 如果表中没有允许存储 NULL 的列，则 NULL 值列表也不存在了，否则将每个允许存储 NULL 的列对应一个二进制位，二进制位按照列的顺序逆序排列。二进制位的值为 1 时，代表该列的值为 NULL，为 0 时，代表该列的值不为 NULL。因为表 test 的 c1、c3、c4 都是允许存储 NULL 值的允许为 NULL 的列，所以这 3 个列和二进制位的对应关系就是这样：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uSOX5eic3pDxYrMoLlYeDYyL9tKdTXVicOYYFd9HZWwdEiaeydnOFA99UQ/640?wx_fmt=png)

3.NULL 值列表必须用整数个字节的位表示，如果使用的二进制位个数不是整数个字节，则在字节的高位补 0。

也就是说，表 test 只有 3 个字段允许为 NULL，对应 3 个二进制位，不足 1 字节，那么就在高位补 0 即可。

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uNVkBM18xEl6icgd3jUYfWo5WwxATHVduQtj8IwqXZibKibl7VDMxpWnCw/640?wx_fmt=png)

以此类推，如果表中有 9 个字段都允许为 NULL，那么这个记录的 NULL 值列表就需要 2 个字节来表示，高字节高位补 0。

对于第一条记录，c1、c3、c4 都不为 NULL，对应的为进制位为 0，十六进制表示就是 0x00

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uNvMmNLsWR3h8WLZcLUZw93I6DwldTfDScwHgAUoicptQeYTPshEwicuw/640?wx_fmt=png)

对于第二条记录，c3、c4 都是 NULL，对应的二进制位为 1，十六进制表示就是 0x06

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8ubFQtSaxQJt683ib1TRYLPLmCiaNucTGaIPsQZqnFrmpicgZ4fJdSo4J9A/640?wx_fmt=png)

这两条记录在填充了 NULL 值列表后示意图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbufL11ic2nVEsXiaGBvS3fhc8uNyTpp6wxEicycETVMchSBzSJbBVPQWEC3qt3Bp1icJC4CJegYoTRYWuA/640?wx_fmt=png)

#### 3.5 某个列数据占用的字节数非常多怎么办？——dynamic 行格式的溢出列

如果某个列中存储的数据占用的字节数非常多，该列就可能称为溢出列。

对于占用存储空间非常多的列，在记录真实数据时，该列只会用 20 字节空间，而这 20 字节的空间不存储数据，因为数据都分散存储在其他几行中了。这 20 字节的空间存储的是分散行的地址和占用的字节数。分散行记录是单链表连接的结构。

### 后续

如果大家对 innodb 存储结构其他行格式感兴趣，或者我没说的记录头信息，可以去阅读《MySQL 是怎样运行的》一书，我和书中不同的是，书中讲的 Compact 格式，字符集是 ascii，我选用的是平时开发中用到的默认 dynamic 格式，字符集是 utf8mb4，字符集变化后所有的数据我在文中和图中都有重新计算。大家平时或许没关注过行格式，那么就是按照 dynamic 格式理解就可以，更贴近实际开发。