> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/mGzNT_et0jQj8OgEiCekRA)

仅作学术分享，不代表本公众号立场，侵权联系删除

转载于：机器之心

**一、研究动机**

掩码建模（MIM, MAE）被证明是非常有效的自监督训练方法。然而，如图 1 所示，MIM 对于更大的模型效果相对更好。当模型很小的时候（比如 ViT-T 5M 参数，这样的模型对于现实世界非常重要），MIM 甚至可能一定程度上降低模型的效果。比如用 MAE 训练的 ViT-L 比普通监督训练的模型在 ImageNet 上的分类效果提升 3.3%，但是用 MAE 训练的 ViT-T 比普通监督训练的模型在 ImageNet 上的分类效果降低了 0.6%。

在这篇工作中我们提出了 TinyMIM，其在保持 ViT 结构不变并且不修改结构引入其他归纳偏置（inductive bias）的基础上、用蒸馏的方法迁移大模型上的知识到小模型。

*   论文地址：https://arxiv.org/pdf/2301.01296.pdf
    
*   代码地址：https://github.com/OliverRensu/TinyMIM
    

我们系统性的研究了蒸馏目标、数据增强、正则化、辅助损失函数等对于蒸馏的影响。在严格的只用 ImageNet-1K 作为训练数据的情况下（包括 Teacher model 也只用 ImageNet-1K 训练）和 ViT-B 作为模型，我们的方法实现了当前最好的性能。如图所示：

把我们的方法（TinyMIM）和基于掩码重建的方法 MAE，以及监督式学习的方法从头开始训练的 DeiT 作比较。MAE 在模型比较大的时候有显著的性能提升，但是在模型比较小的时候提升幅度有限甚至会伤害模型的最终效果。我们的方法 TinyMIM 在不同模型的大小上都有大幅提升。

我们的贡献如下：

1. 蒸馏的目标（Distillation targets）:1）蒸馏 token 之间的关系比单独蒸馏 class token 或者特征图（feature map）更有效；2）用中间层作为蒸馏的目标更有效。

2. 数据增强和模型正则化（Data and network regularization）：1）用带掩码的图片效果更差；2）学生模型需要一点 drop path，但是 teacher 模型不需要。

3. 辅助损失函数（auxiliary losses）：MIM 作为辅助损失函数没有意义。

4. 宏观蒸馏策略（Macro distillation strategy）：我们发现序列化的蒸馏（ViT-B -> ViT-S -> ViT-T）效果最好。

**二、方法**

我们系统性的调研了蒸馏的目标，输入的图片，蒸馏目标模块。

**2.1 影响蒸馏效果的因素**

1）特征：

a. 中间 block 特征和输出特征

当 i=L 时，指的是 Transformer 输出层的特征。当 i< L 时，指的是 Transformer 中间层的特征。

b. 注意力（Attention）特征和前馈层（FFN）层特征

Transformer 每一个 block 有 Attention 层和 FFN 层，蒸馏不同的层会带来不同的影响。

c.QKV 特征

在 Attention 层内会有 Q，K，V 特征，这些特征用于计算注意力机制，我们也调研了直接蒸馏这些特征。

2）关系

Q，K，V 用于计算注意力图，这些特征之间的关系也可以作为知识蒸馏的目标。

3）输入：是否带掩码

传统的知识蒸馏是直接输入完整的图片。我们的方法为了探索蒸馏掩码建模模型，所以我们也探索了带掩码的图片是否适合作为知识蒸馏时候的输入。

**2.2 知识蒸馏方法对比**

1）Class Token 蒸馏：

最简单的方法就是类似 DeiT 直接蒸馏 MAE 预训练模型的 class token:

其中![](https://mmbiz.qpic.cn/sz_mmbiz_png/KmXPKA19gW8blftBRIEKQX33Jd1bKxwdDMj13icnAIMXN7aBibaqIAlTlOc97tcQdwyI7mDR9SibPJ01taTEcGTibQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)指学生模型的 class token，而 ![](https://mmbiz.qpic.cn/sz_mmbiz_png/KmXPKA19gW8blftBRIEKQX33Jd1bKxwd73LpZRX0HYWjzB8Zq4IZkINR0LcMFsUxK75JaJ4LKt4soBmd706lMw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)指老师模型的 class token。

2）特征蒸馏：我们直接参考了 feature distillation [1] 作为对比

![](https://mmbiz.qpic.cn/sz_mmbiz_png/KmXPKA19gW8blftBRIEKQX33Jd1bKxwdz1n3segicqR8uVuc2fdmj7bdGH3ibLAvlXPLb26mmnE55Okt6bVbTSEg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/KmXPKA19gW8blftBRIEKQX33Jd1bKxwdACcEbhTbd9AfPb3J6wRibutuQlZicvM7ZcOSKhDbXicW0foIIANeicKia6g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

3）关系蒸馏：我们提出了也是本文默认的蒸馏策略

**三、实验**

**3.1 主要实验结果**

我们的方法在 ImageNet-1K 上预训练，而且教师模型也是在 ImageNet-1K 预训练。然后我们将我们预训练的模型在下游任务（分类、语义分割）上进行了微调。模型表现如图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/KmXPKA19gW8blftBRIEKQX33Jd1bKxwd4AL5ic06UQ67WibWQNHdIxPicF0jWUA7wibh23xUYoic0STQ78VlSkia3KmA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们的方法显著超过之前基于 MAE 的方法，尤其是小模型。具体来讲，对于超小的模型 ViT-T，我们的方法实现了 75.8% 的分类准确性，相比 MAE 基线模型实现了 4.2 的提升。对于小模型 ViT-S，我们实现了 83.0% 的分类准确性，比之前最好的方法提升了 1.4。对于 Base 尺寸的模型，我们的方法分别超过 MAE 基线模型和以前最好的模型 CAE 4.1 和 2.0。

同时我们也测试了模型的鲁棒性，如图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/KmXPKA19gW8blftBRIEKQX33Jd1bKxwduHR2DicdNicbVvWrXICibG0Zr41SWNN67xnGwXlNiaS9TBQ9ff6xmibZ92g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

TinyMIM-B 对比 MAE-B，在 ImageNet-A 和 ImageNet-R 分别提升了 + 6.4 和 +4.6。

**3.2 消融实验**

1）蒸馏不同关系

![](https://mmbiz.qpic.cn/sz_mmbiz_png/KmXPKA19gW8blftBRIEKQX33Jd1bKxwdo3XWl8w2mwOfZpFLZW6L9EeFZLJcibIQ2mkcJYxvFiczYAMTiaSqIGZbQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

同时蒸馏 QK,VV 关系而且在计算关系的时候有 Softmax 实现了最好的效果。

2）不同的蒸馏策略

![](https://mmbiz.qpic.cn/sz_mmbiz_png/KmXPKA19gW8blftBRIEKQX33Jd1bKxwdP6bWoJfon6Unhlv30nmG4c0iceibf3RvvGCKWFXsdYcIFbQ4oC5WBvjQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

TinyMIM 这种蒸馏关系的方法实现了比 MAE 基线模型，class token 蒸馏，特征图蒸馏都更好的效果，在各种尺寸的模型上都是如此。

3）蒸馏中间层

![](https://mmbiz.qpic.cn/sz_mmbiz_png/KmXPKA19gW8blftBRIEKQX33Jd1bKxwdpZuAicbjZk4UXiaxhFOdvjkEpajJDfQQLgpwHzdAibS7Jntv34DHqFkibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们发现蒸馏第十八层实现了最好的效果。

**四、结论**

在本文中，我们提出了 TinyMIM，它是第一个成功地使小模型受益于掩码重建建模（MIM）预训练的模型。我们没有采用掩码重建作为任务，而是通过以知识蒸馏的方式训练小模型模拟大模型的关系来预训练小模型。TinyMIM 的成功可以归功于对可能影响 TinyMIM 预训练的各种因素的全面研究，包括蒸馏目标、蒸馏输入和中间层。通过大量的实验，我们得出结论，关系蒸馏优于特征蒸馏和类标记蒸馏等。凭借其简单性和强大的性能，我们希望我们的方法能够为未来的研究提供坚实的基础。

_[1] Wei, Y., Hu, H., Xie, Z., Zhang, Z., Cao, Y., Bao, J., ... & Guo, B. (2022). Contrastive learning rivals masked image modeling in fine-tuning via feature distillation. arXiv preprint arXiv:2205.14141._