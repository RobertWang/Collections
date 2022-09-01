> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7033384820755398670)*   fluent 方法，一行代码搞定一切
    
*   高度自定义，允许用户定义自己的实现
    
*   词库自定义，适应各种应用场景
    
*   丰富的实现策略
    

默认实现了基于 四角编码 + 拼音 + 汉字结构 + 汉字偏旁 + 笔画数 的相似度比较。

变更日志
====

> [变更日志](https://link.juejin.cn?target=CHANGELOG.md "CHANGELOG.md")

快速开始
====

需要
--

jdk1.7+

maven 3.x+

maven 引入
--------

```
<dependency>
    <groupId>com.github.houbb</groupId>
    <artifactId>nlp-hanzi-similar</artifactId>
    <version>1.0.0</version>
</dependency>
复制代码

```

快速开始
----

### 基本用法

`HanziSimilarHelper.similar` 获取两个汉字的相似度。

```
double rate1 = HanziSimilarHelper.similar('末', '未');
复制代码

```

结果为：

```
0.9629629629629629
复制代码

```

### 自定义权重

默认是根据 四角编码 + 拼音 + 汉字结构 + 汉字偏旁 + 笔画数 进行相似度比较。

如果默认的系统权重无法满足你的需求，你可以通过自定义权重调整：

```
double rate = HanziSimilarBs.newInstance()
                .jiegouRate(10)
                .sijiaoRate(8)
                .bushouRate(6)
                .bihuashuRate(2)
                .pinyinRate(1)
                .similar('末', '未');
复制代码

```

### 自定义相似度

有些情况下，系统的计算是无法满足的。

用户可以在根目录下 `hanzi_similar_define.txt` 进行自定义。

```
入人 0.9
人入 0.9
复制代码

```

这样在计算 `人` 和 `入` 的相似度时，会优先以用户自定义的为准。

```
double rate = HanziSimilarHelper.similar('人', '入');
复制代码

```

此时的结果为用户自定义的值。

引导类
===

说明
--

为了便于用户自定义，`HanziSimilarBs` 支持用户进行自定义配。

HanziSimilarBs 中允许自定义的配置列表如下：

<table><thead><tr><th align="left">序号</th><th align="left">属性</th><th align="left">说明</th></tr></thead><tbody><tr><td align="left">1</td><td align="left">bihuashuRate</td><td align="left">笔画数权重</td></tr><tr><td align="left">2</td><td align="left">bihuashuData</td><td align="left">笔画数数据</td></tr><tr><td align="left">3</td><td align="left">bihuashuSimilar</td><td align="left">笔画数相似度策略</td></tr><tr><td align="left">4</td><td align="left">jiegouRate</td><td align="left">结构权重</td></tr><tr><td align="left">5</td><td align="left">jiegouData</td><td align="left">结构数据</td></tr><tr><td align="left">6</td><td align="left">jiegouSimilar</td><td align="left">结构相似度策略</td></tr><tr><td align="left">7</td><td align="left">bushouRate</td><td align="left">部首权重</td></tr><tr><td align="left">8</td><td align="left">bushouData</td><td align="left">部首数据</td></tr><tr><td align="left">9</td><td align="left">bushouSimilar</td><td align="left">部首相似度策略</td></tr><tr><td align="left">10</td><td align="left">sijiaoRate</td><td align="left">四角编码权重</td></tr><tr><td align="left">12</td><td align="left">sijiaoData</td><td align="left">四角编码数据</td></tr><tr><td align="left">13</td><td align="left">sijiaoSimilar</td><td align="left">四角编码相似度策略</td></tr><tr><td align="left">14</td><td align="left">pinyinRate</td><td align="left">拼音权重</td></tr><tr><td align="left">15</td><td align="left">pinyinData</td><td align="left">拼音数据</td></tr><tr><td align="left">16</td><td align="left">pinyinSimilar</td><td align="left">拼音相似度策略</td></tr><tr><td align="left">17</td><td align="left">hanziSimilar</td><td align="left">汉字相似度核心策略</td></tr><tr><td align="left">18</td><td align="left">userDefineData</td><td align="left">用户自定义数据</td></tr></tbody></table>

所有的配置都可以基于接口，用户进行自定义。

快速体验
====

说明
--

如果 java 语言不是你的主要开发语言，你可以通过下面的 exe 文件快速体验一下。

下载地址
----

> [github.com/houbb/nlp-h…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhoubb%2Fnlp-hanzi-similar%2Freleases%2Fdownload%2Fexe%2Fhanzi-similar.zip "https://github.com/houbb/nlp-hanzi-similar/releases/download/exe/hanzi-similar.zip")

下载后直接解压得到 `hanzi-similar.exe` 免安装的可执行文件。

执行效果
----

界面是使用 java swing 实现的，所以美观什么的，已经完全放弃治疗 T_T。

使用 exe4j 打包。

字符一输入一个汉字，字符二输入另一个汉字，点击计算，则可以获取对应的相似度。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/611bb2a8a8a54c84a4516062814f61ff~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

字典的弊端
=====

这个项目开源，是因为有一位小伙伴有相关的需求，但是他不懂 java。

一开始想把项目设计成为字典的形式，两个字对应一个相似度。

但是有一个问题，2W 汉字，和 2W 汉字的相似度字典，数量已经是近亿的数据量。

空间复杂度过高，同时会导致时间复杂度问题。

所以目前采用的是实时计算，有时间做一下其他语言的迁移 :)

实现原理
====

实现思路
----

不同于文本相似度，汉字相似度的单位是汉字。

所以相似度是对于汉字的拆解，比如笔画，拼音，部首，结构等。

推荐阅读：

> [NLP 中文形近字相似度计算思路](https://link.juejin.cn?target=https%3A%2F%2Fhoubb.github.io%2F2020%2F01%2F20%2Fnlp-chinese-similar-char "https://houbb.github.io/2020/01/20/nlp-chinese-similar-char")

计算思路描述了实现的原理，但是小伙伴反应不会实现，于是才有了本项目。

核心代码
----

核心实现如下，就是各种相似度，进行加权计算。

```
/**
 * 相似度
 *
 * @param context 上下文
 * @return 结果
 * @since 1.0.0
 */
@Override
public double similar(final IHanziSimilarContext context) {
    final String charOne = context.charOne();
    final String charTwo = context.charTwo();

    //1. 是否相同
    if(charOne.equals(charTwo)) {
        return 1.0;
    }

    //2. 是否用户自定义
    Map<String, Double> defineMap = context.userDefineData().dataMap();
    String defineKey = charOne+charTwo;
    if(defineMap.containsKey(defineKey)) {
        return defineMap.get(defineKey);
    }

    //3. 通过权重计算获取
    //3.1 四角编码
    IHanziSimilar sijiaoSimilar = context.sijiaoSimilar();
    double sijiaoScore = sijiaoSimilar.similar(context);

    //3.2 结构
    IHanziSimilar jiegouSimilar = context.jiegouSimilar();
    double jiegouScore = jiegouSimilar.similar(context);

    //3.3 部首
    IHanziSimilar bushouSimilar = context.bushouSimilar();
    double bushouScore = bushouSimilar.similar(context);

    //3.4 笔画
    IHanziSimilar biahuashuSimilar = context.bihuashuSimilar();
    double bihuashuScore = biahuashuSimilar.similar(context);

    //3.5 拼音
    IHanziSimilar pinyinSimilar = context.pinyinSimilar();
    double pinyinScore = pinyinSimilar.similar(context);

    //4. 计算总分
    double totalScore = sijiaoScore + jiegouScore + bushouScore + bihuashuScore + pinyinScore;
    //4.1 避免浮点数比较问题
    if(totalScore <= 0) {
        return 0;
    }

    //4.2 正则化
    double limitScore = context.sijiaoRate() + context.jiegouRate()
            + context.bushouRate() + context.bihuashuRate() + context.pinyinRate();

    return totalScore / limitScore;
}
复制代码

```

具体的细节，如果感兴趣，可以自行阅读源码。

开源地址
----

为了便于大家的学习和使用，本项目已开源。

开源地址：

> [github.com/houbb/nlp-h…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhoubb%2Fnlp-hanzi-similar "https://github.com/houbb/nlp-hanzi-similar")

欢迎大家，fork&star 鼓励一下老马~

算法的优缺点
======

优点
--

为数不多的几篇 paper 是从汉字的结构入手的。

本算法引入了四角编码 + 结构 + 部首 + 笔画 + 拼音的方式，使其更加符合国内的使用直觉。

缺点
--

部首这部分因为当时数据问题，实际上是有缺憾的。

后续准备引入拆字字典，对汉字的所有组成部分进行对比，而不是目前一个简单的部首。

后期 Road-MAP
===========

*    丰富相似度策略
    
*    优化默认权重
    
*    优化 exe 界面
    

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b3a4ec163d44e55a433298b14309d86~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)