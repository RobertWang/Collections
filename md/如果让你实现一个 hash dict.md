> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/55100060)

要考虑到的的问题：

1. hash 冲突了之后怎么解决

tips：采用单向链表的方式的话，如何才能最快？

2. 在初始化 hashtable 之后负载因子越来越大之后怎么处理 （负载因子 = used / size）

tips：如果通过两张表来维护的话，该如何完成表的迁移（表足够大），扩容 | 缩容的策略

3. 在设计一个 hash dict 的时候如何设计来确保层次清楚（不通的 dicttype，不同的方式）

4. 如何能快速索引和查找（hash 算法的选择）