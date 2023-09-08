> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hBcsLpfW7e3MYP6fGnvD0g)

> 今天要为大家介绍几个神奇的 Python 数据库连接库，让你轻松搞定数据库操作！

**欢迎来到本篇文章，今天要为大家介绍几个神奇的 Python 数据库连接库，让你轻松搞定数据库操作！**

**SQLite3：轻巧便捷的数据库连接**

**首先，让我们认识一下 SQLite3 这个轻巧便捷的库。它是 Python 中自带的数据库模块，适用于小型应用和快速原型开发。**

**让我们来看看 SQLite3 的魔法：**

```
import sqlite3
# 连接数据库
conn = sqlite3.connect("mydatabase.db")
# 创建表格
cursor = conn.cursor()
cursor.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT, age INTEGER)")
# 插入数据
cursor.execute("INSERT INTO users (name, age) VALUES (?, ?)", ("Alice", 25))
cursor.execute("INSERT INTO users (name, age) VALUES (?, ?)", ("Bob", 30))
# 查询数据
cursor.execute("SELECT * FROM users")
rows = cursor.fetchall()
for row in rows:
    print(row)
# 关闭连接
conn.close()

```

**运行上述代码，你将看到插入的数据和查询结果输出在终端上，SQLite3 让数据库连接变得如此简单！**

**MySQL Connector：连接 MySQL**

**接下来，我们要介绍 MySQL Connector 这个强大的库。它是用于连接 MySQL 数据库的官方驱动，适用于中小型应用和生产环境。**

**MySQL Connector 的安装非常简单，使用以下命令：**

```
pip install mysql-connector-python

```

**让我们看看 MySQL Connector 的魔法：**

```
import mysql.connector
# 连接数据库
conn = mysql.connector.connect(
    host="localhost",
    user="root",
    password="your_password",
    database="mydatabase"
)
# 创建表格
cursor = conn.cursor()
cursor.execute("CREATE TABLE IF NOT EXISTS users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), age INT)")
# 插入数据
cursor.execute("INSERT INTO users (name, age) VALUES (%s, %s)", ("Alice", 25))
cursor.execute("INSERT INTO users (name, age) VALUES (%s, %s)", ("Bob", 30))
# 查询数据
cursor.execute("SELECT * FROM users")
rows = cursor.fetchall()
for row in rows:
    print(row)
# 关闭连接
conn.close()

```

**运行上述代码，你将看到插入的数据和查询结果输出在终端上，MySQL Connector 让数据库连接变得如此高效！**

**SQLAlchemy：强大的 ORM 框架**

**最后，让我们来认识 SQLAlchemy 这个强大的库。它是 Python 中最流行的 ORM（Object-Relational Mapping）框架，适用于复杂的数据库操作和应用。**

**SQLAlchemy 的安装非常简单，使用以下命令：**

```
pip install sqlalchemy

```

**让我们看看 SQLAlchemy 的魔法：**

```
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
# 连接数据库
engine = create_engine("sqlite:///mydatabase.db", echo=True)
Base = declarative_base()
# 定义数据表模型
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    age = Column(Integer)
# 创建数据表
Base.metadata.create_all(engine)
# 创建会话
Session = sessionmaker(bind=engine)
session = Session()
# 插入数据
user1 = User(, age=25)
user2 = User(, age=30)
session.add_all([user1, user2])
session.commit()
# 查询数据
users = session.query(User).all()
for user in users:
    print(user.name, user.age)
# 关闭会话
session.close()

```

**运行上述代码，你将看到插入的数据和查询结果输出在终端上，SQLAlchemy 让数据库操作变得如此高端！**

**数据库连接案例**

**下面我们来看一个实际应用的数据库连接案例：从 CSV 文件中读取数据并存储到数据库。**

```
import csv
# 连接数据库（使用 SQLite3 示例）
conn = sqlite3.connect("mydatabase.db")
cursor = conn.cursor()
# 创建表格
cursor.execute("CREATE TABLE IF NOT EXISTS books (id INTEGER PRIMARY KEY, title TEXT, author TEXT)")
# 从 CSV 文件中读取数据
with open("books.csv", "r") as file:
    reader = csv.reader(file)
    next(reader)  # 跳过标题行
    for row in reader:
        title, author = row
        cursor.execute("INSERT INTO books (title, author) VALUES (?, ?)", (title, author))
# 查询数据
cursor.execute("SELECT * FROM books")
rows = cursor.fetchall()
for row in rows:
    print(row)
# 关闭连接
conn.close()

```

**运行上述代码，你将看到读取的数据和查询结果输出在终端上，通过这样简单的数据库连接，你可以更好地管理和查询数据。**

**— **完** —**