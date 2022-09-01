> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7138027830512173069)*   思考每一个 commit 是否必须。一定要记住，不要多写一行多余的代码，否则未来你会很纠结；
*   学会归纳。合并同类项。

message 格式
==========

```
<type>(<scope>): <subject>
<空行>
<body>
<空行>
<footer>
复制代码

```

type
----

type 就是 commit 的类型：

*   docs: 文档变更
*   feat: 新需求特性
*   fix: 修复 bug
*   perf: 性能提升
*   style: 代码风格相关
*   test: 测试代码修改
*   refactor: 重构代码
*   chore: 构建过程或辅助工具的变动，日常的修改

scope
-----

scope 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。这个大家根据业务情况自行约定规范即可。

subject
-------

commit message 的核心部分，说明干了一件什么事，标题性质，简单的修改要做到让大家一看标题就能理解改了什么。

body
----

对本次 commit 的详细描述，可以分成多行，说明代码变动的动机，以及与以前行为的对比，详细的背景要在这里说清楚。比如

```
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. 

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
复制代码

```

footer
------

只有在 break change 或者关闭 issue 时使用，场景较少，日常业务开发可以忽略。事实上，能把 subject，body 写好就不容易了。

内容多了怎么办
=======

很多时候大家都直接用 `git commit -m "xxx"` 来代替，其实建议大家带上 body，很多 commit 光看 subject 跟没看是一样的。

这种情况下，大家直接用 `git commit` 来提交，跳出文本编辑器，支持大家输入 body。

一定记住，body 不是可有可无，一个优秀的 commit message 要能做到看了就知道前因后果，知道改动原因。而不是留一句片汤话在 subject 上。

语言选择
====

这一点很多团队会忽视，笔者待过的几乎每个团队，都是中英文混杂，这一点很不好。建议统一语言，如果公司规定开发必须用【英语】，ok 没问题，大家做好准备，提高英语素养，找到每个业务概念最精准的表达，形成规范，落地就行。

但是，但是！！！

绝大多数团队是做不到这一点的，因为没有动力，而且大家都是中国人，很多语言表达即便过了四六级，也很难很精准，于是写的五花八门。

与其这样，不如明确，除了 type，其他全用中文代替。