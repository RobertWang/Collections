> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Y602E1ra-PXTBlI4xCih9g)

> 作为一名 Python 开发者，psutil 库无疑是你的必备工具之一！

大家好，我是杨叔。每周进步一点点，关注我的微信公众号【程序员杨叔】，获取更多测试开发技术知识！本周分享的内容是：Python 神器：psutil 库使用详解！  

如果本文对你有帮助，麻烦点一点在看 + 点赞 + 分享，你的支持就是作者更新最大的动力~

一、背景

在 Python 的世界里，有一些库因其强大的功能和易用性而备受开发者们的喜爱。今天，我们要介绍的就是其中的一员——psutil 库。psutil(python system and process utilities) 是一个跨平台的第三方库，用于获取系统运行时的进程和系统利用率（包括 CPU、内存、磁盘、网络等）信息。它主要用于系统监控，性能分析，进程管理等场景。

二、安装 & 基本使用

psutil 安装：

```
pip install psutil

```

安装完成后，我们就可以开始使用 psutil 库了。下面，我们将介绍一些常用的功能。

1、获取 CPU 信息

psutil 库可以获取 CPU 的使用情况。例如，我们可以使用 psutil.cpu_percent(interval=1) 来获取 CPU 的使用率。

```
import psutil
cpu_percent = psutil.cpu_percent(interval=1)
print(f'CPU usage: {cpu_percent}%')

```

2、获取内存信息

我们可以使用 psutil.virtual_memory() 来获取系统的内存使用情况。

```
import psutil
mem_info = psutil.virtual_memory()
print(f'Total memory: {mem_info.total / (1024**3):.2f} GB')
print(f'Used memory: {mem_info.used / (1024**3):.2f} GB')
print(f'Memory usage: {mem_info.percent}%')

```

3、获取磁盘信息

psutil 库也可以获取磁盘的使用情况。例如，我们可以使用 psutil.disk_usage(‘/’) 来获取根目录的磁盘使用情况。

```
import psutil
disk_usage = psutil.disk_usage('/')
print(f'Total disk space: {disk_usage.total / (1024**3):.2f} GB')
print(f'Used disk space: {disk_usage.used / (1024**3):.2f} GB')
print(f'Disk usage: {disk_usage.percent}%')

```

4、获取进程信息

psutil 库还可以获取系统中运行的所有进程的信息。例如，我们可以使用 psutil.pids() 来获取所有进程的 PID。

```
import psutil
pids = psutil.pids()
print(f'Total processes: {len(pids)}')

```

三、项目应用实战

假设有这样的一个需求：长时间运行 Pycharm 程序，监控 Pycharm 程序的 CPU / 内存占用，以验证 Pycharm 程序在长时间打开的情况下，程序是否会存在 CPU 占用率升高或内存泄漏的情况。

基于这样的需求，我们可以使用 psutil 库和 pandas 库来完成，脚本如下：

1、获取电脑整体的 CPU、内存占用情况

```
def getMemory():
    data = psutil.virtual_memory()
    memory = str(int(round(data.percent))) + "%"
    print("系统整体memory占用:"+memory)
    return memory
def getCpu():
    cpu_list=psutil.cpu_percent(percpu=True)
    average_cpu = round(sum(cpu_list) / len(cpu_list),2)
    cpu=str(average_cpu) + "%"
    print("系统整体cpu占用:"+cpu)
    return cpu

```

2、获取指定进程的 CPU 和内存占用信息代码

```
def getMemSize(pid):
    process = psutil.Process(pid)
    memInfo = process.memory_info()
    return memInfo.rss / 1024 / 1024
def getCpuPercent(pid):
    p = psutil.Process(pid)
    p_cpu = p.cpu_percent(interval=0.1)
    cpu = round(p_cpu,2)
    return cpu
def getTotalM(processName):
    totalM = 0
    for i in psutil.process_iter():
        if i.name() == processName:
            totalM += getMemSize(i.pid)
    print('进程占用内存：%.2f MB' % totalM)
    finalM=round(totalM,2)
    return finalM
def getTotalCPU(processName):
    totalCPU = 0
    for i in psutil.process_iter():
        if i.name() == processName:
            totalCPU += getCpuPercent(i.pid)
    totalCPU_convert=round(totalCPU,2)
    finalCPU=str(totalCPU_convert)+'%'
    print("进程占用CPU："+finalCPU)
    return totalCPU_convert

```

3、将测试结果数据写入 csv 文件

```
def writeExcel(caseName,cpu,mem,pycharmcpu,pycharmmem):
    timestamp = time.strftime('%Y-%m-%d-%H:%M:%S', time.localtime(time.time()))
    dict = {'caseName': [caseName], 'Sys_CPU': [cpu], 'Sys_Memory': [mem], 'Pycharm_Cpu': [pycharmcpu], 'Pycharm_Mem': [pycharmmem],'OperationTime':[timestamp]}
    dataframe = pd.DataFrame(dict)
    dataframe['OperationTime'] = pd.to_datetime(dataframe['OperationTime'])
    dataframe.to_csv("cpuAndMemtest.csv", date_format='%Y-%m-%d-%H:%M:%S', mode='a',index=False,header=False,encoding='GBK')

```

4、封装方法为函数，以便后续直接调用

```
def getCpuAndMem(caseName,processName1):
    memory = getMemory()
    cpu = getCpu()
    pycharmmem = getTotalM(processName1)
    pycharmcpu = str(getTotalCPU(processName1))+'%'
    time.sleep(1)
    writeExcel(caseName,cpu,memory,pycharmcpu,pycharmmem)
    print("系统整体CPU占用：%s     系统整体内存占用：%s   进程_CPU占用：%s  进程内存占用：%s"%(cpu, memory, pycharmcpu, pycharmmem))
    print("===============================================================")

```

5、运行脚本

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vx46Ba7qgDtNHpHH1AZLItAondWKsiaFjqIiaqW3WPjLMiaEBeKlVCRuDliaAURnpicdMEFbYiaa6JI2elwBqYLyoU4Q/640?wx_fmt=png)

6、生成的 csv 文件内容

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vx46Ba7qgDtNHpHH1AZLItAondWKsiaFjoUsp0mmm6LklkTzhFANGGWiaZThFCjUzLwZG7M4KyPtQia3n0F9WeM3A/640?wx_fmt=png)

7、依赖测试数据生成趋势图，用于测试报告使用

![](https://mmbiz.qpic.cn/sz_mmbiz_png/Vx46Ba7qgDtNHpHH1AZLItAondWKsiaFjj9ibfn1vuVzG4DxO53OLB1Sau4357pEyWeQyRicdQl3IajjVacSRN6Cg/640?wx_fmt=png)

完整的脚本代码我也上传到了我的百度网盘，需要的同学可以添加杨叔微信，加入杨叔测试交流群免费获取~

以上就是 psutil 库的一些基本用法。实际上，psutil 库的功能远不止这些，它还可以获取网络接口信息，系统启动时间，当前用户信息等等。如果你是一名 Python 开发者，那么 psutil 库无疑是你的必备工具。

END

以上就是本次的全部内容，如果对你有帮助，麻烦点一点在看 + 点赞 + 分享，你的支持就是作者更新最大的动力！