> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3NDgxMzI0Mw==&mid=2247499337&idx=2&sn=51235bd526527579bf0fa3b4cec98b33&chksm=fd2e1b1dca59920b96ad3662b511050826dc27316b6f1c1a53e470a596067070561b322f5768&mpshare=1&scene=1&srcid=0817f6TDevziaXKLSxjUI1gs&sharer_sharetime=1629144872021&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来自：blog.csdn.net/mhmyqn/article/details/48653157

在看高性能 MySQL 第 3 版（4.1.7 节）时，作者建议**当存储 IPv4 地址时，应该使用 32 位的无符号整数（UNSIGNED INT）来存储 IP 地址，而不是使用字符串。** 但是没有给出具体原因。为了搞清楚这个原因，查了一些资料，记录下来。

相对字符串存储，使用无符号整数来存储有如下的好处：

*   节省空间，不管是数据存储空间，还是索引存储空间
    
*   便于使用范围查询（BETWEEN...AND），且效率更高
    

通常，在保存 IPv4 地址时，一个 IPv4 最小需要 7 个字符，最大需要 15 个字符，所以，使用`VARCHAR(15)`即可。MySQL 在保存变长的字符串时，还需要额外的一个字节来保存此字符串的长度。而如果使用无符号整数来存储，只需要 4 个字节即可。

另外还可以使用 4 个字段分别存储 IPv4 中的各部分，但是通常这不管是存储空间和查询效率应该都不是很高（可能有的场景适合使用这种方式存储）。

使用字符串和无符号整数来存储 IP 的具体性能分析及 benchmark，可以看这篇文章。

> https://bafford.com/2009/03/09/mysql-performance-benefits-of-storing-integer-ip-addresses/

使用无符号整数来存储也有缺点：

*   不便于阅读
    
*   需要手动转换
    

对于转换来说，MySQL 提供了相应的函数来把字符串格式的 IP 转换成整数`INET_ATON`，以及把整数格式的 IP 转换成字符串的`INET_NTOA`。如下所示：

```
mysql> select inet_aton('192.168.0.1');
+--------------------------+
| inet_aton('192.168.0.1') |
+--------------------------+
|               3232235521 |
+--------------------------+
1 row in set (0.00 sec)

mysql> select inet_ntoa(3232235521);
+-----------------------+
| inet_ntoa(3232235521) |
+-----------------------+
| 192.168.0.1           |
+-----------------------+
1 row in set (0.00 sec)
```

对于 IPv6 来说，使用`VARBINARY`同样可获得相同的好处，同时 MySQL 也提供了相应的转换函数，即`INET6_ATON`和`INET6_NTOA`。

对于转换字符串 IPv4 和数值类型，可以放在应用层，下面是使用 java 代码来对二者转换：

```
package com.mikan;

/**
 * @author Mikan
 */
public class IpLongUtils {
    /**
     * 把字符串IP转换成long
     *
     * @param ipStr 字符串IP
     * @return IP对应的long值
     */
    public static long ip2Long(String ipStr) {
        String[] ip = ipStr.split("\\.");
        return (Long.valueOf(ip[0]) << 24) + (Long.valueOf(ip[1]) << 16)
                + (Long.valueOf(ip[2]) << 8) + Long.valueOf(ip[3]);
    }

    /**
     * 把IP的long值转换成字符串
     *
     * @param ipLong IP的long值
     * @return long值对应的字符串
     */
    public static String long2Ip(long ipLong) {
        StringBuilder ip = new StringBuilder();
        ip.append(ipLong >>> 24).append(".");
        ip.append((ipLong >>> 16) & 0xFF).append(".");
        ip.append((ipLong >>> 8) & 0xFF).append(".");
        ip.append(ipLong & 0xFF);
        return ip.toString();
    }

    public static void main(String[] args) {
        System.out.println(ip2Long("192.168.0.1"));
        System.out.println(long2Ip(3232235521L));
        System.out.println(ip2Long("10.0.0.1"));
    }

}
```

输出结果为：

```
192.168.0.1
```

<END>