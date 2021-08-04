> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1628071894&ver=3232&signature=V9TbRl8-75i9dHKm7*lCG-4JSz4o-SG9*kNw0VwDPOVPjkd*P5V0D-d4xgWH*KHPe65xNx6VbWJtWZJ5SAqpJ0Q4*cYwno7PVt0tIpknqXfzmxyp-foYlulk3nF92zXV&new=1)

```
import logging
import sys
# 创建一个logger日志器
logger = logging.getLogger("test")
# 设置日志级别
logger.setLevel(logging.DEBUG)
#设置日志的处理器handler为console流,即在屏幕输出
handler = logging.StreamHandler(sys.stdout)
# 设置日志格式
formatter = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
handler.setFormatter(formatter)
# 向日志器添加处理器
logger.addHandler(handler)
# 日志调试
logger.debug("debug信息")
logger.info("info信息")
logger.warning("warning信息")
logger.error("error信息")

```

日志输出  

```
2020-02-17 15:53:52,127 - test - DEBUG - debug信息
2020-02-17 15:53:52,127 - test - INFO - info信息
2020-02-17 15:53:52,127 - test - WARNING - warning信息
2020-02-17 15:53:52,128 - test - ERROR - error信息

```

2、使用配置文件和 fileConfig() 函数实现日志配置

首先创建日志配置文件，文件名 logging_dev.conf

```
[loggers]
keys = root,test
[handlers]
keys = consoleHandler,fileHandler
[formatters]
keys = testFormatter
[logger_root]
level = DEBUG
handlers = consoleHandler
[logger_test]
level = DEBUG
handlers = fileHandler
qualname = test1
propagate = 1
[handler_consoleHandler]
class = StreamHandler
args = (sys.stdout,)
formatter = testFormatter
[handler_fileHandler]
class = FileHandler
args = ("test.log","a")
formatter = testFormatter
[formatter_testFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=

```

接下来，通过 fileConfig() 函数读取配置文件  

```
import logging.config
# 读取Log配置文件
logging.config.fileConfig("logging_dev.conf")
# 读取配置文件中的logger,读取qualname
logger = logging.getLogger("test1")
logger.debug("debug信息")
logger.info("info信息")
logger.warning("warning信息")
logger.error("error信息")

```

日志输出，分别为 console 输出和 test.log 输出：  

关于 fileConfig() 函数的说明：  

该函数定义在 loging.config 模块下：

```
logging.config.fileConfig(fname,defaults=None,disable_existing_loggers=True)

```

参数：

fname：表示配置文件的文件名或文件对象；

defaults：指定传给 ConfigParser 的默认值；

disable_existing_loggers：这是一个布尔型值，默认值为 True（为了向后兼容）表示禁用已经存在的 logger，除非它们或者它们的祖先明确的出现在日志配置中；如果值为 False 则对已存在的 loggers 保持启动状态。

配置文件格式说明：

1）配置文件中一定要包含 loggers、handlers、formatters 这些 section，它们通过 keys 这个 option 来指定该配置文件中已经定义好的 loggers、handlers 和 formatters，多个值之间用逗号分隔；另外 loggers 这个 section 中的 keys 一定要包含 root 这个值；  

如本例中创建的 loggers

```
[loggers]
keys = root,test

```

2）loggers、handlers、formatters 中所指定的日志器、处理器和格式器都需要在下面以单独的 section 进行定义。seciton 的命名规则为

[logger_loggerName]

[formatter_formatterName]

[handler_handlerName]

3）定义 logger 的 section 必须指定 level 和 handlers 这两个 option，level 的可取值为 DEBUG、INFO、WARNING、ERROR、CRITICAL、NOTSET，其中 NOTSET 表示所有级别的日志消息都要记录，包括用户定义级别；handlers 的值是以逗号分隔的 handler 名字列表，这里出现的 handler 必须出现在 [handlers] 这个 section 中，并且相应的 handler 必须在配置文件中有对应的 section 定义；

4）对于非 root logger 来说，除了 level 和 handlers 这两个 option 之外，还需要一些额外的 option，其中 **qualname** 是必须提供的 option，**它表示在 logger 层级中的名字，在应用代码中通过这个名字得到 logger**；**propagate** 是可选项，其**默认是为 1**，表示消息将会传递给高层次 logger 的 handler，通常我们需要指定其值为 0，对于非 root logger 的 level 如果设置为 NOTSET，系统将会查找高层次的 logger 来决定此 logger 的有效 level。

5）定义 handler 的 section 中必须指定 class 和 args 这两个 option，level 和 formatter 为可选 option；class 表示用于创建 handler 的类名，args 表示传递给 class 所指定的 handler 类初始化方法参数，它必须是一个元组（tuple）的形式，即便只有一个参数值也需要是一个元组的形式；

如在本例中：

```
[handler_consoleHandler]
class = StreamHandler
args = (sys.stdout,)
formatter = testFormatter
[handler_fileHandler]
class = FileHandler
args = ("test.log","a")
formatter = testFormatter

```

level 与 logger 中的 level 一样, 可以不写，也可以分别指定，而 formatter 指定的是该处理器所使用的格式器，这里指定的格式器名称必须出现在 formatters 这个 section 中，且在配置文件中必须要有这个 formatter 的 section 定义；如果不指定 formatter 则该 handler 将会以消息本身作为日志消息进行记录，而不添加额外的时间、日志器名称等信息；

6）定义 formatter 的 sectioin 中的 option 都是可选的，其中包括 format 用于指定格式字符串，默认为消息字符串本身；datefmt 用于指定 asctime 的时间格式，默认为'%Y-%m-%d %H:%M:%S'；class 用于指定格式器类名，默认为 logging.Formatter；

propagate 默认为 1，一般我们定义为 1 或不写，我们可以将以上例子的 propagate 修改为 0 看一下效果。

修改后，**console 无输出，仅在 test.log 中有输出**。  

**3、使用字典配置信息和 dictConfig() 函数实现日志配置**

相对于上面所讲的基于配置文件来保存 logging 配置信息的方式来说，功能更加强大，也更加灵活，因为我们可把很多的数据转换成字典。比如，我们可以使用 JSON 格式的配置文件、YAML 格式的配置文件，然后将它们填充到一个配置字典中；或者，我们也可以用 Python 代码构建这个配置字典，或者通过 socket 接收 pickled 序列化后的配置信息。总之，你可以使用你的应用程序可以操作的任何方法来构建这个配置字典。

首先，我们先来创建一个字典文件，文件名为 logging_dict.py：

```
log_conf = {
    "version": 1,
    "formatters": {
        "test": {
            "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
        }
    },
    "filters": {},
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "formatter": "test"
        },
        "file": {
            "class": "logging.FileHandler",
            "formatter": "test",
            "filename": "test.log"
        }
    },
    "loggers": {
        "test": {
            "handlers": ["console","file"],
            "level": "DEBUG"
        }
    }
}

```

接下来，我们使用 dictConfig() 来读取这个字典文件

```
import logging.config
from lib.logging_dict import log_conf
logging.config.dictConfig(log_conf)
logger = logging.getLogger("test")
logger.debug("debug信息")
logger.info("info信息")
logger.warning("warning信息")
logger.error("error信息")

```

日志输出：  

```
2020-02-17 16:07:18,064 - test - DEBUG - debug信息
2020-02-17 16:07:18,064 - test - INFO - info信息
2020-02-17 16:07:18,065 - test - WARNING - warning信息
2020-02-17 16:07:18,065 - test - ERROR - error信息

```

```
logging.config.dictConfig(config)

```

该函数可以从一个字典对象中获取日志配置信息，config 参数就是这个字典对象。

**配置字典说明**

无论是上面提到的配置文件，还是这里的配置字典，它们都要描述出日志配置所需要创建的各种对象以及这些对象之间的关联关系。比如，可以先创建一个名额为 “test” 的格式器 formatter；然后创建一个名为 “console” 的处理器 handler，并指定该 handler 输出日志所使用的格式器为 "test"；然后再创建一个日志器 logger，并指定它所使用的处理器为 "console"。

传递给 dictConfig() 函数的字典对象只能包含下面这些 keys，其中 version 是必须指定的 key，其它 key 都是可选项：

![](https://mmbiz.qpic.cn/mmbiz_png/uVuXibmDicWpVuCK2oWuZnSy9V3u95820lewuBGRMjvg1hAlJ2f80lGEZabXicNXN0FMHYdpGTUC1mtnm0fwpWVSQ/640?wx_fmt=png)

文章参考：  

```
python之配置日志的几种方式 https://www.cnblogs.com/yyds/p/6885182.html
logging.config — Logging configuration https://docs.python.org/3.5/library/logging.config.html

```