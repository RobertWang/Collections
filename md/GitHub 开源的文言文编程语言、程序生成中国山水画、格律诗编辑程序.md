> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/371081217)

一位叫做 Lingdong 的大四学生在 [GitHub](https://link.zhihu.com/?target=https%3A//github.com/LingDong-) 上开源了一系列非常有意思的项目，其中文言文编程语言、程序生成中国山水画、格律诗编辑程序，吸人眼球，符合主旋律，弘扬传统文化。

![](https://pic2.zhimg.com/v2-967d9e668dfc876da8fbf690f3d706c5_b.jpg)

1、文言文编程语言
---------

文言文编程语言使编程不再是英文的专属了，当然目前这个也只是玩玩。

![](https://pic1.zhimg.com/v2-e27bf39d5517eed324de7d2b41fd80a8_b.jpg)

举个例子，怎么定义一个变量：

```
吾有一數。曰三。名之曰「甲」。var a = 3;
有數五十。名之曰「大衍」。                   var dayan = 50;
昔之「甲」者。今「大衍」是也。a = dayan;
吾有一言。曰「「噫吁戲」」。名之曰「乙」。var b = "alas!";
吾有一爻。曰陰。名之曰「丙」。var c = false;
吾有一列。名之曰「丁」。var d = [];
吾有三數。曰一。曰三。曰五。名之曰「甲」曰「乙」曰「丙」。var a=1,b=3,c=5;
```

怎么定义一些函数运算呢，我们的 if for while 语句，他也能用文言文翻译出来 ：

```
若三大於二者。乃得「「想當然耳」」也。if (3>2){ return "of course"; }
若三不大於五者。乃得「「想當然耳」」。若非。乃得「「怪哉」」也。if(3<=5){return "of course"}else{return "no way"}
為是百遍。⋯⋯ 云云。for (var i = 0; i < 100; i++){ ... }
恆為是。⋯⋯ 云云。while (true) { ... }
凡「天地」中之「人」。⋯⋯ 云云。for (var human of world){ ... }
乃止。break;
```

作者提供了一个在线的 IDE，目前支持 Python 以及 JavaScript。

![](https://pic4.zhimg.com/v2-72b9d99be572cd8339521ce0f30f08fb_b.jpg)

​GitHub 网址：[https://github.com/LingDong-/wenyan-lang](https://link.zhihu.com/?target=https%3A//github.com/LingDong-/wenyan-lang)

在线体验：[http://wenyan-lang.lingdong.works/ide.html](https://link.zhihu.com/?target=http%3A//wenyan-lang.lingdong.works/ide.html)

2、程序生成中国山水画
-----------

基于 javascript 编写，程序无限生成的中国山水画，使用噪音和数学函数从头开始建模山峰和树木，并输出可缩放矢量图形（SVG）格式。

![](https://pic3.zhimg.com/v2-5c152a4a9e578a615df03c503b9aa436_b.jpg)

GitHub 网址：[https://github.com/LingDong-/shan-shui-inf](https://link.zhihu.com/?target=https%3A//github.com/LingDong-/shan-shui-inf)

在线体验：[http://shan-shui-inf.lingdong.works/](https://link.zhihu.com/?target=http%3A//shan-shui-inf.lingdong.works/)

3、格律诗编辑程序
---------

一个帮助创作，编辑，管理格律诗作品的工具，可自动识别音调模式并检测错误。自动检查平仄等格律规则，使用数据分析提供对用户诗歌的见解，机器学习分析用户作品并与《全唐诗》唐人诗句比对。

![](https://pic3.zhimg.com/v2-c19b69b694cfff60cd569864321450fa_b.jpg)

GitHub 网址：[https://github.com/LingDong-/cope](https://link.zhihu.com/?target=https%3A//github.com/LingDong-/cope)