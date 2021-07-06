> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwNjU4NTcwNw==&mid=2652924543&idx=1&sn=57d6d91fc7e689f0ce95f14876816e7a&chksm=80dfe67bb7a86f6d027b11763eecba8e59a86bd114a3df8b877b37e7334593f52fc36b720070&scene=21#wechat_redirect)

> 很好的教程

NumPy 是 Python 中用于数据分析、机器学习、科学计算的重要软件包。它极大地简化了向量和矩阵的操作及处理。python 的不少数据处理软件包依赖于 NumPy 作为其基础架构的核心部分（例如 scikit-learn、SciPy、pandas 和 tensorflow）。除了数据切片和数据切块的功能之外，掌握 numpy 也使得开发者在使用各数据处理库调试和处理复杂用例时更具优势。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRia5G6jaFd6PcdUYK9M72ibRPNoNKCJic7vGldqspVlbbagdRC1YIeib8Kg/640?wx_fmt=jpeg)

在本文中，将介绍 NumPy 的主要用法，以及它如何呈现不同类型的数据（表格，图像，文本等），这些经 Numpy 处理后的数据将成为机器学习模型的输入。

**NumPy 中的数组操作**

**创建数组**  

我们可以通过将 python 列表传入 np.array() 来创建一个 NumPy 数组（也就是强大的 ndarray）。在下面的例子里，创建出的数组如右边所示，通常情况下，我们希望 NumPy 为我们初始化数组的值，为此 NumPy 提供了诸如 ones()，zeros() 和 random.random() 之类的方法。我们只需传入元素个数即可：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRqDZyRtDPYicU3gdmXq9Ju9wA17SrXOYWzc6zE9dRcFol3bicAT49pntA/640?wx_fmt=jpeg) 

一旦我们创建了数组，我们就可以用其做点有趣的应用了，文摘菌将在下文展开说明。

**数组的算术运算**

让我们创建两个 NumPy 数组，分别称作 data 和 ones：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fR7XNPdVlTIicz3Qo2zGSCicIhYlrg01SKVkwBgZ3A6ug5eXYtBJNrpO2g/640?wx_fmt=png)

若要计算两个数组的加法，只需简单地敲入 data + ones，就可以实现对应位置上的数据相加的操作（即每行数据进行相加），这种操作比循环读取数组的方法代码实现更加简洁。

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRVN6qnQejRZktlibIdopjB1kMKHrM90khxZPU693E1k9mpKRKpyiamdvg/640?wx_fmt=png)

当然，在此基础上举一反三，也可以实现减法、乘法和除法等操作：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRcnhWOHibcNz1GhZzkeJic7PBQO9psibgkzmsZ4ZR7tetzlF4LLE3Q9N8A/640?wx_fmt=jpeg)

许多情况下，我们希望进行数组和单个数值的操作（也称作向量和标量之间的操作）。比如：如果数组表示的是以英里为单位的距离，我们的目标是将其转换为公里数。可以简单的写作 data * 1.6：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRciaHiafOA755jjIYN4OrjToy2xaLmCUMCX6jZZRiaMp41geFvktbaOlhw/640?wx_fmt=png)

NumPy 通过数组广播（broadcasting）知道这种操作需要和数组的每个元素相乘。

**数组的切片操作**

我们可以像 python 列表操作那样对 NumPy 数组进行索引和切片，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRRzx6n3H1eHVibsrROicpznI4DK1FELlsdNgfQia8TwIrPXia3U5tOyV9ew/640?wx_fmt=png)

**聚合函数**

NumPy 为我们带来的便利还有聚合函数，聚合函数可以将数据进行压缩，统计数组中的一些特征值：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRnF0UyxGA1ny6je7jkzsrZ82hI5sicxic0HsEW1IPwxfuTtpTmfBFmJicg/640?wx_fmt=jpeg)

除了 min，max 和 sum 等函数，还有 mean（均值），prod（数据乘法）计算所有元素的乘积，std（标准差），等等。上面的所有例子都在一个维度上处理向量。除此之外，NumPy 之美的一个关键之处是它能够将之前所看到的所有函数应用到任意维度上。

**NumPy 中的矩阵操作**

**创建矩阵**

我们可以通过将二维列表传给 Numpy 来创建矩阵。

np.array([[1,2],[3,4]])

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fR9TX8b0YMPTkq0o4mDXkhU4XZ4ibK06DFjWY5kGWXficCiaYYx5mOUGibhQ/640?wx_fmt=png)

除此外，也可以使用上文提到的 ones()、zeros() 和 random.random() 来创建矩阵，只需传入一个元组来描述矩阵的维度：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRhBR3cicCafJq1CAoocUngvZt5zQQX2Q70Ucibc4XnOlY8nbhCE20LWbQ/640?wx_fmt=jpeg)

**矩阵的算术运算**

对于大小相同的两个矩阵，我们可以使用算术运算符（+-*/）将其相加或者相乘。NumPy 对这类运算采用对应位置（position-wise）操作处理：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRuPFC6GiclYJXe8DSdaOJprPrVeZpTqzANsE08Ghat791Ot4U3Oq9skA/640?wx_fmt=png)

对于不同大小的矩阵，只有两个矩阵的维度同为 1 时（例如矩阵只有一列或一行），我们才能进行这些算术运算，在这种情况下，NumPy 使用广播规则（broadcast）进行操作处理：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRumKIBbB11RO6fxUrycI30M8GaGBa1PNN2Oxbg5znuqGLmLRtLnS8ZQ/640?wx_fmt=jpeg)

与算术运算有很大区别是使用点积的矩阵乘法。NumPy 提供了 dot() 方法，可用于矩阵之间进行点积运算：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRiaNtSQPJs8nufvkUgDKEomq9o7pN6nOiax025SZY3ScC7Zia1UcW5QD3Q/640?wx_fmt=png)

上图的底部添加了矩阵尺寸，以强调运算的两个矩阵在列和行必须相等。可以将此操作图解为如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRg2jcY2ic3h2Fu875M9YIsshRTSWPJgAyTyc9PVrxwF8lFVDH1FZaNlg/640?wx_fmt=jpeg)

**矩阵的切片和聚合**

索引和切片功能在操作矩阵时变得更加有用。可以在不同维度上使用索引操作来对数据进行切片。

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fR35YMhOMn4GjnJrSAfsYqsTU96rcolyJHGC6IuBiaj7Dwp6FOUtv8rrg/640?wx_fmt=png)

我们可以像聚合向量一样聚合矩阵：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fR2XqU8mLXu6Q5c4rrDiazxGFnHnJTEMVgF3icAI0SMBVNQDhSyf9tlfYA/640?wx_fmt=jpeg)

不仅可以聚合矩阵中的所有值，还可以使用 axis 参数指定行和列的聚合：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRIOUZzBUtElM4PjIM0VLA7w4NK8qdZmTbxZ0b9H1eW0pyRNdiaWbDP3w/640?wx_fmt=jpeg)

**矩阵的转置和重构**

处理矩阵时经常需要对矩阵进行转置操作，常见的情况如计算两个矩阵的点积。NumPy 数组的属性 T 可用于获取矩阵的转置。

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fR75QebejeBhurtDmn7plyd7eOy4iasBYqKBvDqib1lh3T39VjWHLGvIDQ/640?wx_fmt=png)

在较为复杂的用例中，你可能会发现自己需要改变某个矩阵的维度。这在机器学习应用中很常见，例如模型的输入矩阵形状与数据集不同，可以使用 NumPy 的 reshape() 方法。只需将矩阵所需的新维度传入即可。也可以传入 - 1，NumPy 可以根据你的矩阵推断出正确的维度：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRibGqOYNxrZlKeLLnlGs0p6oSicdVUSW8EvicOT9a09VaESpEuQOIBAH8w/640?wx_fmt=png)

上文中的所有功能都适用于多维数据，其中心数据结构称为 ndarray（N 维数组）。

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fR8F9NGnRDEFic8oOGKL6icYGyK9vpN7K7CJMFviaKia073bZbNobpkqwA6A/640?wx_fmt=png)

很多时候，改变维度只需在 NumPy 函数的参数中添加一个逗号，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRnOibYMwGdHxHX3ibBNEWVYnPl2HOyzV33vSF9uhLOXiaibzeqYobxk5ZGQ/640?wx_fmt=jpeg)

**NumPy 中的公式应用示例**

NumPy 的关键用例是实现适用于矩阵和向量的数学公式。这也 Python 中常用 NumPy 的原因。例如，均方误差是监督机器学习模型处理回归问题的核心：  

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRkSuyWtgtaznia1M9mkf252bSqibrOiaA1FibX4NM2GXf2cr9Sl0wGc5zXA/640?wx_fmt=png)

在 NumPy 中可以很容易地实现均方误差：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRxHeGvtn7stvCrpsbONickf3iawhiahzbbRic41BlXY8icSpq4ibaviaqKtSzQ/640?wx_fmt=jpeg)

这样做的好处是，numpy 无需考虑 predictions 与 labels 具体包含的值。文摘菌将通过一个示例来逐步执行上面代码行中的四个操作：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRYU5L4IxCo2QqEp0Tt00KQ54rtNC5jn3No1icUY6Fcv75YYhdUjkGlMA/640?wx_fmt=png)

预测（predictions）和标签（labels）向量都包含三个值。这意味着 n 的值为 3。在我们执行减法后，我们最终得到如下值：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRicSyqRs0djiaLm7rxmAtvrc9gk5A4Z4EVqoZuWN4Hr4sTTyiayA0SCibrw/640?wx_fmt=png)

然后我们可以计算向量中各值的平方：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRe9sGe75ZUV6m6ayBJPaXTFNia3Z3YCXDzxqzt6V41BdHgFvHqbSrQlA/640?wx_fmt=png)

现在我们对这些值求和：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fR3WfZ83Sqb3rVj2Ld6iaDwecko07PibxBqo4XvmrWsHSSAMrtdkUqLNMA/640?wx_fmt=png)

最终得到该预测的误差值和模型质量分数。

**用 NumPy 表示日常数据**

日常接触到的数据类型，如电子表格，图像，音频...... 等，如何表示呢？Numpy 可以解决这个问题。

**表和电子表格**

电子表格或数据表都是二维矩阵。电子表格中的每个工作表都可以是自己的变量。python 中类似的结构是 pandas 数据帧（dataframe），它实际上使用 NumPy 来构建的。

![](https://mmbiz.qpic.cn/mmbiz_jpg/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRIsibbc9Zr6X6dQCJJhEZLcFvFE6adMYVznolf6eo7DoSDmianufZ2ibwQ/640?wx_fmt=jpeg)

**音频和时间序列**

音频文件是一维样本数组。每个样本都是代表一小段音频信号的数字。CD 质量的音频每秒可能有 44,100 个采样样本，每个样本是一个 - 65535 到 65536 之间的整数。这意味着如果你有一个 10 秒的 CD 质量的 WAVE 文件，你可以将它加载到长度为 10 * 44,100 = 441,000 个样本的 NumPy 数组中。想要提取音频的第一秒？只需将文件加载到我们称之为 audio 的 NumPy 数组中，然后截取 audio[:44100]。

以下是一段音频文件：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRvMAs0ia3n9TwhvUo2C3JtZLsPg6FQkAsqV5pdfpSaKlprnhJ8I7qXEg/640?wx_fmt=jpeg)

时间序列数据也是如此（例如，股票价格随时间变化的序列）。

**图像**

图像是大小为（高度 × 宽度）的像素矩阵。如果图像是黑白图像（也称为灰度图像），则每个像素可以由单个数字表示（通常在 0（黑色）和 255（白色）之间）。如果对图像做处理，裁剪图像的左上角 10 x 10 大小的一块像素区域，用 NumPy 中的 image[:10,:10] 就可以实现。

这是一个图像文件的片段：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRmastCBykBNypNibL6fb8DCmEykncWCMrysRVLkuCtzdODXmKiaWSdK1g/640?wx_fmt=jpeg)

如果图像是彩色的，则每个像素由三个数字表示 ：红色，绿色和蓝色。在这种情况下，我们需要第三维（因为每个单元格只能包含一个数字）。因此彩色图像由尺寸为 (高 x 宽 x 3）的 ndarray 表示。

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRvV5WML33viaxtSE6vWiano8e0NYic9ASVpsX8QeBZnzeQRAEm9icRaiaszA/640?wx_fmt=jpeg)

**语言**

如果我们处理文本，情况就会有所不同。用数字表示文本需要两个步骤，构建词汇表（模型知道的所有唯一单词的清单）和嵌入（embedding）。让我们看看用数字表示这个（翻译的）古语引用的步骤：“Have the bards who preceded me left any theme unsung?”

模型需要先训练大量文本才能用数字表示这位战场诗人的诗句。我们可以让模型处理一个小数据集，并使用这个数据集来构建一个词汇表（71,290 个单词）：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRpW6hAwQVCqoMacVt3tSDIIQIiblOfJnOFPBE0voZ7ibsibRKXN8yJbWMA/640?wx_fmt=png) 

然后可以将句子划分成一系列 “词”token（基于通用规则的单词或单词部分）：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRmZfL8ghuXRjG9ibxv3ZdMvuCajBI0wCkCezzbDzYD4IrxC57zH9PqmA/640?wx_fmt=jpeg)

然后我们用词汇表中的 id 替换每个单词：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fR5AJSSOXIibj5vG32FupugaRJtcEoNp62qoXG9eym0tcVNMtia2KCNgBQ/640?wx_fmt=jpeg)

这些 ID 仍然不能为模型提供有价值的信息。因此，在将一系列单词送入模型之前，需要使用嵌入（embedding）来替换 token / 单词（在本例子中使用 50 维度的 word2vec 嵌入)：

![](https://mmbiz.qpic.cn/mmbiz_png/wc7YNPm3YxWEXib7ibcRpDVoraXzZh13fRJzBQ8UO1OgbWQk9uOZpZ1FkGibxe4pE0mLPuXmxj24BqvH2TAss3fKg/640?wx_fmt=jpeg)

你可以看到此 NumPy 数组的维度为 [embedding_dimension x sequence_length]。

在实践中，这些数值不一定是这样的，但我以这种方式呈现它是为了视觉上的一致。出于性能原因，深度学习模型倾向于保留批数据大小的第一维（因为如果并行训练多个示例，则可以更快地训练模型）。很明显，这里非常适合使用 reshape()。例如，像 BERT 这样的模型会期望其输入矩阵的形状为：[batch_size，sequence_length，embedding_size]。

这是一个数字合集，模型可以处理并执行各种有用的操作。我留空了许多行，可以用其他示例填充以供模型训练（或预测）。

事实证明，在我们的例子中，那位诗人的话语比其他诗人的诗句更加名垂千古。尽管生而为奴，诗人安塔拉（Antarah）的英勇和语言能力使他获得了自由和神话般的地位，他的诗是伊斯兰教以前的阿拉伯半岛《悬诗》的七首诗之一。

来源丨大数据文摘  

原文链接丨 https://jalammar.github.io/visual-numpy/