> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650834221&idx=4&sn=95cd0ed864fa5638fe9b6acba9bc37da&chksm=bd0110638a7699753b858b8f1dfc52f545de7472b1ebe6c3797b8b4a22b388da0029f1770020&scene=21#wechat_redirect)

来源 | 大迁世界 （ID：qq449245885）

已获得原公众号的授权转载

它简单、强大，而且是声明式的。我们可以很容易地实现复杂的事情，如暗黑 / 光明模式。然而，对它有很多误解和错误的使用。这些会把 CSS 标记变成复杂的不可读且不可扩展的代码。

我们如何才能防止这种情况的发生？通过遵循最佳实践，避免最常见的错误。在这篇文章中，我们将总结出 5 个最常见的错误以及如何避免它们。

### 1. 不预先设计

不经过思考，立马动手，这样可能会更快的完成任务，这也给了我们一种速度和成就感。但，从长远来看，这会有相反的效果。

在写代码之前，必须要先想清楚。我们将采取什么方式来设计组件？我们想以原子的方式建立我们的组件吗？我们是否愿意创建一个可组合的实用系统？我们想要一个已经内置的 UI 库吗？我们希望我们的 CSS 是全局作用域的还是按组件作用域的？

**有一个明确的目标将帮助我们选择最好的工具。这将使我们免于冗余和违反 DRY。** 有许多有效的方法来设计一个应用程序。最常见的无效的是即兴创作。

我们的代码必须是可预测的，易于扩展和维护。

看个例子：

```
/* ❌ 到处添加离散值 */
.card {
  color: #edb361;
  background-color: #274530;
  padding: 1rem;
}

/* ✅ 定义基于主题的属性 */
:root {
  --primary-bg-color: #274530;
  --accent-text-color: #edb361;
  --spacing-unit: 0.5;
}

.card {
  color: var(--accent-text-color);
  background-color: var(--primary-bg-color);
  padding: calc(var(--spacing-unit) * 2rem);
}
```

在上面的例子中，我们可以看到当使用 CSS 变量进行主题设计时，一切都变得可读和清晰。第一个 `.card` 定义看起来完全是随机的，这个组件不容易被扩展。

### 2. CSS Code Smells

> Code Smell 中文译名一般为 “代码异味”，或 “代码味道”，它是提示代码中某个地方存在错误的一个暗示，开发人员可以通过这种 smell（异味）在代码中追捕到问题。

Code smells 不是 bug。它们也不会妨碍系统的正常工作。它们只是一些不好的做法，会使我们的代码更难阅读和维护。

在这里，列举一些最常见的以及如何克服它们：

#### :: 符号

在伪元素和伪类中使用 `::` 符号是很常见的。这是旧的 CSS 规范的一部分，浏览器继续支持它作为一种回退。然而，我们应该在伪元素中使用 `::`，比如 `::before`, `::after`, `::frist-line`...，在伪类中使用`:`，比如`:link`, `:visited`, `:first-child`...

#### 使用字符串连接类

使用 Sass 预处理器来帮助处理我们的 CSS 代码库是非常流行的。有时在尝试 DRY 时，我们通过连接`&`操作符来创建类。

```
.card {
  border: 0.5 solid rem #fff;
  
  /* ❌ failed attempt to be dry */
  &-selected {
    border-color: #000;
  }
}
```

在开发人员试图在代码库中搜索`.card-selected`类之前，似乎没有什么问题。开发者将很难找到这个类。

#### 不正确地使用缩写

CSS 的简写非常好，可以让我们避免代码过于冗长。但是，有时我们并没有刻意地使用它们。大多数情况下，background 简写是偶然使用的。

```
/* ❌ 由于我们只是在设置一个属性，所以不需要使用简写。*/
.foo {
  background: #274530;
}

/* ✅ 使用正确的CSS属性 */
.foo {
  background-color: #274530;
}
```

#### !important  的错误使用

`!important` 规则用于覆盖特定性规则。它的使用主要集中在覆盖一个不能以任何其他方式覆盖的样式。

它通常用于更具体的选择器可以完成任务的场景。

```
<div class="inner">
  <p>This text is in the inner div.</p>
</div>


<style>
  .inner {
    color: blue;
  }
  
  /* ❌ 重写 color */
  .inner {
    color: orange !important;
  }
</style>


<style>
  .inner {
    color: blue;
  }
  
  /* ✅ 使用一个更具体的选择器规则，该规则将优先于更一般的规则。 */
  .inner p {
    color: orange;
  }
</style>
 
```

#### 强制使用属性值

在 CSS 代码库中出现一个神奇的数字是很常见的。它们带来了相当多的混乱。有时，我们可能会在代码中发现长的数字，因为开发者是为了覆盖一个他不确定的属性。

```
/* ❌ Brute 强制使这个元素位于z轴的最前面 */
.modal-confirm-dialog {
  z-index: 9999999;
}

/* ✅ 提前计划并定义所有可能的用例 */
.modal-confirm-dialog {
  z-index: var(--z-index-modal-type);
}
```

#### 3. 不对 CSS 类名进行作用域划分

由于 CSS 语言的特性，很容易出现元素在无意中被一个糟糕的类名定型的情况。这个问题非常频繁，所以有相当多的解决方案来解决这个问题。

在我看来，最好的两个是：

*   使用命名约定
    
*   CSS Modules
    

#### 命名约定

最流行的命名方式是 BEM 101。它代表了 `Block`、`Element`、`Modifier`方法。

```
[block]__[element]--[modifier]
/* Example */
.menu__link--blue {
  ...
}
```

其目的是通过让开发者了解 HTML 和 CSS 之间的关系来创建独特的名称。

#### CSS Modules

我对 BEM 方法最大的担心是，它很耗时，而且要依靠开发人员来实现。CSS 模块发生在预处理器一侧，这使得它没有错误。它为我们的 CSS 模块类名生成了随机的前缀 / 名称。

### 4. 使用 px 单位

像素的使用相当频繁，因为它起初看起来很容易和直观的使用。事实恰恰相反。很久以来，像素已经不再基于硬件了。它们只是基于一个光学参考单元。

`px`是一个绝对单位。这意味着什么呢？那就是我们不能适当地缩放以满足更多的人。

我们应该用什么来代替？相对单位是要走的路。我们可以依靠这些来更好地表达我们的动态布局。例如，我们可以使用`ch`来表达一个基于字符数的`div`宽度。

```
.article-column {
  /* ✅  我们的元素将最多容纳20个继承的字体大小的字符。 */
  max-width: 20ch;
}
```

通常情况下，`px`最常用的替换单位是`rem`和`em`。它们以一种从框到文本的相对方式来表示字体的相对大小。

*   `rem` 表示相对于根 `font-size` 的大小。
    
*   `em` 表示相对于元素大小的大小。
    

通过使用 `rem`，我们将能够根据用户偏好的字体大小来表达布局。

![](https://mmbiz.qpic.cn/mmbiz_png/LDPLltmNy55zpicFCLKicppmSXyVdicRZSaMzs2aMsB2baVWyy0mtrcs4DY5nxy9nSetl4Aicg8MCo5HTbD7XDQFwA/640?wx_fmt=png)

在上面的截图中，我们可以看到基于 `rem` 单元的布局如何能够扩展并适应不同的默认字体大小。  

### 5. 忽略浏览器支持

当开始开发一个网站时，定义我们的目标客户是至关重要的。跳过这一步，直接进行编码是很常见的。

为什么它至关重要? 它帮助我们了解我们的应用程序将在哪种设备上使用。之后，我们可以定义我们将支持哪些浏览器和哪些版本。

只要我们能提供适当的后备方案，我们仍然可以致力于接受像`subgrid`这样的后期功能。定义一个渐进的功能体验总是一个好主意。当一个特性得到更多的支持时，我们可以逐步抛弃它的后备方案。

像 caniuse.com 或 browserslist.dev 这样的工具在这方面很有帮助。像`postcss`这样的工具自带的`autoprefixer`功能将帮助我们的 CSS 得到更广泛的支持。

### 总结

我们已经看到了如何改进我们的 CSS 代码。遵循一些简单的指导原则，我们可以实现一个声明式、可重用和可读的代码库。我们应该在 CSS 中投入和在 Javascript 中一样多的精力。

作者：Jose Granja  译者：前端小智  来源：medium 原文：https://levelup.gitconnected.com/top-5-css-mistakes-to-avoid-963f76892954

**<END>**