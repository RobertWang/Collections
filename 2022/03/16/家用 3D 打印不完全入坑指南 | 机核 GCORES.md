> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.gcores.com](https://www.gcores.com/articles/147983)

> 机核从 2010 年开始一直致力于分享游戏玩家的生活，以及深入探讨游戏相关的文化。

最近发现开了 3D 打印话题，突然开心，算算自己接触打印也有个 5 年多了，自己家里堆了四台打印机，故起意想汇总一些信息，说不定能帮到想要入坑此领域的朋友。当然，我自己也是野生自行研究的，所以以下信息仅供各位参考，如果有说的不对的，肯请大家帮我指出，定会完善修改。本文主要内容

*   家用 3D 打印最常见的两种原理及各自的特性
*   FDM（其中一种原理）的硬件部分极简基础知识

一、家用 3D 打印机常用原理简介
=================

1、熔融堆积（FDM）简单来说就是 “挤牙膏”。它使用的材料是一卷卷的塑料线，会有一个“挤出机” 把塑料线压进一个加热的喷头里面，然后跟随喷头的移动，融化的材料就会被挤到指定的位置。喷头就好像一只画笔一样，随着喷头移动，挤出机控制材料的挤出，就这样 “画” 出一层，然后调整喷头和底板的距离，再挤下一层，逐层堆积成为最后的 3D 模型。下图为 FDM 打印件的一些照片。可以看到如果不经过打磨的话层纹还是比较明显的，而且由于常用的喷头口径为 0.4mm，所以 FDM 对模型细节的 “分辨率” 并不高。[![](https://image.gcores.com/0ab04706-06f4-4d20-a8c8-2f54a3863445.jpg?x-oss-process=style/content_watermark)

这个蜘蛛侠的头大概只有 2.5cm 高，因为模型的纹理掩盖了层纹，所以第一眼看效果很惊艳。

](https://image.gcores.com/0ab04706-06f4-4d20-a8c8-2f54a3863445.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/c6a03216-ebeb-4867-94be-db802c85a495.jpg?x-oss-process=style/content_watermark)

请放大仔细看层纹，此为逐层成型的特有纹理

](https://image.gcores.com/c6a03216-ebeb-4867-94be-db802c85a495.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/6f9d173a-3e95-4e24-8dbd-006338e3baf1.jpg?x-oss-process=style/content_watermark)

本模型打印后经过重度砂纸打磨，故呈现出哑光无层纹的质感

](https://image.gcores.com/6f9d173a-3e95-4e24-8dbd-006338e3baf1.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/79a2073b-bcb7-4563-bbfb-b049b487a8aa.jpg?x-oss-process=style/content_watermark)

此模型层高设置较小，故层纹细密，光泽柔顺~

](https://image.gcores.com/79a2073b-bcb7-4563-bbfb-b049b487a8aa.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/26888ae9-5ea6-4843-9b7e-7cc9afe36f63.jpg?x-oss-process=style/content_watermark)

此机器成型尺寸较大，故稳定性比较差，可放大观瞧其表面的具体层纹效果

](https://image.gcores.com/26888ae9-5ea6-4843-9b7e-7cc9afe36f63.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/e7b5e055-5ad0-45ee-b44c-aee8f6688cf2.jpg?x-oss-process=style/content_watermark)

这是木制质感的 PLA 材料，据说掺了木粉，所以也常会有堵喷头的风险。

](https://image.gcores.com/e7b5e055-5ad0-45ee-b44c-aee8f6688cf2.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/0ab04706-06f4-4d20-a8c8-2f54a3863445.jpg?x-oss-process=style/content_watermark)

这个蜘蛛侠的头大概只有 2.5cm 高，因为模型的纹理掩盖了层纹，所以第一眼看效果很惊艳。

](https://image.gcores.com/0ab04706-06f4-4d20-a8c8-2f54a3863445.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/c6a03216-ebeb-4867-94be-db802c85a495.jpg?x-oss-process=style/content_watermark)

请放大仔细看层纹，此为逐层成型的特有纹理

](https://image.gcores.com/c6a03216-ebeb-4867-94be-db802c85a495.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/6f9d173a-3e95-4e24-8dbd-006338e3baf1.jpg?x-oss-process=style/content_watermark)

本模型打印后经过重度砂纸打磨，故呈现出哑光无层纹的质感

](https://image.gcores.com/6f9d173a-3e95-4e24-8dbd-006338e3baf1.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/79a2073b-bcb7-4563-bbfb-b049b487a8aa.jpg?x-oss-process=style/content_watermark)

此模型层高设置较小，故层纹细密，光泽柔顺~

](https://image.gcores.com/79a2073b-bcb7-4563-bbfb-b049b487a8aa.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/26888ae9-5ea6-4843-9b7e-7cc9afe36f63.jpg?x-oss-process=style/content_watermark)

此机器成型尺寸较大，故稳定性比较差，可放大观瞧其表面的具体层纹效果

](https://image.gcores.com/26888ae9-5ea6-4843-9b7e-7cc9afe36f63.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/e7b5e055-5ad0-45ee-b44c-aee8f6688cf2.jpg?x-oss-process=style/content_watermark)

这是木制质感的 PLA 材料，据说掺了木粉，所以也常会有堵喷头的风险。

](https://image.gcores.com/e7b5e055-5ad0-45ee-b44c-aee8f6688cf2.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/0ab04706-06f4-4d20-a8c8-2f54a3863445.jpg?x-oss-process=style/content_watermark)

这个蜘蛛侠的头大概只有 2.5cm 高，因为模型的纹理掩盖了层纹，所以第一眼看效果很惊艳。

](https://image.gcores.com/0ab04706-06f4-4d20-a8c8-2f54a3863445.jpg?x-oss-process=image/quality,q_90)2 / 62、光固化（LCD）光固化的材料用的是 “光敏树脂”，是一种照紫外线就会固化的液态材料。我们家用的树脂机器主要是 LCD 原理的。其思路也非常简单：一个底部透明的树脂池放在一片透明的 LCD 显示器上，然后再下面是一个大功率紫外线光源。如此我们就可以通过控制 LCD 显示的图案来决定哪些部分紫外线可以照到树脂，哪些地方照不到。开始打印的时候，一个可升降的平台 “怼” 在池底，于是照到的部分就会固化，粘到平台上，接下来平台提升，带着已经固化的树脂离开池底，让池子底部重新布满液态树脂，然后再下降，留下薄薄一层，再次固化，再次提升。如此往复。也是逐层堆垒，下图为光固化打印件的一些照片。可以看到，由于 LCD 的最小分辨率不是喷头直径，而是那一片 LCD 显示器的像素密度，所以它对模型细节纹理的表现力明显高于 FDM。[![](https://image.gcores.com/ef3230a3-8da6-4c0b-bcb8-b5a18bbaca77.jpg?x-oss-process=style/content_watermark)

请参照掌纹查看树脂打印的分辨率，这种细度的歪斜杆件在 FDM 中几乎不可能打出来。

](https://image.gcores.com/ef3230a3-8da6-4c0b-bcb8-b5a18bbaca77.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/e7956fed-66e5-4932-b61e-37a6b503376c.png?x-oss-process=style/content_watermark)

请看树脂打印的精细度（此图片来自网络非笔者打印）

](https://image.gcores.com/e7956fed-66e5-4932-b61e-37a6b503376c.png?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/b3cb2674-6db4-400c-bfd4-e37bc3e28bae.png?x-oss-process=style/content_watermark)

请看树脂打印的精细度（此图片来自网络，非笔者打印）

](https://image.gcores.com/b3cb2674-6db4-400c-bfd4-e37bc3e28bae.png?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/22229604-ffde-4291-b23d-722f544504fb.jpg?x-oss-process=style/content_watermark)

树脂的好处之一在于可以打印出透明的效果，但刚刚清洗之后依然是磨砂表面的，如果要透明则需要细致的打磨和上光油。

](https://image.gcores.com/22229604-ffde-4291-b23d-722f544504fb.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/ef3230a3-8da6-4c0b-bcb8-b5a18bbaca77.jpg?x-oss-process=style/content_watermark)

请参照掌纹查看树脂打印的分辨率，这种细度的歪斜杆件在 FDM 中几乎不可能打出来。

](https://image.gcores.com/ef3230a3-8da6-4c0b-bcb8-b5a18bbaca77.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/e7956fed-66e5-4932-b61e-37a6b503376c.png?x-oss-process=style/content_watermark)

请看树脂打印的精细度（此图片来自网络非笔者打印）

](https://image.gcores.com/e7956fed-66e5-4932-b61e-37a6b503376c.png?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/b3cb2674-6db4-400c-bfd4-e37bc3e28bae.png?x-oss-process=style/content_watermark)

请看树脂打印的精细度（此图片来自网络，非笔者打印）

](https://image.gcores.com/b3cb2674-6db4-400c-bfd4-e37bc3e28bae.png?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/22229604-ffde-4291-b23d-722f544504fb.jpg?x-oss-process=style/content_watermark)

树脂的好处之一在于可以打印出透明的效果，但刚刚清洗之后依然是磨砂表面的，如果要透明则需要细致的打磨和上光油。

](https://image.gcores.com/22229604-ffde-4291-b23d-722f544504fb.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/ef3230a3-8da6-4c0b-bcb8-b5a18bbaca77.jpg?x-oss-process=style/content_watermark)

请参照掌纹查看树脂打印的分辨率，这种细度的歪斜杆件在 FDM 中几乎不可能打出来。

](https://image.gcores.com/ef3230a3-8da6-4c0b-bcb8-b5a18bbaca77.jpg?x-oss-process=image/quality,q_90)3 / 4[![](https://image.gcores.com/2088bae0-e321-43a2-ae82-748073ff69d2.jpg?x-oss-process=image/resize,limit_1,m_lfit,w_1400/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)](https://image.gcores.com/2088bae0-e321-43a2-ae82-748073ff69d2.jpg?x-oss-process=image/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)FDM 推荐给这样的朋友：

*   希望较低成本体验 3D 打印；
*   不希望接触有机化学气体（但国外有研究称 FDM 的加热过程会产生小于 PM0.1 的极微小颗粒物，可经肺进入血液，故不论哪种打印都请一定做好通风。）；
*   需要打印件结实耐久；
*   有一定研究机械和数控的兴趣

LCD 推荐给这样的朋友：

*   打印的东西需要相当高的精度和细节表现力；
*   不在意打印件的机械性能；
*   有条件折腾挥发性有机化学液体（树脂有刺激性气味，可能含有甲醛等有机气体，且打印件需要工业酒精清洗，工业酒精里一般是有甲醇的）；
*   希望在维护调试机器方面尽量少投入精力。

二、FDM 的上手建议
===========

由于笔者对 FDM 了解比较多，接下来仅就个人的认知，为选择 FDM 入门的朋友提供一些基础信息。首先，请一定做好心理准备，因为 3D 打印其实并不是一个很舒适惬意的爱好。贸然入坑的很多小伙伴都有过（不止一次）被各种莫名其妙的问题折磨的心力交瘁的经历。因为 3D 打印的技术和办公室里的 A4 纸打印机相比起来，还远远不够成熟。打比方来说，就好像你要复印一张身份证，机器却需要你输入扫描灯的亮度，移动速度，然后打印纸张的时候墨粉的流量，硒鼓的温度，出纸的速度等等…… 这种感觉。所以，操作 3D 打印机的过程（尤其是 FDM）会涉及到非常多的具体参数的调整（比如层厚，壁层数，执行每种动作的时候喷头移动的速度，温度，同一层上不同部分的打印先后顺序，以及 “回抽”“补偿” 等等细小的动作的具体设置之类的）而且不止如此，这些参数往往互相之间还有所牵制联系，针对不同的打印材料又有些微不同，再加上 FDM 打印机的硬件机械部件比较多，各个部件之间的传动连接工作状态是否正常等等，所有这些因素都会对打印件最后的效果造成影响。所以弄一台打印机回家之后的折腾基本是免不了的，甚至一开始的时候遇到问题要判断出来是软件还是硬件的原因都很难，故上文提到了一个推荐给 “有一定研究机械和数控的兴趣” 的朋友。但作为半个过来人，我想说这一切 “受苦” 都是值得的~那么如果看到这里还想继续前进的话，接下来我们研究一下 FDM 机器的构架选择。FDM 的原理，说穿了只要是能在水平方向上，控制喷头和打印底板的相对位置，然后竖直方向上控制喷头和打印底板的距离，再加上一个可以控制打印耗材流量的 “挤出机”，就齐活了。但是到底怎么做到这几个相对位置的精确控制，人类想出好多种不同的构架，而且每种构架自己也都会不断的改良融合，显得相当复杂。但实际上基本可以（粗暴地）归为两种：

*   第一种：打印平台只在 Z 方向（高低）移动
*   第二种：打印平台只在 Y 方向（前后）移动。

[![](https://image.gcores.com/d96501f1-d883-4dea-9c39-d937cfffea59.gif)

第二种原理：平台前后动，喷头负责 X 轴运动和 Z 轴运动。

](https://image.gcores.com/d96501f1-d883-4dea-9c39-d937cfffea59.gif)[![](https://image.gcores.com/78573b93-c940-4364-8336-bf480438a5fa.gif)

第一种构架：喷头负责在 XY 平面移动，平台随着打印进度下降（Z 轴）。

](https://image.gcores.com/78573b93-c940-4364-8336-bf480438a5fa.gif)[![](https://image.gcores.com/d96501f1-d883-4dea-9c39-d937cfffea59.gif)

第二种原理：平台前后动，喷头负责 X 轴运动和 Z 轴运动。

](https://image.gcores.com/d96501f1-d883-4dea-9c39-d937cfffea59.gif)[![](https://image.gcores.com/78573b93-c940-4364-8336-bf480438a5fa.gif)

第一种构架：喷头负责在 XY 平面移动，平台随着打印进度下降（Z 轴）。

](https://image.gcores.com/78573b93-c940-4364-8336-bf480438a5fa.gif)[![](https://image.gcores.com/d96501f1-d883-4dea-9c39-d937cfffea59.gif)

第二种原理：平台前后动，喷头负责 X 轴运动和 Z 轴运动。

](https://image.gcores.com/d96501f1-d883-4dea-9c39-d937cfffea59.gif)2 / 2笔者自己是第一种架构入坑的，目前家中三台 FDM 有两台是第一种架构的，比较偏爱这种，原因如下：打印过程中，高度的提升速度是很缓慢的，但是 xy 平面上的动作非常频繁而且快速。一般来说，打印平台上有一块玻璃板，这导致平台的重量一般都不会很轻便。第二种构架中，负责拉动平台前后动作的传动皮带，在对抗较强大的惯性力时，可能会出现不稳定的抖动或动作延迟，具体反映在打印件上就是表面效果的一些类似波纹或是 “冲过头” 等等的小问题，如果想要避免这些小问题，就只有降低打印速度，以弱化惯性的影响。但降速之后也会同时带来漏料等等的其他问题。而如果像第一种构架，打印平台只负责 Z 方向（实则是非常缓慢地）的移动，这一问题就完全不存在了。但是第一种构架也有其问题，就是负责 xy 平面移动的那一套机械，往往比第二种要复杂一些，检修起来也会更麻烦一些。但换回的是又快又好的打印质量，个人认为还是相当值得的。那么构架推荐之后，就是一些 FDM 机器重要组成部件的解读：

*   打印平台：

就是负责承载第一层模型材料的那个平台。首先为了平整，往往会用到玻璃。其次为了粘的牢固，往往会在玻璃的底下放一个电热板（热床），因为通常 FDM 用的打印材料在 60 度左右就会有点软，粘性比较好，粘的牢固。然后再在玻璃的上面做一些 “晶格” 之类的粗糙涂层，进一步加固打印的首层。但此处笔者推荐的方案是” 弹簧钢板 + PEI 涂层 “。（选购时可以注意机型关于平添的描述，也可以买回来之后自己配）这一方式的好处在于，打印结束之后可以把弹簧钢板直接取下来，只要稍作弯曲，打印件就会自动从钢板上脱落，免去了传统不能弯曲的平台粘得太牢取不下来的烦恼。（相反，如果粘得不够牢，则会导致打印失败，因为打印上部的时候模型底部会移动导致 “错层”，或者更严重的后果 “炒粉”。）[![](https://image.gcores.com/7a9f15d4-7901-4f69-b932-4c838b012117.png?x-oss-process=image/resize,limit_1,m_lfit,w_1400/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)](https://image.gcores.com/7a9f15d4-7901-4f69-b932-4c838b012117.png?x-oss-process=image/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)某个打印失败现场其次，关于打印平台，“调平”是个绕不开的话题。因为我们需要保证喷头移动到平台的各个部位的时候，和平台平面和喷嘴之间的间隙都是几乎一样的，如此才能让打印的首层材料服服帖帖地均匀粘在平台上——如果太远，挤出的材料还没被 “涂抹” 到平台上就冷却了，粘不牢。如果太近，平台会把喷嘴堵住，导致没有料挤出来，甚至喷头还可能会划伤平台表面（或者反过来平台把喷嘴磨坏），所以调平成为了一个必修课。调平基本上都是通过平台上分布于四个脚落的旋钮实现的，一般开始的时候新手会使用 “A4 纸” 方法：依次让喷头来到平台的四个脚落，然后在平台和喷头之间塞一张 A4 纸，如此调整多轮，最终做到四个角点 “平台和喷头之间的空隙恰好能有点阻力地夹住 A4 纸的厚度，还不至于夹死不能动” 的微妙距离。但实际上，熟练的老手基本都是在打印第一层的同时，看着打印出来的线条形态实时调平的，也就 10 几秒的样子。做到这一点需要对首层线条的形态非常熟悉，其实不难，各位可以自行尝试~不过好在一般来说调平并非每次打印都需要，如果不去搬动或者拆装打印机部件，原则上是不需要重新调平的。而且现在比较新型号的机器也有在做自动调平了，对新手逐渐友好~

*   热块、喷嘴、喉管散热：

这部分的功能就是负责把挤出机送过来的材料通过喉管送到热块，加热到熔融，然后再从喷嘴挤出去。热块是一大块金属，上面插着电热棒和温敏电阻，喷嘴就拧在它上面。这里有一个非常有趣的事情，就是热块负责恒温到 180~220 度（取决于设置，稳定后正负波动不应超过 1 度）左右的高温，但紧贴着热块的上面，往往就是一个散热器，这个散热器还需要一个风冷风扇一直吹着——这个就是喉管的散热片和风扇。为啥要下面一边加热，上面一边散热呢？🤔[![](https://image.gcores.com/e5964d4e-f110-457d-a131-7514dceb89f3.png?x-oss-process=image/resize,limit_1,m_lfit,w_1400/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)](https://image.gcores.com/e5964d4e-f110-457d-a131-7514dceb89f3.png?x-oss-process=image/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)原因如上图：打印线材的直径是略小于喉管的，我们需要一个在短距离内剧烈变化的温度差，来让一些材料封住喷嘴附近的高温区域，以此保证喷嘴里面的熔融材料的压力只有喷嘴一个”泄压口 “。如果上半部分的散热不及时，熔融的材料就会从线材与喉管内壁之间的空隙向上” 反向挤出“，然后在上面凝固，裹住耗材，极大地增加线材挤出的压力——表现为类似堵头的症状。请记得这一关键原理，遇到无法解释的堵头类似症状时或许有用。

*   模型冷却风扇：

这个东西负责冷却从喷嘴挤出来的高温材料。原因是因为挤出来之后的材料越快冷却，它就会越快稳定在刚刚被挤出来的那个位置和形状——显然，刚被挤出来的时候它还是热的，具有流动性的。但不吹人家也会冷却的，所以这个风扇并非是一个必需品，但是它可以提升打印的品质。[![](https://image.gcores.com/6791e317-9c55-4c46-b3ba-2b463015e00f.png?x-oss-process=image/resize,limit_1,m_lfit,w_1400/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)](https://image.gcores.com/6791e317-9c55-4c46-b3ba-2b463015e00f.png?x-oss-process=image/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)图中左侧轴流风扇为喉管散热风扇，右侧离心风扇便是模型冷却风扇。（出风口对着喷嘴下方）但是它也会造成一些问题，比如上层过快地冷却会导致热胀冷缩效应的 “上紧下松”，有时候会导致上面的材料把下面的材料拉起来的“翘边” 情况。此时可以考虑在打印参数里面让模型冷却风扇开启的时间延后，比如到第 10 层或者 20 层才开。当然，翘边问题的成因很多，比如平台上有油脂导致底层粘不牢，或者热传温度过低粘得不牢等等，并非只有模型风扇一条。

*   挤出机：

由一个步进电机和一个齿轮组组成。负责把打印材料塑料线用齿轮 “咬” 住，然后通过减速齿轮用很大的力量推向喷头。FMD 的挤出机根据位置的不同大体分为 “近程” 和“远程”。[![](https://image.gcores.com/de9aecd3-2fb8-4e87-8678-07fb40525fa3.png?x-oss-process=image/resize,limit_1,m_lfit,w_1400/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)](https://image.gcores.com/de9aecd3-2fb8-4e87-8678-07fb40525fa3.png?x-oss-process=image/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)其中近程挤出机（上图）由于步进电机和挤出齿轮组都会跟着喷头在平面方向移动，故会增加其惯性，导致快速打印的时候有喷头转直角弯冲过头或者急停震动等等情况。但是近程挤出的优势在于挤出机的动作可以非常迅速地反应成喷头内部的压力，所以对喷头端挤出的动作控制得更加精确，也会有助于提升打印质量。另外像是 TPU 这种软性材料，就只能近程挤出机打了。[![](https://image.gcores.com/9a3e9e45-4796-4a04-b615-85add500f64d.png?x-oss-process=image/resize,limit_1,m_lfit,w_1400/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)](https://image.gcores.com/9a3e9e45-4796-4a04-b615-85add500f64d.png?x-oss-process=image/quality,q_90/watermark,image_d2F0ZXJtYXJrLnBuZw,g_se,x_10,y_10)远程挤出机（上图），则是在挤出机和喷头之间连一根特氟龙管，打印线材走内部（自行车闸线原理），如此喷头得以轻量化，惯性小，可以高速打印，但由于材料的弹性和特氟龙管内壁的容差，挤出机动作反映到喷头会有延迟，也就是会有点挤出动作和喷头的移动 “音画不同步” 的问题。（这一问题比较先进的主板会提供一个叫做 “挤出机线性系数” 的设置来尽可能矫正，也就是挤出机的一切动作都提前零点几秒做出。但是依然，单就这一点上，远程还是比不上近程挤出机。）另外，挤出机的设计基本都有一个可调整的 “按压弹簧”，这个弹簧的作用是控制把料夹住施力的那一对齿轮的夹力有多大。太松的话遇到一点点阻力料就会打滑，太紧的话对齿轮组轴承和步进电机都是一种消耗，每款挤出机都不一样，所以这边需要根据打印的经验去调整了。挤出机的步进电机基本没啥大讲究，但是齿轮组却有非常多种不同的设计。（但这一点如果不入坑到自己魔改打印机的程度，应该是涉及不到的。）这边仅作简要提及：个人觉得好的挤出机设计应该是，在发生堵头或者缠料等巨大阻力时，宁可把料刨断，挤出齿轮组也不打滑。因为料刨断了最多就是拆开挤出机清理一下碎屑，但基础齿轮组打滑的话会让齿轮本身磨损，久而久之就会越来越容易打滑。笔者最近在用的一个叫做 BMG 的挤出机，有一个主动轮和一个从动轮，两边都有齿，就有一些这种磨损的问题，需要定期更换齿轮。

*   步进电机的驱动：

步进电机的驱动是一个插在主板上的小片片（也有直接集成在主板上的），这一点在选购的时候建议留意一下产品参数。主要是驱动的细分数量。这个细分指的是将步进电机的基本步距角再细分成几个中间值，换句话说就是让它从大步前进变成小碎步倒腾。因为步进电机的驱动细分数量决定了它运作的时候噪音的大小。如果是 16 或者 32 细分，那么步进电机会有较大的噪音，笔者不是很推荐。目前静音驱动市面上比较多的是 128 或者 256 细分的，基本都能满足步进电机静音的需求。笔者目前用的型号是 TMC2209，它会把主板送过来的 16 细分信号用算法细化成 256 细分之后再发给步进电机，如此打印的时候主要的噪音来源基本上就只有风扇了。另外也有一些说法认为细分数量低会导致打印件上产生波纹，因为打印头是 “一格一格” 地移动的，实际上以笔者自身的经验来看，这种问题并不明显，推测是因为从步进电机到喷头经过的诸多层皮带等等的传动系统，已经把这种一格格的细微方波给抹平了。

*   机械动力系统：

这部分基本上都是由步进电机、传动皮带、丝杆、光轴这些机械部件组成的，其实大家仔细看下它运作的方式基本就能理解到，而且不同构架设计的打印机不尽相同，故不做展开介绍。不过值得一提的是 “限位” 这个东西。限位，是一个微动开关，一共有三个，分别在喷头的 xyz 轴处于零坐标的时候会被按下去。每次开始打印的时候，机器做的第一件事就是 “三轴归零”，比如 x 轴，它会一直往 x 的负方向移动，直到按下 x 轴的限位开关——此时打印机就知道 “啊这里是 x 的零坐标了”。YZ 同理，它保证了打印机在每次打印的时候都明确知道自己的喷头在构建范围内。所以，如果限位器坏了，就会发生在归零的时候喷头已经撞到机器壁了还在不断撞的情况，如果出现了类似问题，记得检查限位开关。（目前比较先进的型号限位开关已经从机械微动变成了激光式的。还有一些限位是直接做到了步进电机驱动里，通过步进电机的运行阻力来判断是否撞到极限位置，这个笔者个人觉得有点暴力且不稳定，不是很好）

*   主板：

那就是整个打印机的主控板啦，上面一般会有 tf 或者 sd 卡槽（或者 USB 口) 用来读取打印文件或者连接电脑。连接电脑后会有两种可能的打印模式，这个我们放在软件部分去聊吧~综上所述，插在主板上面的线大概有如下这么些：热块加热管，喷头温度传感器，喉管散热风扇，模型冷却风扇，四个步进电机（挤出机、X、Y、Z），三个限位器（X、Y、Z），热床加热板，热床温度传感器。还有可能会存在的比如：驱动散热风扇，自动调平信号感应器，LED 灯，WIFI 模块，断料检测模块，等等的非必需品。硬件部分介绍至此，笔者觉得掌握了如上信息之后，各位基本上应该对 FDM 的硬件构架内心有数了。当然，这些信息只能算是冰山一角，针对每种不同构架的打印机，其影响打印效果的点和容易出问的点也都不尽相同。更相信的还是得各位上手机器之后慢慢摸索了~ 当然有任何事情都可以发在我们的 3D 打印话题里，相信氛围也会越来越好的。如果后面有空的话笔者会尝试再聊聊软件部分。

一个建议
====

上手 FDM 最快的方法之一，就是搬个凳子坐在它边上看它打印。这是一个非常耗时的行为，但是当你看到软件里面设置参数时候自己的设想，在被打印机忠实地在真实的物理世界里执行出来之后，结果却和自己想象中的样子不太一致时，就会开始真正地掌握 FDM 打印了。最后放一个笔者曾帮朋友做的小物件，3D 打印还是有很多奇技淫巧的玩法的，后面希望也能写一篇专门介绍这种奇奇怪怪打印玩法的东西~ 哈哈哈哈[![](https://image.gcores.com/7104308b-2761-4468-af85-5d8ee665a7cb.jpg?x-oss-process=style/content_watermark)

水塔老陈醋（请往左滑）

](https://image.gcores.com/7104308b-2761-4468-af85-5d8ee665a7cb.jpg?x-oss-process=image/quality,q_90)[![](https://image.gcores.com/d47c8cfc-074d-400d-b469-a7a1043b0da6.jpg?x-oss-process=style/content_watermark)](https://image.gcores.com/d47c8cfc-074d-400d-b469-a7a1043b0da6.jpg?x-oss-process=image/quality,q_90)1 / 2肖像是某知名主播，故熟悉我的朋友请收八卦之魂哈哈哈哈以上，祝各位一切顺利。