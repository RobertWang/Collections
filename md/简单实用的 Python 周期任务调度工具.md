> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650808857&idx=3&sn=f9f941d446bb533c4cfe338e0193a8b2&chksm=bd01b7578a763e417f7afbc669e962507a656f6b952b07b99886370c753b5e64c00a791f27dc&mpshare=1&scene=1&srcid=0708cT7k2S13B0HLvu8rJnem&sharer_sharetime=1625740233554&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

_________出处：Python 实用宝典（ID：pythondict）_________

_________作者__________________：Ckend_________

_________如若转载请联系原公众号_________

如果你想周期性地执行某个 Python 脚本，最出名的选择应该是 Crontab 脚本，但是 Crontab 具有以下缺点：

*   1. 不方便执行**秒级任务**。
    
*   2. 当需要执行的定时任务有上百个的时候，Crontab 的**管理就会特别不方便**。
    

还有一个选择是 Celery，但是 Celery 的配置比较麻烦，如果你只是需要一个轻量级的调度工具，Celery 不会是一个好选择。

在你想要使用一个轻量级的任务调度工具，而且希望它尽量简单、容易使用、不需要外部依赖，最好能够容纳 Crontab 的所有基本功能，那么 Schedule 模块是你的不二之选。

使用它来调度任务可能只需要几行代码，感受一下：

```
# Python 实用宝典
import schedule
import time

def job():
    print("I'm working...")

schedule.every(10).minutes.do(job)

while True:
    schedule.run_pending()
    time.sleep(1)

```

上面的代码表示每 10 分钟执行一次 job 函数，非常简单方便。你只需要引入 schedule 模块，通过调用 **`scedule.every(时间数).时间类型.do(job)`**  发布周期任务。

发布后的周期任务需要用 **`run_pending`** 函数来检测是否执行，因此需要一个 **`While`** 循环不断地轮询这个函数。

下面具体讲讲 Schedule 模块的安装和初级、进阶使用方法。

_**1. 准备**_

  
开始之前，你要确保 Python 和 pip 已经成功安装在电脑上，如果没有，可以访问这篇文章：[超详细 Python 安装指南](http://mp.weixin.qq.com/s?__biz=MzI3MzM0ODU4Mg==&mid=2247485004&idx=1&sn=6f89120cf926e71c7eb4788744ff625f&chksm=eb25e4c5dc526dd31f216f56b963179a0bc301a5654644ef98f436aa4740caa6f2774046296f&scene=21#wechat_redirect) 进行安装。  

(可选 1) 如果你用 Python 的目的是数据分析，可以直接安装 Anaconda：[Python 数据分析与挖掘好帮手—Anaconda](http://mp.weixin.qq.com/s?__biz=MzI3MzM0ODU4Mg==&mid=2247486014&idx=1&sn=4519422fbd83b5feffcbb21552226bc3&chksm=eb25e8b7dc5261a1aef2fa400ca7bcaa8c06394ea1f9a5860ab02bcf95d4664f41903b12bbd8&scene=21#wechat_redirect)，它内置了 Python 和 pip.

(可选 2) 此外，推荐大家用 VSCode 编辑器，它有许多的优点：[Python 编程的最好搭档—VSCode 详细指南](http://mp.weixin.qq.com/s?__biz=MzI3MzM0ODU4Mg==&mid=2247485849&idx=1&sn=ec098cf67a55bd1d61d4513397434c94&chksm=eb25eb10dc52620682db716d206c18b00bd53c01743729a9dea381e1791566a04a06f1fabca5&scene=21#wechat_redirect)。

**请选择以下任一种方式输入命令安装依赖**：  
1. Windows 环境 打开 Cmd (开始 - 运行 - CMD)。  
2. MacOS 环境 打开 Terminal (command + 空格输入 Terminal)。  
3. 如果你用的是 VSCode 编辑器 或 Pycharm，可以直接使用界面下方的 Terminal.

```
pip install schedule

```

_**2. 基本使用**_

最基本的使用在文首已经提到过，下面给大家展示更多的调度任务例子：

```
# Python 实用宝典
import schedule
import time

def job():
    print("I'm working...")

# 每十分钟执行任务
schedule.every(10).minutes.do(job)
# 每个小时执行任务
schedule.every().hour.do(job)
# 每天的10:30执行任务
schedule.every().day.at("10:30").do(job)
# 每个月执行任务
schedule.every().monday.do(job)
# 每个星期三的13:15分执行任务
schedule.every().wednesday.at("13:15").do(job)
# 每分钟的第17秒执行任务
schedule.every().minute.at(":17").do(job)

while True:
    schedule.run_pending()
    time.sleep(1)

```

可以看到，从月到秒的配置，上面的例子都覆盖到了。不过**如果你想只运行一次任务**的话，可以这么配：

```
# Python 实用宝典
import schedule
import time

def job_that_executes_once():
    # 此处编写的任务只会执行一次...
    return schedule.CancelJob

schedule.every().day.at('22:30').do(job_that_executes_once)

while True:
    schedule.run_pending()
    time.sleep(1)

```

**参数传递**

如果你有参数需要传递给作业去执行，你只需要这么做：

```
# Python 实用宝典
import schedule

def greet(name):
    print('Hello', name)

# do() 将额外的参数传递给job函数
schedule.every(2).seconds.do(greet, name='Alice')
schedule.every(4).seconds.do(greet, name='Bob')

```

**获取目前所有的作业**

如果你想获取目前所有的作业：

```
# Python 实用宝典
import schedule

def hello():
    print('Hello world')

schedule.every().second.do(hello)

all_jobs = schedule.get_jobs()

```

**取消所有作业**

如果某些机制触发了，你需要立即清除当前程序的所有作业：

```
# Python 实用宝典
import schedule

def greet(name):
    print('Hello {}'.format(name))

schedule.every().second.do(greet)

schedule.clear()

```

**标签功能**

在设置作业的时候，为了后续方便管理作业，你可以给作业打个标签，这样你可以通过标签过滤获取作业或取消作业。

```
# Python 实用宝典
import schedule

def greet(name):
    print('Hello {}'.format(name))

# .tag 打标签
schedule.every().day.do(greet, 'Andrea').tag('daily-tasks', 'friend')
schedule.every().hour.do(greet, 'John').tag('hourly-tasks', 'friend')
schedule.every().hour.do(greet, 'Monica').tag('hourly-tasks', 'customer')
schedule.every().day.do(greet, 'Derek').tag('daily-tasks', 'guest')

# get_jobs(标签)：可以获取所有该标签的任务
friends = schedule.get_jobs('friend')

# 取消所有 daily-tasks 标签的任务
schedule.clear('daily-tasks')

```

**  
设定作业截止时间**

如果你需要让某个作业到某个时间截止，你可以通过这个方法：

```
# Python 实用宝典
import schedule
from datetime import datetime, timedelta, time

def job():
    print('Boo')

# 每个小时运行作业，18:30后停止
schedule.every(1).hours.until("18:30").do(job)

# 每个小时运行作业，2030-01-01 18:33 today
schedule.every(1).hours.until("2030-01-01 18:33").do(job)

# 每个小时运行作业，8个小时后停止
schedule.every(1).hours.until(timedelta(hours=8)).do(job)

# 每个小时运行作业，11:32:42后停止
schedule.every(1).hours.until(time(11, 33, 42)).do(job)

# 每个小时运行作业，2020-5-17 11:36:20后停止
schedule.every(1).hours.until(datetime(2020, 5, 17, 11, 36, 20)).do(job)

```

截止日期之后，该作业将无法运行。

**立即运行所有作业，而不管其安排如何**

如果某个机制触发了，你需要立即运行所有作业，可以调用 **`schedule.run_all()`** :

```
# Python 实用宝典
import schedule

def job_1():
    print('Foo')

def job_2():
    print('Bar')

schedule.every().monday.at("12:40").do(job_1)
schedule.every().tuesday.at("16:40").do(job_2)

schedule.run_all()

# 立即运行所有作业，每次作业间隔10秒
schedule.run_all(delay_seconds=10)

```

_**3. 高级使用**_

**装饰器安排作业**

如果你觉得设定作业这种形式太啰嗦了，也可以使用装饰器模式：

```
# Python 实用宝典
from schedule import every, repeat, run_pending
import time

# 此装饰器效果等同于 schedule.every(10).minutes.do(job)
@repeat(every(10).minutes)
def job():
    print("I am a scheduled job")

while True:
    run_pending()
    time.sleep(1)

```

**并行执行**

默认情况下，Schedule 按顺序执行所有作业。其背后的原因是，很难找到让每个人都高兴的并行执行模型。

不过你可以通过多线程的形式来运行每个作业以解决此限制：

```
# Python 实用宝典
import threading
import time
import schedule

def job1():
    print("I'm running on thread %s" % threading.current_thread())
def job2():
    print("I'm running on thread %s" % threading.current_thread())
def job3():
    print("I'm running on thread %s" % threading.current_thread())

def run_threaded(job_func):
    job_thread = threading.Thread(target=job_func)
    job_thread.start()

schedule.every(10).seconds.do(run_threaded, job1)
schedule.every(10).seconds.do(run_threaded, job2)
schedule.every(10).seconds.do(run_threaded, job3)

while True:
    schedule.run_pending()
    time.sleep(1)

```

**日志记录**

Schedule 模块同时也支持 logging 日志记录，这么使用：

```
# Python 实用宝典
import schedule
import logging

logging.basicConfig()
schedule_logger = logging.getLogger('schedule')
# 日志级别为DEBUG
schedule_logger.setLevel(level=logging.DEBUG)

def job():
    print("Hello, Logs")

schedule.every().second.do(job)

schedule.run_all()

schedule.clear()

```

效果如下：

```
DEBUG:schedule:Running *all* 1 jobs with 0s delay in between
DEBUG:schedule:Running job Job(interval=1, unit=seconds, do=job, args=(), kwargs={})
Hello, Logs
DEBUG:schedule:Deleting *all* jobs

```

**异常处理**

Schedule 不会自动捕捉异常，它遇到异常会直接抛出，这会导致一个严重的问题：**后续所有的作业都会被中断执行**，因此我们需要捕捉到这些异常。

你可以手动捕捉，但是某些你预料不到的情况需要程序进行自动捕获，加一个装饰器就能做到了：

```
# Python 实用宝典
import functools

def catch_exceptions(cancel_on_failure=False):
    def catch_exceptions_decorator(job_func):
        @functools.wraps(job_func)
        def wrapper(*args, **kwargs):
            try:
                return job_func(*args, **kwargs)
            except:
                import traceback
                print(traceback.format_exc())
                if cancel_on_failure:
                    return schedule.CancelJob
        return wrapper
    return catch_exceptions_decorator

@catch_exceptions(cancel_on_failure=True)
def bad_task():
    return 1 / 0

schedule.every(5).minutes.do(bad_task)

```

这样，**`bad_task`** 在执行时遇到的任何错误，都会被 **`catch_exceptions`** 捕获，这点在保证调度任务正常运转的时候非常关键。

我们的文章到此就结束啦，感谢阅读与支持！