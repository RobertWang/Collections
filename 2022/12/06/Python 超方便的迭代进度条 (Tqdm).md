> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2MDY2NzQwMQ==&mid=2247497338&idx=1&sn=c45459f9c1fc4c01b8d0c7a3f8e293c5&chksm=ea64971cdd131e0a39637fb62a6e90fd06b317c3cdb1159204c24d7f6bd9a90bb60b6b1ecbd6&mpshare=1&scene=1&srcid=09081ppjhtdFKRa6BWOcbLCp&sharer_sharetime=1662619214890&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

![](https://mmbiz.qpic.cn/mmbiz_jpg/P33BNFicia1AlI5mHYiayuSdkeGSbEP6MJ8oThthQngPxnxlFIVpbAADSOhASmMSWQKN9YC4oKLZhhOEsasTGoKDw/640?wx_fmt=jpeg&random=0.567562745219758)

  
Tqdm 是一个智能进度表。它能够显示所有可迭代对象当前执行的进度。

你只需要用 tqdm 对可迭代对象进行封装后再遍历即可实现进度条功能，比如说：

显示效果如下：

76%|████████████████████████ | 7568/10000 [00:33<00:10, 229.00it/s]

**请选择以下任一种方式输入命令安装依赖**：  
1. Windows 环境 打开 Cmd (开始 - 运行 - CMD)。  
2. MacOS 环境 打开 Terminal (command + 空格输入 Terminal)。  
3. 如果你用的是 VSCode 编辑器 或 Pycharm，可以直接使用界面下方的 Terminal.

```
pip install tqdm

```

_**2. 基本使用**_

tqdm 非常灵活，可以使用多种方式调用。下面给出了两种主要的形式。

#### **2.1 迭代的形式**

使用 **tqdm()** 封装可迭代的对象：

```
from tqdm import tqdm
from time import sleep

text = ""
for char in tqdm(["a", "b", "c", "d"]):
    sleep(0.25)
    text = text + char

```

**trange(i)** 是特殊的关键字，是封装了 range 的 tqdm 对象：

```
from tqdm import trange

for i in trange(100):
    sleep(0.01)

```

通过 set_description 方法，你能控制进度条显示当前步骤的名称：

Processing d: 100%|█████████████████████████████████████████████| 4/4 [00:01<00:00, 3.99it/s]

#### **2.2 手动的形式**

除了迭代的形式，你可以手动控制进度，加一个 tqdm 上下文即可：

```
with tqdm(total=100) as pbar:
    for i in range(10):
        sleep(0.1)
        pbar.update(10)

```

  
上述例子中，pbar 是 tpdm 的 “进度”，每一次对 pbar 进行 update 10 都相当于进度加 10。  

Total 的值即是总进度，这里 total 的值是 100，那么 pbar 加到 100 的时候进度也就结束了。

你也可以选择不使用上下文的形式调用，但要记得结束后对对象进行关闭操作：

```
pbar = tqdm(total=100)
for i in range(10):
    sleep(0.1)
    pbar.update(10)
pbar.close()

```

_**3. 模块结合**_

Tqdm 最妙的地方在于能在命令行中结合使用：

```
$ find . -name '*.py' -type f -exec cat \{} \; |
    tqdm --unit loc --unit_scale --total 857366 >> /dev/null
100%|█████████████████████████████████| 857K/857K [00:04<00:00, 246Kloc/s]

```

只需在管道之间插入 tqdm（或 python -m tqdm），即可将进度条显示到终端上。

备份大目录：

```
$ tar -xcf - docs/ | tqdm --bytes --total `du -sb docs/ | cut -f1` \
  > backup.tgz
 44%|██████████████▊ | 153M/352M [00:14<00:18, 11.0MB/s]

```

这可以进一步美化：

```
$ BYTES="$(du -sb docs/ | cut -f1)"
$ tar -cf - docs/ \
  | tqdm --bytes --total "$BYTES" --desc Processing | gzip \
  | tqdm --bytes --total "$BYTES" --desc Compressed --position 1 \
  > ~/backup.tgz
Processing: 100%|██████████████████████| 352M/352M [00:14<00:00, 30.2MB/s]
Compressed: 42%|█████████▎ | 148M/352M [00:14<00:19, 10.9MB/s]

```

我们的文章到此就结束啦，如果你喜欢今天的 Python 实战教程，就给本文右下角点个赞吧。