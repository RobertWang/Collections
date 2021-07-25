> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU5Nzg5ODQ3NQ==&mid=2247511126&idx=2&sn=684a62cf38ddb5c34d415555474cb299&chksm=fe4e89d2c93900c4c052eaf1212e3d492c7f9d8b1cb99cce7869d055016c96820c61cc2c6bd2&scene=21#wechat_redirect)

大家好，我是小五🐶

好多小伙伴用的 python 编辑器还是`jupyter notebook`，有可能会遇到一个问题。

就是 jupyter notebook 默认存储路径在 C 盘，而我们的 python 脚本在其他文件位置，想运行就非常麻烦。

所以就希望能在指定文件夹快速打开`jupyter notebook`。目前常见的方法，就是在 CMD 命令窗口执行以下语句

```
jupyter notebook 指定文件夹路径
```

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLh8q1D5jVVKy1ib15CDIlS7MXcAZ1kSuiaS8CNX5rjSlnkicveGRe4A9Txf4ZdRZmicYrCTWyw8iclSxlg/640?wx_fmt=png)

执行图

那有没有更简单快捷的方法呢？

有的

举个例子，目前我的`D:\python_code`目录如下图所示，现在我想在该路径下快捷打开 jupyter notebook。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLh8q1D5jVVKy1ib15CDIlS7MksiaaALMhFicQKB9cTtfkmPnKIoib903jesxpuSZI4A4wmecSO9NOXoyQ/640?wx_fmt=png)

只需在上方文件路径位置，直接输入`jupyter notebook`后回车即可

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLh8q1D5jVVKy1ib15CDIlS7MVKST3u4DzYgQnIXbYpLq58vU7WEgUIe3t7Q51UHpTLIWYYhOlKIvkQ/640?wx_fmt=png)

会发现`jupyter notebook`已经自动打开，并且工作路径正是我们所需的`D:\python_code`。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLh8q1D5jVVKy1ib15CDIlS7MJLmLicdZ4JHhgjV7tCm6wrJa0icia596Egtk3xtWuNicCYPQEjMNbB1QpA/640?wx_fmt=png)

### 前提条件

这种方法只需一个前提条件，即在 anaconda 安装时勾选了将 anaconda 添加到 path 环境中。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLh8q1D5jVVKy1ib15CDIlS7Mp2scjesIoVbO6DXzo7Z1MqmyAWnnmkicVO7U5bPIQyNxFu6xupS3jRA/640?wx_fmt=png)

当然如果当时没有勾选，也可以自行百度搜索如何**添加环境变量**。

### 将偷懒进行到底

上面已经将步骤精简为只需输入两个单词就好了。

可是我们要输入这么长的单词，一不小心万一输错了呢。于是乎小五准备进一步偷懒，可以将`jupyter notebook`这个单词设置为常用短语。

以搜狗输入法为例，打开【工具箱】-【属性设置】-【高级】-【自定义短语设置】，按照下图所示设置短语，至于缩写字符，大家可以根据自己喜好来填写。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLh8q1D5jVVKy1ib15CDIlS7MztB5HUprmoHoqyAoD1RAPCrgke3qSmWpYd1wNibc1m546ybu24KMANA/640?wx_fmt=png)

这样设置后，我只需要敲完`jup`三个字母，输入法就会显示出`jupyter notebook`这个备选单词了。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLh8q1D5jVVKy1ib15CDIlS7MqoKXbu9jHSZEB7fkh7vxGkjpVavPhnsia4biaPGVDpno1Cx6gRwaicr5A/640?wx_fmt=png)

选定该短语，再回车，就轻松打开了`jupyter notebook`，是不是更简单啦。

![](https://mmbiz.qpic.cn/mmbiz_png/tXYict40xfLh8q1D5jVVKy1ib15CDIlS7MJLmLicdZ4JHhgjV7tCm6wrJa0icia596Egtk3xtWuNicCYPQEjMNbB1QpA/640?wx_fmt=png)

以上就是小五今天分享给大家的小技巧了，求点赞支持一波。