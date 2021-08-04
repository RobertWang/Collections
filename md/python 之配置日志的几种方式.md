> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/yyds/p/6885182.html)

作为开发者，我们可以通过以下 3 种方式来配置 logging:

*   1）使用 Python 代码显式的创建 loggers, handlers 和 formatters 并分别调用它们的配置函数；
*   2）创建一个日志配置文件，然后使用`fileConfig()`函数来读取该文件的内容；
*   3）创建一个包含配置信息的 dict，然后把它传递个`dictConfig()`函数；

需要说明的是，`logging.basicConfig()`也属于第一种方式，它只是对 loggers, handlers 和 formatters 的配置函数进行了封装。另外，第二种配置方式相对于第一种配置方式的优点在于，它将配置信息和代码进行了分离，这一方面降低了日志的维护成本，同时还使得非开发人员也能够去很容易地修改日志配置。

一、使用 Python 代码实现日志配置
--------------------

代码如下：

```
# 创建一个日志器logger并设置其日志级别为DEBUG
logger = logging.getLogger('simple_logger')
logger.setLevel(logging.DEBUG)

# 创建一个流处理器handler并设置其日志级别为DEBUG
handler = logging.StreamHandler(sys.stdout)
handler.setLevel(logging.DEBUG)

# 创建一个格式器formatter并将其添加到处理器handler
formatter = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
handler.setFormatter(formatter)

# 为日志器logger添加上面创建的处理器handler
logger.addHandler(handler)

# 日志输出
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')


```

运行输出：

```
2017-05-15 11:30:50,955 - simple_logger - DEBUG - debug message
2017-05-15 11:30:50,955 - simple_logger - INFO - info message
2017-05-15 11:30:50,955 - simple_logger - WARNING - warn message
2017-05-15 11:30:50,955 - simple_logger - ERROR - error message
2017-05-15 11:30:50,955 - simple_logger - CRITICAL - critical message


```

二、使用配置文件和 fileConfig() 函数实现日志配置
-------------------------------

现在我们通过配置文件的方式来实现与上面同样的功能：

```
# 读取日志配置文件内容
logging.config.fileConfig('logging.conf')

# 创建一个日志器logger
logger = logging.getLogger('simpleExample')

# 日志输出
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')


```

配置文件`logging.conf`内容如下：

```
[loggers]
keys=root,simpleExample

[handlers]
keys=fileHandler,consoleHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=DEBUG
handlers=fileHandler

[logger_simpleExample]
level=DEBUG
handlers=consoleHandler
qualname=simpleExample
propagate=0

[handler_consoleHandler]
class=StreamHandler
args=(sys.stdout,)
level=DEBUG
formatter=simpleFormatter

[handler_fileHandler]
class=FileHandler
args=('logging.log', 'a')
level=ERROR
formatter=simpleFormatter

[formatter_simpleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=


```

运行输出：

```
2017-05-15 11:32:16,539 - simpleExample - DEBUG - debug message
2017-05-15 11:32:16,555 - simpleExample - INFO - info message
2017-05-15 11:32:16,555 - simpleExample - WARNING - warn message
2017-05-15 11:32:16,555 - simpleExample - ERROR - error message
2017-05-15 11:32:16,555 - simpleExample - CRITICAL - critical message


```

### 1. 关于`fileConfig()`函数的说明：

该函数实际上是对`configparser`模块的封装，关于`configparser`模块的介绍请参考 [<<Python 之 xml 文档及配置文件处理>>](http://www.cnblogs.com/yyds/p/6627208.html) 。

##### 函数定义：

该函数定义在 loging.config 模块下：

```
logging.config.fileConfig(fname, defaults=None, disable_existing_loggers=True)


```

##### 参数：

*   fname：表示配置文件的文件名或文件对象
*   defaults：指定传给 ConfigParser 的默认值
*   disable_existing_loggers：这是一个布尔型值，默认值为 True（为了向后兼容）表示禁用已经存在的 logger，除非它们或者它们的祖先明确的出现在日志配置中；如果值为 False 则对已存在的 loggers 保持启动状态。

### 2. 配置文件格式说明：

上面提到过，`fileConfig()`函数是对`ConfigParser/configparser`模块的封装，也就是说`fileConfig()`函数是基于`ConfigParser/configparser`模块来理解日志配置文件的。换句话说，`fileConfig()`函数所能理解的配置文件基础格式是与`ConfigParser/configparser`模块一致的，只是在此基础上对文件中包含的`section`和`option`做了一下规定和限制，比如：

*   1）配置文件中一定要包含`loggers`、`handlers`、`formatters`这些 section，它们通过`keys`这个 option 来指定该配置文件中已经定义好的 loggers、handlers 和 formatters，多个值之间用逗号分隔；另外`loggers`这个 section 中的 keys 一定要包含 root 这个值；
    
*   2）`loggers`、`handlers`、`formatters`中所指定的日志器、处理器和格式器都需要在下面以单独的 section 进行定义。seciton 的命名规则为`[logger_loggerName]`、`[formatter_formatterName]`、`[handler_handlerName]`
    
*   3）定义 logger 的 section 必须指定`level`和`handlers`这两个 option，`level`的可取值为`DEBUG`、`INFO`、`WARNING`、`ERROR`、`CRITICAL`、`NOTSET`，其中`NOTSET`表示所有级别的日志消息都要记录，包括用户定义级别；`handlers`的值是以逗号分隔的 handler 名字列表，这里出现的 handler 必须出现在 [handlers] 这个 section 中，并且相应的 handler 必须在配置文件中有对应的 section 定义；
    
*   4）对于非 root logger 来说，除了`level`和`handlers`这两个 option 之外，还需要一些额外的 option，其中`qualname`是必须提供的 option，它表示在 logger 层级中的名字，在应用代码中通过这个名字得到 logger；`propagate`是可选项，其默认是为 1，表示消息将会传递给高层次 logger 的 handler，通常我们需要指定其值为 0，这个可以看下下面的例子；另外，对于非 root logger 的 level 如果设置为 NOTSET，系统将会查找高层次的 logger 来决定此 logger 的有效 level。
    
*   5）定义 handler 的 section 中必须指定`class`和`args`这两个 option，`level`和`formatter`为可选 option；`class`表示用于创建 handler 的类名，`args`表示传递给`class`所指定的 handler 类初始化方法参数  
    ，它必须是一个元组（tuple）的形式，即便只有一个参数值也需要是一个元组的形式；`level`与 logger 中的 level 一样，而`formatter`指定的是该处理器所使用的格式器，这里指定的格式器名称必须出现在`formatters`这个 section 中，且在配置文件中必须要有这个 formatter 的 section 定义；如果不指定 formatter 则该 handler 将会以消息本身作为日志消息进行记录，而不添加额外的时间、日志器名称等信息；
    
*   6）定义 formatter 的 sectioin 中的 option 都是可选的，其中包括`format`用于指定格式字符串，默认为消息字符串本身；`datefmt`用于指定 asctime 的时间格式，默认为`'%Y-%m-%d %H:%M:%S'`；`class`用于指定格式器类名，默认为 logging.Formatter；
    

> _**说明：**_
> 
> 配置文件中的`class`指定类名时，该类名可以是相对于 logging 模块的相对值，如：`FileHandler`、`handlers.TimeRotatingFileHandler`；也可以是一个绝对路径值，通过普通的 import 机制来解析，如自定义的 handler 类`mypackage.mymodule.MyHandler`，但是 mypackage 需要在 Python 可用的导入路径中 --sys.path。

### 3. 对于 propagate 属性的说明

##### 实例 1：

我们把`logging.conf`中 simpleExample 这个 handler 定义中的 propagate 属性值改为 1，或者删除这个 option（默认值就是 1）：

```
[logger_simpleExample]
level=DEBUG
handlers=consoleHandler
qualname=simpleExample
propagate=1


```

现在来执行同样的代码：

```
# 读取日志配置文件内容
logging.config.fileConfig('logging.conf')

# 创建一个日志器logger
logger = logging.getLogger('simpleExample')

# 日志输出
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')


```

我们会发现，除了在控制台有输出信息时候，在 logging.log 文件中也有内容输出:

```
2017-05-15 16:06:25,366 - simpleExample - ERROR - error message
2017-05-15 16:06:25,367 - simpleExample - CRITICAL - critical message


```

这说明 simpleExample 这个 logger 在处理完日志记录后，把日志记录传递给了上级的 root logger 再次做处理，所有才会有两个地方都有日志记录的输出。通常，我们都需要显示的指定 propagate 的值为 0，防止日志记录向上层 logger 传递。

##### 实例 2：

现在，我们试着用一个没有在配置文件中定义的 logger 名称来获取 logger：

```
# 读取日志配置文件内容
logging.config.fileConfig('logging.conf')

# 用一个没有在配置文件中定义的logger名称来创建一个日志器logger
logger = logging.getLogger('simpleExample1')

# 日志输出
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')


```

运行程序后，我们会发现控制台没有任何输出，而 logging.log 文件中又多了两行输出：

```
2017-05-15 16:13:16,810 - simpleExample1 - ERROR - error message
2017-05-15 16:13:16,810 - simpleExample1 - CRITICAL - critical message


```

这是因为，当一个日志器没有被设置任何处理器是，系统会去查找该日志器的上层日志器上所设置的日志处理器来处理日志记录。simpleExample1 在配置文件中没有被定义，因此`logging.getLogger(simpleExample1)`这行代码这是获取了一个 logger 实例，并没有给它设置任何处理器，但是它的上级日志器 --root logger 在配置文件中有定义且设置了一个 FileHandler 处理器，simpleExample1 处理器最终通过这个 FileHandler 处理器将日志记录输出到 logging.log 文件中了。

三、使用字典配置信息和 dictConfig() 函数实现日志配置
---------------------------------

Python 3.2 中引入的一种新的配置日志记录的方法 -- 用字典来保存 logging 配置信息。这相对于上面所讲的基于配置文件来保存 logging 配置信息的方式来说，功能更加强大，也更加灵活，因为我们可把很多的数据转换成字典。比如，我们可以使用 JSON 格式的配置文件、YAML 格式的配置文件，然后将它们填充到一个配置字典中；或者，我们也可以用 Python 代码构建这个配置字典，或者通过 socket 接收 pickled 序列化后的配置信息。总之，你可以使用你的应用程序可以操作的任何方法来构建这个配置字典。

这个例子中，我们将使用 YAML 格式来完成与上面同样的日志配置。

首先需要安装 PyYAML 模块：

```
pip install PyYAML


```

Python 代码：

```
import logging
import logging.config
import yaml

with open('logging.yml', 'r') as f_conf:
    dict_conf = yaml.load(f_conf)
logging.config.dictConfig(dict_conf)

logger = logging.getLogger('simpleExample')
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')


```

logging.yml 配置文件的内容：

```
version: 1
formatters:
  simple:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
handlers:
  console:
    class: logging.StreamHandler
    level: DEBUG
    formatter: simple
    stream: ext://sys.stdout
  console_err:
    class: logging.StreamHandler
    level: ERROR
    formatter: simple
    stream: ext://sys.stderr
loggers:
  simpleExample:
    level: DEBUG
    handlers: [console]
    propagate: yes
root:
  level: DEBUG
  handlers: [console_err]


```

输出结果：

```
2017-05-21 14:19:31,089 - simpleExample - DEBUG - debug message
2017-05-21 14:19:31,089 - simpleExample - INFO - info message
2017-05-21 14:19:31,089 - simpleExample - WARNING - warn message
2017-05-21 14:19:31,089 - simpleExample - ERROR - error message
2017-05-21 14:19:31,090 - simpleExample - CRITICAL - critical message


```

### 1. 关于`dictConfig()`函数的说明：

该函数实际上是对`configparser`模块的封装，关于`configparser`模块的介绍请参考 [<<Python 之 xml 文档及配置文件处理>>](http://www.cnblogs.com/yyds/p/6627208.html) 。

##### 函数定义：

该函数定义在 loging.config 模块下：

```
logging.config.dictConfig(config)


```

该函数可以从一个字典对象中获取日志配置信息，config 参数就是这个字典对象。关于这个字典对象的内容规则会在下面进行描述。

### 2. 配置字典说明

无论是上面提到的配置文件，还是这里的配置字典，它们都要描述出日志配置所需要创建的各种对象以及这些对象之间的关联关系。比如，可以先创建一个名额为 “simple” 的格式器 formatter；然后创建一个名为 “console” 的处理器 handler，并指定该 handler 输出日志所使用的格式器为 "simple"；然后再创建一个日志器 logger，并指定它所使用的处理器为 "console"。

传递给 dictConfig() 函数的字典对象只能包含下面这些 keys，其中`version`是必须指定的 key，其它 key 都是可选项：

<table><thead><tr><th>key 名称</th><th>描述</th></tr></thead><tbody><tr><td>version</td><td>必选项，其值是一个整数值，表示配置格式的版本，当前唯一可用的值就是 1</td></tr><tr><td>formatters</td><td>可选项，其值是一个字典对象，该字典对象每个元素的 key 为要定义的格式器名称，value 为格式器的配置信息组成的 dict，如 format 和 datefmt</td></tr><tr><td>filters</td><td>可选项，其值是一个字典对象，该字典对象每个元素的 key 为要定义的过滤器名称，value 为过滤器的配置信息组成的 dict，如 name</td></tr><tr><td>handlers</td><td>可选项，其值是一个字典对象，该字典对象每个元素的 key 为要定义的处理器名称，value 为处理器的配置信息组成的 dcit，如 class、level、formatter 和 filters，其中 class 为必选项，其它为可选项；其他配置信息将会传递给 class 所指定的处理器类的构造函数，如下面的 handlers 定义示例中的 stream、filename、maxBytes 和 backupCount 等</td></tr><tr><td>loggers</td><td>可选项，其值是一个字典对象，该字典对象每个元素的 key 为要定义的日志器名称，value 为日志器的配置信息组成的 dcit，如 level、handlers、filters 和 propagate（yes</td></tr><tr><td>root</td><td>可选项，这是 root logger 的配置信息，其值也是一个字典对象。除非在定义其它 logger 时明确指定 propagate 值为 no，否则 root logger 定义的 handlers 都会被作用到其它 logger 上</td></tr><tr><td>incremental</td><td>可选项，默认值为 False。该选项的意义在于，如果这里定义的对象已经存在，那么这里对这些对象的定义是否应用到已存在的对象上。值为 False 表示，已存在的对象将会被重新定义。</td></tr><tr><td>disable_existing_loggers</td><td>可选项，默认值为 True。该选项用于指定是否禁用已存在的日志器 loggers，如果 incremental 的值为 True 则该选项将会被忽略</td></tr></tbody></table>

handlers 定义示例：

```
handlers:
  console:
    class : logging.StreamHandler
    formatter: brief
    level   : INFO
    filters: [allow_foo]
    stream  : ext://sys.stdout
  file:
    class : logging.handlers.RotatingFileHandler
    formatter: precise
    filename: logconfig.log
    maxBytes: 1024
    backupCount: 3



```

### 3. 关于外部对象的访问

需要说明的是，上面所使用的对象并不限于 loggging 模块所提供的对象，我们可以实现自己的 formatter 或 handler 类。另外，这些类的参数也许需要包含 sys.stderr 这样的外部对象。如果配置字典对象是使用 Python 代码构造的，可以直接使用 sys.stdout、sys.stderr；但是当通过文本文件（如 JSON、YAML 格式的配置文件）提供配置时就会出现问题，因为在文本文件中，没有标准的方法来区分`sys.stderr`和字符串`'sys.stderr'`。为了区分它们，配置系统会在字符串值中查找特定的前缀，例如'ext://sys.stderr'中'ext://'会被移除，然后`import sys.stderr`。

*   参考文档：[https://docs.python.org/3.5/library/logging.config.html](https://docs.python.org/3.5/library/logging.config.html)