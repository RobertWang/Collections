> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/tM8wxd1_-9RLGN0XFNyKWw)

后台进程是在后台运行的程序或任务，它们不会阻塞主程序的执行，并可以在后台处理一些耗时或周期性的任务。在本文中，我们将探讨如何在 Python 中启动后台进程，并介绍一些内置模块和第三方库来实现这一目标。

### 同步 vs. 异步

在开始之前，我们需要了解同步和异步编程的区别。在同步编程中，程序按顺序执行，每个操作完成后才进行下一个操作。而在异步编程中，程序可以在等待某个操作完成的同时继续执行其他操作。

后台进程通常是异步的，因为它们在后台执行，不会阻塞主程序的运行。异步编程的基本概念包括回调、协程、异步 / 等待等，Python 提供了一些内置模块和第三方库来支持异步编程。

### 使用内置模块启动后台进程

Python 提供了一些内置模块，可以用于启动后台进程。以下是其中一些常用的模块：

#### subprocess 模块

subprocess 模块允许你在 Python 中启动外部进程。你可以使用`subprocess.run()`函数来执行外部命令，并将其设置为在后台运行。例如，下面的代码启动一个后台的 ping 命令：

```
import subprocess

subprocess.run(["ping", "-c", "10", "example.com"], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)


```

#### threading 模块

threading 模块允许你在 Python 中启动线程，从而在后台执行任务。你可以创建一个 Thread 对象，并将要执行的函数传递给它。例如，下面的代码启动一个后台线程来执行一个耗时的任务：

```
import threading

def long_running_task():
    # 执行耗时的任务

thread = threading.Thread(target=long_running_task)
thread.start()


```

### 使用第三方库启动后台进程

除了内置模块，Python 还有许多强大的第三方库可用于启动后台进程。

#### multiprocessing 模块

multiprocessing 模块允许你在 Python 中启动并发进程。它提供了类似于 threading 模块的接口，但它使用多个进程而不是线程。以下是一个使用 multiprocessing 模块的示例：

```
import multiprocessing

def long_running_task():
    # 执行耗时的任务

if __name__ == "__main__":
    process = multiprocessing.Process(target=long_running_task)
    process.start()


```

#### celery 库

Celery 是一个功能强大的分布式任务队列库，用于在后台执行任务。它允许你将任务分发给多个工作者（workers），并提供任务调度、结果跟踪和错误处理等功能。以下是一个简单的 Celery 示例：

```
from celery import Celery

app = Celery('tasks', broker='amqp://guest@localhost//')

@app.task
def long_running_task():
    # 执行耗时的任务

if __name__ == '__main__':
    long_running_task.delay()


```

### 进程间通信和数据共享

在后台进程中，有时需要进行进程间通信和数据共享。Python 提供了不同的机制来实现这一目标。

#### 队列

队列是一种常见的进程间通信机制，用于在进程之间传递数据。Python 的 multiprocessing 模块提供了`Queue`类来实现进程间的安全数据传输。以下是一个使用队列进行进程间通信的示例：

```
from multiprocessing import Process, Queue

def producer(queue):
    # 将数据放入队列
    queue.put('data')

def consumer(queue):
    # 从队列中获取数据
    data = queue.get()

if __name__ == '__main__':
    queue = Queue()
    p1 = Process(target=producer, args=(queue,))
    p2 = Process(target=consumer, args=(queue,))
    p1.start()
    p2.start()


```

#### 共享内存

共享内存是一种用于在进程之间共享数据的机制。Python 的 multiprocessing 模块提供了`Value`和`Array`等类来实现共享内存。以下是一个使用共享内存的示例：

```
from multiprocessing import Process, Value

def increment_counter(counter):
    counter.value += 1

if __name__ == '__main__':
    counter = Value('i', 0)
    processes = [Process(target=increment_counter, args=(counter,)) for _ in range(10)]
    for process in processes:
        process.start()
    for process in processes:
        process.join()
    print(f"Counter: {counter.value}")


```

案例研究
----

### 案例 1：定时任务

定时任务是一种常见的需求，特别是在自动化任务和计划任务方面。在 Python 中，有一些定时任务库可以帮助我们启动后台进程来执行这些任务。其中，`schedule`和`APScheduler`是两个流行的库。

`schedule`库提供了简单而直观的 API，可以帮助我们定义和调度定时任务。以下是一个使用`schedule`库的示例，执行每小时备份数据库的任务：

```
import schedule
import time

def backup_database():
    # 执行备份数据库的任务
    pass

# 每小时执行一次备份任务
schedule.every().hour.do(backup_database)

while True:
    schedule.run_pending()
    time.sleep(1)


```

`APScheduler`库提供了更多高级功能和灵活性，如支持多种调度方式（固定时间间隔、定时表达式等）和多种触发器（时间触发器、日期触发器等）。以下是一个使用`APScheduler`库的示例，执行每天发送报告的任务：

```
from apscheduler.schedulers.blocking import BlockingScheduler

def send_report():
    # 执行发送报告的任务
    pass

scheduler = BlockingScheduler()
scheduler.add_job(send_report, 'cron', day_of_week='mon-fri', hour=17)
scheduler.start()


```

### 案例 2：并发处理

当我们需要处理大量数据或执行耗时的计算时，后台进程可以帮助我们提高处理效率。在 Python 中，`multiprocessing`库可以用于启动多个进程并并发地处理任务。以下是一个使用`multiprocessing`库的示例，计算一个数列的平方和：

```
import multiprocessing

def square(number):
    return number ** 2

if __name__ == '__main__':
    numbers = [1, 2, 3, 4, 5]
    with multiprocessing.Pool() as pool:
        results = pool.map(square, numbers)
        total = sum(results)
        print(f"The sum of squares is: {total}")


```

在上面的例子中，我们使用`multiprocessing.Pool`创建了一个进程池，并使用`map`方法并发地计算数列中每个数的平方，然后使用`sum`函数求和。

### 案例 3：长时间运行的任务

有些任务需要较长的时间才能完成，如爬取大量网页数据或训练复杂的机器学习模型。将这些任务放在后台进程中运行可以确保主程序的响应性。以下是一个使用`multiprocessing`库的示例，模拟一个长时间运行的任务：

```
import time
import multiprocessing

def long_running_task():
    # 模拟耗时任务
    time.sleep(60)
    print("Long running task completed.")

if __name__ == '__main__':
    process = multiprocessing.Process(target=long_running_task)
    process.start()
    print("Main program continues to execute.")


```

在这个例子中，我们使用`multiprocessing.Process`创建了一个新的进程，并在其中执行一个模拟的长时间运行的任务。主程序在启动后台进程后继续执行。

结论
--

在本文中，我们讨论了如何在 Python 中启动后台进程。我们介绍了使用内置模块（如`subprocess`和`threading`等）以及一些常用的第三方库（如`multiprocessing`和`celery`）来启动后台进程。我们还介绍了进程间通信和数据共享的机制，如队列和共享内存。

在案例研究中，我们探讨了几个实际应用场景，展示了如何使用后台进程来处理定时任务、并发处理和长时间运行的任务。这些案例研究帮助我们理解在不同情境下如何应用后台进程来提高程序的效率和可靠性。