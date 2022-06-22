> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652594083&idx=1&sn=148b680498f2bd13b26d8bd459bb0643&chksm=846577e9b312feff6bc722309ac811429826c928ec03af2b44e4ce378f3515e7f03f107c7d1f&scene=21#wechat_redirect)

今天要分享的主题是：改变类定义的神器 - metaclass  

我们将带大家设计一个简单的 orm 框架，并简单剖析一下 YAML 这个序列化工具的原理。

Python 类的上帝 - type
------------------

1.  **对象是类的实例**
    
2.  **类是 type 的实例**
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/P33BNFicia1AlqCV8FqDMlzQV6KnWwwBoVjQx6YTlnCFY8cEv3LO7wn2RqXwvsZqOU8AGXKibzJmvTdm91qu0g6yA/640?wx_fmt=jpeg)

总之，类就是创建对象的模板。

而 type 又是创建类的模板，那么我们就可以通过 type 创建自己想要的类。

比如定义一个 Hello 的 class：

当 Python 解释器载入 hello 模块时，就会依次执行该模块的所有语句，执行结果就是动态创建出一个 Hello 的 class 对象。

type() 函数既可以查看一个类型或变量的类型，也可以根据参数创建出新的类型，比如上面那段类的定义本质上就是：

```
class Hello(object):    def hello(self, name='world'):     print('Hello, %s.' % name)
```

type() 函数创建 class 对象，依次传入 3 个参数：

*   class 类的名称；
    
*   继承的父类集合，注意 Python 支持多重继承，如果只有一个父类，别忘了 tuple 的单元素写法；
    
*   class 的方法名称与函数绑定以及字段名称与对应的值，这里我们把函数 fn 绑定到方法名 hello 上。
    

通过 type() 函数创建的类和直接写 class 是完全一样的，因为 Python 解释器遇到 class 定义时，仅仅是扫描一下 class 定义的语法，然后调用 type() 函数创建出 class。

metaclass 到底是什么
---------------

对于类 MyClass：

```
def hello(self, name='world'):    print('Hello, %s.' % name)Hello = type('Hello', (object,), dict(hello=hello))
```

其实相当于：

```
class MyClass(): pass
```

一旦我们把它的 metaclass 设置成 MyMeta：

```
class MyClass(metaclass = type): pass
```

MyClass 就不再由原生的 type 创建，而是会调用 MyMeta 的`__call__`运算符重载。

```
class MyClass(metaclass = MyMeta): pass
```

对于具有继承关系的类：

```
class = type(classname, superclasses, attributedict) ## 变为了class = MyMeta(classname, superclasses, attributedict)
```

Python 做了如下的操作：

*   Foo 中有__metaclass__这个属性吗？如果是，Python 会通过__metaclass__创建一个名字为 Foo 的类 (对象)
    
*   如果 Python 没有找到__metaclass__，它会继续在 Bar（父类）中寻找__metaclass__属性，并尝试做和前面同样的操作。
    
*   如果 Python 在任何父类中都找不到__metaclass__，它就会在模块层次中去寻找__metaclass__，并尝试做同样的操作。
    
*   如果还是找不到__metaclass__，Python 就会用内置的 type 来创建这个类对象。
    

假想一个很傻的例子，你决定在你的模块里所有的类的属性都应该是大写形式。有好几种方法可以办到，但其中一种就是通过在模块级别设定__metaclass__：

```
class Foo(Bar): pass
```

简易 ORM 框架的设计
------------

ORM 全称 “Object Relational Mapping”，即对象 - 关系映射，就是把关系数据库的一行映射为一个对象，也就是一个类对应一个表，这样，写代码更简单，不用直接操作 SQL 语句。

现在设计一下 ORM 框架的调用接口，比如用户想通过`User`类来操作对应的数据库表`User`，我们期待他写出这样的代码：

```
class UpperAttrMetaClass(type):    ## __new__ 是在__init__之前被调用的特殊方法    ## __new__是用来创建对象并返回之的方法    ## 而__init__只是用来将传入的参数初始化给对象    ## 你很少用到__new__，除非你希望能够控制对象的创建    ## 这里，创建的对象是类，我们希望能够自定义它，所以我们这里改写__new__    ## 如果你希望的话，你也可以在__init__中做些事情    ## 还有一些高级的用法会涉及到改写__call__特殊方法，但是我们这里不用    def __new__(cls, future_class_name, future_class_parents, future_class_attr):        ##遍历属性字典，把不是__开头的属性名字变为大写        newAttr = {}        for name,value in future_class_attr.items():            if not name.startswith("__"):                newAttr[name.upper()] = value        ## 方法1：通过'type'来做类对象的创建        ## return type(future_class_name, future_class_parents, newAttr)        ## 方法2：复用type.__new__方法，这就是基本的OOP编程        ## return type.__new__(cls, future_class_name, future_class_parents, newAttr)        ## 方法3：使用super方法        return super(UpperAttrMetaClass, cls).__new__(cls, future_class_name, future_class_parents, newAttr)class Foo(object, metaclass = UpperAttrMetaClass):    bar = 'bip'print(hasattr(Foo, 'bar'))## 输出: Falseprint(hasattr(Foo, 'BAR'))## 输出:Truef = Foo()print(f.BAR)## 输出:'bip'
```

上面的接口通过常规方法很难或几乎很难实现，但通过 metaclass 就会相对比较简单。核心思想就是通过 metaclass 修改类的定义，将类的所有 Field 类型的属性，用一个额外的字典去保存，然后从原定义中删除。对于 User 创建对象时传入的参数（id=12345, name='xiaoxiaoming'等）可以模仿字典的实现或直接继承 dict 类保存起来。

其中，父类`Model`和属性类型`StringField`、`IntegerField`是由 ORM 框架提供的，剩下的魔术方法比如`save()`全部由 metaclass 自动完成。虽然 metaclass 的编写会比较复杂，但 ORM 的使用者用起来却异常简单。

首先定义 Field 类，它负责保存数据库表的字段名和字段类型：

```
class User(Model):    ## 定义类的属性到列的映射：    id = IntegerField('id')    name = StringField('username')    email = StringField('email')    password = StringField('password')## 创建一个实例：u = User(id=12345, name='xiaoxiaoming', email='test@orm.org', password='my-pwd')## 保存到数据库：u.save()
```

```
class Field(object):    def __init__(self, name, column_type):        self.name = name        self.column_type = column_type    def __str__(self):        return '<%s:%s>' % (self.__class__.__name__, self.name)
```

```
class StringField(Field):    def __init__(self, name):        super(StringField, self).__init__(name, 'varchar(100)')class IntegerField(Field):    def __init__(self, name):        super(IntegerField, self).__init__(name, 'bigint')
```

```
class ModelMetaclass(type):    def __new__(cls, name, bases, attrs):        if name == 'Model':            return type.__new__(cls, name, bases, attrs)        print('Found model: %s' % name)        mappings = dict()        for k, v in attrs.items():            if isinstance(v, Field):                print('Found mapping: %s ==> %s' % (k, v))                mappings[k] = v        for k in mappings.keys():            attrs.pop(k)        attrs['__mappings__'] = mappings  ## 保存属性和列的映射关系        attrs.setdefault('__table__', name) ## 当未定义__table__属性时，表名直接使用类名        return type.__new__(cls, name, bases, attrs)
```

在`ModelMetaclass`中，一共做了几件事情：

1.  在当前类（比如`User`）中查找定义的类的所有属性，如果找到一个 Field 属性，就把它保存到一个`__mappings__`的 dict 中，同时从类属性中删除该 Field 属性（避免实例的属性遮盖类的同名属性）；
    
2.  当类中未定义`__table__`字段时，直接将类名保存到`__table__`字段中作为表名。
    

在`Model`类中，就可以定义各种操作数据库的方法，比如`save()`，`delete()`，`find()`，`update`等等。

我们实现了`save()`方法，把一个实例保存到数据库中。因为有表名，属性到字段的映射和属性值的集合，就可以构造出`INSERT`语句。

测试：

```
class Model(dict, metaclass=ModelMetaclass):    def __init__(self, **kw):        super(Model, self).__init__(**kw)    def __getattr__(self, key):        try:            return self[key]        except KeyError:            raise AttributeError(r"'Model' object has no attribute '%s'" % key)    def __setattr__(self, key, value):        self[key] = value    def save(self):        fields = []        params = []        args = []        for k, v in self.__mappings__.items():            fields.append(v.name)            params.append('?')            args.append(getattr(self, k, None))        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join(params))        print('SQL: %s' % sql)        print('ARGS: %s' % str(args))
```

输出如下：

```
u = User(id=12345, name='xiaoxiaoming', email='test@orm.org', password='my-pwd')u.save()
```

测试 2：

```
Found model: UserFound mapping: id ==> <IntegerField:id>Found mapping: name ==> <StringField:username>Found mapping: email ==> <StringField:email>Found mapping: password ==> <StringField:password>SQL: insert into User (id,username,email,password) values (?,?,?,?)ARGS: [12345, 'xiaoxiaoming', 'test@orm.org', 'my-pwd']
```

输出：

```
class Blog(Model):    __table__ = 'blogs'    id = IntegerField('id')    user_id = StringField('user_id')    user_name = StringField('user_name')    name = StringField('user_name')    summary = StringField('summary')    content = StringField('content')b = Blog(id=12345, user_id='user_id1', user_)b.save()
```

可以看到，`save()`方法已经打印出了可执行的 SQL 语句，以及参数列表，只需要真正连接到数据库，执行该 SQL 语句，就可以完成真正的功能。

YAML 序列化工具的实现原理浅析
-----------------

YAML 是一个家喻户晓的 Python 工具，可以方便地序列化 / 逆序列化结构数据。

官方文档：https://pyyaml.org/wiki/PyYAMLDocumentation

安装：

```
Found model: BlogFound mapping: id ==> <IntegerField:id>Found mapping: user_id ==> <StringField:user_id>Found mapping: user_name ==> <StringField:user_name>Found mapping: name ==> <StringField:user_name>Found mapping: summary ==> <StringField:summary>Found mapping: content ==> <StringField:content>SQL: insert into blogs (id,user_id,user_name,user_name,summary,content) values (?,?,?,?,?,?)ARGS: [12345, 'user_id1', 'xxm', 'orm框架的基本运行机制', '简单讲述一下orm框架的基本运行机制', '此处省略一万字...']
```

YAMLObject 的任意子类支持序列化和反序列化（serialization & deserialization）。比如说下面这段代码：

```
pip install pyyaml<br style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;" data-darkmode-bgcolor-16559241920512="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16559241920512="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-16559241920512="rgb(171, 178, 191)" data-darkmode-original-color-16559241920512="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

运行结果：

```
import yamlclass Monster(yaml.YAMLObject):    yaml_tag = '!Monster'    def __init__(self, name, hp, ac, attacks):        self.name = name        self.hp = hp        self.ac = ac        self.attacks = attacks    def __repr__(self):        return f"{self.__class__.__name__}(name={self.name}, hp={self.hp}, ac={self.ac}, attacks={self.attacks})"monster1 = yaml.load("""--- !Monstername: Cave spiderhp: [2,6]ac: 16attacks: [BITE, HURT]""")print(monster1, type(monster1))monster2 = Monster(name='Cave lizard', hp=[3, 6], ac=16, attacks=['BITE', 'HURT'])print(yaml.dump(monster2))
```

这里面调用统一的 yaml.load()，就能把任意一个 yaml 序列载入成一个 Python Object；而调用统一的 yaml.dump()，就能把一个 YAMLObject 子类序列化。

对于 load() 和 dump() 的使用者来说，他们完全不需要提前知道任何类型信息，这让超动态配置编程成了可能。比方说，在一个智能语音助手的大型项目中，我们有 1 万个语音对话场景，每一个场景都是不同团队开发的。作为智能语音助手的核心团队成员，我不可能去了解每个子场景的实现细节。

在动态配置实验不同场景时，经常是今天我要实验场景 A 和 B 的配置，明天实验 B 和 C 的配置，光配置文件就有几万行量级，工作量不可谓不小。而应用这样的动态配置理念，就可以让引擎根据配置文件，动态加载所需要的 Python 类。

对于 YAML 的使用者也很方便，只要简单地继承 yaml.YAMLObject，就能让你的 Python Object 具有序列化和逆序列化能力。

据说即使是在大厂 Google 的 Python 开发者，发现能深入解释 YAML 这种设计模式优点的人，大概只有 10%。而能知道类似 YAML 的这种动态序列化 / 逆序列化功能正是用 metaclass 实现的人，可能只有 1% 了。而能够将 YAML 怎样用 metaclass 实现动态序列化 / 逆序列化功能讲出一二的可能只有 0.1% 了。

对于 YAMLObject 的 load 和 dump() 功能，简单来说，我们需要一个全局的注册器，让 YAML 知道，序列化文本中的`!Monster`需要载入成 Monster 这个 Python 类型，Monster 这个 Python 类型需要被序列化为`!Monster`标签开头的字符串。

一个很自然的想法就是，那我们建立一个全局变量叫 registry，把所有需要逆序列化的 YAMLObject，都注册进去。比如下面这样：

```
Monster(name=Cave spider, hp=[2, 6], ac=16, attacks=['BITE', 'HURT']) <class '__main__.Monster'>!Monsterac: 16attacks: [BITE, HURT]hp: [3, 6]name: Cave lizard
```

然后，在 Monster 类定义后面加上下面这行代码：

```
registry = {} def add_constructor(target_class):    registry[target_class.yaml_tag] = target_class
```

这样的缺点很明显，对于 YAML 的使用者来说，每一个 YAML 的可逆序列化的类 Foo 定义后，都需要加上一句话`add_constructor(Foo)`。这无疑给开发者增加了麻烦，也更容易出错，毕竟开发者很容易忘了这一点。

更优雅的实现方式自然是通过 metaclass 解决了这个问题，YAML 的源码正是这样实现的：

```
add_constructor(Monster)<br style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;" data-darkmode-bgcolor-16559241920512="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16559241920512="#fff|rgb(255, 255, 255)|rgb(40, 44, 52)" data-darkmode-color-16559241920512="rgb(171, 178, 191)" data-darkmode-original-color-16559241920512="#fff|rgb(0, 0, 0)|rgb(171, 178, 191)">
```

可以看到，YAMLObject 把 metaclass 声明成了 YAMLObjectMetaclass，YAMLObjectMetaclass 则会改变 YAMLObject 类和其子类的定义，就是下面这行代码将 YAMLObject 的子类加入到了 yaml 的两个全局注册表中：

```
class YAMLObjectMetaclass(type):    def __init__(cls, name, bases, kwds):        super(YAMLObjectMetaclass, cls).__init__(name, bases, kwds)        if 'yaml_tag' in kwds and kwds['yaml_tag'] is not None:            cls.yaml_loader.add_constructor(cls.yaml_tag, cls.from_yaml)            cls.yaml_dumper.add_representer(cls, cls.to_yaml)    ## 省略其余定义 class YAMLObject(metaclass=YAMLObjectMetaclass):    yaml_loader = Loader    yaml_dumper = Dumper    ## 省略其余定义
```

YAML 应用 metaclass，拦截了所有 YAMLObject 子类的定义。也就是说，在你定义任何 YAMLObject 子类时，Python 会强行插入运行上面这段代码，把我们之前想要的`add_constructor(Foo)`和`add_representer(Foo)`给自动加上。所以 YAML 的使用者，无需自己去手写`add_constructor(Foo)`和`add_representer(Foo)`。

总结
--

这次分享主要是简单的浅析了 metaclass 的实现机制。通过实现一个 orm 框架并解读 YAML 的源码，相信你已经对 metaclass 有了不错的理解。

metaclass 是 Python 黑魔法级别的语言特性，它可以改变类创建时的行为，这种强大的功能使用起来务必小心。

看完本文，你觉得装饰器和 metaclass 有什么区别呢？欢迎下方留言讨论。

- EOF -

1、[Python 处理超大 JSON 文件，这个方法简单！](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652593771&idx=1&sn=be6181848a3a08dc064a7febdf81f54a&chksm=84657621b312ff37c28f21c2c98c14325a61aa783bf6086d5c59390ff12fe8e716ceaf4cb5fe&scene=21#wechat_redirect)

2、[又一机器学习模型解释神器：Shapash](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652593817&idx=1&sn=6bf78668fa0df361bcf2a0b87b49b79e&chksm=846576d3b312ffc56b9cd6fe7ebf828648354a0895690a5b17abb01cb0c8151dcb7d4d593366&scene=21#wechat_redirect)

3、[你以为的万能爬虫方法，其实一行代码就能识别！](http://mp.weixin.qq.com/s?__biz=MzA4MjEyNTA5Mw==&mid=2652593836&idx=3&sn=987528c0d4604a74fbb19fd5a0e48b40&chksm=846576e6b312fff0305e5a1b2b381fe4b35ecdcef81cf8d6c2b03b43cabe8f240cff8702440a&scene=21#wechat_redirect)