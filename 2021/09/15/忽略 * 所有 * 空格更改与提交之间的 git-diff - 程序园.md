> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.voidcn.com](http://www.voidcn.com/article/p-zadqtnnn-btq.html)

> 我正在经历一个代码库并修复空白怪异, 并通常修正缩进等等, 我想确保我没有无意中进行任何其他更改, 所以我正在进行 git diff -w 显示所有更改中的差异文件, 同时忽略空白差异. 问题在于, 这并不是实际上忽略......

我正在经历一个代码库并修复空白怪异, 并通常修正缩进等等, 我想确保我没有无意中进行任何其他更改, 所以我正在进行 git diff -w 显示所有更改中的差异文件, 同时忽略空白差异. 问题在于, 这并不是实际上忽略了所有的空白差异 – 至少我认为只是空白的差异. 例如, 在 git diff -w 的以下输出中,

```
-"Links":
-{
-
-    "Thermal":
-
-{
-
+  "Links": {
+    "Thermal": {

```

你可以看到我只有

> 删除多余的空白行,  
> 把大括号放在他们打开的价值钥匙的末尾,  
> 缩进以适应上下文

This question 看起来可能首先提供答案, 但它处理两个特定文件之间的差异, 而不是两个特定的提交之间的差异. 搜索结果的其他一切也是一个死胡同. 例如, this question 是关于合并, 不显示差异, this question 涉及显示字级差异等等.

也许有一个更好的答案, 但是迄今为止我发现的最好的解决方案就是这样.

首先, 您必须控制 Git 目前使用的 “空格” 的定义. 创建或编辑项目中的. gitconfig, 以便包含

```
[core]
    whitespace = -trailing-space,-indent-with-non-tab,-tab-in-indent

```

接下来, 您必须控制所用单词的定义. 而不是仅仅使用 git diff -w, 添加–word-diff-regex = [^ [：space：]]：

```
git diff -w  --word-diff-regex=[^[:space:]]

```

你仍然会看到上下文 (在我的情况下, 因为我试图确保没有差异, 除了空格差异) 是没有帮助的. 你可以使用 - U0 来告诉 Git 给你 0 行的上下文, 像这样,

```
git diff -w -U0 --word-diff-regex=[^[:space:]]

```

但是你仍然会得到类似于上下文的输出, 但是仔细和手动查看所有更改还是要好一些, 以确保它们只是空格的改变.