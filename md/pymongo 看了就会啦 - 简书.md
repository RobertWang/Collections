> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/438832d64653)

前言
--

4 个多月之前，写过一篇关于非关系型数据库 mongodb 的博文，介绍了怎么在 Windows 系统下来操作 mongodb 的细节。基础不太牢固的可以点击下面的链接 [http://blog.csdn.net/marksinoberg/article/details/53489989](https://link.jianshu.com?t=http://blog.csdn.net/marksinoberg/article/details/53489989)

来加深一下印象。

但是现在不一样了，因为我终于换系统了。之前得 Windows7 由于各种各样的问题，导致补丁包装不上去，间接导致了很多软件也都装不上去。mongodb 当时用的也是很旧的版本了。现在换成了 Windows10，一切都变的很明朗了。装软件什么的再也不是我的 “拦路虎” 了。于是我安装了最新版的 mongodb。

准备
--

我的本机环境是：

*   Python3.6
*   mongodb3.4.3
*   IDE: PyCharm Professional

因为要使用 Python 来操作数据库，所以还需要安装一个 pymongo 的包。需要注意的是直接用

```
pip install pymongo


```

是没办法使用的，可能是版本的问题，也可能是兼容性的问题。所以我们需要这么做。

到 [http://www.lfd.uci.edu/~gohlke/pythonlibs/#pymongo](https://link.jianshu.com?t=http://www.lfd.uci.edu/~gohlke/pythonlibs/#pymongo)

下找到合适自己 Python 版本的 whl 文件，然后再使用 pip 安装这个 whl 文件即可。

简单操作
----

简单操作嘛，无非增删改查。接下来就逐个尝试尝试吧。在此之前先来看看如何连接本地的 mongodb 数据库。

### 数据库连接

连接数据库有下面两种方式。官方建议使用 MongoClient 的形式，而 Connection 则最好不要使用，我这里就干脆按照官方来吧。

```
# coding: utf8

# @Author: 郭 璞
# @File: mongo-connect.py                                                                 
# @Time: 2017/4/22                                   
# @Contact: 1064319632@qq.com
# @blog: http://blog.csdn.net/marksinoberg
# @Description: 链接Mongodb的测试程序

import pymongo
# 默认没有密码，所以可以这么写。如果设置了密码需要使用授权方法db.auth("用户名","密码")
conn = pymongo.MongoClient()
# 选择一个数据库
db = conn.test
# 选择一个collection
collection = db.user

print(collection)


```

如果您的控制台也输出了如下类似的信息，那说明您已经连接成功了。

```
Collection(Database(MongoClient(host=['localhost:27017'], document_class=dict, tz_aware=False, connect=True), 'test'), 'user')


```

### 增：insert

mongodb 的插入操作也是很方便的。我这里事先在 MongoVUE 中添加了几条数据。

![](http://upload-images.jianshu.io/upload_images/8196049-44a3dc179a048c6d) 初始数据库状态

下面咱们通过代码来插入数据。

#### 插入单条记录

```
import pymongo

conn = pymongo.MongoClient()
# 选择一个数据库
db = conn.test
# 选择一个collection
collection = db.user

def show(collection):
    # 查找
    for item in collection.find():
        print(item)

# 插入
dic = {
    'name': '刀塔传奇',
    'age': 23,
    'address': '北京朝阳',
    'blog': '没有博客'
}
collection.insert(dic)
show(collection)


```

代码运行的结果如下：

```
D:\Software\Python3\python.exe E:/Code/Python/Python3/MyWork/mongo/mongo-connect.py
{'_id': ObjectId('58f9e91295ee7820082b562b'), 'name': '郭 璞', 'age': 20}
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 28, 'Address': '北京海淀', 'blog': 'http://blog.csdn.net/zhangsan'}
{'_id': ObjectId('58f9e98f95ee7820082b562d'), 'name': '李四', 'age': 19, 'Address': '上海滩', 'blog': 'http://blog.csdn.net/lisi'}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fac6c495ee78170c9075c7'), 'name': '可爱的哒哒', 'age': 19, 'address': '北京朝阳', 'blog': '哒哒的博客一般人是不会知道的'}
{'_id': ObjectId('58fae2dd95ee7810044c07cf'), 'name': '刀塔传奇', 'age': 23, 'address': '北京朝阳', 'blog': '没有博客'}


```

对比初始状态，不难发现多了一条新插入的记录。

#### 插入多条记录

也许你会有这样的需求，要一次插入多条记录。这在命令行中很方便，我们写个循环就可以了。不过 pymongo 也给我们提供了这样的一个接口。

```
import pymongo

conn = pymongo.MongoClient()
# 选择一个数据库
db = conn.test
# 选择一个collection
collection = db.user

def show(collection):
    # 查找
    for item in collection.find():
        print(item)

# 插入
dic = {
    'name': '刀塔传奇',
    'age': 23,
    'address': '北京朝阳',
    'blog': '没有博客'
}
# collection.insert(dic)

many = []
import copy
for index in range(1, 8):
    tempdic = copy.deepcopy(dic)
    tempdic['age'] = index**2
    many.append(tempdic)

collection.insert_many(many)
show(collection)


```

运行的结果如下：

```
D:\Software\Python3\python.exe E:/Code/Python/Python3/MyWork/mongo/mongo-connect.py
{'_id': ObjectId('58f9e91295ee7820082b562b'), 'name': '郭 璞', 'age': 20}
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 28, 'Address': '北京海淀', 'blog': 'http://blog.csdn.net/zhangsan'}
{'_id': ObjectId('58f9e98f95ee7820082b562d'), 'name': '李四', 'age': 19, 'Address': '上海滩', 'blog': 'http://blog.csdn.net/lisi'}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fac6c495ee78170c9075c7'), 'name': '可爱的哒哒', 'age': 19, 'address': '北京朝阳', 'blog': '哒哒的博客一般人是不会知道的'}
{'_id': ObjectId('58fae2dd95ee7810044c07cf'), 'name': '刀塔传奇', 'age': 23, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc07'), 'name': '刀塔传奇', 'age': 1, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc08'), 'name': '刀塔传奇', 'age': 4, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc09'), 'name': '刀塔传奇', 'age': 9, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0a'), 'name': '刀塔传奇', 'age': 16, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'age': 25, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'age': 36, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0d'), 'name': '刀塔传奇', 'age': 49, 'address': '北京朝阳', 'blog': '没有博客'}



```

怎么样，还是很方便的吧。

### 改： update

修改数据的话，需要了解的是 update 方法的参数的含义。

*   第一个参数是数据库中某一条记录的一个字段的值。它就是一个 “代号”， 保证我们在数据库中能找到它。
    
*   第二个参数是更新后的数据。
    

而修改的话，也可以有两种方式。

#### 方式一

这种方式其实是需要整条记录参与的。

```
dic['blog'] = '哒哒的博客一般人是不会知道的'
collection.update({'name': "傻瓜哒哒"}, dic)
show(collection)


```

程序运行的结果如下：

```
{'_id': ObjectId('58f9e91295ee7820082b562b'), 'name': '郭 璞', 'age': 20}
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 28, 'Address': '北京海淀', 'blog': 'http://blog.csdn.net/zhangsan'}
{'_id': ObjectId('58f9e98f95ee7820082b562d'), 'name': '李四', 'age': 19, 'Address': '上海滩', 'blog': 'http://blog.csdn.net/lisi'}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fac6c495ee78170c9075c7'), 'name': '可爱的哒哒', 'age': 19, 'address': '北京朝阳', 'blog': '哒哒的博客一般人是不会知道的'}
{'_id': ObjectId('58fae2dd95ee7810044c07cf'), 'name': '刀塔传奇', 'age': 23, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc07'), 'name': '刀塔传奇', 'age': 1, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc08'), 'name': '刀塔传奇', 'age': 4, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc09'), 'name': '刀塔传奇', 'age': 9, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0a'), 'name': '刀塔传奇', 'age': 16, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'age': 25, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'age': 36, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0d'), 'name': '刀塔传奇', 'age': 49, 'address': '北京朝阳', 'blog': '没有博客'}
修改操作之后：
{'_id': ObjectId('58f9e91295ee7820082b562b'), 'name': '郭 璞', 'age': 20}
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 23, 'address': '上海新城区', 'blog': '张飞的博客内容被修改后了的内容'}
{'_id': ObjectId('58f9e98f95ee7820082b562d'), 'name': '李四', 'age': 19, 'Address': '上海滩', 'blog': 'http://blog.csdn.net/lisi'}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fac6c495ee78170c9075c7'), 'name': '可爱的哒哒', 'age': 19, 'address': '北京朝阳', 'blog': '哒哒的博客一般人是不会知道的'}
{'_id': ObjectId('58fae2dd95ee7810044c07cf'), 'name': '刀塔传奇', 'age': 23, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc07'), 'name': '刀塔传奇', 'age': 1, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc08'), 'name': '刀塔传奇', 'age': 4, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc09'), 'name': '刀塔传奇', 'age': 9, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0a'), 'name': '刀塔传奇', 'age': 16, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'age': 25, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'age': 36, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0d'), 'name': '刀塔传奇', 'age': 49, 'address': '北京朝阳', 'blog': '没有博客'}


```

#### 方式二

下面介绍部分修改字段内容的方式。

*   更新一条记录

```
collection.update({'name': '刀塔传奇'}, {'$set': {"name": "可爱的Mongodb"}})
show(collection)


```

等效于

```
collection.update_one({'name': '刀塔传奇'}, {'$set': {"name": "可爱的Mongodb"}})
show(collection)


```

*   更新符合条件的所有记录

```
collection.update_many({'name': '刀塔传奇'}, {'$set': {"name": "可爱的Mongodb"}})
show(collection)


```

### 删： remove

这个操作是需要额外注意的，重要性不说大家估计也懂。一个不小心，估计就 “删库跑路” 了。

```
# 删除操作
collection.remove({'name': '可爱的Mongodb'})
show(collection)


```

默认会删除所有符合条件的记录。  
如下：

```
{'_id': ObjectId('58f9e91295ee7820082b562b'), 'name': '郭 璞', 'age': 20}
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 23, 'address': '上海新城区', 'blog': '张飞的博客内容被修改后了的内容'}
{'_id': ObjectId('58f9e98f95ee7820082b562d'), 'name': '李四', 'age': 19, 'Address': '上海滩', 'blog': 'http://blog.csdn.net/lisi'}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fac6c495ee78170c9075c7'), 'name': '可爱的哒哒', 'age': 19, 'address': '北京朝阳', 'blog': '哒哒的博客一般人是不会知道的'}
{'_id': ObjectId('58fae2dd95ee7810044c07cf'), 'name': '可爱的Mongodb', 'age': 23, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc07'), 'name': '可爱的Mongodb', 'age': 1, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc08'), 'name': '可爱的Mongodb', 'age': 4, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc09'), 'name': '刀塔传奇', 'age': 9, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0a'), 'name': '刀塔传奇', 'age': 16, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'age': 25, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'age': 36, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0d'), 'name': '刀塔传奇', 'age': 49, 'address': '北京朝阳', 'blog': '没有博客'}
修改操作之后：
{'_id': ObjectId('58f9e91295ee7820082b562b'), 'name': '郭 璞', 'age': 20}
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 23, 'address': '上海新城区', 'blog': '张飞的博客内容被修改后了的内容'}
{'_id': ObjectId('58f9e98f95ee7820082b562d'), 'name': '李四', 'age': 19, 'Address': '上海滩', 'blog': 'http://blog.csdn.net/lisi'}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fac6c495ee78170c9075c7'), 'name': '可爱的哒哒', 'age': 19, 'address': '北京朝阳', 'blog': '哒哒的博客一般人是不会知道的'}
{'_id': ObjectId('58fae37b95ee78202cc2cc09'), 'name': '刀塔传奇', 'age': 9, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0a'), 'name': '刀塔传奇', 'age': 16, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'age': 25, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'age': 36, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0d'), 'name': '刀塔传奇', 'age': 49, 'address': '北京朝阳', 'blog': '没有博客'}



```

### 查： find

对于一个数据库来说，最最常用的估计就是查找操作了。查找操作的速度决定了数据库性能的评价。下面对于几个常用的查询做下介绍。

#### 查询所有

```
def show(collection):
    # 查找
    for item in collection.find():
        print(item)


```

#### 查询 某个符合要求的字段

*   大于： $gt
*   小于 : $lt
*   大于等于: $gte
*   小于等于: $lte
*   不等于 : $ne

```
# 带有查询条件的查询
items = collection.find({"age": {'$gt': 20}})
for item in items:
    print(item)


```

运行结果

```
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 23, 'address': '上海新城区', 'blog': '张飞的博客内容被修改后了的内容'}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'age': 25, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'age': 36, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0d'), 'name': '刀塔传奇', 'age': 49, 'address': '北京朝阳', 'blog': '没有博客'}


```

#### 查询限制条数

```
collection.find({"age": {'$gt': 20}}).limit(number)


```

#### 查询某（几）个字段的值

```
# 指定查询字段的查询
items = collection.find({"age": {'$gt': 20}}, ['name', 'blog']).limit(5)
for item in items:
    print(item)


```

运行结果

```
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'blog': '张飞的博客内容被修改后了的内容'}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'blog': '没有博客'}


```

#### 查询集合内共有多少条记录

```
# 查询collection中到底有多少条记录
count = collection.count()
print("user 集合中共有{} 条数据".format(count))


```

运行结果：

```
user 集合中共有11 条数据


```

#### 对查询结果排序输出

```
items = collection.find().sort([('age', pymongo.ASCENDING), ('Address', pymongo.DESCENDING)])
for item in items:
    print(item)


```

运行结果：

```
{'_id': ObjectId('58fae37b95ee78202cc2cc09'), 'name': '刀塔传奇', 'age': 9, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0a'), 'name': '刀塔传奇', 'age': 16, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58f9e98f95ee7820082b562d'), 'name': '李四', 'age': 19, 'Address': '上海滩', 'blog': 'http://blog.csdn.net/lisi'}
{'_id': ObjectId('58fac6c495ee78170c9075c7'), 'name': '可爱的哒哒', 'age': 19, 'address': '北京朝阳', 'blog': '哒哒的博客一般人是不会知道的'}
{'_id': ObjectId('58f9e91295ee7820082b562b'), 'name': '郭 璞', 'age': 20}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 23, 'address': '上海新城区', 'blog': '张飞的博客内容被修改后了的内容'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'age': 25, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'age': 36, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0d'), 'name': '刀塔传奇', 'age': 49, 'address': '北京朝阳', 'blog': '没有博客'}


```

#### 模糊查询

```
# # 模糊查询
items = collection.find({"Address": {"$regex": r'^江苏.*'}})
for item in items:
    print(item)


```

运行结果：

```
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}



```

#### 存在性查询

```
# 存在性查询
items = collection.find({'address': {'$exists': True}})
for item in items:
    print(item)


```

运行结果：

```
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 23, 'address': '上海新城区', 'blog': '张飞的博客内容被修改后了的内容'}
{'_id': ObjectId('58fac6c495ee78170c9075c7'), 'name': '可爱的哒哒', 'age': 19, 'address': '北京朝阳', 'blog': '哒哒的博客一般人是不会知道的'}
{'_id': ObjectId('58fae37b95ee78202cc2cc09'), 'name': '刀塔传奇', 'age': 9, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0a'), 'name': '刀塔传奇', 'age': 16, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'age': 25, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'age': 36, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0d'), 'name': '刀塔传奇', 'age': 49, 'address': '北京朝阳', 'blog': '没有博客'}


```

#### in 查询

```
# in 查询
items = collection.find({'age': {'$in': [18, 19, 22, 28]}})
for item in items:
    print(item)


```

运行结果：

```
{'_id': ObjectId('58f9e98f95ee7820082b562d'), 'name': '李四', 'age': 19, 'Address': '上海滩', 'blog': 'http://blog.csdn.net/lisi'}
{'_id': ObjectId('58fac6c495ee78170c9075c7'), 'name': '可爱的哒哒', 'age': 19, 'address': '北京朝阳', 'blog': '哒哒的博客一般人是不会知道的'}


```

#### not in 查询

```
# not in 查询
items = collection.find({'age': {'$nin': [18, 19, 28]}})
for item in items:
    print(item)


```

运行结果：

```
{'_id': ObjectId('58f9e91295ee7820082b562b'), 'name': '郭 璞', 'age': 20}
{'_id': ObjectId('58f9e96f95ee7820082b562c'), 'name': '张三', 'age': 23, 'address': '上海新城区', 'blog': '张飞的博客内容被修改后了的内容'}
{'_id': ObjectId('58f9e9a895ee7820082b562e'), 'name': '王五', 'age': 23, 'Address': '江苏杭州', 'blog': 'http://blog.csdn.net/wangwu'}
{'_id': ObjectId('58f9e9c595ee7820082b562f'), 'name': '赵六', 'age': 32, 'Address': '江苏南京', 'blog': 'http://blog.csdn.net/zhaoliu'}
{'_id': ObjectId('58fae37b95ee78202cc2cc09'), 'name': '刀塔传奇', 'age': 9, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0a'), 'name': '刀塔传奇', 'age': 16, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0b'), 'name': '刀塔传奇', 'age': 25, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0c'), 'name': '刀塔传奇', 'age': 36, 'address': '北京朝阳', 'blog': '没有博客'}
{'_id': ObjectId('58fae37b95ee78202cc2cc0d'), 'name': '刀塔传奇', 'age': 49, 'address': '北京朝阳', 'blog': '没有博客'}


```

差不多经常用到的简单的查询操作就是这样了。掌握了这些差不多也算可以了。

实战
--

下面做个实战，来加深一下熟练度。比如我要爬取西祠代理网的代理 IP 的信息。那么我可以这么来。

### 爬取模块

```
# coding: utf8

# @Author: 郭 璞
# @File: spider.py                                                                 
# @Time: 2017/4/22                                   
# @Contact: 1064319632@qq.com
# @blog: http://blog.csdn.net/marksinoberg
# @Description: 爬取代理IP模块

import requests
from bs4 import BeautifulSoup

def gethtml(url, headers):
    """
    获取网页源代码
    :param url:
    :param headers:
    :return:
    """
    return requests.get(url=url, headers=headers).text

def parse(data):
    """
    解析网页源码，将获取到的代理IP数据封装到一个大的集合中。
    :param data:
    :return:
    """
    result = []

    soup = BeautifulSoup(data, 'html.parser')
    iptable = soup.find('table', {'id': 'ip_list'}).find_all('tr')[2:]
    for item in iptable:
        try:
            ipaddress = item.find_all('td')[1].get_text()
            port = item.find_all('td')[2].get_text()
            address = item.find_all('td')[3].get_text()
            ip = {
                'ipaddress': ipaddress,
                'port': port,
                'address': address
            }
            result.append(ip)
            ip = None
        except Exception as e:
            print(e)
            continue
    return result

if __name__ == '__main__':
    headers = {
        'Referer': 'http://www.xicidaili.com/',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.110 Safari/537.36'
    }
    url = 'http://www.xicidaili.com'
    result = parse(gethtml(url=url, headers=headers))
    print("IP: {}\t Port: {}\t Location: {}".format(result[0].ipaddress, result[0].port, result[0].address))


```

### 存储模块

```
# coding: utf8

# @Author: 郭 璞
# @File: storage.py                                                                 
# @Time: 2017/4/22                                   
# @Contact: 1064319632@qq.com
# @blog: http://blog.csdn.net/marksinoberg
# @Description: 代理IP存储模块

import pymongo

class DbUtils(object):

    def __init__(self, dbname='test', collectionname=''):
        # 实例化self.db
        exec("self.db = pymongo.MongoClient()."+dbname)
        # 给要操作的集合赋值
        exec("self.collection = self.db."+collectionname)

    def storage(self, data=[]):
        return True if self.collection.insert_many(data) else False

    def update(self, old={}, new={}):
        return True if self.collection.update(old, {'$set': new}) else False

    def select(self, where={}, field=[], limits=3, ordering=[]):
        return self.collection.find(where, field).limit(limits).sort(ordering)

    def delete(self, field={}):
        return True if self.collection.remove(field) else False





if __name__ == '__main__':
    dbutils = DbUtils('test', 'iptable')
    data = [
        {'ipaddress': '127.0.0.1', 'port': 8080, 'address': '辽宁大连'},
        {'ipaddress': 'localhost', 'port': '27017', 'address': '北京朝阳'}
    ]
    # flag = dbutils.storage(data=data)
    # old = {'address': '辽宁大连'}
    # new = {'address': '浙江温州'}
    # flag = dbutils.update(old=old, new=new)
    # field = {'port': 8080}
    # flag = dbutils.delete(field=field)
    # print("OP Result:", flag)

    result = dbutils.select({'ipaddress': 'localhost'}, ['ipaddress', 'address'], 3, [('port', pymongo.ASCENDING)])
    for item in result:
        print(item)





```

### 总管模块

```
# coding: utf8

# @Author: 郭 璞
# @File: Main.py
# @Time: 2017/4/22                                   
# @Contact: 1064319632@qq.com
# @blog: http://blog.csdn.net/marksinoberg
# @Description: 代理IP模块整合

from mongo import spider
from mongo import storage

if __name__ == '__main__':
    headers = {
        'Referer': 'http://www.xicidaili.com/',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.110 Safari/537.36'
    }
    url = 'http://www.xicidaili.com'
    # 获取代理IP列表
    iptable = spider.parse(spider.gethtml(url=url, headers=headers))
    print(iptable)
    # 存储数据
    dbutils = storage.DbUtils('test', 'iptable')
    flag = dbutils.storage(data=iptable)
    if flag:
        print('代理IP已经装填完毕，整装待发！')
    else:
        print('代理IP装填不成功，可能出现了点问题！')




```

### 运行效果

```
[{'ipaddress': '119.5.0.3', 'port': '808', 'address': '四川南充'},。。。。。。。。。。。{'ipaddress': '202.119.199.147', 'port': '1080', 'address': '江苏徐州'}]
代理IP已经装填完毕，整装待发！


```

数据库中的效果如下：

![](http://upload-images.jianshu.io/upload_images/8196049-c860190baf08884c) 爬取到的数据存储结果

总结
--

最后来回顾一下今天的内容。

*   使用 Python 简单操作了下 mongodb。
*   对常用的操作进行了演示和讲解
*   实战：封装了爬虫，封装了数据库工具类。分层分模块思想的应用。

大概就是这些了。现在先拿代理 IP 练练手，后面还可以慢慢添加功能，爬取到合适的某些数据之后，通过发邮件，或者调用即时通信 api 来及时的通知自己。这都是有待添加的内容。