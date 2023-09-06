> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/t4AGxQLRilv5cnci6Q82uQ)

> 决策树、随机森林、bagging、boosting、Adaboost、GBDT、XGBoost 总结

一. 决策树

决策树是一个有监督分类模型，本质是选择一个最大信息增益的特征值进行输的分割，直到达到结束条件或叶子节点纯度达到阈值。下图是决策树的一个示例图：

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDHt9M27Y80TSjL1fAZ81rTRHCUB3BwhMhkT5yQM4CnXVLUAdWAAeNbQ/640?wx_fmt=jpeg)

根据分割指标和分割方法，可分为：ID3、C4.5、CART 算法。

**1.ID3 算法：以信息增益为准则来选择最优划分属性**

信息增益的计算是基于信息熵（度量样本集合纯度的指标）

![](https://mmbiz.qpic.cn/mmbiz_svg/LwcbhAmMnZCAjC4TiaGVE64KzClziajNvjNJibrEibaG4BMAGticx0wCBJX5AV0cvQoFjJnxiaKZvpPx7WNa7E4acZACicz1wRMgN1I/640?wx_fmt=svg) 信息熵越小，数据集 ![](https://mmbiz.qpic.cn/mmbiz_svg/LwcbhAmMnZCAjC4TiaGVE64KzClziajNvjP6cWW0ovTGw2F4zz9KwPkynctj9qGe0OCj3JFeBe1qh6wJ1QCKeKbQSz7uDf6o39/640?wx_fmt=svg) 的纯度越大

假设基于数据集 ![](https://mmbiz.qpic.cn/mmbiz_svg/LwcbhAmMnZCAjC4TiaGVE64KzClziajNvjsk6Ro2kWbSxRmqkNaM0YsastLWtCBNXqd8Pg6nibknibEviazHjEAcDloBqKYALuf8b/640?wx_fmt=svg) 上建立决策树，数据有 ![](https://mmbiz.qpic.cn/mmbiz_svg/LwcbhAmMnZCAjC4TiaGVE64KzClziajNvjCMHaG27GLJDSH6lpkic05iamdFm4dLtNgEVVQPus9mQ19v6Zh9nQDWz1HIrS3PoCNF/640?wx_fmt=svg) 个类别：

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDs1cqc99LCibupNZ4oWSsfyS2Wf79Q6oXNWyRRcrEXFYAOx7ypRGDokA/640?wx_fmt=jpeg)

公式 (1) 中：

![](https://mmbiz.qpic.cn/mmbiz_svg/LwcbhAmMnZCAjC4TiaGVE64KzClziajNvjS0bErDaz2wxrhybwjDNKG3RiafV7zOxM5F3qgSa8mMJUg6cogwOgNjPdOXFqW1Lby/640?wx_fmt=svg) 表示第 K 类样本的总数占数据集 D 样本总数的比例。

公式 (2) 表示是以特征 A 作为分割的属性，得到的信息熵：Di 表示的是以属性 A 为划分，分成 n 个分支，第 i 个分支的节点集合。因此，该公式求得的是以属性 A 为划分，n 个分支的信息熵总和。

公式 (3) 是以 A 为属性划分前和划分后的信息熵差值，也就是信息增益，越大越好。

假设每个记录有一个属性'ID', 若按照 ID 进行分割的话，在这个属性上，能够取得的特征值是样本数，特征数目太多，无论以哪一个 ID 进行划分，叶子节点的值只会有一个，纯度很大，得到的信息增益很大，这样划分出来的决策树没有意义，即，_ID3 偏向于取值较多的属性进行分割，存在一定的偏好_。为减少这一影响，有学者提出了 C4.5 算法。

**2.C4.5 基于信息增益率准则 选择最有分割属性的算法**

信息增益率通过引入一个被称为分裂信息 (Split information) 的惩罚项来惩罚取值较多的属性：

![](https://mmbiz.qpic.cn/mmbiz_svg/LwcbhAmMnZCAjC4TiaGVE64KzClziajNvjJlwHVSrke0hD7jgBmBlbf2wreiaazz1DibhN6swkuxdD8MsJgUnVQicoc8LfxRcxhdH/640?wx_fmt=svg) , ![](https://mmbiz.qpic.cn/mmbiz_svg/LwcbhAmMnZCAjC4TiaGVE64KzClziajNvjauIasQmzY1OqwOToib7rEicjkf34VesDAvCTScB5zsI7TybTO38DKy32E4lJzayicfh/640?wx_fmt=svg)

其中，**IV(a) 是由属性 A 的特征值个数决定的，个数越多，IV 值越大**，信息增益率越小，这样就可以避免模型偏好特征值多的属性，如果简单按这个规则分割，模型又会偏好特征值少的特征，因此 C4.5 决策树先从候选划分属性中找出信息增益高于平均水平的属性，在从中选择增益率最高的。

对于连续值属性来说，可取值数目不再有限，因此可以采用离散化技术（如二分法）进行处理。将属性值从小到大排序，然后选择中间值作为分割点，数值比它小的点被划分到左子树，数值不小于它的点被分到右子树，计算分割的信息增益率，选择信息增益率最大的属性值进行分割。

**3.CART: 以基尼系数为准则选择最优划分属性，可用于分类和回归**

CART 是一棵二叉树，采用二元切分法，每次把数据分成两份，分别进入左子树、右子树。并且每个非叶子节点都有两个孩子，所以 CART 的叶子节点比非叶子节点多一。相比于 ID3 和 C4.5，CART 的应用要多一些，既可以用于分类也可以用于回归。CART 分类时，选择基尼指数（Gini）为最好的分类特征，gini 描述的是纯度，与信息熵含义类似，CART 中每次迭代都会降低基尼系数。

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkD97ql49XiaETEK5u57spT5kjPDRctqp5ecubt9zYsjWYj9mewLO81r9w/640?wx_fmt=jpeg)

Gini(D) 反映了数据集 D 的纯度，值越小，纯度越高。我们在候选集合中选择使得划分后基尼指数最小的属性作为最优化分属性。

**分类树和回归树**

先说分类树，ID3、C4.5 在每一次分支时，是穷举每一个特征属性的每一个阈值，找到使得按照特征值 <= 阈值，和特征值> 阈值分成的两个分支的熵最大的特征和阈值。按照该标准分支得到两个新节点，用同样的方法进行分支，直到所有人被分入性别唯一的叶子节点，或达到预设的终止条件，若最终叶子节点中性别不唯一，则以多数人的性别作为该叶子节点的性别。

回归树总体流程也是类似，不过在每个节点 (不一定是叶子节点) 都会得到预测值，以年龄为例，该预测值等于属于这个节点的所有人年龄的平均值。分支时穷举每个特征的每个阈值，找最好的分割点，但衡量的标准变成了最小化均方误差，即（每个人的年龄 - 预测年龄）^2 的总和 / N，或者说是每个人的预测误差平方和 除以 N。这很好理解，被预测出粗的人数越多，错的越离谱，均方误差越大，通过最小化均方误差找最靠谱的分支依据。分支直到每个叶子节点上的人的年龄都唯一(这太难了)，或者达到预设的终止条件(如叶子个数上限)，若最终叶子节点上人的年龄不唯一，则以该节点上所有人的平均年龄作为该叶子节点的预测年龄。

二. 随机森林
-------

先补充**组合分类器**的概念，将多个分类器的结果进行多票表决或取平均值，以此作为最终的结果。

1. 构建组合分类器的好处：

(1) 提升模型精度：整合各个模型的分类结果，得到更合理的决策边界，减少整体错误呢，实现更好的分类效果：

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDRwbe1EumDXic0mkzJprNYcffaBnLor4E9rhArX0aMslx6rzJbzAkeZQ/640?wx_fmt=jpeg)

(2)处理过大或过小的数据集：数据集较大时，可将数据集划分成多个子集，对子集构建分类器；当数据集较小时，通过自助采样 (bootstrap) 从原始数据集采样产生多组不同的数据集，构建分类器。

(3) 若决策边界过于复杂，则线性模型不能很好地描述真实情况。因此，现对于特定区域的数据集，训练多个线性分类器，再将他们集成。

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDhz3tKJbnCHib9rh6icib1SbNvcllXn0B6T1cZMC4iaFJRhwXZ2Aj5M72VQ/640?wx_fmt=jpeg)

(4) 比较适合处理多源异构数据（存储方式不同（关系型、非关系型），类别不同（时序型、离散型、连续型、网络结构数据））

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDRKcOyOEVXzCeAzY1prnJXAIkx2nd1zUdVe17TgoQahQ5azGibqbgB4g/640?wx_fmt=jpeg)

随机森林是一个多决策树的组合分类器，随机主要体现在两个方面：数据选取的随机性和特征选取的随机性。

(1) 数据的随机选取

第一，从原始数据集中采取有放回的抽样 (bootstrap), 构造子数据集，子数据集扥数量和原始数据集的数量一样。不同的子数据集的元素可以重复，同一个子数据集中的元素也可以重复。

第二，利用子数据集构建子决策树，将这个数据放到每个子决策树中，每个子决策树输出一个结果。最后，如果有了新的数据需啊哟通过随机森林得到分类结果，就可以通过子决策树的判断结果来投票，得到随机森林的输出结果。如下图，假设随机森林中有 3 棵子决策树，2 棵子树的分类结果是 A 类，1 棵子树的分类结果是 B 类，那么随机森林的分类结果就是 A 类。

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDKablbHUZ2Ag0Nn3OW959406f6qYllzIa7H6xYR8eyDYxEV1wnlxJLw/640?wx_fmt=jpeg)

(2) 待选特征的随机选取

类似于数据集的随机选取，随即森林中的子树的每一个分裂过程并未用到所有的待选特征，而是从所有的待选特征中随机选取一定的特征，之后再在随机选取的特征中选择最优的特征。这样能使随机森林中的决策树能不同，提升系统的多样性，从而提升分类性能。

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDSrSO729oSic3oJDcULyuM8xOibEy29cXCRQliaUuyAXb0XuhrcbvrsWlQ/640?wx_fmt=jpeg)

组合树示例图

三. GBDT 和 XGBoost
-----------------

**1. 在讲 GBDT 和 XGBoost 之前先补充 Bagging 和 Boosting 的知识。**

Bagging 是并行的学习算法，思想很简单，即每一次**从原始数据中根据均匀概率分布有放回的抽取和原始数据集一样大小的数据集合。**样本点可以出现重复，然后对每一次产生的数据集构造一个分类器，再对分类器进行组合。

**Boosting 的每一次抽样的样本分布是不一样的**，每一次迭代，都是根据上一次迭代的结果，增加被错误分类的样本的权重。使模型在之后的迭代中更加注重难以分类的样本。这是一个不断学习的过程，也是一个不断提升的过程，这就是 Boosting 思想的本质所在。迭代之后，将每次迭代的基分类器进行集成，那么如何进行样本权重的调整和分类器的集成是我们需要考虑的关键问题。

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDTNiaTPwmNDZ9H2DgewU0GlvUNrEvIicOHicAftVDnuhAqIooibx7jt89DQ/640?wx_fmt=jpeg)

Boosting 算法结构图

以著名的 Adaboost 算法举例：

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDSa72p5Uiclz1UAyUg6Kavjib6MrTxHhISvhwVoicCkVy4ldyrUUmoR6yQ/640?wx_fmt=jpeg)

有一个数据集，样本大小为 N，每一个样本对应一个原始标签起初，我们初始化样本的权重为 1/N

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkD1paFTVq0fduicsllwcoCFF195wcqY1mmhyTIic7vFKGoUsyAJJhqRtmA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_svg/LwcbhAmMnZCAjC4TiaGVE64KzClziajNvj6t2LkuQLyjugX2xS3Dw56cWkuC4rmBZ8YNovAmsRVhiaJ5TvrEmjsiasR1yWE4T198/640?wx_fmt=svg) 计算的是当前数据下，模型的分类误差率，模型的系数值是基于分类误差率的

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDSib1cP4dqMZZoRzNDe0j5qq3lNexAIcSZHyJ8bV5vzgGJMrRVEF7V5A/640?wx_fmt=jpeg)

根据模型的分类结果，更新原始数据中数据的分布，增加被错分的数据被抽中的概率，以便下一次迭代的时候能被模型重新训练

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDZHbib6rLM9ITKBicyGiaGcj9S9ovP8yHVpnXibjO1aIGbOPbASiaVic2I6qw/640?wx_fmt=jpeg)

最终的分类器是各个基分类器的组合

**2.GBDT**

**GBDT 是以决策树 (CART) 为基学习器的 GB 算法，是迭代树而不是分类树，**Boost 是 "提升" 的意思，一般 Boosting 算法都是一个迭代的过程，每一次新的训练都是为了改进上一次的结果。有了前面 Adaboost 的铺垫，大家应该能很容易理解大体思想。

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkD51sUUgia7EKvG1LJ5SVsHbVibicIAlYYgQs2skhmSpynadAryZibDtJtVg/640?wx_fmt=jpeg)

GBDT 的核心是：**每一棵树学习的是之前所有树结论和的残差。这个残差就是一个加预测值后能得真实值的累加量。**比如 A 的真实年龄是 18 岁，但第一棵树的预测年龄是 12 岁，差了 6 岁，即残差为 6 岁。那么在第二棵树里我们把 A 的年龄设为 6 岁去学习，如果第二棵树真的能把 A 分到 6 岁的叶子节点，那累加两棵树的结论就是 A 的真实年龄；如果第二棵树的结论是 5 岁，则 A 仍然存在 1 岁的残差，第三棵树里 A 的年龄就变成 1 岁，继续学习。

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDgPejCsg8zYOq1dibP6qKMqlcyic9fLf3saWiagaUbMGvxR79XUxtoSxoA/640?wx_fmt=jpeg)

**3.XGBoost**

XGBoostt 相比于 GBDT 来说，更加**有效应用了数值优化，最重要是对损失函数**（预测值和真实值的误差）**变得更复杂**。目标函数依然是所有树的预测值相加等于预测值。

损失函数如下，引入了一阶导数，二阶导数：

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkD5DnJghA043RudjnMRUzib88WOES0xJE1AVD1P4h1oWyZiak3ANUg8jlg/640?wx_fmt=jpeg)

好的模型需要具备两个基本要素：一是要有好的精度（即好的拟合程度），二是模型要尽可能的简单（复杂的模型容易出现过拟合，并且更加不稳定）因此，我们构建的目标函数右边第一项是模型的误差项，第二项是正则化项（也就是模型复杂度的惩罚项）

常用的误差项有平方误差和逻辑斯蒂误差，常见的惩罚项有 l1，l2 正则，l1 正则是将模型各个元素进行求和，l2 正则是对元素求平方。

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDhWLy6JLwsjOJZ0MaPp64a7EzpfREcGGg5fpXib7Fa4ErUtA7SOJ9qng/640?wx_fmt=jpeg)

每一次迭代，都在现有树的基础上，增加一棵树去拟合前面树的预测结果与真实值之间的残差

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDqeE6xJVyictkkureKxbibhAOnsialQyQSllmnnlRG8e6YGkib6mlDsW9ag/640?wx_fmt=jpeg)![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDOxCtNtdtNkD3cTfHibO8bVJzicK8o5TONGxicAQzs0EscBiaTAEgIc4WicA/640?wx_fmt=jpeg)

目标函数如上图，最后一行画圈部分实际上就是预测值和真实值之间的残差

先对训练误差进行展开：

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDnePKufCdQDcy0ar2QtTTsJunu6VA5r1eTJBs57el0AZ7lvMkPpASpQ/640?wx_fmt=jpeg)

xgboost 则对代价函数进行了二阶泰勒展开，同时用到了残差平方和的一阶和二阶导数

再研究目标函数中的正则项：

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDoQo6ee5UtZicFvMKfiaYXPiaumTKGpibWyYkA5Utte9Pj1HTuUicz32oAcA/640?wx_fmt=jpeg)

树的复杂度可以用树的分支数目来衡量，树的分支我们可以用叶子结点的数量来表示

那么树的复杂度式子：右边第一项是叶子结点的数量 T，第二项是树的叶子结点权重 w 的 l2 正则化，正则化是为了防止叶子结点过多

此时，每一次迭代，相当于在原有模型中增加一棵树，目标函数中，我们用 wq（x）表示一棵树，包括了树的结构以及叶子结点的权重，w 表示权重（反映预测的概率），q 表示样本所在的索引号（反映树的结构）

将最终得到的目标函数对参数 w 求导，带回目标函数，可知目标函数值由红色方框部分决定：

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDrOnslFSq9Y5PeiaNQCCUEZL5eALtscrbZvakl5xHNLahD8xP1ObYk7g/640?wx_fmt=jpeg)

因此，xgboost 的迭代是以下图中 gain 式子定义的指标选择最优分割点的：

![](https://mmbiz.qpic.cn/mmbiz_jpg/njjfaJS7c9pgyvbGgyJZyThv3gvFtBkDl1y0qkA0aUqQgqJGwUKYvokGcWaiazicntnyrzDI4Wqb8ibM0r17QkJ5A/640?wx_fmt=jpeg)

那么如何得到优秀的组合树呢？

一种办法是贪心算法，遍历一个节点内的所有特征，按照公式计算出按照每一个特征分割的信息增益，找到信息增益最大的点进行树的分割。增加的新叶子惩罚项对应了树的剪枝，当 gain 小于某个阈值的时候，我们可以剪掉这个分割。但是这种办法不适用于数据量大的时候，因此，我们需要运用近似算法。

另一种方法：XGBoost 在寻找 splitpoint 的时候，不会枚举所有的特征值，而会对特征值进行聚合统计，按照**特征值的密度分布**，构造直方图计算特征值分布的面积，然后划分分布形成若干个 bucket(桶)，每个 bucket 的面积相同，**将 bucket 边界上的特征值作为 split  
point 的候选，****遍历所有的候选分裂点**来找到最佳分裂点。

上图近似算法公式的解释：将特征 k 的特征值进行排序，计算特征值分布，rk（z）表示的是对于特征 k 而言，其特征值小于 z 的权重之和占总权重的比例，代表了这些特征值的重要程度，我们按照这个比例计算公式，将特征值分成若干个 bucket，每个 bucket 的比例相同，选取这几类特征值的边界作为划分候选点，构成候选集；选择候选集的条件是要使得相邻的两个候选分裂节点差值小于某个阈值

**综合以上所述，我们可以得到 xgboost 相比于 GBDT 的创新之处：**

1、传统 GBDT 以 CART 作为基分类器，xgboost 还支持线性分类器，这个时候 xgboost 相当于带 L1 和 L2 正则化项的逻辑斯蒂回归（分类问题）或者线性回归（回归问题）。

2、传统 GBDT 在优化时只用到一阶导数信息，xgboost 则对代价函数进行了**二阶泰勒展开，同时用到了一阶和二阶导数。**顺便提一下，xgboost 工具支持自定义代价函数，只要函数可一阶和二阶求导。

3、xgboost 在**代价函数里加入了正则项，用于控制模型的复杂度。正则**项里包含了树的叶子节点个数、每个叶子节点上输出的 score 的 L2 模的平方和。从 Bias-variance tradeoff 角度来讲，正则项降低了模型的 variance，使学习出来的模型更加简单，防止过拟合，这也是 xgboost 优于传统 GBDT 的一个特性。

4、**Shrinkage（缩减），相当于学习速率（xgboost 中的 eta）。**每次迭代，增加新的模型，在前面成上一个小于 1 的系数，降低优化的速度，每次走一小步逐步逼近最优模型比每次走一大步逼近更加容易避免过拟合现象；

5、**列抽样**（column subsampling）。xgboost 借鉴了随机森林的做法，支持列抽样（即每次的输入特征不是全部特征），不仅能降低过拟合，还能减少计算，这也是 xgboost 异于传统 gbdt 的一个特性。

6、**忽略缺失值：**在寻找 splitpoint 的时候，不会对该特征为 missing 的样本进行遍历统计，只对该列特征值为 non-missing 的样本上对应的特征值进行遍历，通过这个工程技巧来减少了为稀疏离散特征寻找 splitpoint 的时间开销。

7、**指定缺失值的分隔方向：**可以为缺失值或者指定的值指定分支的默认方向，为了保证完备性，会分别处理将 missing 该特征值的样本分配到左叶子结点和右叶子结点的两种情形，分到那个子节点带来的增益大，默认的方向就是哪个子节点，这能大大提升算法的效率。

8、**并行化处理：**在训练之前，预先对每个特征内部进行了排序找出候选切割点，然后保存为 block 结构，后面的迭代中重复地使用这个结构，大大减小计算量。在进行节点的分裂时，需要计算每个特征的增益，最终选增益最大的那个特征去做分裂，那么各个特征的增益计算就可以开多线程进行，即在不同的特征属性上采用多线程并行方式寻找最佳分割点。

作者：ChrisCao

来源：https://zhuanlan.zhihu.com/p/75468124