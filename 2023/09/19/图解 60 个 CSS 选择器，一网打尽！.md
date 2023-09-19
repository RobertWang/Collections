> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU2MTIyNDUwMA==&mid=2247523197&idx=1&sn=89106526ccbf71ea44114630c6433884&chksm=fc7ed526cb095c30637b91c7c2a0da5ac0ca9a9ff3dbb2154e529db51c2117185d5576658e5a&scene=21#wechat_redirect)

CSS 选择器用于选择 HTML 元素并将样式应用于它们。使用这些选择器，可以定义特定条件下应用哪些样式。除了普通的选择器外，还有伪类和伪元素，用于选择具有特定状态或特定部分的元素，并将样式应用于它们。本文将通过图文并茂的方式展示这些选择器的使用方法！

选择器
---

通用选择器使用 `*` 来选择全部元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYy9YjhxgiakWMtVC21vtxAukNC2nRB8o4WoXibDgcnWEZyWnB5BK9uT9A/640?wx_fmt=png)

### 元素选择器

同于选择指定 HTML 元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYZiaEa6XZmWM80FxXIqXd4ObMKDibqibIgOPJ0E8m3Vf8AFrPb7w4Rn8lg/640?wx_fmt=png)

### 类选择器

选择具有指定类名的所有元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYUjNV3GazNmaQMooARic8geH8QwSHfbOXFHAFiaI759CMq0DpSbFQweuQ/640?wx_fmt=png)

### ID 选择器

选择具有指定 ID 的元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcY2UicL00RlaMCC8RYUPHicMwiahNVyAKY7TooRjaI4JBwffDDRxHmlsmoQ/640?wx_fmt=png)

### 组合选择器

连接两个或多个类名或 ID，以选择具有所有指定类名 / ID 的元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYB1cpuiaiaISKlDSt4rne83ialCyuibv6Pe98lOtj4R8rGuHewTZyQdmiaUQ/640?wx_fmt=png)

### 多重选择器

使用逗号将多个选择器声明分开，这样可以很容易地将相同的样式应用于多个选择器声明。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYuUyTNAeeB9KLeqHZns6icjIuBo8U0ViaLcjj9KUMSu8UqtW8T7pINdVA/640?wx_fmt=png)

### 后代选择器

后代选择器表示选择某个元素的所有后代元素，即位于该元素内部的所有子孙元素。在使用后代选择器时，在两个选择器之间加上一个空格，表示前一个选择器所选中的元素中包含的后代元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYHDMJxM5AeOHokzKhYS4Sdc6vdwbX9geHiacotIiapZhXstwCZ4ckIWHg/640?wx_fmt=png)

### 相邻选择器

相邻选择器用于选择紧接在另一个元素后面的直接相邻兄弟元素的选择器，使用加号（+）作为组合符号，将两个选择器连接起来。它选择的是位于第一个选择器后紧邻的同级元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYicVFeGxv8f6q44FXv2riaHVncOV2EsSTnqwIM1mHzrNlaZCDdtKoicTLg/640?wx_fmt=png)

### 子选择器

子选择器用于选择某个元素的直接子元素，使用大于号（>）作为组合符号，将两个选择器连接起来。它选择的是父级元素下的直接子元素，即元素树结构中的一级关系。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYXibkoEhGHiaicK7NDajBIdzicPDhJyPt8OFozn3ChgianlK0JgDKF03dTlA/640?wx_fmt=png)

### 通用兄弟选择器

通用兄弟选择器使用波浪号（即通用兄弟组合符）来选择在第一个选择器之前的所有元素，而不要求它们是第一个选择器的直接兄弟元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcY4ugNuX8n1C5QXZBJrmG6yV5yIuXMuibQRc2wAia9Fol4BJ2VJlLHFdjA/640?wx_fmt=png)

### 猫头鹰选择器

猫头鹰选择器用于选择所有具有前置兄弟元素的元素，用来在一个容器内的元素之间添加间距，但不包括第一个元素，因为第一个元素没有前置兄弟元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYGcZzExmP100Ub9H4z5LaEslmEsTddf5csyiayox2hKy5Xq42fhbAIwA/640?wx_fmt=png)

### 属性选择器

*   使用`[attribute]`来选择所有具有 `attribute` 属性的元素：
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYO8JRDRPy6ZibVrdH3EqqOicyCcUKiaW45en1KWfgicoTnRFSSjfBFlibSQQ/640?wx_fmt=png)

*   使用 `[attribute=value]` 来选择具有指定属性及属性值的元素：
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcY516OWo3JOySt0lv2V9MJg4bn8r3lrykYLXMHmJd8YwJMibicw3r7r9dQ/640?wx_fmt=png)

*   使用 `[attribute~=value]` 来选择具有指定属性，并且该属性的多个值中**包含给定值**的元素：
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcY1jf0hWP1nFWoJDH3SpNf1JMZDgic97v8bZiakkick1ndzNjKBfu8xeR3Q/640?wx_fmt=png)

*   使用 `[attribute*=value]` 来选择具有指定属性，并且该属性的值中**包含特定部分值**的元素。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYoE7PuWNL8LSxzaMQb93eeLWcA2FLwib6qIoD0QjbnhiaS8xdUzl5Uykw/640?wx_fmt=png)

*   使用 `[attribute^=value`] 来选择具有指定属性，并且该属性的值**以给定值开头**的元素。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYVrtfDHP3kHJDLfd9dibk5R56nJsZIU8v5m1gJvTafticVBqRpWk2qStg/640?wx_fmt=png)

*   使用 `[attribute$=value]` 来选择具有指定属性，并且该属性的值**以给定值结尾**的元素。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYRHCd4iaMsxtGH6xVeRyd0YLLNAaXTNOX1vygSJvuHSOoWxqRicdqlO5A/640?wx_fmt=png)

伪类
--

### 链接伪类选择器

*   `:link`：选取未被访问过的链接，用于为用户尚未点击的超链接设置样式。
    
*   `:visited`：选取已经被用户访问过的链接，用于为用户之前点击过的超链接设置样式。
    
*   `:hover`：当用户指针（例如鼠标光标）悬停在某些元素（通常是链接）上时，选取这些元素。
    
*   `:active`：在元素（通常是链接或按钮）被激活时，例如用户点击它们的瞬间，选取这些元素。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYpfEWNw6vL4icpSZordELalHNicicvayd6PicCbMrIZk2Tib3Ary3O9NLVRg/640?wx_fmt=png)

### :focus 伪类

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYoBVmiaJnGgH6BbknmaKT6CSHsyhJWNzKlY3C84A52fxhPXib2CIdALvA/640?wx_fmt=png)

### :checked 伪类

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYuZyeLu0P5Hvc9aP5BcouWpE8a6TPsHJNCweibjicia74CUkdHIiaThA9fA/640?wx_fmt=png)

### :disabled 伪类

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYNg0FicX7T5pSibQIdicVzvfzQtI40v3BSHHXGr2WjaxgRUydKibWKjibITQ/640?wx_fmt=png)

### :enabled 伪类

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYoCeB4t55x1IYUyHTETkrL4Via7mbfhzyQDUWZvmZE0eNGjuEicLvFYug/640?wx_fmt=png)

### :valid 伪类

`:valid` 伪类用于选择具有与其属性（如 `pattern`、`type` 等）所指定要求相匹配的内容的输入元素。

当 `input` 元素的内容符合其属性所指定的要求时，可以使用 `:valid` 伪类选择它们。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYibr8OPd8Cm4OdSTCBuB1LibWwOKfprpjl3lVF7WkrTHnBbde0kHfyuxA/640?wx_fmt=png)

### :invalid 伪类

`:invalid` 伪类用于选择具有内容不符合要求的输入元素。

当`input`元素的内容不符合其要求时，可以使用 `:invalid` 伪类来选择它们。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYOqWDnUzia6S9K5Z0ibUmfKZNupHhZZqh8Q48Qd5FT91lQYPtueeeYYkg/640?wx_fmt=png)

### :required 伪类

`:required` 伪类用于选择具有 `required` 属性的输入元素，该属性表示在提交表单之前必须填写它们。

当 `input` 元素具有 `required` 属性时，可以使用 `:required` 伪类选择它们。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYibiaEJ5QmlocyBDuQGECFGPG5qvZD5oSBzZ9iafbotVuYopvxaicz2U8OQ/640?wx_fmt=png)

### :optional 伪类

`:optional` 伪类用于选择没有 `required` 属性的输入元素，这意味着它们不是必填项。

当`input` 元素没有 required 属性时，可以使用 `:optional` 伪类选择它们。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYUMKpDvdQvOf2ZVMvsbJrFPRSyKxRkhYLxSeMCzsibSbBP9DSEt3xmQA/640?wx_fmt=png)

### :first-child 伪类

`:first-child` 伪类用于选择父元素中的第一个子元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYmR2V6anJ0KmHPlMyjrME3AMMRJMWCajK1dMGCXRblYvndYSicYibRdpA/640?wx_fmt=png)

### :last-child 伪类

`:last-child` 伪类用于选择父元素中的最后一个子元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYElvpnsq5K4pvc5Dar6CnDZ9ibU8nR9y3ibcj6DhVIicRX804pXHKAibz7g/640?wx_fmt=png)

### :nth-child 伪类

`:nth-child` 伪类根据元素在父元素中的位置进行选择，允许进行各种选择。`:nth-child` 还可以自定义模式选择元素：

*   `:nth-child(odd)` 或 `:nth-child(2n+1)` 选择每个奇数位置的子元素
    
*   `:nth-child(even)` 或 `:nth-child(2n)` 选择每个偶数位置的子元素
    

公式中的 `n` 就像一个计数器，可以按重复周期选择元素。例如，`:nth-child(3n)` 将选择每三个元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcY18naMib2F5VxnIKQPAdbDO9apictJjn0sic3lrVaHcQria4vp9bib97dlHg/640?wx_fmt=png)

### :nth-last-child 伪类

`:nth-last-child` 伪类与 `:nth-child` 类似，但是从最后一个子元素向后计数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYxUhmT9s3Ld4stdw2YeUrHC2iaJib6NmF0zHK0T33rOchhic8c9GfyicKfQ/640?wx_fmt=png)

### :only-child 伪类

当需要选择在其父级元素中唯一的一个子元素时，可以使用 `:only-child` 伪类。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYGQQ5NxhD8lOQC0daIFgTyLMDRJyC5hJuhtYZG5wic1KyMrZxyoa01ibw/640?wx_fmt=png)

### :first-of-type 伪类

`:first-of-type` 伪类选择在其父元素中的特定类型的元素中的第一个元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYrTs4JSbvmSMPljo5thFZJL4kytrpaa0fWmmB87fWcUOvnR0ZU9tHzA/640?wx_fmt=png)

### :last-of-type 伪类

`:last-of-type` 伪类选择在其父元素中的特定类型的元素中的最后一个元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYkUATMhK3gULDRwGyibWTzafRSl48KiahBh8bbib8fSThR1OiaXJDFDmm8g/640?wx_fmt=png)

### :nth-of-type 伪类

当需要根据元素在兄弟元素中的类型和位置选择元素时，可以使用 `:nth-of-type` 伪类。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYYve7d041OG2RnllLcziaXeKFF516HY4iaW7HDwFvwqaO4olJNkaITy4g/640?wx_fmt=png)

### :nth-last-of-type 伪类

`:`当需要根据元素在兄弟元素中的类型和位置选择元素，并且从末尾开始计数时，可以使用 `:nth-last-of-type` 伪类。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcY7hEic6mHYibT4ibPCVsqj6W0D7icKKhYiaeUW3S8HzHjMopkm3HU3OcgZpQ/640?wx_fmt=png)

### :only-of-type 伪类

当需要选择在其兄弟元素中为特定类型的元素仅有一个的元素时，可以使用 `:only-of-type` 伪类。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYO2q14MlLQkGFGjHgSvSdmdZFAb2mcIJzDPUM2nTxP3ISfGZiaCQXJicg/640?wx_fmt=png)

### :target 伪类

`:target` 用于选择具有与 URL 片段匹配的 ID 属性的元素。URL 片段是指 URL 中的 # 符号后面的部分。当从页面中的链接点击跳转到带有特定片段的 URL 时，`:target` 伪类将会被应用于与片段匹配的对应元素上。这样可以利用 `:target` 来为被直接链接到的页面部分设置不同的样式，从而提供视觉上的反馈和突出显示。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYDu9WXvLHTo1ELCIagrd7YXKwSbmGDpGCaxrNywaPx0NS0pM2E6SYmw/640?wx_fmt=png)

### :not() 伪类

使用 `:not()` 伪类可以选择不符合指定选择器或条件的元素。这在需要排除某些特定元素时非常有用。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYICeWyibBiax3uVIrAyevuvrexSV2EpduHLFlTxN2RN9QibNOQnUUibsm5g/640?wx_fmt=png)

### has() 伪类

使用 `:has()` 伪类可以选择包含某个特定元素或选择器的父级元素，并为其应用样式。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYMC9YMoxBcKP8VT94zsOgXvz6KZG1kC8CgibumChu4BicrYQY17aCEsvA/640?wx_fmt=png)

### 其他伪类

除了上面提到的伪类，CSS 中还包含以下伪类：

*   `:root`：选择文档中最高级别的父元素，通常是 HTML 文档中的 `<html>` 元素。可用于定义对所有页面元素可用的 CSS 变量。
    
*   `:is()`：匹配可以是多个选择器之一的元素，使得长的选择器列表更加简短和易读。例如，`:is(h1, h2, h3)` 会匹配这三个标题元素之一。
    
*   `:where()`：与 `:is()` 类似，但允许根据条件选择元素，而不影响选择器的特异性。
    
*   `:default`：匹配设置为默认选择状态的用户界面元素（如单选按钮或复选框）。
    
*   `:empty`：选择没有子元素（包括文本节点）的元素。
    
*   `:fullscreen`：选择当前以全屏模式显示的元素。
    
*   `:in-range`：匹配值在指定范围内的表单元素（使用 `min` 和 `max` 属性指定范围）。
    
*   `:out-of-range`：匹配值在指定范围之外的表单元素。
    
*   `:indeterminate`：选择状态不确定的表单元素，例如既未选中也未取消选中的复选框（在树状结构中经常见到）。
    
*   `:read-only`：匹配由于 readonly 属性而不可编辑的表单元素。
    
*   `:read-write`：选择可由用户编辑的表单元素，意味着它们不是只读的。
    
*   `:lang()`：根据元素的语言属性进行匹配。例如，`:lang(en)` 选择使用英语定义的元素。
    

伪元素
---

### ::before 伪元素

`::before` 伪元素用于在元素内容之前插入内容。它可以用于添加装饰性内容、图标或其他不需要出现在实际 DOM 中的元素。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYaWLPlme8tVeMiahJ7fKhgiaK6QfYNvmOf3GwiczaZkufeLnuHryTRWqQg/640?wx_fmt=png)

### ::after 伪元素

`::after` 伪元素与 `::before` 伪元素类似，用于在元素的内容之后插入内容。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYic2ZNiaN5u2PNvTG7gjkicRvaUOfUoHw6YqJfpeK3JnDUKfz1wNvNYQzw/640?wx_fmt=png)

### ::first-letter 伪元素

`::first-letter` 伪元素用于选择并修改块级元素的第一个字母，从而应用特定的样式。这个伪元素只能选择每个块级元素的第一个字母，并且仅在有文本内容的时候生效。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYTbIJUMlAD7ibbo01HuTEoAgjruRmuJBiapbCl4ggibnCwyfDeVom3xXAg/640?wx_fmt=png)

### ::first-line 伪元素

`::first-line` 伪元素用于选择并修改块级元素的第一行，从而应用特定的样式。这个伪元素只能选择每个块级元素的第一行，并且仅在有文本内容的时候生效。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYibL6Zfytb0u2frWhSxrAMEam1BIjQoz8MgiaMwoSdXyBiaqtW3Ojvfvpg/640?wx_fmt=png)

### ::placeholder 伪元素

`::placeholder` 伪元素用于选择并修改表单字段的占位符文本，从而应用特定的样式。占位符文本是在用户未输入任何内容时显示的默认文本。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcY6PxGVtxtXFJsYTibYv4SNK5aHtM3gUrnBibX1ibib1UI046PaNiaQm0es9Q/640?wx_fmt=png)

### ::selection 伪元素

`::selection` 伪元素用于选择并修改用户所选文本的样式。可以应用于包含文本内容的任何元素，如段落、标题、按钮等。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYiaA4ekHUQZRKicJXnEcCoTzheKUOwl9eaiaEQ3T2vGibXUBzMeJJLpFlfA/640?wx_fmt=png)

### ::marker 伪元素

`::marker` 伪元素用于为列表项中的标记符设置样式，这些标记符通常包含无序列表的项目符号或有序列表的数字 / 字母。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/EO58xpw5UMMCVuhQIYyPiae8U3gUyAmcYR9nk2a2lqeRc0URIIAJKHUgsmNKpWoEgUibQmVQgSM1iciccN2nLQtPPg/640?wx_fmt=png)

### 其他伪元素

除了上述伪元素之外，CSS 还提供了以下伪元素：

*   `::file-selector-button`：用于选择文件的按钮元素的伪元素。它可以用来样式化文件上传控件中的选择按钮。
    
*   `::cue`：用于样式化媒体元素（如音频或视频）中的字幕或注释的伪元素。可以用来设置字幕的样式，比如字体、颜色等。
    
*   `::part()`：用于自定义 Web 组件的内部部件的伪元素。它可以针对组件的具体部分应用样式，并通过组件的 shadow DOM 提供的 Custom Elements API 进行访问。
    
*   `::slotted()`：用于样式化插槽内容的伪元素。插槽是一种在 Web 组件中用于插入内容的机制，`::slotted()` 可以应用样式到被插入的内容，以实现更精确的样式控制。