> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iIf01y6WBuIbdCK536I38A)

> MNN 对 ConvolutionDepthwise Int8 量化算子在 ARM V8(64 位) 和 ARM V8.2 上的性能做了较大的优化，主要优化方法包括改变数据的排列方式以及使用 Sdot 指令。

本文介绍了 Depthwise Convolution 的 Int8 算子在移动端 CPU 上的性能优化方案。ARM 架构的升级和相应指令集的更新不断提高移动端各算子的性能上限，结合数据重排和 Sdot 指令能给 DepthwiseConv 量化算子的性能带来较大提升。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju8X1wEorjS3bDLnHiar4vtV5RkRoYd65guD5FtbNgFoz71Fzyp1yc7WklYCvES93U4NELnJf4lFzgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

背景

MNN 对 ConvolutionDepthwise Int8 量化算子在 ARM V8(64 位) 和 ARM V8.2 上的性能做了较大的优化，主要优化方法包括改变数据的排列方式以及使用 Sdot 指令。为了方便叙述，后文统一用 DepthwiseConvInt8 来表示 ConvolutionDepthwiseInt8 量化算子。在优化前在安卓手机上测得该量化算子的耗时比使用 FP16 推理多三倍，严重影响量化模型在端侧的性能。本文从 ARM 汇编算子的角度出发讲解 DepthwiseConvInt8 算子的优化方案。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju8X1wEorjS3bDLnHiar4vtV5s4iaFibfqswhDiaUmcuk0ibG6v33ybaPY8N6ZVvedwxAbibQ1ib6BIlnJtRw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

性能瓶颈定位及优化方案

#### **▐** **ARM V8 优化数据排列方式**

*   ### 方案介绍
    

MNN 中 DepthwiseConvInt8 的输入数据格式是![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicCImeqevzQNV6hZibsIloAgmFLQVTS3zEJjjTFibLklTMRQjXA0Qgzl7Po0LhiagxjJibmLVqLa7pYzQ/640?wx_fmt=png&from=appmsg)，后文简称这种数据排列数据方式为 C4），即每 4 个 Channel 的数据排列在一起。同理，我们用 C16 表示每 16 个 Channel 的数据连续排列的数据排布方式。下图直观地描述了 C4 的数据排列方式，假设![](https://mmbiz.qpic.cn/mmbiz_jpg/33P2FdAnjuicCImeqevzQNV6hZibsIloAgHHJJvTMBVOcVCGy46H3zS3PIu4iaW8uwqia57W8iaAL1cw2jExAnNEPEg/640?wx_fmt=jpeg)，图中每一个小正方形代表一个数据，四种颜色分别代表 feature map 上四个坐标点，正方形内的数字编号![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicCImeqevzQNV6hZibsIloAg3xbq1XeW7SvabWaDdibvLQYibicq9tUS0C7R7WRWBuORwDBbkbgd69bYw/640?wx_fmt=png&from=appmsg)表示它们在内存中的排列次序。

![](https://mmbiz.qpic.cn/mmbiz_jpg/33P2FdAnjuicCImeqevzQNV6hZibsIloAgal81kqzvzMOFJ8jaBfb72gjuKdH72ibU0808uVy3yxFxwQEKnOPJQfw/640?wx_fmt=jpeg)

一般地，我们会对![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicCImeqevzQNV6hZibsIloAgt65Xlw3CCHaaNjXPVib5hfWuRpl3YB0QtfD4MokISm6LSdJLL8UQIsQ/640?wx_fmt=png&from=appmsg)采取多线程计算，对![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicCImeqevzQNV6hZibsIloAglnIELQQia7rUuRptjUG3NQf1bZrfhZP1P1Nvibo2zl7SgqfrFJ46e0rA/640?wx_fmt=png&from=appmsg)采取 ARM 汇编并行计算。本文关注单线程时对![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicCImeqevzQNV6hZibsIloAgOKAQic0kap8UEcfXRB6oN6d3aVn33w9xWd06tasQsCcIqiamicAAZO40g/640?wx_fmt=png&from=appmsg)维度做性能优化。在 ARM 汇编中通常会同时计算输出的特征图上多个点的结果，这要求我们尽量能够同时读取输入特征图上的点数据。例如当 Depthwise 层的 Stride 参数等于 1 时，同时计算输出特征图中 4 个点坐标的数据（一共 4(坐标点)*4(channel)=16 个数据），我们需要读取上图中编号为![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnjuicCImeqevzQNV6hZibsIloAgAPHdCI08MdnrIIPWTBMJgKb8FiaIMdStDjydZ6R17vzabX90a6vICOA/640?wx_fmt=png&from=appmsg)的正方形数据。当每个数据占 4 个字节（32 位）时，4 个输入数据正好填满一个向量寄存器。但是在 Int8 量化算子中，每个输入数据占 8 位（1 个字节），如果仍然采取 C4 的数据排列，则 4 个数据填不满一个向量寄存器。特别地，当 Depthwise 层的 Stride 参数不是 1 时，采用 C4 的数据排列方式会导致 feature map 上多个数据时不能连续读取，进而导致性能损失。所以对于 Int8 类型的输入数据，在 ARM 平台上我们首先考虑将 C4 的数据排列方式改为 C16 的排列，这样保证可以连续地读取 16 个 Int8 类型数据来填满一个向量寄存器。我们用汇编代码来表示这两种数据读取方式的差异。

```
/* x0: source address, 读取feature map上的4个点 */
/* pack=4, stridex = 2, sizeof(inputData)=1 */
ld1 {v0.s}[0], [x0], #8
ld1 {v1.s}[0], [x0], #8
ld1 {v2.s}[0], [x0], #8
ld1 {v3.s}[0], [x0], #8
/* pack=16, stridex = 2, sizeof(inputData)=1 */
ld1 {v0.4s}, [x0], #32
ld1 {v1.4s}, [x0], #32
ld1 {v2.4s}, [x0], #32
ld1 {v3.4s}, [x0], #32

```

从代码中可以看到同样是使用 4 条指令读取数据，C16 比 C4 多读取 3 倍的数据。所以对于 ARM V8 平台，我们将 pack=4 更改为 pack=16，虽然该方案额外增加了推理时数据排布的转换时间，但是在 ARM 汇编内部更好地并行化带来的收益更大。

*   ### ARM V8 性能提升结果
    

本文中所有的性能数据均来源于美颜模型在华为 Mate40 Pro 上的测试结果。美颜模型中一共含有 23 个 Convolution Depthwise 算子，所有算子都是 3x3 kernel，其中 19 个算子的 stride=1，4 个算子的 stride=2. 表格中记录的算子耗时是美颜模型中所有 23 个 convolution depthwise 算子在一次推理中的耗时总和，单位：ms.

<table width="498"><colgroup><col width="187"><col width="152"><col width="159"></colgroup><tbody><tr data-cangjie-key="71" data-sticky="false"><td data-cangjie-key="73" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><p>华为 Mate40 Pro ARM V8</p></td><td data-cangjie-key="78" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><p>优化前 C4 数据排列</p></td><td data-cangjie-key="83" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><p>优化后 C16 数据排列</p></td></tr><tr data-cangjie-key="88" data-sticky="false"><td data-cangjie-key="90" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><p>Convolution Depthwise Int8 量化算子</p></td><td data-cangjie-key="95" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><p>4.46 ms</p></td><td data-cangjie-key="100" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true"><p>2.78 ms</p></td></tr></tbody></table>

改变数据排列方式的方案性能加速比是 1.6.

#### **▐** **ARM V8.2 使用 sdot 指令提高性能**

*   ### 为什么 sdot 指令能提高算子性能？
    

虽然在 ARM V8 平台上 DepthwiseConvInt8 算子的性能加速已经有明显效果，但 ARM V8.2 指令集为 ARM 平台上 CPU 算子的性能加速提供了更大的空间。与此同时，随着 ARM V8.2 指令集在旗舰型手机上的普及，浮点模型使用 fp16 推理极大地提高了移动端性能，量化模型想要保持竞争力也需要进一步优化。在 DepthwiseConvInt8 算子含有大量乘加计算，以 3x3kernel 为例，先进行 9 次乘法再进行 1 次加法，此时需要 9 条指令才能得到一个输出数据。ARM V8.2 平台提供的 sdot 指令能实现 3 条指令即可得到 1 个输入数据。汇编代码能够直观展示两者差异：

```
// 3x3kernel 不使用sdot指令，进行9次循环
Loop_Kernel_H3:
  Loop_Kernel_W3:
    smlal v0.4s, v1.4h, v2.4s // 累加结果存储在v0中
// 3x3kernel 使用sdot指令
sdot v0.4s, v1.16b, v3.16b
sdot v0.4s, v2.16b, v4.16b
smlal v0.4s, v5.4h, v6.4h

```

*   ### 使用 sdot 指令会带来额外开销吗？
    

我们先关注 sdot 指令是如何加速若干个元素乘加操作的，下面的代码展示了 sdot 指令计算原理。

```
// v0.16b : [0,1,2,3, 4,5,6,7, 8,9,10,11, 12,13,14,15]
// v1.16b : [0,1,2,3, 4,5,6,7, 8,9,10,11, 12,13,14,15]
// v2.s[0]=v0.b[0]*v1.b[0]+v0.b[1]*v1.b[1]
//         +v0.b[2]*v1.b[2]+v0.b[3]*v1.b[3]
sdot v2.4s, v0.16b, v1.16b // v2.4s: [14,126,366,734]

```

由代码示例得知，进行累加的 4 个元素在内存中是连续排列的。上文中已经介绍了算子的输入数据排列是 C4，同一个 Kernel 中的所有元素在内存中肯定不能连续读取。

sdot 指令的使用之前需要重排数据才能得到正确的结果，注意此时的数据重排和 ARM V8 优化中的数据重排完全不同。上文中我们提到的 ARM V8 上的数据重排仅仅是在 Channel 维度将 C4 修改为 C16，而 ARM V8.2 上 sdot 指令之前的数据重排需要将一个 kernel 中的 9 个元素每 4 个连续排列在一起。我们用一张图来解释重排规则。不妨假设 kernel size 是 3x3，后文中均采取此假设。

![](https://mmbiz.qpic.cn/mmbiz_jpg/33P2FdAnjuicCImeqevzQNV6hZibsIloAgfiaBB2qLkVQhAR4Q7WiafYTaZibQDbFgXDHT0icXn9JyN2VBMo3K3e3bRA/640?wx_fmt=jpeg)

显然数据重排的时间会极大影响算子性能，高效地进行数据重排是 ARM V8.2 DepthwiseConvInt8 性能优化的关键步骤。

*   ### ARM V8.2 如何进行数据重排
    

思考重排问题时我们将问题简化，不需要写代码就能得到最高效的解决方案（前提是足够熟悉 ARM 汇编指令，因为指令集了解地越多，解决方法就越多）。我们把问题抽象为如何高效重排一个向量寄存器中的 16 个 int8_t 数据。重排前后的数据排列对比图如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/33P2FdAnjuicCImeqevzQNV6hZibsIloAgyE74ecgzB9gLCuGgibUtgJlxm9CxibBgMVSSFZibg5nKW0QicUHQzgWr4g/640?wx_fmt=jpeg)

从抽象问题回到对 3x3 kernel 中的数据进行重排，我们最终需要 9 个元素相乘再累加，所以我们对 kernel 中的每 4 个元素进行重排，剩下的 1 个再使用 smlal 指令累加结果即可。当然我们推理前就可以对权值矩阵进行对应元素的重排，不占用推理时间。跟 ARM V8 上的优化不同的是，ARM V8.2 的重排是基于 C4 进行的而不是 C16. 无论是 C8 还是 C16 都会给数据重排带来更多的开销，具体来说就是需要的重排指令条数更多，这里就不详细讲解 C8 和 C16 场景下的最优重排方案了。

因为数据重排这一步是必不可少的，所以在该算子中使用 sdot 指令的前提是 kernel size 是已知的。当前 MNN 对于 DepthwiseConvInt8 在 ARM V8.2 平台上的优化仅支持 3x3 kernel.

*   ### ARM V8.2 性能提升结果
    

<table width="657"><tbody><tr data-cangjie-key="169" data-sticky="false"><td data-cangjie-key="171" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" data-style="box-sizing: inherit; padding: 8px 7px; border-color: rgb(217, 217, 217);"><section>华为 Mate40 Pro ARM V8</section></td><td data-cangjie-key="176" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" data-style="box-sizing: inherit; padding: 8px 7px; border-color: rgb(217, 217, 217);"><section>优化前 C4 数据排列</section></td><td data-cangjie-key="181" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" data-style="box-sizing: inherit; padding: 8px 7px; border-color: rgb(217, 217, 217);"><section>ARM V8 使用 C16 优化后</section></td><td data-cangjie-key="186" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" data-style="box-sizing: inherit; padding: 8px 7px; border-color: rgb(217, 217, 217);"><section>ARM V8 使用 sdot 指令优化后</section></td></tr><tr data-cangjie-key="191" data-sticky="false"><td data-cangjie-key="193" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" data-style="box-sizing: inherit; padding: 8px 7px; border-color: rgb(217, 217, 217);"><section>Convolution Depthwise Int8 量化算子</section></td><td data-cangjie-key="198" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" data-style="box-sizing: inherit; padding: 8px 7px; border-color: rgb(217, 217, 217);"><section>4.46 ms</section></td><td data-cangjie-key="203" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" data-style="box-sizing: inherit; padding: 8px 7px; border-color: rgb(217, 217, 217);"><section>2.78 ms</section></td><td data-cangjie-key="208" data-type="table-cell" rowspan="1" colspan="1" data-container-block="true" data-style="box-sizing: inherit; padding: 8px 7px; border-color: rgb(217, 217, 217);"><section>1.75 ms</section></td></tr></tbody></table>

通过使用 sdot 指令，算子性能加速比达到了 2.55.

#### **▐** **ARM V8.2 和 ARM V8 的优化方案差异总结**

在 ARM V8.2 上优化 DepthwiseConvInt8 算子性能的核心是重排输入数据以方便使用 sdot 指令进行累加。因为 Depthwise 算子的计算复杂度比 Convolution 低，所以数据重排的耗时对算子性能影响更大。经过稿纸推演和实际测试，我们确定了耗时最低的数据重排方案，即在输入数据是 NC4HW4 时，用 tbl 指令将与 3x3kernel 中 9 个数据分成 3 组，第一组和第二组分别有 4 个数据连续排列，最后一个数据单独排列。在 ARM V8 上的优化受限于指令种类数量，目前仅从减少数据读取时间角度优化算子性能。

#### **▐** **总结**

ARM 架构的升级和相应指令集的更新不断提高移动端各算子的性能上限，目前在 ARM V8.2 上对于 Convolution depthwise Int8 量化算子的性能已经接近最优，但受限于数据重排带来的额外负载，Int8 量化算子的性能仍然无法超越半浮点精度推理性能。

![](https://mmbiz.qpic.cn/mmbiz_png/33P2FdAnju8X1wEorjS3bDLnHiar4vtV5Mcf2mWYYibJt6RwM7zgbBS247KgYR9yVeZewdqR7qYwa7Rp0eCKm7JA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

团队介绍

大淘宝技术 Meta Team，负责面向消费场景的 3D/XR 基础技术建设和创新应用探索，通过技术和应用创新找到以手机及 XR 新设备为载体的消费购物 3D/XR 新体验。团队在端智能、商品三维重建、3D 引擎、XR 引擎等方面有深厚的技术积累。先后发布端侧推理引擎 MNN，端侧实时视觉算法库 PixelAI，商品三维重建工具 Object Drawer 等技术。团队在 OSDI、MLSys、CVPR、ICCV、NeurIPS、TPAMI 等顶级学术会议和期刊上发表多篇论文。