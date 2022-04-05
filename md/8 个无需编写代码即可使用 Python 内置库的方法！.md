> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzU0MzU0Nw==&mid=2651403371&idx=4&sn=f89eb9a1dd48dda1c2a9d125ed6e869e&chksm=bd25ff7f8a527669705ec710e404ea6a48219bce30378708082e8d88cce11d720c9298f8084b&mpshare=1&scene=1&srcid=04030XK3UXy4CANMU48vC5ZO&sharer_sharetime=1648982152641&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

（点击上方快速关注并设置为星标，一起学 Python）  

> 作者：Christopher Tao  
> 译者：王坤祥 @InfoQ  
> 原文：Eight “No-Code” Features In Python (https://towardsdatascience.com/eight-no-code-features-in-python-15744e8c01f4)


===================================================================================================================================================================

近几年 Python 语言之所以流行，是因为我们可以使用它编写更少的代码来实现复杂的功能。Python 开发者社区非常欢迎那些封装了复杂实现但是对使用者十分友好的工具包。

然而，Python 的简便性不止如此。你能相信我们可以在不写任何代码的情况下使用 Python 吗？在接下来的文章中，我会介绍 8 个无需编写任何代码即可使用 Python 内置功能的例子。

0. Python CLI “-m” 参数
---------------------

我们首先从 Python CLI（命令行界面）开始谈起。虽然我们不必编写代码来使用稍后介绍的功能，但是为了让 Python 知道我们要执行的内容，我们需要使用 Python 命令行来进行操作。

只要我们的电脑上安装了 Python 环境，我们就可以在 Python 命令行界面输入`python --help`显示所有支持的参数。

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWW8jgybZTibJGUaza8FSoxc7acAKtKfet0zGibhJohnOibZg14ictDEzbTqQ/640?wx_fmt=jpeg)

由于命令输出的内容太长，上图仅显示了部分内容。这里最想强调的是`-m mod`参数，它会将 Python 模块以脚本的形式运行。因此，如果该模块的实现支持命令行操作，我们就可以在命令行直接使用它。接下来就让我们体验一下:)

1. 服务端口测试
---------

有时候，我们想测试 ip 端口的出站网络流量，通常 telnet 命令是一个不错的选择。在 Windows 平台上默认没有安装 telnet 软件，使用前需要手动安装。如果只是进行简单的测试，未来使用场景也不多，安装它可能是一种资源浪费。

但是，如果安装了 Python，那就不必下载安装 telnet，因为 Python 内置了 telnet 对应的模块。我们可以对 Google 搜索网站的 443 端口进行测试。

```
python -m telnetlib -d 142.250.70.174 443<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWW08RaBiaPpBYge5gqKXBdQMaZC0RfKtF5hwJWGzWlNnwNpSr5m0pGVJg/640?wx_fmt=jpeg)

如上图所示，网络流量显示正常，我们甚至收到了来自 Google 空字符的响应。如果我们尝试访问 ip 的随机一个端口，则会抛出错误，如下图所示。

```
python -m telnetlib -d 142.250.70.174 999<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWhhZdJKbR2DVa8eDibg1sxyuBxiaL4icAbsWicH8pTgtgRBkPNDdZLewy6g/640?wx_fmt=jpeg)

2. 本地启动 web 服务
--------------

很多 Python 使用者不知道这一点，当第一次听说后会感到惊讶。是的，我们可以使用 Python 启动 web 服务，而无需编写任何代码，只需按如下方式在命令行执行如下命令。

```
python -m http.server<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWW59vLHf9fOtibzicTPzloyEJBWD0I0IWzia9fDdL82kd86ISvXNnYjR45w/640?wx_fmt=jpeg)

运行后，显示该服务监听了本地的 8000 端口，然后，我们就可以尝试从浏览器进行访问 http://localhost:8000/。

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWW8QzCR68StIMRuibMbS2V2ibgA2d7U32f5k90y9SrHKAy2icAzSrP3lSnw/640?wx_fmt=jpeg)

该 web 服务会以根目录的形式展示在命令启动路径下的本地文件系统，换句话说，我们无法访问它的父级目录。

你可能会问，这个功能的使用场景是什么。举一个例子，如果你想跟你的好伙伴们分享你电脑某个目录下的许多文本 / PDF / 图像文件 / 子目录文件等，那么使用这个方法就可以非常轻松地进行共享了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWVkWiavicZe2zFsqmJP7aQhL1mRCVdW9HZB0og25gcia5YAdibiae8wKGmGg/640?wx_fmt=jpeg)

如果你想知道更多关于这个话题的内容，可以参考 3 Lines of Python Code to Write A Web Server 这篇文章。如果你按照上面的文章实现了一个 “低代码” 的解决方案，那么就可以向它添加更多的自定义功能了。

3. 验证及格式化 JSON 字符串
------------------

如果你有一个非常长且未经格式化的 JSON 字符串，那么阅读起来会非常困难。通常，我会使用一些带有 JSON 插件的文本编辑器，比如 Sublime 或者 VS Code，来格式化 JSON 字符串。但是，如果手头没有这些工具，Python 可以临时一用。比如下面会以这个简短的 JSON 字符串进行展示。

```
echo '{"name": {"first_name":"Chris", "last_name":"Tao"} "age":33}'
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWbPWpYAM1VolwbhYs1rJhslBXEmbDhK7oCc4v62dQSkT3ibVyGJricFuQ/640?wx_fmt=jpeg)

可以看到，当前操作系统的命令行工具只能按照原字符串的原始格式进行展示。但是，如果借助 Python 的 `json.tool`工具，JSON 字符串就会被很好的格式化。

```
echo '{"name": {"first_name":"Chris", "last_name":"Tao"} "age":33}' | python -m json.tool
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWuWJ90v5WZhLUxzTXZcbr8qGrNvW1cWRqXib6ErtgzVo2O8uMGjg7kkA/640?wx_fmt=jpeg)

Oops！JSON 字符串无效，并且 json.tool 帮助我们定位了问题。我们在名称对象后面漏掉了一个逗号。所以添加逗号以使该 JSON 合法有效。

```
echo '{"name": {"first_name":"Chris", "last_name":"Tao"}, "age":33}' | python -m json.tool
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWBH1f6MVzw8vg8VqhpPo6SR4cw3dUdtVsccsAFiceRbJekuVbaIc0Hicw/640?wx_fmt=jpeg)

现在，JSON 字符串具有了完美缩进的格式化输出！更加方便阅读。

4. 创建文本编辑器
----------

你没看错，我们可以使用 Python 来” 创建” 一个文本编辑器。当然，它的功能非常有限，但是如果当前没有更好的选择，使用它会方便很多。另外，功能上肯定无法与 Vim 和 Nanos 相比，但是它完全是基于 UI 编辑器而不是命令行文本形式。这个编辑器由基于 Tkinter 实现的`idlelib` 模块创建，所以它是可以跨平台运行的。

假设我们要编写一个简单的 Python 程序来显示当前的时间，我想快速编写代码而不想下载和安装庞大的代码编辑工具。现在让我们运行下面这个命令。

```
mkdir get_time_apppython -m idlelib get_time_app/print_time.py<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

如果文件目录不存在，`idlelib`将无法创建，因此如果必要，我们需要创建一个。我们运行完这个命令之后，print_time.py 只有执行保存的情况下才会创建到本地。现在应该会弹出编辑器，我们可以在里面写一些代码， 可以看到代码是支持语法高亮的。

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWW4vDGflLibicRHThWnhYeIjEwoGfBEsaDbzR7hAIxHsXmu9hhTkhfP4TA/640?wx_fmt=jpeg)

现在我们使用`ctrl+s`快捷键对编辑好的代码进行保存，并关闭编辑窗口。接下来使用命令行查看一下编辑好的代码文件进行验证，没有任何问题。

```
cat get_time_app/print_time.py<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWM415aYic836pMqibiclp3yl61piajDhTq9OcPgInepXChAm5iclRoPW9QMA/640?wx_fmt=jpeg)

5. 创建可执行应用程序
------------

如果我们想要创建一个简单的应用，比如前面写的获取当前时间的应用程序，我们不必再需要像 PyInstaller 这样的第三方工具包，Python 内置的 Zipapp 就可以做到。假设我们要打包成一个 "Get Time" 的应用，我们可以在命令行运行下面的命令。

```
python -m zipapp get_time_app -m "print_time:main"
```

在该命令中，我们只需要给 `zipapp`设置`get_time_app`名称，指定 Python 程序的入口文件及其程序入口函数即可。以`.pyz`为扩展名的文件就是我们创建的应用程序，至此我们就可以将项目作为单个文件而不是文件夹进行分发。

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWBn41pDn0yMR3PqlWA7iaOlJexrpkE2wZ8ibecKjWT797TREQQhqDlDlw/640?wx_fmt=jpeg)

该程序的启动方式也很简单，直接使用 Python 进行调用即可。

```
python get_time_app.pyz<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWPD8AYiciciaRm6MCVejTApd7HBVibB4g5iawbqPZBnreib742OIoQfpibV8Jw/640?wx_fmt=jpeg)

6. 编码和解码字符串或文件
--------------

通过 Python CLI，我们可以加密字符串或文件。我们以有趣的 ROT13 加密算法为例进行展示。ROT13 是一种偏移 13 位的凯撒密码，它的加密原理如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWW9mbeDSOcMdwicCAZGItpJLV2puE6ZBCBDvcbrmHXPdyPTRzqnicgvoqw/640?wx_fmt=png)

我们可以使用 `encodings.rot_13` 来加密一个字符串，命令如下。

```
echo "I am Chris" | python -m encodings.rot_13
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWLbb1ibWdelp6Z5b0RWDFxibNVjMWqWfYiak3ibe3BQu6N9JLHvegfJUo7Q/640?wx_fmt=jpeg)

切记，不要将其用于任何真正的加密内容。因为英文有 26 个字母，所以再次运行这个算法我们可以很容易地破译这个加密字符串:)

```
echo 'V nz Puevf' | python -m encodings.rot_13
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWP63EYvejBypC152EEPsKvl4IAWPiaPtoWricia0EDE6MdBXr5cnVDNbzQ/640?wx_fmt=jpeg)

现在让我们尝试一个更常见的场景——base64 编码。我们可以对字符串进行 base64 编码，如下所示。

```
echo "I am Chris" | python -m base64
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWmdlTqibYicWwMu0GdpClXGQsk64J6kbrHxqfUceuiawb1EWhYhJEe7FxA/640?wx_fmt=jpeg)

接下来，我们也可以使用`-d`参数对加密字符串进行解码。

```
echo "SSBhbSBDaHJpcwo=" | python -m base64 -d
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWicyXr5m7EL2YfgatgOSzJv0d4K7ALXfj1AaRWl4UEJL9ibxIq6pyNEEA/640?wx_fmt=jpeg)

base64 也经常用在对图像文件的编码和解码上。我们也可以对文件进行如下编码。

```
python -m base64 get_time_app/print_time.py  <br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWyfk5JianYjPf5vmONzpl8bouKwYv3BqbyOGvibCw4a8Q9h2OGJ7Bib1kQ/640?wx_fmt=jpeg)

非常有趣的是，解码后的 Python 脚本可以即时执行，不会报错。。

```
echo "ZnJvbSBkYXRldGltZSBpbXBvcnQgZGF0ZXRpbWUKCgpkZWYgbWFpbigpOgogICAgY3VycmVudF90aW1lID0gZGF0ZXRpbWUubm93KCkKICAgIHByaW50KGYnQ3VycmVudCB0aW1lIGlzIHtjdXJyZW50X3RpbWV9LicpCgoKaWYgX19uYW1lX18gPT0gJ19fbWFpbl9fJzoKICAgIG1haW4oKQo=" | python -m base64 -d | python
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWTjlPh5hp4uricUI5ZKCM7Gv12y0R2HVUEFe6KLqNeibeDLfh20XUdG5g/640?wx_fmt=jpeg)

7. 获取系统元数据
----------

如果我们想获取当前的系统信息，Python 提供了一种非常简便的方法。我们只需要运行下面的命令即可。

```
python -m sysconfig<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWzuH8taJ55vVapxvQY9Y7Nhia4fPic5M9ZVThOMjmKoLhRgicvIZckmLIg/640?wx_fmt=jpeg)

可以看到，这个命令执行后会显示所有的系统配置信息，比如 Python 环境路径和环境变量等。上面的截图仅仅展示了一部分内容，实际显示的内容会非常丰富。如果我们只想展示 Python 环境路径和当前工作路径，我们可以执行下面的命令。

```
python -m site<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWdq2Wvtb5mkK8WBMWSDyX8uEA8kqPs9ibs7DHCdes5sQlPPerEJwibylA/640?wx_fmt=jpeg)

8. 文件压缩
-------

我们可以使用 Python 来压缩文件，而无需下载 tar/zip/gzip 等工具。举个例子，如果我们想压缩我们刚刚在第 4 节中编写的应用程序，我们可以运行以下命令将文件夹压缩到 zip 文件中。在命令中，选项 -c 代表的是 “create” 即创建的含义。

```
python -m zipfile -c get_time_app.zip get_time_app<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWWI0fhrd6WL5htl7ea38ZicTfcehTZtc3vMibEG0A2KNZ0rDRm1ZNUNhBQ/640?wx_fmt=jpeg)

当然，我们也可以对压缩文件进行解压。紧接这上面的操作，我们把文件夹解压出来放到一个新目录中，这样就不会和原来的目录冲突了。在下面的命令中，选项 -e 代表 “extract” 即解压的含义。

```
python -m zipfile -e get_time_app.zip get_time_app_extracted<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

如果不放心，我们可以检验一下。

```
ls get_time_app_extractedcat get_time_app_extracted/get_time_app/print_time.py<br data-darkmode-bgcolor-16491854091481="rgb(49, 54, 63)" data-darkmode-original-bgcolor-16491854091481="#fff|rgb(40, 44, 52)" data-darkmode-color-16491854091481="rgb(171, 178, 191)" data-darkmode-original-color-16491854091481="#fff|rgb(171, 178, 191)">
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/LLRiaS9YfFTMhyj5RuTTnd9EGP55jTibWW8UzIOdm2n4ia40xRD29cJB2BBXN6MFpMCLUrvcylGOID5qQHuZAZMjw/640?wx_fmt=jpeg)

我们刚刚以 zip 文件为例进行了展示，Python 除了支持 zip 格式的解压缩以外，还支持 tar 和 gzip 的解压缩。

总结
--

该篇文章中介绍了一种无需编写任何代码即可使用 Python 内置库的方法。如果在某些场景下能够想到使用这些方法，毫无疑问可以给我们提供很多的便利。希望这篇文章能够给大家带来启发和帮助。