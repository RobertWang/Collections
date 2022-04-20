> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.sulvblog.cn](https://www.sulvblog.cn/posts/tech/redis_5_data_types/)

> Redis 中 5 种常用的数据类型，字符串、列表、集合、散列、有序集合

> 本文参考《Redis 实战》

### 一、5 种数据结构命令

#### 1. 字符串

字符串可以存储三种类型的值：**字节串** (byte string)，**整数**，**浮点数**。

存储方式：键值对

字符串命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>get</td><td>获取存储在给定键中的值</td></tr><tr><td>set</td><td>设置存储在给定键中的值</td></tr><tr><td>del（这个命令适用于 5 种数据结构）</td><td>删除存储在给定键中的值</td></tr></tbody></table>

```
127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> get key1
"value1"
127.0.0.1:6379> del key1
(integer) 1
127.0.0.1:6379> get key1
(nil)
127.0.0.1:6379> del key1
(integer) 0


```

自增和自减命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>incr</td><td>将键存储的值加 1</td></tr><tr><td>decr</td><td>将键存储的值减 1</td></tr><tr><td>incrby</td><td>将键存储的值加上整数</td></tr><tr><td>decrby</td><td>将键存储的值减去整数</td></tr><tr><td>incrbyfloat</td><td>将键存储的值加上浮点数</td></tr></tbody></table>

```
127.0.0.1:6379> incr key1
(integer) 1
127.0.0.1:6379> incrby key1 10
(integer) 11
127.0.0.1:6379> decr key1
(integer) 10
127.0.0.1:6379> decrby key1 5
(integer) 5
127.0.0.1:6379> incrbyfloat key1 10.5
"15.5"
127.0.0.1:6379> incrby key1 10.5
(error) ERR value is not an integer or out of range


```

处理子串和二进制位的命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>append</td><td>将值追加到给定键当前存储的末尾</td></tr><tr><td>getrange</td><td>获取一个由偏移量 start 到偏移量 end 范围内所有字符组成的子串，包括 start 和 end 在内 (闭区间)</td></tr><tr><td>setrange</td><td>将从 start 偏移量开始的子串设置为给定值</td></tr><tr><td>getbit</td><td>将字节串看作是二进制位串，并返回位串中偏移量为 offset 的二进制位的值</td></tr><tr><td>setbit</td><td>将字节串看作是二进制位串，并将位串中偏移量为 offset 的二进制位的值设置为 value</td></tr><tr><td>bitcount</td><td>统计二进制位串里面值为 1 的二进制位的数量，可设置 start 和 end 指定查询范围</td></tr><tr><td>bitop</td><td>可对一个或多个二进制位串执行 and、or、异或 (xor)、not 等按位运算操作，并将结果保存在键里</td></tr></tbody></table>

```
127.0.0.1:6379> append key1 a
(integer) 1
127.0.0.1:6379> append key1 1
(integer) 2
127.0.0.1:6379> get key1
"a1"
127.0.0.1:6379> getrange key1 0 1
"a1"
127.0.0.1:6379> setrange key1 1 bbb
(integer) 4
127.0.0.1:6379> get key1
"abbb"
127.0.0.1:6379> del key1
(integer) 1
//A对应的十进制为65，二进制为01000001
127.0.0.1:6379> set key1 A
OK
127.0.0.1:6379> get key1
"A"
127.0.0.1:6379> getbit key1 0
(integer) 0
127.0.0.1:6379> getbit key1 1
(integer) 1
127.0.0.1:6379> getbit key1 2
(integer) 0
127.0.0.1:6379> getbit key1 3
(integer) 0
127.0.0.1:6379> getbit key1 4
(integer) 0
127.0.0.1:6379> getbit key1 5
(integer) 0
127.0.0.1:6379> getbit key1 6
(integer) 0
127.0.0.1:6379> getbit key1 7
(integer) 1
//现在把从左往右数下标为6的位改为1，二进制为010000110，十进制为67，字符为C
127.0.0.1:6379> setbit key1 6 1
(integer) 0
127.0.0.1:6379> get key1
"C"
127.0.0.1:6379> bitcount key1
(integer) 3


```

#### 2. 列表

列表和数据结构中的队列类似。

常用命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>rpush</td><td>将一个或多个值推入列表的右端</td></tr><tr><td>lpush</td><td>将一个或多个值推入列表的左端</td></tr><tr><td>rpop</td><td>移除并返回列表最左端的元素</td></tr><tr><td>lpop</td><td>移除并返回列表最右端的元素</td></tr><tr><td>lindex</td><td>返回列表中偏移量 offset 的元素</td></tr><tr><td>lrange</td><td>返回列表中偏移量从 start 到 end 范围内的所有元素，闭区间</td></tr><tr><td>ltrim</td><td>只保留偏移量 start 到 end 范围内的元素，闭区间</td></tr></tbody></table>

```
127.0.0.1:6379> rpush key1 r1 r2 r3
(integer) 3
127.0.0.1:6379> lpush key1 l1 l2 l3
(integer) 6
127.0.0.1:6379> lrange key1 0 -1
1) "l3"
2) "l2"
3) "l1"
4) "r1"
5) "r2"
6) "r3"
127.0.0.1:6379> rpop key1
"r3"
127.0.0.1:6379> lpop key1
"l3"
127.0.0.1:6379> lrange key1 0 -1
1) "l2"
2) "l1"
3) "r1"
4) "r2"
127.0.0.1:6379> lindex key1 1
"l1"
127.0.0.1:6379> ltrim key1 1 2
OK
127.0.0.1:6379> lrange key1 0 -1
1) "l1"
2) "r1"


```

阻塞式的列表弹出命令以及在列表之间移动元素的命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>blpop</td><td>从第一个非空列表中弹出位于最左端的元素，或者在 timeout 秒之内阻塞并等待可弹出的元素出现</td></tr><tr><td>brpop</td><td>从第一个非空列表中弹出位于最右端的元素，或者在 timeout 秒之内阻塞并等待可弹出的元素出现</td></tr><tr><td>rpoplpush</td><td>从 source 列表中弹出位于最左端的元素，并推入 dest 列表的最右端，并向用户返回这个元素</td></tr><tr><td>brpoplpush</td><td>从 source 列表中弹出位于最左端的元素，并推入 dest 列表的最右端，并向用户返回这个元素，如果 source 列表为空，那么在 timeout 秒之内阻塞并等待可弹出的元素出现</td></tr></tbody></table>

```
127.0.0.1:6379> lrange key1 0 -1
1) "1"
2) "2"
127.0.0.1:6379> blpop key1 5
1) "key1"
2) "1"
127.0.0.1:6379> blpop key1 5
1) "key1"
2) "2"
127.0.0.1:6379> blpop key1 5
(nil)
(5.06s)
127.0.0.1:6379> lrange key2 0 -1
1) "6"
2) "7"
3) "8"
4) "9"
5) "0"
127.0.0.1:6379> blpop key1 key2 5
1) "key2"
2) "6"
127.0.0.1:6379> rpoplpush key2 key1
"0"
127.0.0.1:6379> lrange key1 0 -1
1) "0"
127.0.0.1:6379> lrange key2 0 -1
1) "7"
2) "8"
3) "9"
127.0.0.1:6379> brpoplpush key1 key2 5
"0"
127.0.0.1:6379> brpoplpush key1 key2 5
(nil)
(5.00s)
127.0.0.1:6379> lrange key1 0 -1
(empty array)
127.0.0.1:6379> lrange key2 0 -1
1) "0"
2) "7"
3) "8"
4) "9"


```

#### 3. 集合

集合与列表的不同之处在于，列表可以存储多个相同的字符串，而集合则通过使用散列表来保证每个字符串都是不相同的，集合使用无序的方式存储元素，不能像列表一样从一端插入，另一端弹出。

常用命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>sadd</td><td>将一个或多个元素添加到集合，并返回与集合元素不重复的元素数量</td></tr><tr><td>srem</td><td>将一个或多个元素从集合移除，并返回被移除元素的数量</td></tr><tr><td>sismember</td><td>检查某个元素是否存在于集合中</td></tr><tr><td>scard</td><td>返回集合包含的元素的数量</td></tr><tr><td>smembers</td><td>返回集合包含的所有元素</td></tr><tr><td>srandmember</td><td>从集合里随机返回一个或多个元素，当 count 为正数时，返回的随机元素不会重复，当 count 为负数时，返回的随机元素可能会重复</td></tr><tr><td>spop</td><td>随机的移除集合中的一个元素，并返回被移除的元素</td></tr><tr><td>smove</td><td>如果 source 集合包含元素 item，则从 source 集合移除元素 item，并将元素添加到集合 dest，移除成功返回 1，失败返回 0</td></tr></tbody></table>

```
127.0.0.1:6379> sadd key1 1 2 3 4
(integer) 4
127.0.0.1:6379> srem key1 1 2
(integer) 2
127.0.0.1:6379> sismember key1 2
(integer) 0
127.0.0.1:6379> sismember key1 3
(integer) 1
127.0.0.1:6379> scard key1
(integer) 2
127.0.0.1:6379> smembers key1
1) "3"
2) "4"
127.0.0.1:6379> srandmember key1 1
1) "4"
127.0.0.1:6379> srandmember key1 2
1) "3"
2) "4"
127.0.0.1:6379> srandmember key1 -1
1) "4"
127.0.0.1:6379> srandmember key1 -2
1) "3"
2) "3"
127.0.0.1:6379> srandmember key1 -2
1) "3"
2) "4"
127.0.0.1:6379> spop key1
"4"
127.0.0.1:6379> smembers key1
1) "3"
127.0.0.1:6379> del key1
(integer) 1
127.0.0.1:6379> sadd key1 a b c d
(integer) 4
127.0.0.1:6379> sadd key2 e f g h
(integer) 4
127.0.0.1:6379> smove key1 key2 a
(integer) 1
127.0.0.1:6379> smove key1 key2 z
(integer) 0
127.0.0.1:6379> smembers key1
1) "b"
2) "c"
3) "d"
127.0.0.1:6379> smembers key2
1) "h"
2) "g"
3) "f"
4) "e"
5) "a"


```

用于组合和处理多个集合的命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>sdiff</td><td>返回那些存在于第一个集合但不存在于其他集合中的元素（差集）</td></tr><tr><td>sdiffstore</td><td>将那些存在于第一个集合但不存在于其他集合中的元素存储到 dest 键里面（差集）</td></tr><tr><td>sinter</td><td>返回那些同时存在于所有集合中的元素（交集）</td></tr><tr><td>sinterstore</td><td>将那些同时存在于所有集合中的元素存储到 dest 键里（交集）</td></tr><tr><td>sunion</td><td>返回那些至少存在于一个集合中的元素（并集）</td></tr><tr><td>sunionstore</td><td>将那些至少存在于一个集合中的元素存储到 dest 键里（并集）</td></tr></tbody></table>

```
127.0.0.1:6379> sadd key1 a b c d e f g
(integer) 7
127.0.0.1:6379> sadd key2 a b c h i j k
(integer) 7
127.0.0.1:6379> sdiff key1 key2
1) "g"
2) "e"
3) "f"
4) "d"
127.0.0.1:6379> sdiffstore dest key1 key2
(integer) 4
127.0.0.1:6379> smembers dest
1) "g"
2) "e"
3) "f"
4) "d"
127.0.0.1:6379> sinter key1 key2
1) "a"
2) "b"
3) "c"
127.0.0.1:6379> sinterstore dest2 key1 key2
(integer) 3
127.0.0.1:6379> smembers dest2
1) "b"
2) "c"
3) "a"
127.0.0.1:6379> sunion key1 key2
 1) "g"
 2) "d"
 3) "e"
 4) "i"
 5) "j"
 6) "b"
 7) "f"
 8) "a"
 9) "k"
10) "c"
11) "h"
127.0.0.1:6379> sunionstore dest3 key1 key2
(integer) 11
127.0.0.1:6379> smembers dest3
 1) "g"
 2) "d"
 3) "e"
 4) "i"
 5) "j"
 6) "b"
 7) "f"
 8) "a"
 9) "k"
10) "c"
11) "h"


```

#### 4. 散列

Redis 的散列可以存储多个键值对之间的映射，和字符串一样，散列存储的值既可以是字符串也可以是数字值，并且同样可以对数字值执行自增和自减的操作。

常用命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>hset</td><td>在散列里面关联起给定的键值对</td></tr><tr><td>hget</td><td>获取指定散列键的值</td></tr><tr><td>hgetall</td><td>获取散列包含的所有键值对</td></tr><tr><td>hdel</td><td>如果给定键存在于散列里面，那么移除这个键</td></tr></tbody></table>

```
127.0.0.1:6379> hset key1 k1 v1
(integer) 1
127.0.0.1:6379> hset key1 k2 v1
(integer) 1
127.0.0.1:6379> hset key1 k3 v3
(integer) 1
127.0.0.1:6379> hgetall key1
1) "k1"
2) "v1"
3) "k2"
4) "v1"
5) "k3"
6) "v3"
127.0.0.1:6379> hget key1 k2
"v1"
127.0.0.1:6379> hdel key1 k2
(integer) 1
127.0.0.1:6379> hgetall key1
1) "k1"
2) "v1"
3) "k3"
4) "v3"


```

用于添加和删除键值对的散列操作

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>hmget</td><td>从散列里面获取一个或多个键的值</td></tr><tr><td>hmset</td><td>为散列里面的一个或多个键设置值</td></tr><tr><td>hdel</td><td>删除散列里面的一个或多个键值对，返回成功找到并删除的键值对的数量</td></tr><tr><td>hlen</td><td>返回散列包含的键值对数量</td></tr></tbody></table>

```
127.0.0.1:6379> hmset key2 k1 v1 k2 v2 k3 v3 k4 v4
OK
127.0.0.1:6379> hmget key2 k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> hlen key2
(integer) 4
127.0.0.1:6379> hdel key2 k1 k2 k3 k4
(integer) 4
127.0.0.1:6379> hlen key2
(integer) 0


```

散列更高级的特性

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>hexists</td><td>检查给定键是否存在与散列中</td></tr><tr><td>hkeys</td><td>获取散列包含的所有键</td></tr><tr><td>hvals</td><td>获取散列包含的所有值</td></tr><tr><td>hgetall</td><td>获取散列包含的所有键值对</td></tr><tr><td>hincrby</td><td>将键 key 存储的值加上整数 increment</td></tr><tr><td>hincrbyfloat</td><td>将键 key 存储的值加上浮点数 increment</td></tr></tbody></table>

```
127.0.0.1:6379> hmset key1 k1 1 k2 2 k3 3 k4 4
OK
127.0.0.1:6379> hexists key1 k3
(integer) 1
127.0.0.1:6379> hkeys key1
1) "k1"
2) "k2"
3) "k3"
4) "k4"
127.0.0.1:6379> hvals key1
1) "1"
2) "2"
3) "3"
4) "4"
127.0.0.1:6379> hgetall key1
1) "k1"
2) "1"
3) "k2"
4) "2"
5) "k3"
6) "3"
7) "k4"
8) "4"
127.0.0.1:6379> hincrby key1 k1 10
(integer) 11
127.0.0.1:6379> hincrbyfloat key1 k1 10
"21"
127.0.0.1:6379> 


```

#### 5. 有序集合

有序集合的键称为成员 (member)，值称为分值 (score)，分值必须为浮点数，有序集合成员不允许重复，但分值却可以重复，有序集合通过分值为集合中的成员进行从大到小的排序。

常用命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>zadd</td><td>将带有给定分值的成员添加到有序集合里面</td></tr><tr><td>zrem</td><td>从有序集合里面移除给定的成员，并返回被移除成员的数量</td></tr><tr><td>zcard</td><td>返回有序集合包含的成员数量</td></tr><tr><td>zincrby</td><td>将 member 成员的分值加上 increment</td></tr><tr><td>zcount</td><td>返回分值介于 min 和 max(这里指的是分值) 之间的成员数量</td></tr><tr><td>zrank</td><td>返回成员 member 在有序集合中的排名</td></tr><tr><td>zscore</td><td>返回成员 member 的分值</td></tr><tr><td>zrange</td><td>返回有序集合中排名介于 start 和 stop(这里指的是下标索引，而不是分值) 之间的成员，如果给定了可选的 withscores 选项，那么命令会将成员的分值也一并返回</td></tr></tbody></table>

```
127.0.0.1:6379> zadd key1 5 m5 4 m4 3 m3 2 m2 1 m1
(integer) 5
127.0.0.1:6379> zcard key1
(integer) 5
127.0.0.1:6379> zrange key1 0 4 withscores
 1) "m1"
 2) "1"
 3) "m2"
 4) "2"
 5) "m3"
 6) "3"
 7) "m4"
 8) "4"
 9) "m5"
10) "5"
127.0.0.1:6379> zrem key1 m5
(integer) 1
127.0.0.1:6379> zrange key1 0 3 withscores
1) "m1"
2) "1"
3) "m2"
4) "2"
5) "m3"
6) "3"
7) "m4"
8) "4"
127.0.0.1:6379> zincrby key1 10 m1
"11"
127.0.0.1:6379> zrange key1 0 3 withscores
1) "m2"
2) "2"
3) "m3"
4) "3"
5) "m4"
6) "4"
7) "m1"
8) "11"
127.0.0.1:6379> zcount key1 0 11
(integer) 4
127.0.0.1:6379> zrank key1 m2
(integer) 0
127.0.0.1:6379> zrank key1 m3
(integer) 1
127.0.0.1:6379> zrank key1 m4
(integer) 2
127.0.0.1:6379> zrank key1 m1
(integer) 3
127.0.0.1:6379> zscore key1 m1
"11"


```

有序集合的范围型数据获取命令和范围型数据删除命令，以及并集命令的交集命令

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>zrevrank</td><td>返回有序集合里成员 member 的排名，成员按照分值从大到小排序</td></tr><tr><td>zrevrange</td><td>返回有序集合给定排名范围内的成员，成员按照分值从大到小排序</td></tr><tr><td>zrangebyscore</td><td>返回有序集合中分值介于 min 和 max 之间的所有成员，从小到大</td></tr><tr><td>zrevrangebyscore</td><td>返回有序集合中分值介于 max 和 min 之间的所有成员，从大到小</td></tr><tr><td>zremrangebyrank</td><td>移除有序集合中排名介于 start 和 stop 之间的所有成员</td></tr><tr><td>zremrangebyscore</td><td>移除有序集合中分数介于 min 和 max 之间的所有成员</td></tr><tr><td>zinterstore</td><td>对指定的有序集合执行类似于集合的交集运算，这里的交集运算表示对指定数量的有序集合中键的分值相加，并把结果输出到 destination 组成新的有序集合</td></tr><tr><td>zunionstore</td><td>对给定的有序集合执行类似于集合的并集运算，这里的并集运算指在指定数量个有序集合都包含同一个成员的情况下，可在末尾加上 aggregate sum|min|max，会将 和 | 最小 | 最大 的那个分值输出到 destination 组成新的有序集合</td></tr></tbody></table>

```
127.0.0.1:6379> zadd key1 1 k1 3 k3 2 k2 5 k5 4 k4
(integer) 5
127.0.0.1:6379> zrevrank key1 k5
(integer) 0
127.0.0.1:6379> zrevrank key1 k1
(integer) 4
127.0.0.1:6379> zrevrange key1 0 4
1) "k5"
2) "k4"
3) "k3"
4) "k2"
5) "k1"
127.0.0.1:6379> zrevrange key1 0 4 withscores
 1) "k5"
 2) "5"
 3) "k4"
 4) "4"
 5) "k3"
 6) "3"
 7) "k2"
 8) "2"
 9) "k1"
10) "1"
127.0.0.1:6379> zrangebyscore key1 1 5
1) "k1"
2) "k2"
3) "k3"
4) "k4"
5) "k5"
127.0.0.1:6379> zrangebyscore key1 1 5 withscores
 1) "k1"
 2) "1"
 3) "k2"
 4) "2"
 5) "k3"
 6) "3"
 7) "k4"
 8) "4"
 9) "k5"
10) "5"
127.0.0.1:6379> zrevrangebyscore key1 5 1 withscores
 1) "k5"
 2) "5"
 3) "k4"
 4) "4"
 5) "k3"
 6) "3"
 7) "k2"
 8) "2"
 9) "k1"
10) "1"
127.0.0.1:6379> zremrangebyrank key1 1 2
(integer) 2
127.0.0.1:6379> zrange key1 0 -1 withscores
1) "k1"
2) "1"
3) "k4"
4) "4"
5) "k5"
6) "5"
127.0.0.1:6379> zremrangebyscore key1 4 5
(integer) 2
127.0.0.1:6379> zrange key1 0 -1 withscores
1) "k1"
2) "1"

127.0.0.1:6379> del key1
(integer) 1

127.0.0.1:6379> zadd key1 1 k1 2 k2 3 k3 4 k4 5 k5
(integer) 5
127.0.0.1:6379> zadd key2 1 k1 2 k2 3 k3 4 k4 5 k5
(integer) 5
127.0.0.1:6379> zinterstore dest 2 key1 key2	//这里的数字2表示后面跟的键的个数
(integer) 5
127.0.0.1:6379> zrange dest 0 -1 withscores
 1) "k1"
 2) "2"
 3) "k2"
 4) "4"
 5) "k3"
 6) "6"
 7) "k4"
 8) "8"
 9) "k5"
10) "10"

127.0.0.1:6379> zadd key3 1 k1 2 k2 3 k3
(integer) 3
127.0.0.1:6379> zadd key4 0 k4 1 k3 4 k2
(integer) 3
127.0.0.1:6379> zunionstore dest2 2 key3 key4 aggregate sum
(integer) 4
127.0.0.1:6379> zrange dest2 0 -1 withscores
1) "k4"
2) "0"
3) "k1"
4) "1"
5) "k3"
6) "4"
7) "k2"
8) "6"
127.0.0.1:6379> zunionstore dest3 2 key3 key4 aggregate min
(integer) 4
127.0.0.1:6379> zrange dest3 0 -1 withscores
1) "k4"
2) "0"
3) "k1"
4) "1"
5) "k3"
6) "1"
7) "k2"
8) "2"
127.0.0.1:6379> zunionstore dest4 2 key3 key4 aggregate max
(integer) 4
127.0.0.1:6379> zrange dest4 0 -1 withscores
1) "k4"
2) "0"
3) "k1"
4) "1"
5) "k3"
6) "3"
7) "k2"
8) "4"


```

### 二、其他命令

#### 1. 排序

sort 命令可以对列表、集合、有序集合进行排序，类似 SQL 语言里的 order by 子句。

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>sort</td><td>sort source-key [by pattern] [limit offset count] [get pattern [get pattern …]] [asc|desc] [alpha] [store dest-key]，根据给定的选项，对输入列表、集合或有序集合进行排序，然后返回或者存储排序的结果</td></tr></tbody></table>

```
127.0.0.1:6379> rpush key1 1 9 2 8 3 7 4 6 5 0
(integer) 10
127.0.0.1:6379> sort key1 asc store dest
(integer) 10
127.0.0.1:6379> lrange dest 0 -1
 1) "0"
 2) "1"
 3) "2"
 4) "3"
 5) "4"
 6) "5"
 7) "6"
 8) "7"
 9) "8"
10) "9"
127.0.0.1:6379> sort key1 desc store dest1
(integer) 10
127.0.0.1:6379> lrange dest1 0 -1
 1) "9"
 2) "8"
 3) "7"
 4) "6"
 5) "5"
 6) "4"
 7) "3"
 8) "2"
 9) "1"
10) "0"
127.0.0.1:6379> rpush key2 7 15 23 110
(integer) 4
//根据字母表顺序对元素进行排序
127.0.0.1:6379> sort key2 alpha store dest4
(integer) 4
127.0.0.1:6379> lrange dest4 0 -1
1) "110"
2) "15"
3) "23"
4) "7"
127.0.0.1:6379> lrange key2 0 -1
1) "7"
2) "15"
3) "23"
4) "110"


```

#### 2. 基本的 Redis 事务

为了可以在不同类型之间移动元素，Redis 有 5 个命令可以让用户在不被打断的情况下对多个键执行操作，分别是：watch、multi、exec、unwatch、discard。

#### 4. 键的过期时间

有些数据可能在某个时间点之后就不再有用了，可以设置 Redis 的过期时间让一个键在给定的时限之后自动被删除。

<table><thead><tr><th>命令</th><th>描述</th></tr></thead><tbody><tr><td>persist</td><td>移除键的过期时间</td></tr><tr><td>ttl</td><td>查看给定键距离过期还有多少秒</td></tr><tr><td>expire</td><td>让给定键在指定的秒数之后过期</td></tr><tr><td>expireat</td><td>将给定键的过期时间设置为给定的 UNIX 时间戳</td></tr><tr><td>pttl</td><td>查看给定键距离过期时间还有多少毫秒</td></tr><tr><td>pexpire</td><td>让给定键在指定的毫秒数之后过期</td></tr><tr><td>pexpireat</td><td>将一个毫秒级精度的 UNIX 时间戳设置为给定键的过期时间</td></tr></tbody></table>

```
127.0.0.1:6379> rpush key1 123
(integer) 1
127.0.0.1:6379> expire key1 600
(integer) 1
127.0.0.1:6379> ttl key1
(integer) 597
127.0.0.1:6379> ttl key1
(integer) 596
127.0.0.1:6379> ttl key1
(integer) 595
127.0.0.1:6379> pttl key1
(integer) 590098
127.0.0.1:6379> persist key1
(integer) 1
127.0.0.1:6379> ttl key1
(integer) -1

//我现在是2021-10-21 14:11:19，把停止时间设置为15:00:00，时间戳为1634799600，相差49分钟41秒左右，即2909秒左右
127.0.0.1:6379> rpush key1 1234
(integer) 1
127.0.0.1:6379> expireat key1 1634799600
(integer) 1
127.0.0.1:6379> ttl key1
(integer) 2909


```