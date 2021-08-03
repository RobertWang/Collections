> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_42907802/article/details/106806374)

### 文章目录

*   [Python 连接数据库](#Python_1)
*   *   [连接 SQLite](#SQLite_5)
    *   [连接 MySQL](#MySQL_181)
    *   [使用 SQLAlchemy](#SQLAlchemy_261)

Python 连接数据库
============

> 本文为 jupyter Notebook 导出文档，可能显示效果不太好

连接 SQLite
---------

要操作关系数据库，首先需要连接到数据库，一个数据库连接称为 Connection；

连接到数据库后，需要打开游标，称之为 Cursor，通过 Cursor 执行 SQL 语句，然后，获得执行结果。

Python 定义了一套操作数据库的 API 接口，任何数据库要连接到 Python，只需要提供符合 Python 标准的数据库驱动即可。

由于 SQLite 的驱动内置在 Python 标准库中，可以直接来操作 SQLite 数据库。

```
# 导入SQLite驱动:
import sqlite3

```

```
# 连接到SQLite数据库
# 数据库文件是test.db
# 如果文件不存在，会自动在当前目录创建:
conn = sqlite3.connect('test.db')

```

```
# 创建一个Cursor:
cursor = conn.cursor()

```

```
# 执行一条SQL语句，创建user表:
cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')

```

```
<sqlite3.Cursor at 0x2040e1eff10>

```

```
# 继续执行一条SQL语句，插入一条记录:
cursor.execute('insert into user (id, name) values ("1", "vigilr")')

```

```
<sqlite3.Cursor at 0x2040e1eff10>

```

```
# 通过rowcount获得插入的行数:
cursor.rowcount

```

```
1

```

```
# 关闭Cursor:
cursor.close()
# 提交事务:
conn.commit()
# 关闭Connection:
conn.close()

```

```
conn = sqlite3.connect('test.db')
cursor = conn.cursor()
# 执行查询语句:
cursor.execute('select * from user where id=?',('1',))

```

```
<sqlite3.Cursor at 0x2040e224810>

```

```
# 获得查询结果集:
values = cursor.fetchall()
values

```

```
[('1', 'vigilr')]

```

```
cursor.close()
conn.close()

```

使用 Python 的 DB-API 时，只要搞清楚`Connection`和`Cursor`对象，打开后一定记得关闭，就可以放心地使用。

使用`Cursor`对象执行`insert`，`update`，`delete`语句时，执行结果由`rowcount`返回影响的行数，就可以拿到执行结果。

使用`Cursor`对象执行`select`语句时，通过`featchall()`可以拿到结果集。结果集是一个`list`，每个元素都是一个`tuple`，对应一行记录。

如果 SQL 语句带有参数，那么需要把参数按照位置传递给`execute()`方法，有几个? 占位符就必须对应几个参数，例如：

```
cursor.execute('select * from user where name=? and pwd=?', ('abc', 'password'))

```

```
# -*- coding: utf-8 -*-

import sqlite3


# 初始数据:
conn = sqlite3.connect('test1.db')
cursor = conn.cursor()
cursor.execute('create table user(id varchar(20) primary key, name varchar(20), score int)')
cursor.execute(r"insert into user values ('A-001', 'Adam', 95)")
cursor.execute(r"insert into user values ('A-002', 'Bart', 62)")
cursor.execute(r"insert into user values ('A-003', 'Lisa', 78)")
cursor.close()
conn.commit()
conn.close()

```

```
def get_score_in(low, high):
    '''返回指定分数区间的名字，按分数从低到高排序'''
    conn = sqlite3.connect('test1.db')
    cursor = conn.cursor()
    cursor.execute('select name from user where score>=? and score<=? ORDER BY score',(low,high))
    temp=cursor.fetchall()
    result=[]
    for t in temp:
        for i in t:
            result.append(i)
    cursor.close()
    conn.close()
    return result

```

```
# 测试:
assert get_score_in(80, 95) == ['Adam'], get_score_in(80, 95)
assert get_score_in(60, 80) == ['Bart', 'Lisa'], get_score_in(60, 80)
assert get_score_in(60, 100) == ['Bart', 'Lisa', 'Adam'], get_score_in(60, 100)

print('Pass')


```

```
Pass

```

连接 MySQL
--------

安装 MySQL 驱动

由于 MySQL 服务器以独立的进程运行，并通过网络对外服务，所以，需要支持 Python 的 MySQL 驱动来连接到 MySQL 服务器。  
MySQL 官方提供了`mysql-connector-python`驱动，但是安装的时候需要给 pip 命令加上参数`--allow-external`：  
`pip install mysql-connector-python --allow-external mysql-connector-python`

如果上面的命令安装失败，可以试试另一个驱动：  
`pip install mysql-connector`

除了使用`mysql.connector`还可以使用`pymysql`

```
# 导入pymysql模块
import pymysql

```

```
# 连接database
conn = pymysql.connect(
    host="127.0.0.1",
    port=3308,
    user="root",password="123456",
    database="test",
    charset="utf8")
cursor = conn.cursor()
# 创建user表:
cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')
cursor.close()
conn.close()

```

```
conn = pymysql.connect(
    host="127.0.0.1",
    port=3308,
    user="root",password="123456",
    database="test",
    charset="utf8")
cursor = conn.cursor()
# 插入一行记录，注意MySQL的占位符是%s:
cursor.execute('insert into user (id, name) values (%s, %s)', ['1', 'wasd'])
cursor.execute('insert into user (id, name) values (%s, %s)', ['2', 'zxc'])
print('受影响行数：',cursor.rowcount)

# 提交事务:
conn.commit()
cursor.close()
conn.close()

```

```
受影响行数： 1

```

```
# 连接database
conn = pymysql.connect(
    host="127.0.0.1",
    port=3308,
    user="root",password="123456",
    database="test",
    charset="utf8")
# 运行查询:
cursor = conn.cursor()
cursor.execute('select * from user')
values = cursor.fetchall()
print(values)
# 关闭Cursor和Connection:
cursor.close()
conn.close()

```

```
(('1', 'wasd'), ('2', 'zxc'))

```

使用 SQLAlchemy
-------------

ORM 技术：Object-Relational Mapping，把关系数据库的表结构映射到对象上。

在 Python 中，最有名的 ORM 框架是 SQLAlchemy。

首先通过 pip 安装 SQLAlchemy：`pip install sqlalchemy`

```
# 第一步，导入SQLAlchemy，并初始化DBSession：

# 导入:
from sqlalchemy import Column, String, create_engine,ForeignKey
from sqlalchemy.orm import sessionmaker,relationship
from sqlalchemy.ext.declarative import declarative_base

```

```
# 创建对象的基类:
Base = declarative_base()

# 定义User对象:
class User(Base):
    # 表的名字:
    __tablename__ = 'users'

    # 表的结构:
    id = Column(String(20), primary_key=True)
    name = Column(String(20))

# 以上代码完成SQLAlchemy的初始化和具体每个表的class定义。如果有多个表，就继续定义其他class，例如Scho
class School(Base):
    __tablename__ = 'school'
    id = Column(String(20), primary_key=True)
    name = Column(String(20))

# 初始化数据库连接:mysqlconnector和pymysql都可以用
# engine = create_engine('mysql+mysqlconnector://root:123456@localhost:3308/test')
engine = create_engine('mysql+pymysql://root:123456@localhost:3308/test')
# 创建DBSession类型:
DBSession = sessionmaker(bind=engine)

```

create_engine() 用来初始化数据库连接。SQLAlchemy 用一个字符串表示连接信息：  
`数据库类型+数据库驱动名称://用户名:密码@数据库地址:端口号/数据库名`

由于有了 ORM，我们向数据库表中添加一行记录，可以视为添加一个 User 对象：

```
# 创建所有定义的表到数据库中
Base.metadata.create_all(engine)
# 创建session对象:
session = DBSession()
# 创建新User对象:
user1 = User(id='1', name='wasd')
user2 = User(id='2', name='zxc')
user3 = User(id='3', name='qwe')
user4 = User(id='4', name='rty')
user5 = User(id='5', name='vbn')
user6 = User(id='6', name='fgh')
# 添加到session:
session.add(user1)
session.add(user2)
session.add(user3)
session.add(user4)
session.add(user5)
session.add(user6)
# 提交即保存到数据库:
session.commit()
# 关闭session:
session.close()

# 关键是获取session，然后把对象添加到session，最后提交并关闭。DBSession对象可视为当前数据库连接。

```

```
E:\Users\Administrator\Anaconda3\lib\site-packages\pymysql\cursors.py:170: Warning: (1366, "Incorrect string value: '\\xD6\\xD0\\xB9\\xFA\\xB1\\xEA...' for column 'VARIABLE_VALUE' at row 489")
  result = self._query(query)

```

```
# 查询数据

# 创建Session:
session = DBSession()
# 创建Query查询，filter是where条件，最后调用one()返回唯一行，如果调用all()则返回所有行:
user = session.query(User).filter(User.id=='1').one()
users = session.query(User).filter(User.id!='1').all()
# 打印类型和对象的name属性:
print('type:', type(user))
print('name:', user.name)
for user in users:
    print('type:', type(user))
    print('name:', user.name)
# 关闭Session:
session.close()

```

```
type: <class '__main__.User'>
name: wasd
type: <class '__main__.User'>
name: zxc
type: <class '__main__.User'>
name: qwe
type: <class '__main__.User'>
name: rty
type: <class '__main__.User'>
name: vbn
type: <class '__main__.User'>
name: fgh

```

ORM 就是把数据库表的行与相应的对象建立关联，互相转换。

由于关系数据库的多个表还可以用外键实现一对多、多对多等关联，相应地，ORM 框架也可以提供两个对象之间的一对多、多对多等功能。

例如，如果一个 User 拥有多个 Book，就可以定义一对多关系如下：

```
class User(Base):
    __tablename__ = 'user'
    __table_args__ = {'extend_existing': True}
    id = Column(String(20), primary_key=True)
    name = Column(String(20))
    # 一对多:
    books = relationship('Book')

class Book(Base):
    __tablename__ = 'book'
    id = Column(String(20), primary_key=True)
    name = Column(String(20))
    # “多”的一方的book表是通过外键关联到user表的:
    user_id = Column(String(20), ForeignKey('user.id'))

# 当我们查询一个User对象时，该对象的books属性将返回一个包含若干个Book对象的list。
# 创建所有定义的表到数据库中
Base.metadata.create_all(engine)

```

```
E:\Users\Administrator\Anaconda3\lib\site-packages\sqlalchemy\ext\declarative\clsregistry.py:129: SAWarning: This declarative base already contains a class with the same class name and module name as __main__.User, and will be replaced in the string-lookup table.
  % (item.__module__, item.__name__)

```

更多用法可参考：这篇文章 https://www.jianshu.com/p/65903a69d61d