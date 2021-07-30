> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=Mzg4NTUzNzE5OQ==&mid=2247493773&idx=1&sn=3f5ea21aecfe0ca9e2c5b6d3fafb4a12&chksm=cfa5c84df8d2415b11a70f3a5e7d00b0fb8275f59aeefbd20a3249e7f4f4b51826a0c7b382bc&mpshare=1&scene=1&srcid=07303vHFMWykZt9QKQjKrEhw&sharer_sharetime=1627635323661&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

2. 向量降维和矩阵降维的含义

4. 基向量个数的确定

7. PCA 算法总结  

如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfs0PnCSnsxiaI4lZXib1oqddiaZL280eoC2WNibWGZqVQ5jH6wZzAD3iaLMg/640?wx_fmt=png)

向量 a 在向量 b 的投影为：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfa5aXMQ5KDuHnZzhbpNQ6fDia3hg50WtBRYhHqsgWFbBicyJFUPmhcibKg/640?wx_fmt=png)

其中，θ是向量间的夹角 。

向量 a 在向量 b 的投影表示向量 a 在向量 b 方向的信息，若θ=90° 时，向量 a 与向量 b 正交，向量 a 无向量 b 信息，即向量间无冗余信息 。因此，向量最简单的表示方法是用基向量表示，如下图：  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfRWopYsn8CrG6OOtGEEjI9cLTyZmkfSdxaZGMIbmbhdPL3hmx02EIDQ/640?wx_fmt=png)

向量表示方法：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfoRkVzeZ2WE8SfI1SPWFLicFP3r0byqCKp4rk8g8U5GAHNVGKd6WzZ5A/640?wx_fmt=png)

其中，c1 是![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfDJkcTqyy9Jl0I1703mK1VEEazMWaGgSHphGFlHWxoKwNLMdS94LomQ/640?wx_fmt=png)在 e1 方向的投影，c2 是![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfDJkcTqyy9Jl0I1703mK1VEEazMWaGgSHphGFlHWxoKwNLMdS94LomQ/640?wx_fmt=png)在 e2 方向的投影，e1 和 e2 是基向量

我们用向量的表示方法扩展到矩阵，若矩阵![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfibXx59NmVMObDxC6CPlI2w8Mu1LSp9M9EIYLxIUQnZsNLyO45hq75fA/640?wx_fmt=png)的秩 r(A)=n，![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfTQ7npsSyJC1ftMGrxURAB8MaYRHHiaN2napCILfcGB6a1HysHemWeew/640?wx_fmt=png)

，其中 ai（i=1,2,...,n）为 n 个维度的列向量，那么矩阵 A 的列向量表示为：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfvpXnjbZL61sRQPy9P3GaFMW0DBrudqzospUHlB19FtZN3gO8ruLc8g/640?wx_fmt=png)

其中，e1，e2，...，en 为矩阵 A 的特征向量 。

若矩阵 A 是对称矩阵，那么特征向量为正交向量，我们对上式结合成矩阵的形式：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfWDEkor9svM28iciaZj0dRoNoj5D1wdGOdQc3zNYCadXWrOLcbpl6FXcQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf0dRtJ9DQ0gicvXB9YUB8qICibQuwdvF7Tz2gT6WC5mj3Sbx4UNIT5mNA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYffFNJRMXQBm6wCcHNZzfMo4upxeF1Vg7XichmbQjDVORZPGicrzHfmc8g/640?wx_fmt=png)

由上式可知，对称矩阵 A 在各特征向量的投影等于矩阵列向量展开后的系数，特征向量可理解为基向量。

向量降维可以通过投影的方式实现，N 维向量映射为 M 维向量转换为 N 维向量在 M 个基向量的投影，如 N 维向量![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfyYWsfT0U7Jeicx44f2naia4qiccsSCK08MjtDazfJYdOX0rnjdtxJHGiaQ/640?wx_fmt=png)，M 个基向量分别为![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYft30tJJTPD1Lzia9bG8eibQU0Xrt5xI7Foua4d7307m6b3Qlwg3RKLQzA/640?wx_fmt=png)，![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf1eOujliaAoicdJTicqwEFbFG5nGA1V8VNfAAchGWKeDQ4nqibp2bbh5ofw/640?wx_fmt=png)在基向量的投影：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfLsZBkPn1p4ZPVLAY27aX3pFrcxseKQTah7NibQXXyTl9CTpz2muoS3A/640?wx_fmt=png)

通过上式完成了降维，降维后的坐标为：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfetuvic9MAXSo1z04iavOmfOWS2ZcjX2DMgZsTgUIPZHIpK5icyZY7Tsvg/640?wx_fmt=png)

矩阵是由多个列向量组成的，因此矩阵降维思想与向量降维思想一样，只要求得矩阵在各基向量的投影即可，基向量可以理解为新的坐标系，投影就是降维后的坐标，那么问题来了，如何选择基向量？

已知样本集的分布，如下图：  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfcKxSahia68nY5ib539cRouD33cyZ488rICoCOkKxXh3DXKJSvic9fOVWQ/640?wx_fmt=png)

样本集共有两个特征 x1 和 x2，现在对该样本数据从二维降到一维，图中列了两个基向量 u1 和 u2，样本集在两个向量的投影表示了不同的降维方法，哪种方法好，需要有评判标准：（1）降维前后样本点的总距离足够近，即最小投影距离；（2）降维后的样本点（投影）尽可能的散开，即最大投影方差 。因此，根据上面两个评判标准可知选择基向量 u1 较好。

我们知道了基向量的选择标准，下面介绍基于这两个评判标准来推导基向量：

（1）基于最小投影距离

假设有 n 个 n 维数据，记为 X。现在对该数据从 n 维降到 m 维，关键是找到 m 个基向量，假设基向量为 {w1,w2,...,wm}，记为矩阵 W，矩阵 W 的大小是 n×m。

原始数据在基向量的投影：![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfKIVYOGqrH9xTFVibM5icribDIAsV03RriaK8fRrmfbKgxv5n3GVlEdWJsQ/640?wx_fmt=png)

投影坐标计算公式：  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfJBo7giaJqzlltDvyB7lJrF4BeZ4w5GfzovhZBL2Ndibh1QibGicichz5USQ/640?wx_fmt=png)

根据投影坐标和基向量，得到该样本的映射点：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfE5BzbQLpkicXbYdSTFO4oeHj0Z5b6UglAbclQSHJeqylvmkxaczJezQ/640?wx_fmt=png)

最小化样本和映射点的总距离：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfUrbicWV2L35lmvcgTadKabqlXjp03lmFdiaXcMBeDpoWiciciaOiak2VtNVQ/640?wx_fmt=png)

推导上式，得到最小值对应的基向量矩阵 W，推导过程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf9y0GmlLPqUh3A7Dic63rsvrHV4QTa9rVoB37WxxBoI0E2HqXYqC2DFQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf09glo2bW0v5eRicibSk4cKLSWE5A97kcypqJ8TJ0KaBWOjqOQ83ojeGQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfU3qn4EX06A9klj8GR0Lbow5lo9hYrbeXj6gqAnribZpsv94MvMWorWQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfPpN5CsmIcOrpSyKYRkNGicewwqcTnRicBkKVHZPEztBBfd49Cicibpos8w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf5JqKAYkicAeurjibXlLEP11KUkIHl3gibPDWhCkLQMb7pwzictWF7JgtjQ/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfEKIKG7pDq5Nh2kjTcibFDTTIfjBUt9mjeKmSlfzh5BVRm2o8gN480hw/640?wx_fmt=png)

所以我们选择![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf7CMI0clwgRdo1icYrDib5qCbG6yc1Xj0w1d3ib1Qo0Oph0wlQ6D6icCicdA/640?wx_fmt=png)的特征向量作为投影的基向量 。  

(2) 基于最大投影方差

我们希望降维后的样本点尽可能分散，方差可以表示这种分散程度。  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfic6pj2gMvhBI2HuzedRJcOicOGcNyNWhVQjPZWdte1ylUN7R3iaKbypww/640?wx_fmt=png)

如上图所示，![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfhiaRHnZeznvIH03c77nDj2K1b5Jzsmk7vp5c16A6yiae0Iia9veaeIFeA/640?wx_fmt=png)表示原始数据，![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfvFqhTxeepYueD81B14icbpQGILJA2smEZoJpeoftt2EgyRxnGQicawEg/640?wx_fmt=png)表示投影数据，![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfX1ZpfqELxtb5QV5LP2TibGuXQQg56eib7essPeR99jm76S0xrOQ4jMYQ/640?wx_fmt=png)表示投影数据的平均值。所以最大化投影方差表示为：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfibIcymFsEbDAchDEqPfQDYlZYayvUlzmw6lNDxBRJDE2ibwGEo1JnFeg/640?wx_fmt=png)

下面推导上式，得到相应的基向量矩阵 W，推导过程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfKdxH59MOGQdePV9LxJVRrFRqviaNhweaDZCZ4dqS1hqan0Gq8sTRpFA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfJmHZtUQnoTWfU2FnTXbtTVcrnqsksQr1Q1dR7XcOiaLOOjfH7BnicqYA/640?wx_fmt=png)

我们发现 (4) 式与上一节的 (13) 式是相同的。  

因此，基向量矩阵 W 满足下式：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfPHjKSjc2Xd8NTnuhgWvuecMkOpRg4zx8uZiaXvOKMMptd3r0FPqpxmg/640?wx_fmt=png)

假设向量 wi，λi 分别为的特征向量和特征值，表达式如下：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfg73Ru8ArIccoufoett7hu8aZrD3Zu56AkvTiaiahV3gZRGsiaAuBFHJlw/640?wx_fmt=png)

对应的图：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf9c3hhiaicdRJWCz8Ov2ichRjHFAC65joAtuSHdwnlwRAJtkVYozmEpyTw/640?wx_fmt=png)

由上图可知，![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf8bDkiaTSrxvVR99TFibDLSJj1nhowib90Ppnib4N3E0ERGNaPGcyMZh66g/640?wx_fmt=png)没有改变特征向量 wi 的方向，只在 wi 的方向上伸缩或压缩了λi 倍。特征值代表了![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf8bDkiaTSrxvVR99TFibDLSJj1nhowib90Ppnib4N3E0ERGNaPGcyMZh66g/640?wx_fmt=png)在该特征向量的信息分量。特征值越大，包含矩阵![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf8bDkiaTSrxvVR99TFibDLSJj1nhowib90Ppnib4N3E0ERGNaPGcyMZh66g/640?wx_fmt=png)的信息分量亦越大。因此，我们可以用λi 去选择基向量个数。我们设定一个阈值 threshold，该阈值表示降维后的数据保留原始数据的信息量，假设降维后的特征个数为 m，降维前的特征个数为 n，m 应满足下面条件：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfN2M17QF5omZrujFepQX2YohGqzneyJDzricDWuJbSJJ0oOvdrjyUKFQ/640?wx_fmt=png)

因此，通过上式可以求得基向量的个数 m，即取前 m 个最大特征值对应的基向量 。

投影的基向量：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfr6v99klYLiafQkw25x8KVKlKZz7xVXUaHn7CtVDFuQL2q3Pic9sjsxWg/640?wx_fmt=png)

投影的数据集：  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYflKORsZ1Xs2yFm65e9eeYvibMFm4UkntJSQwCXG0qyUScJGo4UcLGJyw/640?wx_fmt=png)

我们在计算协方差矩阵![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfsw6cmK8ibkcsku4PFB9qlsEWj2mAhpZRpthvTjDuLUoILX2bRibpS59w/640?wx_fmt=png)的特征向量前，需要对样本数据进行中心化，中心化的算法如下：  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfVm8wsxNFmETvOIJHGScicNmTZZJt72mJMmYv981zz0z7N3snib861dxw/640?wx_fmt=png)

中心化数据各特征的平均值为 0，计算过程如下：

对上式求平均：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf238DicXuK7JibCUwhMabrczBnfs1eMLDgx20zRBvLtLeQgQpjbZwZicTw/640?wx_fmt=png)

中心化的目的是简化算法，我们重新回顾下协方差矩阵，以说明中心化的作用 。

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfl4HtWZxRUsN61Fx1opicWVYEm9JjviaA5UP2GHezNia5QWuicWr1iaTlemw/640?wx_fmt=png)，X 表示共有 n 个样本数。

每个样本包含 n 个特征，即：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfkRRrWHjZRuwIY8u9wia07xKsqQuvH9TvZEuvhKDrYK4q1aica5FeHA9Q/640?wx_fmt=png)

展开![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf5WEm9Z7bPXtexib54yJHiaufymgicaY4yVf3LlEWojVhmxxKyuyiafmJxw/640?wx_fmt=png):

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYftOZ8fDmzrLiaDjhmZcsEShFAesFL68eMvTV4M4jjBa3LsZiaYMxodODQ/640?wx_fmt=png)

为了阅读方便，我们只考虑两个特征的协方差矩阵：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf3iawuzZ4Htv3slMxqRuyKTZjT3LytaGY8UjaAcIGthgtEicee52bSl4g/640?wx_fmt=png)

由 (3) 式推导 (2) 式得：  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfwCtXj9wQZSRH369npO8sj8fSbZdpOVq7Yc2jiclSicrI7zDR8FBeSO5w/640?wx_fmt=png)

所以![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf5kc1eCdB1OG5Kbs68AkJuh0ROGtsibdVAUloArGCd1yRsaBwunJcTSg/640?wx_fmt=png)是样本数据的协方差矩阵，但是，切记必须事先对数据进行中心化处理 。

**6. PCA 算法流程**

  

  

  

1）样本数据中心化。

2）计算样本的协方差矩阵![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfVj7qtJqY9w3iazmPOYYNyNdDrwmHt4J3tSeciaPNtSGqicNNOIJwDZVjw/640?wx_fmt=png)。

3）求协方差矩阵![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfVj7qtJqY9w3iazmPOYYNyNdDrwmHt4J3tSeciaPNtSGqicNNOIJwDZVjw/640?wx_fmt=png)的特征值和特征向量，并对该向量进行标准化（基向量）。

3）根据设定的阈值，求满足以下条件的降维数 m。  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfN2M17QF5omZrujFepQX2YohGqzneyJDzricDWuJbSJJ0oOvdrjyUKFQ/640?wx_fmt=png)

4）取前 m 个最大特征值对应的向量，记为 W。

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfXGnOSicWlcWxickDtxq74ibgd5OO2aibx8XgI65xYE9F7NcntgbN4SJLZw/640?wx_fmt=png)

5）对样本集的每一个样本![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfbX5rdAeGocKwrWJEdUgcvG1TicmA5Sb6nuj6ib0YLtUMRQfutOEdGD8Q/640?wx_fmt=png)，映射为新的样本![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfxKvTIfVibgTlABoUAF9FRcmgibf6qAkVyrrIrnwj2Kz8jlfvoQLb41Kw/640?wx_fmt=png)。

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYf0u26CtZEo0UzCqialcN9sKeMMUYxQYOicycoeMIY8EGcuCmYa3iaf2AXg/640?wx_fmt=png)

6）得到映射后的样本集 D'。

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfp2AlBZBk7vEsNTrtjUqJHAyNT8KweicpUu4c4r9Zl6bNOlEIc5h4nlA/640?wx_fmt=png)

**7. 核主成分分析（KPCA）介绍**

  

  

  

因为![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfgia44EruVYA13ribrcBN8SbgfSckMkqa3T0d3ctGNPoZicXOPqHqFAp0w/640?wx_fmt=png)可以用样本数据内积表示：  

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfic7nM9AtGJxA7InxhXWraaeDfsiaSHBlj6Siba2tCXy6jiarZVP9ezCJeA/640?wx_fmt=png)

由核函数定义可知，可通过核函数将数据映射成高维数据，并对该高维数据进行降维：

![](https://mmbiz.qpic.cn/mmbiz_png/jupejmznDC8D91gHR76PSSe84swAxMYfW8sFcBkXfVMM3G8CbwxCTJgibwYpIHlSUnaxAzT1iacX3wW6M67bVuaQ/640?wx_fmt=png)

KPCA 一般用在数据不是线性的，无法直接进行 PCA 降维，需要通过核函数映射成高维数据，再进行 PCA 降维 。

PCA 是一种非监督学习的降维算法，只需要计算样本数据的协方差矩阵就能实现降维的目的，其算法较易实现，但是降维后特征的可解释性较弱，且通过降维后信息会丢失一些，可能对后续的处理有重要影响。  

参考

https://www.cnblogs.com/pinard/p/6239403.html#undefined

A Singularly Valuable Decompostion: The SVD of a Matrix

![](https://mmbiz.qpic.cn/mmbiz_jpg/jupejmznDCicCEDfm4Q5koCraSm45XoTnY8A5RQMIFlLNVKlC8bo97y7Pibp6VwDZmUGebhLN3akM0R19icNU6tCw/640?wx_fmt=jpeg)