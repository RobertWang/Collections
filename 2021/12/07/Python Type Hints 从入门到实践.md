> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666559579&idx=3&sn=48d7f8221d2b27e82460cb69d3e693ce&chksm=80dcbcf0b7ab35e64b02e625e27dc4fb0ef20ec54873d823a02777773ca1df54d4dff2291059&mpshare=1&scene=1&srcid=1206thHz843el2t2IrFhCVVj&sharer_sharetime=1638763798359&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd) 以 PyCharm 为例，在编写代码的过程中 IDE 会根据函数的类型标注，对传递给函数的参数进行类型检查。如果发现实参类型与函数的形参类型标注不符就会有如下提示：

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUHt4icYsW9sh3zwbiblAQhAXfHPbcrKBTSqC34neibtvTvEMusmTJPGRvZjZCRgfVHd892zaIpbfgl4g/640?wx_fmt=png)

  

**常见数据结构的 Type Hints 写法**

  

上面通过一个 greeting 函数展示了 Type Hints 的用法，接下来我们就 Python 常见数据结构的 Type Hints 写法进行更加深入的学习。  

**默认参数**

Python 函数支持默认参数，以下是默认参数的 Type Hints 写法，只需要将类型写到变量和默认参数之间即可。

```
def greeting(name: str = "world") -> str:
    return "Hello " + name

greeting()
```

**自定义类型**

对于自定义类型，Type Hints 同样能够很好的支持。它的写法跟 Python 内置类型并无区别。  

```
class Student(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age


def student_to_string(s: Student) -> str:
    return f"student name: {s.name}, age: {s.age}."

student_to_string(Student("Tim", 18))
```

当类型标注为自定义类型时，IDE 也能够对类型进行检查。

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUHt4icYsW9sh3zwbiblAQhAXf0zQQ8iaZ2aq16bgvcicWnBn79TxLicUyTD10RaEYicRclPasIZBqlykDcA/640?wx_fmt=png)

**容器类型**

当我们要给内置容器类型添加类型标注时，由于类型注解运算符 [] 在 Python 中代表切片操作，因此会引发语法错误。所以不能直接使用内置容器类型当作注解，需要从 typing 模块中导入对应的容器类型注解（通常为内置类型的首字母大写形式）。

```
from typing import List, Tuple, Dict

l: List[int] = [1, 2, 3]

t: Tuple[str, ...] = ("a", "b")

d: Dict[str, int] = {
    "a": 1,
    "b": 2,
}
```

不过 PEP 585[https://www.python.org/dev/peps/pep-0585/] 的出现解决了这个问题，我们可以直接使用 Python 的内置类型，而不会出现语法错误。

```
l: list[int] = [1, 2, 3]

t: tuple[str, ...] = ("a", "b")

d: dict[str, int] = {
    "a": 1,
    "b": 2,
}
```

**类型别名**

有些复杂的嵌套类型写起来很长，如果出现重复，就会很痛苦，代码也会不够整洁。  

```
config: list[tuple[str, int], dict[str, str]] = [
    ("127.0.0.1", 8080),
    {
        "MYSQL_DB": "db",
        "MYSQL_USER": "user",
        "MYSQL_PASS": "pass",
        "MYSQL_HOST": "127.0.0.1",
        "MYSQL_PORT": "3306",
    },
]

def start_server(config: list[tuple[str, int], dict[str, str]]) -> None:
    ...

start_server(config)
```

此时可以通过给类型起别名的方式来解决，类似变量命名。

```
Config = list[tuple[str, int], dict[str, str]]


config: Config = [
    ("127.0.0.1", 8080),
    {
        "MYSQL_DB": "db",
        "MYSQL_USER": "user",
        "MYSQL_PASS": "pass",
        "MYSQL_HOST": "127.0.0.1",
        "MYSQL_PORT": "3306",
    },
]

def start_server(config: Config) -> None:
    ...

start_server(config)
```

这样代码看起来就舒服多了。

**可变参数**

Python 函数一个非常灵活的地方就是支持可变参数，Type Hints 同样支持可变参数的类型标注。  

```
def foo(*args: str, **kwargs: int) -> None:
    ...

foo("a", "b", 1, x=2, y="c")
```

IDE 仍能够检查出来。  

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUHt4icYsW9sh3zwbiblAQhAXfBmABlnt6qZNlGxl3bWFGCyqczbOqMS0uQZhNUkYdIXhfQc4LkUptZQ/640?wx_fmt=png)

**泛型**

使用动态语言少不了泛型的支持，Type Hints 针对泛型也提供了多种解决方案。  

**TypeVar**

使用 TypeVar 可以接收任意类型。  

```
from typing import TypeVar

T = TypeVar("T")

def foo(*args: T, **kwargs: T) -> None:
    ...

foo("a", "b", 1, x=2, y="c")
```

**Union**

如果不想使用泛型，只想使用几种指定的类型，那么可以使用 Union 来做。比如定义 concat 函数只想接收 str 或 bytes 类型。  

```
from typing import Union

T = Union[str, bytes]

def concat(s1: T, s2: T) -> T:
    return s1 + s2

concat("hello", "world")
concat(b"hello", b"world")
concat("hello", b"world")
concat(b"hello", "world")
```

IDE 的检查提示如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUHt4icYsW9sh3zwbiblAQhAXfdP8xNdTkktHDbwBVw5hu40vku4THCFnibyGMssjCXpvu1QU2vgCFhLA/640?wx_fmt=png)

**TypeVar 和 Union 区别**

TypeVar 不只可以接收泛型，它也可以像 Union 一样使用，只需要在实例化时将想要指定的类型范围当作参数依次传进来来即可。跟 Union 不同的是，使用 TypeVar 声明的函数，多参数类型必须相同，而 Union 不做限制。  

```
from typing import TypeVar

T = TypeVar("T", str, bytes)

def concat(s1: T, s2: T) -> T:
    return s1 + s2

concat("hello", "world")
concat(b"hello", b"world")
concat("hello", b"world")
```

以下是使用 TypeVar 做限定类型时的 IDE 提示：

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUHt4icYsW9sh3zwbiblAQhAXfxNZyI6htdg3icmjiabkl6fToibj2vsxmYZTXyibiabMxcqMeMrxWSibXF8eQ/640?wx_fmt=png)

**Optional**

Type Hints 提供了 Optional 来作为 Union[X, None] 的简写形式，表示被标注的参数要么为 X 类型，要么为 None，Optional[X] 等价于 Union[X, None]。  

```
from typing import Optional, Union

# None => type(None)
def foo(arg: Union[int, None] = None) -> None:
    ...


def foo(arg: Optional[int] = None) -> None:
    ...
```

**Any**

Any 是一种特殊的类型，可以代表所有类型。未指定返回值与参数类型的函数，都隐式地默认使用 Any，所以以下两个 greeting 函数写法等价：  

```
from typing import Any

def greeting(name):
    return "Hello " + name


def greeting(name: Any) -> Any:
    return "Hello " + name
```

当我们既想使用 Type Hints 来实现静态类型的写法，也不想失去动态语言特有的灵活性时，即可使用 Any。

Any 类型值赋给更精确的类型时，不执行类型检查，如下代码 IDE 并不会有错误提示：

```
from typing import Any

a: Any = None
a = []  # 动态语言特性
a = 2

s: str = ''
s = a  # Any 类型值赋给更精确的类型
```

**可调用对象（函数、类等）**

Python 中的任何可调用类型都可以使用 Callable 进行标注。如下代码标注中 Callable[[int], str]，[int] 表示可调用类型的参数列表，str 表示返回值。  

```
from typing import Callable

def int_to_str(i: int) -> str:
    return str(i)

def f(fn: Callable[[int], str], i: int) -> str:
    return fn(i)

f(int_to_str, 2)
```

**自引用**

当我们需要定义树型结构时，往往需要自引用。当执行到 __init__ 方法时 Tree 类型还没有生成，所以不能像使用 str 这种内置类型一样直接进行标注，需要采用字符串形式 “Tree” 来对未生成的对象进行引用。  

```
class Tree(object):
    def __init__(self, left: "Tree" = None, right: "Tree" = None):
        self.left = left
        self.right = right

tree1 = Tree(Tree(), Tree())
```

IDE 同样能够对自引用类型进行检查。

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUHt4icYsW9sh3zwbiblAQhAXfKBgBCeQ3VdazzNDOkbWHmAESdaYHtKicTtib3icAylsnbtouc8guZa4Kg/640?wx_fmt=png)

此形式不仅能够用于自引用，前置引用同样适用。

**鸭子类型**

Python 一个显著的特点是其对鸭子类型的大量应用，Type Hints 提供了 Protocol 来对鸭子类型进行支持。定义类时只需要继承 Protocol 就可以声明一个接口类型，当遇到接口类型的注解时，只要接收到的对象实现了接口类型的所有方法，即可通过类型注解的检查，IDE 便不会报错。这里的 Stream 无需显式继承 Interface 类，只需要实现了 close 方法即可。  

```
from typing import Protocol

class Interface(Protocol):
    def close(self) -> None:
        ...

# class Stream(Interface):
class Stream:
    def close(self) -> None:
        ...

def close_resource(r: Interface) -> None:
    r.close()

f = open("a.txt")
close_resource(f)

s: Stream = Stream()
close_resource(s)
```

由于内置的 open 函数返回的文件对象和 Stream 对象都实现了 close 方法，所以能够通过 Type Hints 的检查，而字符串 “s” 并没有实现 close 方法，所以 IDE 会提示类型错误。

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUHt4icYsW9sh3zwbiblAQhAXf5EghXkkNEShueeGKdAlhfPlGMkF6r9SJKCxLdbfuMLjXjhm0dDggQA/640?wx_fmt=png)

  

**Type Hints 的其他写法**

  

实际上 Type Hints 不只有一种写法，Python 为了兼容不同人的喜好和老代码的迁移还实现了另外两种写法。  

**使用注释编写**

来看一个 tornado 框架的例子（tornado/web.py）。适用于在已有的项目上做修改，代码已经写好了，后期需要增加类型标注。  

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUHt4icYsW9sh3zwbiblAQhAXf6bnLS1rMQTokFt4cOfeic9y8vmcJn35HhIxsWaPyLiasclpAVaXjicIbA/640?wx_fmt=png)

**使用单独文件编写（.pyi）**

可以在源代码相同的目录下新建一个与 .py 同名的 .pyi 文件，IDE 同样能够自动做类型检查。这么做的优点是可以对原来的代码不做任何改动，完全解耦。缺点是相当于要同时维护两份代码。  

![](https://mmbiz.qpic.cn/mmbiz_png/OdIoEOgFgUHt4icYsW9sh3zwbiblAQhAXfKPoGLLGT1o0YCiayY2ibias0PSqIPzXOsoobRPDRDayCmNvTyrxE4PCibA/640?wx_fmt=png)

  

**Type Hints 实践**

  

基本上，日常编码中常用的 Type Hints 写法都已经介绍给大家了，下面就让我们一起来看看如何在实际编码中中应用 Type Hints。

**dataclass——数据类**

dataclass 是一个装饰器，它可以对类进行装饰，用于给类添加魔法方法，例如 __init__() 和 __repr__() 等，它在 PEP 557[https://www.python.org/dev/peps/pep-0557/] 中被定义。

```
from dataclasses import dataclass, field


@dataclass
class User(object):
    id: int
    name: str
    friends: list[int] = field(default_factory=list)


data = {
    "id": 123,
    "name": "Tim",
}

user = User(**data)
print(user.id, user.name, user.friends)
# > 123 Tim []
```

以上使用 dataclass 编写的代码同如下代码等价：

```
class User(object):
    def __init__(self, id: int, name: str, friends=None):
        self.id = id
        self.name = name
        self.friends = friends or []


data = {
    "id": 123,
    "name": "Tim",
}

user = User(**data)
print(user.id, user.name, user.friends)
# > 123 Tim []
```

注意：dataclass 并不会对字段类型进行检查。

可以发现，使用 dataclass 来编写类可以减少很多重复的样板代码，语法上也更加清晰。

**Pydantic**

Pydantic 是一个基于 Python Type Hints 的第三方库，它提供了数据验证、序列化和文档的功能，是一个非常值得学习借鉴的库。以下是一段使用 Pydantic 的示例代码：  

```
from datetime import datetime
from typing import Optional

from pydantic import BaseModel


class User(BaseModel):
    id: int
    name = 'John Doe'
    signup_ts: Optional[datetime] = None
    friends: list[int] = []


external_data = {
    'id': '123',
    'signup_ts': '2021-09-02 17:00',
    'friends': [1, 2, '3'],
}
user = User(**external_data)

print(user.id)
# > 123
print(repr(user.signup_ts))
# > datetime.datetime(2021, 9, 2, 17, 0)
print(user.friends)
# > [1, 2, 3]
print(user.dict())
"""
{
    'id': 123,
    'signup_ts': datetime.datetime(2021, 9, 2, 17, 0),
    'friends': [1, 2, 3],
    'name': 'John Doe',
}
"""
```

注意：Pydantic 会对字段类型进行强制检查。

Pydantic 写法上跟 dataclass 非常类似，但它做了更多的额外工作，还提供了如 .dict() 这样非常方便的方法。

再来看一个 Pydantic 进行数据验证的示例，当 User 类接收到的参数不符合预期时，会抛出 ValidationError 异常，异常对象提供了 .json() 方法方便查看异常原因。

```
from pydantic import ValidationError

try:
    User(signup_ts='broken', friends=[1, 2, 'not number'])
except ValidationError as e:
    print(e.json())
"""
[
  {
    "loc": [
      "id"
    ],
    "msg": "field required",
    "type": "value_error.missing"
  },
  {
    "loc": [
      "signup_ts"
    ],
    "msg": "invalid datetime format",
    "type": "value_error.datetime"
  },
  {
    "loc": [
      "friends",
      2
    ],
    "msg": "value is not a valid integer",
    "type": "type_error.integer"
  }
]
"""
```

所有报错信息都保存在一个 list 中，每个字段的报错又保存在嵌套的 dict 中，其中 loc 标识了异常字段和报错位置，msg 为报错提示信息，type 则为报错类型，这样整个报错原因一目了然。

**MySQLHandler**

MySQLHandler[https://github.com/jianghushinian/python-scripts/blob/main/scripts/mysql_handler_type_hints.py] 是我对 pymysql 库的封装，使其支持使用 with 语法调用 execute 方法，并且将查询结果从 tuple 替换成 object，同样也是对 Type Hints 的应用。

```
class MySQLHandler(object):
    """MySQL handler"""

    def __init__(self):
        self.conn = pymysql.connect(
            host=DB_HOST,
            port=DB_PORT,
            user=DB_USER,
            password=DB_PASS,
            database=DB_NAME,
            charset=DB_CHARSET,
            client_flag=CLIENT.MULTI_STATEMENTS,  # execute multi sql statements
        )
        self.cursor = self.conn.cursor()

    def __del__(self):
        self.cursor.close()
        self.conn.close()

    @contextmanager
    def execute(self):
        try:
            yield self.cursor.execute
            self.conn.commit()
        except Exception as e:
            self.conn.rollback()
            logging.exception(e)

    @contextmanager
    def executemany(self):
        try:
            yield self.cursor.executemany
            self.conn.commit()
        except Exception as e:
            self.conn.rollback()
            logging.exception(e)

    def _tuple_to_object(self, data: List[tuple]) -> List[FetchObject]:
        obj_list = []
        attrs = [desc[0] for desc in self.cursor.description]
        for i in data:
            obj = FetchObject()
            for attr, value in zip(attrs, i):
                setattr(obj, attr, value)
            obj_list.append(obj)
        return obj_list

    def fetchone(self) -> Optional[FetchObject]:
        result = self.cursor.fetchone()
        return self._tuple_to_object([result])[0] if result else None

    def fetchmany(self, size: Optional[int] = None) -> Optional[List[FetchObject]]:
        result = self.cursor.fetchmany(size)
        return self._tuple_to_object(result) if result else None

    def fetchall(self) -> Optional[List[FetchObject]]:
        result = self.cursor.fetchall()
        return self._tuple_to_object(result) if result else None
```

**运行期类型检查**

Type Hints 之所以叫 Hints 而不是 Check，就是因为它只是一个类型的提示而非真正的检查。上面演示的 Type Hints 用法，实际上都是 IDE 在帮我们完成类型检查的功能，但实际上，IDE 的类型检查并不能决定代码执行期间是否报错，仅能在静态期做到语法检查提示的功能。  

要想实现在代码执行阶段强制对类型进行检查，则需要我们通过自己编写代码或引入第三方库的形式（如上面介绍的 Pydantic）。下面我通过一个 type_check 函数实现了运行期动态检查类型，来供你参考：

```
from inspect import getfullargspec
from functools import wraps
from typing import get_type_hints


def type_check(fn):
    @wraps(fn)
    def wrapper(*args, **kwargs):
        fn_args = getfullargspec(fn)[0]
        kwargs.update(dict(zip(fn_args, args)))
        hints = get_type_hints(fn)
        hints.pop("return", None)
        for name, type_ in hints.items():
            if not isinstance(kwargs[name], type_):
                raise TypeError(f"expected {type_.__name__}, got {type(kwargs[name]).__name__} instead")
        return fn(**kwargs)

    return wrapper


# name: str = "world"
name: int = 2

@type_check
def greeting(name: str) -> str:
    return str(name)

print(greeting(name))
# > TypeError: expected str, got int instead
```

只要给 greeting 函数打上 type_check 装饰器，即可实现运行期类型检查。

**附录**

如果你想继续深入学习使用 Python Type Hints，以下是一些我推荐的开源项目供你参考：

*   Pydantic [https://github.com/samuelcolvin/pydantic]
    
*   FastAPI [https://github.com/tiangolo/fastapi]
    
*   Tornado [https://github.com/tornadoweb/tornado]
    
*   Flask [https://github.com/pallets/flask]
    
*   Chia-pool [https://github.com/Chia-Network/pool-reference]
    
*   MySQLHandler [https://github.com/jianghushinian/python-scripts/blob/main/scripts/mysql_handler_type_hints.py]