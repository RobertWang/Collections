> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzMDY5MjM3NQ==&mid=2247484257&idx=1&sn=e44b30ae6396752c96f072cc398fde3f&chksm=e8aec019dfd9490f655a2c602a4292ba9e645a5e50f21e26d163ff04c48f9d67cc725664fd1d&scene=132#wechat_redirect)

一、多进程：

**1、multiprocessing**

multiprocessing.Process  无法批量开启子进程

multiprocessing.Pool      可以批量开启子进程

**2、执行方式**  

尽管 Pool 和 Process 都可以并行执行任务，但是它们并行执行任务的方式却不同。

Pool 类使用 FIFO(First Input First Output  先进先出) 调度将任务分配给可用处理器。它的工作方式类似于 map 缩减架构。它将输入映射到不同的处理器，并收集所有处理器的输出。执行代码后，它将以列表或数组的形式返回输出。它等待所有任务完成，然后返回输出。执行中的进程存储在内存中，其他未执行的进程存储在内存之外。

Process 类将所有进程放入内存中，并使用 FIFO 策略安排执行时间。进程暂停后，它将抢占并安排新进程执行。

**3、使用场景**  

Pool 可以批量并行执行，比如【批量报名】就可以使用 Pool，最后把结果汇总  

Process 适用于每个任务执行一次，对输出结果要求不高的场景

**4、代码实例**

Process 代码（每个任务执行一次，不能批量并行执行）

```
# 此处传的studentid是列表，根据学生数量决定开启的进程数量
# args 多个参数传递
for i in range(len(studentid)): 
    p = Process(target=batch_reg, args=(env, area, classid, studentid[i]))
    p.start()
    p.join()
```

```
# 可以根据cpu的数量决定开启的进程数
res = []
    p = Pool(processes=multiprocessing.cpu_count())
    for i in range(4):
        multi = p.apply_async(batch_reg, (env, area, classid, studentid))
        res.append(multi)
    p.close()
    p.join()
```

```
[<multiprocessing.pool.ApplyResult object at 0x7fc34e5be0d0>, <multiprocessing.pool.ApplyResult object at 0x7fc34e5be1d0>, <multiprocessing.pool.ApplyResult object at 0x7fc34e5be290>, <multiprocessing.pool.ApplyResult object at 0x7fc34e5be350>]
```

二、协程  

grequests （不得不说 greuests 真香啊，可惜和 flask 运行不兼容一直报错）  

> 运行结果和多进程的 apply_async 对比后没什么差别，也是批量并行执行，获取结果的方式都是一样

gevent   

```
def func1(x):
    time.sleep(1)
    print('-----' + str(x))
    return x*x

if __name__ == '__main__':
    begin = time.time()
    p = Pool(10)
    result = p.imap(func1, [i for i in range(10)])
    p.close()
    p.join()
    during = time.time() - begin
    print(during)
```

> 执行耗时对比如我们所见，Pool 仅在内存中分配正在执行的进程，而 Process 在内存中分配所有任务，因此，当任务数较小时，我们可以使用 Process 类；当任务数较大时，我们可以使用 Pool。在大型任务中，如果我们使用 Process，可能会发生内存问题，从而引起系统干扰。Pool，由于创建它会产生开销，因此，对于较小的任务数，使用 Pool 会影响性能。

IO 操作  

> IO 操作 Pool 以 FIFO 方式在可用内核之间分配进程。在每个内核上，分配的进程按顺序执行。因此，如果有很长的 IO 操作，它将等待 IO 操作完成，并且不会安排其他进程。这导致执行时间增加。Process 类则挂起执行 IO 操作的进程并安排另一个进程。因此，在长时间的 IO 操作的情况下，建议使用进程类。