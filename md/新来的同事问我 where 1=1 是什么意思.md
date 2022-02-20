> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA5ODM5MDU3MA==&mid=2650881891&idx=1&sn=9bad7075cb7070ad29f6b45310a915ec&chksm=8b67d826bc1051308cc744a2f658a39cf9b28a8e811044b4e1ee89079282c74fd28f00f79eb9&mpshare=1&scene=1&srcid=0220r31NVHgrrbtns2HbMR2K&sharer_sharetime=1645335778825&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

新的同事来之后问我 **where 1=1** 是什么有意思，这样没意义啊，我笑了。
------------------------------------------

今天来说明下。  

----------

**先来看一段代码**  

--------------

```
<select parameterType="com.ths.platform.entity.BookInfo" resultType="java.lang.Integer">
select count(id) from t_book t where 1=1
<if test="title !=null and title !='' ">
AND title = #{title}
</if>
<if test="author !=null and author !='' ">
AND author = #{author}
</if>
</select>
```

  

**实 测**

> EXPLAIN SELECT * FROM t_book WHERE  title = '且在人间';

![](https://mmbiz.qpic.cn/mmbiz/1J6IbIcPCLYDicABS9TxA7JufWCbDsMnIPcaibw50AeIPpibPia2ibDfTnTdt6kw0mu3L0893CVczCic7KBXUjmVial7Q/640?wx_fmt=jpeg)

> EXPLAIN SELECT * FROM t_book WHERE 1=1 AND title = '且在人间';

![](https://mmbiz.qpic.cn/mmbiz/1J6IbIcPCLYDicABS9TxA7JufWCbDsMnIhVvwa2kkpamJRCrl8ianNkIHfwG5Lbd0JbOEZZA4F1MvERQicOLywY6A/640?wx_fmt=jpeg)

对比上面两种我们会发现 可以看到 **possible_keys（可能使用的索引）  和  key（实际使用的索引）都使用到了索引进行检索**。

  

**结 论**

**where 1=1** **也会走索引，不影响查询效率，我们写的 sql 指令会被 mysql 进行解析优化成自己的处理指令，在这个过程中** **1 = 1** **这类无意义的条件将会被优化。**
------------------------------------------------------------------------------------------------------

**使用 explain EXTENDED  sql 进行校对，发现确实 where1=1 这类条件会被 mysql 的优化器所优化掉。**
----------------------------------------------------------------------

那么我们在 mybatis 当中可以改变一下写法，因为毕竟 mysql 优化器也是需要时间的，虽然是走了索引，但是当数据量很大时，还是会有影响的，所以**我们建议代码这样写：**

```
<select parameterType="com.ths.platform.entity.BookInfo" resultType="java.lang.Integer">
select count(*) from t_book t
<where>
<if test="title !=null and title !='' ">
title = #{title}
</if>
<if test="author !=null and author !='' ">
AND author = #{author}
</if>
</where>
</select>
```

我们用 **where 标签** 代替。

> 转自：苏世
> 
> 链接：https://juejin.cn/post/7030076565673213989

- EOF -