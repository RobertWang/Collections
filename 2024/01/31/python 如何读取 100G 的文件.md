> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxNTEyMzQ3OQ==&mid=2247484125&idx=1&sn=091fa925f1ba424d72211fb9dc4ffc6c&chksm=9b8997a5acfe1eb3caf1d0d9beef00a30bcdafc05e21e000b368546e973f95827d12cfaede59&mpshare=1&scene=1&srcid=01291oMP3ICMbXBzRk80mAUr&sharer_shareinfo=4b262b19667228356a7bddcba934992c&sharer_shareinfo_first=4b262b19667228356a7bddcba934992c#rd)

Python 提供了多种方法来读取文件。在这里，我将介绍一些读取大文件的方法，可以按项目需求使用

一种常见的方法是使用 Python 的标准文件读取流程，即使用 open() 函数打开文件，然后使用 readline() 或 readlines() 方法逐行读取文件内容。  
下面是一个使用 readline() 方法的示例代码：

```
def read_from_file(filename, block_size=1024*8):
    with open(filename, 'r') as fp:
        while True:
            chunk = fp.read(block_size)
            if not chunk:
                break
            # 处理文件内容块

```

如果您想一次性读取所有行，可以使用 readlines() 方法。下面是一个使用 readlines() 方法的示例代码

```
def read_from_file(filename):
    with open(filename, 'r') as fp:
        lines = fp.readlines()
        for line in lines:
            # 处理文件内容

```

这些方法可能会导致内存不足的问题，因为它们需要将整个文件读入内存中。如果您的文件大小超过 100G，这种方法可能不适用

如果您需要处理大文件，可以使用 file.read() 方法。与前一种方法不同，file.read() 方法每次返回一个固定大小的文件内容块，而不是一行一行地读取文件。这种方法可以避免内存不足的问题，但是需要更多的代码来处理文件内容块。

下面是一个使用 file.read() 方法的示例代码：

```
def read_from_file(filename, block_size=1024*8):
    with open(filename, 'r') as fp:
        while True:
            chunk = fp.read(block_size)
            if not chunk:
                break
            # 处理文件内容块

```

如果您想进一步优化代码，可以使用生成器函数来解耦数据生成和数据消费的逻辑。下面是一个使用生成器函数的示例代码：

```
def chunked_file_reader(fp, block_size=1024*8):
    while True:
        chunk = fp.read(block_size)
        if not chunk:
            break
        yield chunk
def read_from_file_v2(filename, block_size=1024*8):
    with open(filename, 'r') as fp:
        for chunk in chunked_file_reader(fp, block_size):
            # 处理文件内容块

```