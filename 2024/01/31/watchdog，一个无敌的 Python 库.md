> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA5MTkxNTMzNg==&mid=2650306460&idx=1&sn=ef10d725b5dd83f0c0e9ea8afea7dfbf&chksm=8879c2bbbf0e4badeddc278c77aead3bfcf9a68add8662633d40a576c97f0ad8b90f47006da5&mpshare=1&scene=1&srcid=0130xKGmOHUFTdn57CdNJz4b&sharer_shareinfo=1ae1d7ce4e138efcb81a5e10d114efa8&sharer_shareinfo_first=1ae1d7ce4e138efcb81a5e10d114efa8#rd)

更多 Python 学习内容：ipengtao.com

大家好，今天为大家分享一个无敌的 Python 库 - watchdog。

Github 地址：https://github.com/gorakhargosh/watchdog

* * *

在软件开发和系统管理领域，经常需要监控文件和目录的变化，以便在文件被创建、修改或删除时触发相应的操作。Python Watchdog 是一个强大的 Python 库，它提供了简单而灵活的方式来监控文件系统的变化。本文将详细介绍 Python Watchdog 的用法和功能，包括安装、基本用法、事件处理以及实际应用场景，并提供丰富的示例代码。

什么是 Python Watchdog？
--------------------

Python Watchdog 是一个用于监控文件系统事件的 Python 库。它可以检测文件和目录的变化，如创建、修改、删除、移动等，并触发相应的事件处理。Python Watchdog 非常适用于开发需要实时监控文件系统变化的应用，如自动化构建、日志分析、文件同步等。

安装 Python Watchdog
------------------

要使用 Python Watchdog，首先需要安装它。

可以使用 pip 来安装：

```
pip install watchdog


```

安装完成后，就可以开始使用 Python Watchdog 来监控文件系统了。

基本用法
----

### 监控单个文件

以下是一个简单的示例，演示如何使用 Python Watchdog 监控单个文件的变化：

```
import time
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

# 创建一个自定义事件处理器
class MyHandler(FileSystemEventHandler):
    def on_modified(self, event):
        if not event.is_directory:
            print(f"File {event.src_path} has been modified")

# 创建一个观察者并启动
observer = Observer()
event_handler = MyHandler()
observer.schedule(event_handler, path="path/to/your/file", recursive=False)
observer.start()

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    observer.stop()

observer.join()


```

上述示例中，创建了一个自定义的事件处理器`MyHandler`，并重写了`on_modified`方法，该方法在文件被修改时触发。然后，创建了一个观察者`Observer`，将事件处理器与文件路径关联，并启动观察者。最后，使用`try`和`except`来捕获 Ctrl+C 中断信号，以便在程序退出时停止观察者。

### 监控目录及其子目录

Python Watchdog 还支持监控整个目录及其子目录的变化。

以下是一个示例：

```
import time
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

# 创建一个自定义事件处理器
class MyHandler(FileSystemEventHandler):
    def on_modified(self, event):
        if not event.is_directory:
            print(f"File {event.src_path} has been modified")

# 创建一个观察者并启动
observer = Observer()
event_handler = MyHandler()
observer.schedule(event_handler, path="path/to/your/directory", recursive=True)
observer.start()

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    observer.stop()

observer.join()


```

在这个示例中，将`recursive`参数设置为`True`，以便监控指定目录及其所有子目录的变化。

事件处理
----

Python Watchdog 提供了多种事件类型，可以根据需要选择并处理。以下是一些常用的事件类型：

*   `on_created`: 当文件或目录被创建时触发。
    
*   `on_deleted`: 当文件或目录被删除时触发。
    
*   `on_modified`: 当文件被修改时触发。
    
*   `on_moved`: 当文件或目录被移动时触发。
    

可以根据需要重写这些事件处理方法，并在其中添加自定义的处理逻辑。例如，可以在文件被创建时执行某些操作，或在目录被删除时触发通知。

实际应用场景
------

当应用 Python Watchdog 时，可以根据不同场景编写事件处理逻辑。以下是一些实际应用场景示例，每个场景都包含相应的示例代码。

### 1. 自动化构建

场景：监控项目目录中的代码变化，以便自动触发构建和测试操作。

```
import time
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

# 自定义事件处理器
class BuildHandler(FileSystemEventHandler):
    def on_modified(self, event):
        if not event.is_directory:
            print(f"File {event.src_path} has been modified. Triggering build...")

        # 在此处添加构建和测试的操作

# 创建观察者并启动
observer = Observer()
event_handler = BuildHandler()
observer.schedule(event_handler, path="path/to/your/project", recursive=True)
observer.start()

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    observer.stop()

observer.join()


```

在上述示例中，当项目目录中的文件被修改时，事件处理器触发构建和测试操作。

### 2. 文件同步

场景：监控源目录的变化，以便将新增、修改或删除的文件同步到目标目录。

```
import time
import shutil
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

source_dir = "path/to/source"
target_dir = "path/to/target"

# 自定义事件处理器
class SyncHandler(FileSystemEventHandler):
    def on_created(self, event):
        if not event.is_directory:
            src_file = event.src_path
            dest_file = src_file.replace(source_dir, target_dir)
            shutil.copy2(src_file, dest_file)
            print(f"File {src_file} has been created. Syncing to {dest_file}")

    def on_modified(self, event):
        if not event.is_directory:
            src_file = event.src_path
            dest_file = src_file.replace(source_dir, target_dir)
            shutil.copy2(src_file, dest_file)
            print(f"File {src_file} has been modified. Syncing to {dest_file}")

    def on_deleted(self, event):
        if not event.is_directory:
            src_file = event.src_path
            dest_file = src_file.replace(source_dir, target_dir)
            try:
                os.remove(dest_file)
                print(f"File {src_file} has been deleted. Removing from {dest_file}")
            except FileNotFoundError:
                pass

# 创建观察者并启动
observer = Observer()
event_handler = SyncHandler()
observer.schedule(event_handler, path=source_dir, recursive=True)
observer.start()

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    observer.stop()

observer.join()


```

在上述示例中，当源目录中的文件被创建、修改或删除时，事件处理器将同步相应的操作到目标目录。

### 3. 日志分析

场景：监控日志文件的变化，以便实时分析和处理新的日志条目。

```
import time
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

log_file = "path/to/your/logfile.log"

# 自定义事件处理器
class LogHandler(FileSystemEventHandler):
    def on_modified(self, event):
        if not event.is_directory and event.src_path == log_file:
            with open(log_file, "r") as file:
                new_entries = file.readlines()
                for entry in new_entries:
                    # 在这里添加日志分析和处理的操作
                    print(f"New log entry: {entry}")

# 创建观察者并启动
observer = Observer()
event_handler = LogHandler()
observer.schedule(event_handler, path="path/to/your", recursive=False)
observer.start()

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    observer.stop()

observer.join()


```

在上述示例中，当日志文件被修改时，事件处理器会读取新的日志条目并进行分析和处理。

总结
--

Python Watchdog 是一个强大而灵活的文件系统事件监控工具，它可以用于多种应用场景，包括自动化构建、文件同步、日志分析等。通过本文的介绍和示例代码，应该已经了解了如何安装、基本用法和事件处理，以及如何在实际项目中应用 Python Watchdog 来实现文件和目录的实时监控。

如果你觉得文章还不错，请大家 点赞、分享、留言 下，因为这将是我持续输出更多优质文章的最强动力！