> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/bjbz_cxy/article/details/109165532?utm_term=linux%E7%BB%88%E7%AB%AF%E6%98%BE%E7%A4%BA%E5%9B%BE%E7%89%87&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-0-109165532&spm=3001.4430)

这是我的一个开源项目，目前是使用 python 编写，系统上需要有 python3 的支持

首先从 git 上下载代码：

```
git clone git@github.com:beiszhihao/spirit.git
```

下载后代码如下：

![](https://img-blog.csdnimg.cn/20201019175145424.png)

然后执行 config，注意这里要使用 sudo

```
sudo ./config
```

执行完成后在 shell 上输入 spirit 图像路径

原图：

![](https://img-blog.csdnimg.cn/20201019175535739.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JqYnpfY3h5,size_16,color_FFFFFF,t_70)

转换后：

![](https://img-blog.csdnimg.cn/20201019175314658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JqYnpfY3h5,size_16,color_FFFFFF,t_70)

目前代码较少，功能不是很完善，后续会逐渐完成这个功能，此功能主要是为了不想在 linux 上打开图像而实现的功能

核心思路就是用 python img 模块读取图像矩阵，在转换成对应的字符即可。

渐变使用的是 lolcat 插件。