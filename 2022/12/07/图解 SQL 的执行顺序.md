> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwNjMxMTA5Mw==&mid=2651350400&idx=1&sn=3fcf86663f6a0bbdd65617ae38a2e87f&chksm=80f3985ab784114cf15d94adb60f2ca0cb208aab5a4c5c62848ebcbacb79e532716fd850b86a&mpshare=1&scene=1&srcid=1117oDR9l3dixL1uhlFK5IiF&sharer_sharetime=1668637824138&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

> 图解 SQL 的执行顺序，一目了然

↓推荐关注↓

> 来源：blog.csdn.net/weixin_44141495/article/details/108744720

这是一条标准的查询语句:

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8MpLabn3VTHvK28W2xiaRmJnfIibnhnzQN82ezbUIa4Kf0dcTjeianQhlNA/640?wx_fmt=png&random=0.8376624600892231) 这是我们实际上 SQL 执行顺序：

*   我们先执行 from,join 来确定表之间的连接关系，得到初步的数据
    
*   where 对数据进行普通的初步的筛选
    
*   group by 分组
    
*   各组分别执行 having 中的普通筛选或者聚合函数筛选。
    
*   然后把再根据我们要的数据进行 select，可以是普通字段查询也可以是获取聚合函数的查询结果，如果是集合函数，select 的查询结果会新增一条字段
    
*   将查询结果去重 distinct
    
*   最后合并各组的查询结果，按照 order by 的条件进行排序
    

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8MliaCnbDiaeeFlM6kDNJDjtquKzlIIHN0opcuCsAXzazRbUtaKXd7zVzQ/640?wx_fmt=png&random=0.9020206554951093)

### **数据的关联过程**

数据库中的两张表

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8M7JiarOd3myPym6BjuZZDic4Tx568nfMNEuDUoT0DIiaASgXEBCanvKxAg/640?wx_fmt=png&random=0.6173145097025767)

### **from&join&where**

用于确定我们要查询的表的范围，涉及哪些表。

选择一张表，然后用 join 连接

```
from table1 join table2 on table1.id=table2.id

```

选择多张表，用 where 做关联条件

```
from table1,table2 where table1.id=table2.id

```

我们会得到满足关联条件的两张表的数据，不加关联条件会出现笛卡尔积。

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8MQ4kqWiaWbAmb7wNN0EV4g8RlOkyf7iaxRUEJkmYDH2RHkXbIJlDe9NVA/640?wx_fmt=png&random=0.12067204242260776)

### **group by**

按照我们的分组条件，将数据进行分组，但是不会筛选数据。

比如我们按照即 id 的奇偶分组

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8MibMyojzzn1zMMib5GDz5fI1K02sufJxS9mKcNN8jX3jR82apoZLXTHAQ/640?wx_fmt=png&random=0.6046945431133164)

### **having&where**

having 中可以是普通条件的筛选，也能是聚合函数。而 where 只能是普通函数，一般情况下，有 having 可以不写 where，把 where 的筛选放在 having 里，SQL 语句看上去更丝滑。

##### 使用 where 再 group by

先把不满足 where 条件的数据删除，再去分组

##### 使用 group by 再 having

先分组再删除不满足 having 条件的数据，这两种方法有区别吗，几乎没有！

举个例子：

`100/2=50`，此时我们把 100 拆分`(10+10+10+10+10…)/2=5+5+5+…+5=50`, 只要筛选条件没变，即便是分组了也得满足筛选条件，所以 where 后 group by 和 group by 再 having 是不影响结果的！

不同的是，having 语法支持聚合函数, 其实 having 的意思就是针对每组的条件进行筛选。我们之前看到了普通的筛选条件是不影响的，但是 having 还支持聚合函数，这是 where 无法实现的。

当前数据分组情况

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8MibMyojzzn1zMMib5GDz5fI1K02sufJxS9mKcNN8jX3jR82apoZLXTHAQ/640?wx_fmt=png&random=0.5118015173602497)执行 having 的筛选条件，可以使用聚合函数。筛选掉工资小于各组平均工资的`having salary<avg(salary)`

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8Mr2MqRTqFxibByXo27ahmhw6pz5PuFvdn6AIItjYujib5f5Fuib9MjzODw/640?wx_fmt=png&random=0.16104042459074108)

### **select**

分组结束之后，我们再执行 select 语句，因为聚合函数是依赖于分组的，聚合函数会单独新增一个查询出来的字段，这里用紫色表示，这里我们两个 id 重复了，我们就保留一个 id，重复字段名需要指向来自哪张表，否则会出现唯一性问题。最后按照用户名去重。

```
select employee.id,distinct name,salary, avg(salary)

```

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8MOickUy3MbrRY3yJOib87lovfgXibKkYuSLHZvMYqjasf9nqDiapYa7Licpw/640?wx_fmt=png&random=0.7707866383011122)将各组 having 之后的数据再合并数据。

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8MQn9YRISZKmBicnnH8eicakx8q9KZU0ibpNquXQ3YaOl6Ye6ZHOP29PiceQ/640?wx_fmt=png&random=0.26377685340116797)

### **order by**

最后我们执行 order by 将数据按照一定顺序排序，比如这里按照 id 排序。如果此时有 limit 那么查询到相应的我们需要的记录数时，就不继续往下查了。

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8MSIeicAibWnoJSrMBZ21ez6fYzyACSMmVrSx8mSXficCGiasIIL6E1KRrCw/640?wx_fmt=png&random=0.8145033475826966)

### **limit**

记住 limit 是最后查询的，为什么呢？假如我们要查询年级最小的三个数据，如果在排序之前就截取到 3 个数据。实际上查询出来的不是最小的三个数据而是前三个数据了，记住这一点。

我们如果 limit 0,3 窃取前三个数据再排序，实际上最少工资的是 2000,3000,4000。你这里只能是 4000,5000,8000 了。

![](https://mmbiz.qpic.cn/mmbiz_png/8KKrHK5ic6XBWqRJK93s6iaF9XhibickkT8M1WXX6yU9QRpWW5icOibQ6Ij4KNSiaM7joGtEdvuVHHDkSAkHPKHrc4aGw/640?wx_fmt=png&random=0.6708150334017671)

- EOF -