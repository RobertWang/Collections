> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/wy123/p/10039966.html)

dbpath 下是清一色的 collection-n-*** 与 index-n-*** 开头的物理文件，如何知道某一个集合与其对应与其对应的物理文件？ 

![](https://img2018.cnblogs.com/blog/380271/201811/380271-20181129175455527-441251274.png)

db.collection_name.stats()

返回的结果包含集合数据对应的物理文件

db.collection_name.stats({indexDetails:true})

返回的结果包含集合数据和索引对应的物理文件

![](https://img2018.cnblogs.com/blog/380271/201811/380271-20181129175821824-479127978.png)

官方有 db.collection.stats 用法的详细信息：[https://docs.mongodb.com/manual/reference/method/db.collection.stats/](https://docs.mongodb.com/manual/reference/method/db.collection.stats/)