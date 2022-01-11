> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.11meigui.com](https://www.11meigui.com/2020/shell-copy-paste.html)

> 如果您要在终端和任何其他应用程序之间来回切换，那么从命令行访问系统剪贴板内容将非常有用。

嘻嘻发布于 2020-12-09   最后更新于 2020 年 12 月 8 日  

1,364

浏览

如果您要在终端和任何其他应用程序之间来回切换，那么从命令行访问系统剪贴板内容将非常有用。如果您使用的是 Mac，则有内置命令 pbcopy 和 pbpaste。

```
ls | pbcopy
```

或者

```
echo "Copy this into Mac's clipboard" | pbcopy
```

如果随后还需要使用终端将其粘贴到文件中，请使用 pbpaste

```
pbpaste > ~/some-file.txt
```

但是在 Ubuntu 中，这些功能不可用。您需要安装一个名为 xclip 的小型实用程序。继续安装。

#### 安装

```
sudo apt-get install xclip
```

#### 使用

现在，您可以使用 xclip 将任何文本（或输出一个命令）复制到剪贴板中。要将 fruits.txt 的内容复制到剪贴板，

```
cat fruits.txt | xclip
```

如果要查看剪贴板的内容，可以使用

```
xclip -o
```

此复制和粘贴将仅在终端中可用，如果您切换到另一个应用程序并尝试粘贴到该应用程序中，它将无法工作。

如果要粘贴到另一个应用程序中，则需要像这样复制

```
cat fruits.txt | xclip -selection clipboard
```

现在，您可以切换到任何其他应用程序，也可以粘贴（`CTRL + V`）内容。

#### 关于我

![](https://www.11meigui.com/wp-content/uploads/2020/02/elephant-85x85.jpeg)

##### 嘻嘻

嘻嘻 IT: 笔者是一个工作七八年的程序猿老鸟，从事涉及的技术栈主要包括 PHP、Linux、Devops 等，喜欢研究新技术，尝试新技术，提升技术自动化和开发效率，致力于 write less，do more! 技术每年都会层出不穷，领域划分的越来越细，不可能学习所有的东西，保持对技术的好奇心，理解技术中核心思想，做一个有深度，有思想的开发！