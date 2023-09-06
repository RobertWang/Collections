> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/84vz2GbcoaG0g32pUlndhQ)

Python 中的并行批处理 

使用 joblib 分批处理并使用 tqdm 显示进度 Joblib 是一个很好的并行化工具，但有时最好是批量处理工作负载，而不是默认的迭代方式。在本文中 将展示：

所有代码都可以在 Github 上找到 源码链接

> https://github.com/dennisbakhuis/tqdm_batch

代码也可以作为 Pypi 包使用：pip install tqdm_batch

#### 一 、使用 joblib 进行并行化的直接方法

在处理大型数据集时，我们都遇到过这样的情况：我们不能使用所有核心并加快处理速度吗？当然，答案是肯定的，但多处理一点也不简单。

我们都遇到过这样的情况：我们不能使用所有核心并加快处理速度吗？

Python 具有内置的多处理和多线程模块 multiprocessing and threading。这就是开始使用多处理和多线程所需的全部内容，但它需要一些样板代码。这可能令人望而生畏，尤其是从数据科学的角度来看，我们有这个串行处理循环并且只想并行运行它。

为了简化我们的多核处理负担，我们可以使用很棒的库，例如 joblib 或 pathos。在 joblib 的网站上，作者说循环是 “令人尴尬的并行”，他们确实是正确的，因为它非常容易上手。让我们创建一个例子：

```
def batch_process_function(row, order, payload):
    """
    Simulate process function
    
    Row and payload are ignored.
    
    Approximate pi
    """
    k, pi = 1, 0
    for i in range(10**6):
        if i % 2 == 0: # even
            pi += 4 / k
        else:  # odd 
            pi -= 4 / k 
        k += 2
    return pi


# Settings
order=6
N = 1_000
items = range(N)

# Serial run
result = [batch_process_function(row, order, None) for row in items]

# Parallel using joblib and a progress bar using tqdm
result = Parallel(n_jobs=8)(
    delayed(batch_process_function)
    (row, order, None) 
    for row in tqdm(items)
)


```

这只是一个虚拟函数，它近似于 Pi，但阶数为 6，计算时间约为 200 毫秒。还有两个变量。第一个变量是一行，这将是要处理的单个项目。这可以是方法需要打开和加载的文件名，也可以是我们需要规范化的字符串。在这个虚拟函数中，我们只是忽略它。有效负载将用于显示限制，现在也可以忽略。让我们为这个函数获取一些运行时数据：

![](https://mmbiz.qpic.cn/mmbiz_png/MdNXuAibcZSEJx1icA2Wmf5JpLgz7qpLmnxgF1ULpCvibBYvPzwaa5PGuuGCcjwf1WKggoEIMTAGMxlef14mczgBw/640?wx_fmt=png)图 1：运行时串行运行和使用 joblib 时。

串行运行，即只处理每一行的 for 循环，只需不到 100 秒。在这个例子中我们可以清楚地看到 joblib 的美妙之处。我们不需要更改任何东西来并行处理它，结果在使用 8 核时几乎快了五倍。这通常效果很好，但我们必须注意其局限性。

### 二、限制以及为什么它并不总是有效

多处理是一种并行运行任务并使用自己的 Python 解释器创建隔离进程的方法。多处理任务不存在 GIL 问题。但是，因为我们正在运行一个独立的进程，任务无法访问与主程序相同的内存，所有需要的数据都需要发送（序列化）到这个进程。序列化本身是使用 Python 内置的 Pickle（或 Dill) 这使得所有所需对象的冻结内存副本。有些对象无法序列化，你会得到一个错误。例如，进程 1 对 UI 的引用不能共享给进程 2（这也没有意义）。这种序列化，即将对象的状态冻结成字节以发送给进程，是开销（额外的内存和 IO）的原因。例如，当每个进程需要一个字典来进行某些翻译时，多处理库将为每个进程制作该字典的副本。如果这个字典很大，这种开销比串行处理花费更多时间的情况并不少见。

这种开销比串行处理花费更多时间的情况并不少见。

另一种并行运行任务的方法是多线程。最大的好处是它与主程序共享内存并且不需要序列化，因此开销更少。但代价是你必须使用 GIL。如果任务使用大量不需要 GIL 的已编译库（例如 Numpy） ，这不一定是个问题。

![](https://mmbiz.qpic.cn/mmbiz_png/MdNXuAibcZSEJx1icA2Wmf5JpLgz7qpLmnSRhCbalAFKtibeyZ8Qv8zKtpUaJSpicwhqGFnpuEdibIjQVqPZOHhrEiag/640?wx_fmt=png)

```
# Random payload to simulate a model
matrix = np.random.normal(size=(500, 500, 100))

# Use default joblib
result = Parallel(n_jobs=8)(
    delayed(batch_process_function)
    (row, order, matrix) 
    for row in tqdm(items)
)


```

### 三、使用批处理并行化

我们已经看到，对于多处理，由于每个任务的大数据序列化需求，开销可能是巨大的。解决方案很简单：减少序列化的数量。我们将创建一个额外的包装函数，而不是为每个项目序列化，该包装函数适用于流程内的批处理。这导致每个进程只序列化一次数据。

```
n_workers = 8

# Create a batch function
def proc_batch(batch, order, matrix):
    return [
        batch_process_function(row, order, matrix)
        for row in batch
    ]

# Divide data in batches
batch_size = ceil(len(items) / n_workers)
batches = [
    items[ix:ix+batch_size]
    for ix in range(0, len(items), batch_size)
]

# divide the work
result = Parallel(n_jobs=n_workers)(
    delayed(proc_batch)
    (batch, order, matrix) 
    for batch in tqdm(batches)
)


```

![](https://mmbiz.qpic.cn/mmbiz_png/MdNXuAibcZSEJx1icA2Wmf5JpLgz7qpLmn2syOEoYzEnbGzUbfR4CSMd7GMUPWRMd4CicUNS2HrA6nst2Nb8uzDCA/640?wx_fmt=png)图 4：减少间接费用可以恢复我们的常规储蓄

这为我们节省了与没有大数据要求的串行方法类似的节省。魔术 %%time 函数还向我们展示了一些有趣的东西：我们在此过程中只花费了 1.33 秒的时间。其他 23s 用于其他无法访问的进程 %%time。

您可能已经注意到的另一件事是进度条不再起作用。Tqdm 与批次耦合并发送所有批次，必须等待流程完成。我们在项目级别上看不到到目前为止已经处理了多少。由于我们无法访问内存，因此显示单个（或多个）进度条并非易事。

### 四、让我们修复进度条

在多处理时创建进度条不是一件容易的事。在使用串行方式时，我们可以将 tqdm 保持在主进度（集体进度）中。它会看到传递给流程的每个项目。通过序列化连接进度条是不可能的，因为它需要进程写入主程序内存的一部分。这在定义上是不允许的，这也是当您尝试将 progress_bar 对象添加到函数参数时出现序列化错误的原因。

为了解决这个问题，我们可以使用队列，一种使进程可以相互通信的列表。为了使它更简单，让我们创建一个完整的包装器，以便我们可以提供一个项目列表和一个函数，在我们的包装器中将工作分配给所有进程。

此代码也可在 Github 上获得，并作为一个包在 PyPi 上获得。

```
def progress_bar(
    totals: Union[int, List[int]],
    queue : Queue,
) -> None:
    if isinstance(totals, list):
        splitted = True
        pbars = [
            tqdm(
                desc=f'Worker {pid + 1}',
                total=total,
                position=pid,
            )
            for pid, total in enumerate(totals)
        ]
    else:
        splitted = False
        pbars = [
            tqdm(total=totals)
        ]

    while True:
        try:
            message = queue.get()
            if message.startswith('update'):
                if splitted:
                    pid = int(message[6:])
                    pbars[pid].update(1)
                else:
                    pbars[0].update(1)
            elif message == 'done':
                break
        except:
            pass
    for pbar in pbars:
        pbar.close()

        
def task_wrapper(pid, function, batch, queue, *args, **kwargs):
    result = []
    for example in batch:
        result.append(function(example, *args, **kwargs))
        queue.put(f'update{pid}')
    return result

        
def batch_process(
    items: list,
    function: Callable,
    n_workers: int=8,
    sep_progress: bool=False,
    *args,
    **kwargs,
    ) -> List[Dict[str, Union[str, List[str]]]]:
    # Divide data in batches
    batch_size = ceil(len(items) / n_workers)
    batches = [
        items[ix:ix+batch_size]
        for ix in range(0, len(items), batch_size)
    ]

    # Check single or multiple progress bars
    if sep_progress:
        totals = [len(batch) for batch in batches]
    else:
        totals = len(items)

    # Start progress bar in separate thread
    manager = Manager()
    queue = manager.Queue()
    progproc = Thread(target=progress_bar, args=(totals, queue))
    progproc.start()

    # Parallel process the batches
    result = Parallel(n_jobs=n_workers)(
        delayed(task_wrapper)
        (pid, function, batch, queue, *args, **kwargs)
        for pid, batch in enumerate(batches)
    )

    # Stop the progress bar thread
    queue.put('done')
    progproc.join()

    # Flatten result
    flattened = [item for sublist in result for item in sublist]

    return flattened


```

![](https://mmbiz.qpic.cn/mmbiz_png/MdNXuAibcZSEJx1icA2Wmf5JpLgz7qpLmnnGPupyeHQMzoDYpptbWaCrtwj4vkDXf5o6sIwOmPIBIXInpUNjU37g/640?wx_fmt=png)图 5：在多个 CPU 上分配工作并显示进度条

使用此方法，我们创建了一个直接替换，您只需为其编写一个处理单行的方法。batch_process 将该函数包装到批处理程序中，该批处理程序还会更新进度条。函数的所有参数都可以使用参数添加，因为这些参数被传递给处理函数，包括所需的大数据和模型。结果与原始运行相似，但开销较小。tqdm_batch 已添加到 pypi，因此您可以 pip install tqdm_batch 自己尝试一下。

工人的数量也不是您所期望的，因为它们不是线性扩展的。在某个时候，增加更多的工人并没有带来任何好处。我还注意到我的超线程内核也没有提供任何性能提升。以下是所需计算时间与工作人员数量的概述：

![](https://mmbiz.qpic.cn/mmbiz_png/MdNXuAibcZSEJx1icA2Wmf5JpLgz7qpLmnf4STSZibF9mm8LcYXH5UbwmuspMStrCTnichibV5ofjAZDBdxM5Q2d4uw/640?wx_fmt=png)图 6：超过 6 个作品并没有增加任何好处

原文链接：

> https://towardsdatascience.com/parallel-batch-processing-in-python-8dcce607d226