> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fHdBCIJX1fiWtF0wDeROng)

什么是大 key
========

*   String: 通常超过 10KB 可以认为是大 key
    
*   List: 长度过长, 如超过 10 万
    
*   Hash:field 过多, 如超过 10 万个 field
    
*   Set: 元素过多, 如超过 10 万
    
*   Zset: 元素过多, 如超过 10 万
    

大 key 的危害
=========

*   占用过多内存, 集群模式下造成数据分片不均衡
    
*   执行大 key 命令的客户端本身，耗时明显增加，甚至超时
    
*   执行大 key 相关读取或者删除操作时，会严重占用带宽和 CPU，影响其他客户端
    
*   低版本执行删除时, 阻塞其他请求的执行
    
*   如果大 key 是热点, 频繁读写影响系统性能
    

如何发现大 key
=========

*   Redis 内存监控, 观察内存增长异常
    
*   增加时延、带宽等指标监控，由于大 key 出现，必然会导致操作时延和带宽消耗变大
    
*   slowlog 查看慢查询, 判断是否由大 key 造成
    
*   使用 redis-rdb-tools 离线分析工具来扫描 RDB 持久化文件, 生成报表用来分析 Redis 的内存使用详情
    
*   使用 bigkeys 命令以遍历的方式分析 Redis 实例中的所有 Key，并返回整体统计信息与每个数据类型中 Top1 的大 Key
    
*   scan 命令迭代所有 key, 发现内存异常的 key
    

### bigkeys 命令

执行 redis-cli 命令时带上–bigkeys 选项，对整个数据库中的键值对大小情况进行统计分析，统计每种数据类型的键值对个数以及平均大小。此外，这个命令执行后，会输出每种数据类型中最大的 bigkey 的信息：

*   对于 String 类型来说，会输出最大 bigkey 的字节长度
    
*   对于集合类型来说，会输出最大 bigkey 的元素个数
    

`./redis-cli  --bigkeys  
-------- summary -------  
Sampled 32 keys in the keyspace!  
Total key length in bytes is 184 (avg len 5.75)  
//统计每种数据类型中元素个数最多的bigkey  
Biggest   list found 'product1' has 8 items  
Biggest   hash found 'dtemp' has 5 fields  
Biggest string found 'page2' has 28 bytes  
Biggest stream found 'mqstream' has 4 entries  
Biggest    set found 'userid' has 5 members  
Biggest   zset found 'device:temperature' has 6 members  
//统计每种数据类型的总键值个数，占所有键值个数的比例，以及平均大小  
4 lists with 15 items (12.50% of keys, avg size 3.75)  
5 hashs with 14 fields (15.62% of keys, avg size 2.80)  
10 strings with 68 bytes (31.25% of keys, avg size 6.80)  
1 streams with 4 entries (03.12% of keys, avg size 4.00)  
7 sets with 19 members (21.88% of keys, avg size 2.71)  
5 zsets with 17 members (15.62% of keys, avg size 3.40)  
`

工具是通过扫描数据库来查找 bigkey 的，所以，在执行的过程中，会对 Redis 实例的性能产生影响。

*   主从集群，建议在从节点上执行该命令。避免阻塞主节点。
    
*   没有从节点情况下，在 Redis 实例业务压力的低峰阶段进行扫描查询，以免影响到实例的正常运行；
    
*   使用 -i 参数控制扫描间隔，避免长时间扫描降低 Redis 实例的性能。例如，我们执行`./redis-cli --bigkeys -i 0.1`，控制每扫描 100 次暂停 100 毫秒（0.1 秒）。
    

##### bigkeys 不足

*   只能返回每种类型中最大的那个 bigkey，无法得到大小排在前 N 位的 bigkey；
    
*   对于集合类型来说，只统计集合元素个数的多少，而不是实际占用的内存量。但是，一个集合中的元素个数多，并不一定占用的内存就多。因为如果每个元素占用的内存很小，即使元素个数有很多，总内存开销也不大。
    

### SCAN 命令

使用 SCAN 命令对数据库扫描，然后用 TYPE 命令获取返回的每一个 key 的类型。

##### String 类型

直接使用 STRLEN 命令获取字符串的长度，也就是占用的内存空间字节数。

##### 集合类型

*   如果预先从业务层知道集合元素的平均大小，使用命令获取集合元素的个数，乘以集合元素的平均大小，获得集合占用的内存大小了。
    
*   不能提前知道写入集合的元素大小，可以使用 MEMORY USAGE 命令（需要 Redis 4.0 及以上版本），查询一个键值对占用的内存空间。(`MEMORY USAGE key`)
    

解决大 key 问题
==========

*   设置内存淘汰策略, 及时清除不常用大 key
    
*   对大 key 设置过期时间
    
*   删除大 key 时, 用 UNLINK(惰性删除) 代替 DEL; 低版本没有 UNLINK 命令时，使用 scan 渐进式遍历删除
    
*   vaule 是 string 时，考虑使用序列化、压缩算法将 key 的大小控制在合理范围内
    
*   分片: 按业务拆分大 key
    
*   限制 value 的大小  
    总之, 大 key 会对 Redis 性能造成较大影响, 需要定期监控, 并采取分割、限制或清除的手段进行治理。