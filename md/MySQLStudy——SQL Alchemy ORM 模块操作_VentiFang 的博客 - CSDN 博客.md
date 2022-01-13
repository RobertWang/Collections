> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_35324294/article/details/93049127)

引言
==

我一共建立了 2 张表用于演示 SQL Alchemy

               users                                                                               usertype

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

导包语句
====

```
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, ForeignKey, DateTime, Index, UniqueConstraint
from sqlalchemy.orm import sessionmaker, relationship
import datetime
from sqlalchemy import and_,or_,any_,func

```

初始化语句
=====

```
# 创建  引擎  连接mysql
# 这个 db1一开始必须先手动创建完毕  max_overflow是最大sql连接数
#                                           | 密码写在这个冒号后面，我的密码为空所以不填
engine = create_engine('mysql+pymysql://root:@127.0.0.1:3306/db1?charset=utf8', max_overflow=5)
# declarative_base()是一个工厂函数，它为声明性类定义构造基类
Base = declarative_base()

```

定义两张表对应的两个类
===========

```
# 定义数据表 UserType类
class UserType(Base):
    __tablename__ = 'usertype'
    id = Column(Integer, autoincrement=True, primary_key=True)
    name = Column(String(32), nullable=False, default='')


# 定义数据表 Users类
class Users(Base):
    __tablename__ = 'users'
    id = Column(Integer, autoincrement=True, primary_key=True)
    name = Column(String(32), nullable=False, default='')
    extra = Column(String(32), nullable=False, default='')
    type_id = Column(Integer, ForeignKey(UserType.id))
    # 关联 back reference
    usertype = relationship('UserType', backref='xxoo')

    # 所有的索引写在这个位置
    __table_args__ = (
        # 联合唯一索引
        UniqueConstraint('id', 'name', name='uix_id_name'),
        # 组合索引
        Index('ix_name_extra', 'name', 'extra')
    )

```

 自定义方法
======

```
def drop_db():
    # 删除所有表
    Base.metadata.drop_all(engine)
def create_db():
    # 会将当前执行文件中所有继承自Base类的类,生成表
    Base.metadata.create_all(engine)

```

Session 创建
==========

```
# 会话工厂函数 生成Session类 绑定引擎
Session = sessionmaker(bind=engine)
# 生成session对象
session = Session()

```

添加数据
====

```
# 增加数据 实例化 添加一条
obj = UserType(name='普通用户哈哈哈')
session.add(obj)
session.commit()
session.close()

# 添加多条数据
# 方法一 提前写好充满待插数据的列表
data = [
    UserType(name='VIP'),
    UserType(name='SVIP'),
    UserType(name='黑金VIP')
]
session.add_all(data)

# 方法二 充满待插数据的列表直接插
session.add_all([
    UserType(name='内部用户'),
])
# 不要忘记插好之后 提交会话任务和关闭会话
session.commit()
session.close()

```

查询
==

```
# query 查询 session.query 不带任何修饰 直接返回 SQL语句
res = session.query(UserType)
print(res)
# SELECT usertype.id AS usertype_id, usertype.name AS usertype_name
# FROM usertype

# 查询多个
# 语法 session.query(类名).all()
res = session.query(UserType).all()
print(res)
# [<__main__.UserType object at 0x000001DB68F5FBA8>,
# <__main__.UserType object at 0x000001DB68F5FA20>,
# <__main__.UserType object at 0x000001DB68F5FB38>,
# <__main__.UserType object at 0x000001DB68F5FA58>,
# <__main__.UserType object at 0x000001DB68F5F898>]
for row in res:
    print(row.id,row.name)
# 1 普通用户
# 2 VIP
# 3 SVIP
# 4 黑金VIP
# 5 内部用户
#
#
# 查询第一条数据 获取到对象
res = session.query(UserType).first()
print(res)
# <__main__.UserType object at 0x00000205F3F9FB70>
print(res.id, res.name)
# 1 普通用户

```

```
# where 条件 filter过滤器
# -----------查询name字段是VIP的条目-------1个条件---------
res = session.query(UserType).filter(UserType.name=='VIP')
print(res.values()) # <tuple_iterator object at 0x0000020DFE51FB70>
print(res.name)  # AttributeError: 'Query' object has no attribute 'name'
for row in res:
    print(row.id,row.name)  # 2 VIP
#-------------------------------------------------------
#
#
#------------查询name字段是普通且id字段为5----2个条件-------
res = session.query(UserType).filter(UserType.name=='VIP',UserType.id==2)
res = session.query(UserType).filter(UserType.name=='VIP',UserType.id==2).all()
for row in res:
    print(row.id,row.name)
#---------------------一个逗号就是and---------------------

```

删除
==

```
# 删除
session.query(UserType).filter(UserType.id>=5).delete()
# 这里不要添加all() 否则返回的会是一个列表没有delete属性
session.query(UserType).filter(UserType.id>=5).all().delete()
session.query(UserType).filter(UserType.id.in_([3,4])).delete(synchronize_session=False)
session.commit()

```

修改
==

```
# 修改
session.query(UserType).filter(UserType.id>=1).update({"name":'普通用户'})
session.query(UserType).filter(UserType.id.in_([1,2,3,4])).update({"name":'haha用户'},synchronize_session=False)
session.query(Users).filter(Users.id==1).update({"name":Users.name*2},synchronize_session=False)
# 上述语法会报错,加上
session.commit()
session.close()


```

高级查询
====

分组
--

```
##############################################################################################
res = session.query(UserType.id,
                    func.max(UserType.id),
                    func.min(UserType.id)).group_by(UserType.id).all()
print(res)
##############################################################################################
# SELECT usertype.id AS usertype_id, max(usertype.id) AS max_1, min(usertype.id) AS min_1
# FROM usertype GROUP BY usertype.id
#######  [(1, 1, 1), (2, 2, 2), (3, 3, 3), (4, 4, 4)]
##############################################################################################
ret = session.query(
    func.max(Users.id),
    func.min(Users.id)).group_by(Users.type_id).having(func.min(Users.id) >2).all()
print(ret)
##############################################################################################
# SELECT max(users.id) AS max_1, min(users.id) AS min_1
# FROM users GROUP BY users.type_id
# HAVING min(users.id) > %(min_2)s
#######   [(3, 3), (4, 4)]
# 根据Users.type_id分组得到1,2,3,4四个组

```

分页
--

```
# 分页
ret = session.query(Users)[1:3]
for row in ret:
    print(row.id,row.name)
# 2 ee
# 3 rr

```

排序
--

```
ret = session.query(Users).order_by(Users.name.desc()).all()
print(ret)
# [<__main__.Users object at 0x000002428E8F5A20>,
# <__main__.Users object at 0x000002428E8F5A90>,
# <__main__.Users object at 0x000002428E8F5B00>,
# <__main__.Users object at 0x000002428E8F5B70>]
########################################################
for row in ret:
    print(row.id,row.name)
########################################################
# 4 哈哈
# 3 rr
# 2 ee
# 1 160
########################################################
ret = session.query(Users).order_by(Users.name.asc()).all()
for row in ret:
    print(row.id,row.name)
########################################################
# 1 160
# 2 ee
# 3 rr
# 4 哈哈
########################################################
ret = session.query(Users).order_by(Users.name.desc(), Users.id.asc()).all()
for row in ret:
    print(row.id,row.name)
########################################################
# 4 哈哈
# 3 rr
# 2 ee
# 1 160
########################################################

```

between ... and ...
-------------------

```
# between ... and ...
res = session.query(UserType).filter(UserType.id.between(1,5)).all()
print(res)
print(type(res))
for row in res:
   print(row.id,row.name)
print('--------------------------------')
res = session.query(UserType).filter(UserType.id.between(1,5))
print(res[0],res[1],res[2],res[3])
print(type(res))
print(res.values())
for row in res:
    print(row.id,row.name)
print('--------------------------------')
#
#
# [<__main__.UserType object at 0x0000020B963C9588>, <__main__.UserType object at 0x0000020B963C95F8>, <__main__.UserType object at 0x0000020B963C9668>, <__main__.UserType object at 0x0000020B963C96D8>]
# <class 'list'>
# 1 普通用户
# 2 VIP
# 3 SVIP
# 4 黑金VIP
# --------------------------------
# <__main__.UserType object at 0x0000020B963C9828> <__main__.UserType object at 0x0000020B963C9BE0> <__main__.UserType object at 0x0000020B963C9EF0> <__main__.UserType object at 0x0000020B963C96D8>
# <class 'sqlalchemy.orm.query.Query'>
# <tuple_iterator object at 0x0000020B963C98D0>
# 1 普通用户
# 2 VIP
# 3 SVIP
# 4 黑金VIP
# --------------------------------

```

in 与 not in
-----------

```
# in
res = session.query(UserType).filter(UserType.id.in_([1,2,4])).all()
for row in res:
    print(row.id,row.name)
#
#
#
# not in ~取反
res2 =session.query(UserType).filter(~UserType.id.in_([1,2,4])).all()
for row in res2:
    print(row.id,row.name)

```

连表查询
----

```
# 连表查询
###1. 查询某一个用户的用户类型
### 第一种方法(左连接查询):
res = session.query(Users,UserType).join(UserType, isouter=True).all()
for row in res:
    print(row[0].id, row[0].name,row[1].id,row[1].name)
##################################################################################
# 1 160 1 普通用户
# 2 ee 2 VIP
# 3 rr 3 SVIP
# 4 哈哈 4 黑金VIP
##################################################################################
### 第二种方法(通过联合)
res = session.query(Users).all()
for row in res:
    print(row.id, row.name, row.extra, row.usertype.id,row.usertype.name)
##################################################################################
# 1 160 111 1 普通用户
# 2 ee 222 2 VIP
# 3 rr 333 3 SVIP
# 4 哈哈 444 4 黑金VIP
##################################################################################
### 2. 某一个类型下面的用户
### 第一种方法
res = session.query(UserType).all()
for row in res:
    print(row.id, row.name, session.query(Users).filter(Users.type_id == row.id).all())

### 第二种方法
res = session.query(UserType).all()
for row in res:
    print(row.id, row.title, row.xxoo)

```