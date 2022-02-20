> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTY1MjY0MQ==&mid=2650840065&idx=5&sn=15e489c18aa0477a2651faa6e6be8819&chksm=bd01094f8a768059a69c1ce81ee8345a01e0598a98ac3792cc2680e23b2bacb4a4d662cb366b&mpshare=1&scene=1&srcid=02207R85T10Na5lTpt8vLB11&sharer_sharetime=1645350773029&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

来源 | ELab 团队（ID：gh_b1857b91fe44）

已获得原公众号的授权转载

**CSS**（全称 **C**ascading **S**tyle **S**heets，层叠样式表）为开发人员提供声明式的样式语言，是前端必备的技能之一。基于互联网上全面的资料和简单易懂的语法，CSS 非常易于学习，但其知识点广泛且分散，很难做到精通。在我们日常开发中，受限于原代码混乱、DDL 将近等问题，常常忽视了 CSS 代码的质量，很容易写出杂乱无章的 CSS 文件。  

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6QfpteD6FHibSrd0uBCYV6WY7RFT67zHHuicR48XUcgWUzHAYqcRpb8yvA/640?wx_fmt=png)

代码优化建议
------

1.  ### 使用缩写属性精简代码
    

> 适用于：margin、padding、border、font、background 等
> 
> 但并非所有情况下都必须缩写，因为当一个属性的值缩写时，总是会将所有项都设置一遍，而有时候我们不希望设置值里的某些项，这时候需要开发者自行判断。

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6QElg4YZA0icJOibLCXuCLwqcOarRpST4lV3Ae2RibBGI2eicj6vqMF3mOFw/640?wx_fmt=png)

2.  ### 合并选择器
    

> 使用 ",（逗号）" 连接多个选择器定义公用属性，不仅能减小 css 文件大小，还能增加可读性。
> 
> 为了更易于定位问题，逗号后换行。

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6Qyqhea6ianDXh3iblqCibhrNwPicj4fhy4XnsrLzkkn3wukOa4MkyfJQqkg/640?wx_fmt=png)

3.  ### 使用更语义化的单词命名 class
    

> 命名的时候以 “在你之后开发的人不会产生疑惑” 为目标

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6Qa5S6YJHeaEpylyARtTq6TYj7wUe6I2VniccrVJ8qb2jL4gHVKuiax0ow/640?wx_fmt=png)

4.  ### 属性声明顺序
    

> Reference：Bootstrap property order for Stylelint[1]
> 
> 选择器中属性数量较多时，将相关的属性声明放在一起，并按以下顺序排列：
> 
> 1.  Positioning：定位相关，如 position、top/bottom/left/right、z-index 等
>     
> 2.  Box model：盒模型相关，如 display、float、margin、width/height 等
>     
> 3.  Typographic：排版相关，如 font、color、line-height 等
>     
> 4.  Visual：可视相关，如 background、color 等
>     
> 5.  Misc：其他，如 opacity、animation 等
>     

个人建议：在属性数量较多时可以参考这 5 个类别归类排列，至于顺序没必要太过纠结。

```
.declaration-order {
  /* Positioning */
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 100;

  /* Box-model */
  display: block;
  float: right;
  width: 100px;
  height: 100px;

  /* Typography */
  font: normal 13px "Helvetica Neue", sans-serif;
  line-height: 1.5;
  color: #333;
  text-align: center;

  /* Visual */
  background-color: #f5f5f5;
  border: 1px solid #e5e5e5;
  border-radius: 3px;

  /* Misc */
  opacity: 1;
}
```

5.  ### 使用 & 符号引用父选择器
    

> & 是 Sass 和 Less 中提供的语法糖，用于表示对父选择器的引用，antd design 的源码里应用广泛  
> 优点：非常适合用于编写组件的样式，减少了很多重复单词  
> 缺点：从 HTML 的 class name 中寻找对应样式的成本增加  

```
.header {
    .header-title {/* styles */}
    .header-title:after {/* styles */}
    .header-content {/* styles */}
}

/* 用&引用来优化代码👇 */
.header {
    &-title {
        /* styles */
        &:after {/* styles */}
    }
    &-content {/* styles */}
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6Q367qWX8KU1xiaME2Wfhuc7eHW85GwtR5EuibGsyiaVoMv178AeKMHoKhw/640?wx_fmt=png)

善用相关技术
------

从 The State of CSS 2020: 技术 [2] 调研中我们可以得知当前主流 CSS 工具的用户数量与满意率数据（样本数量 1 万 +，仅用于参考），其中分为了 4 个象限：

> **评估**: 使用率较低，满意度较高。值得关注的技术。  
> **采用**: 使用率高，满意度高。可采用安全技术。【Sass、BEM、PostCSS、Styled Components】  
> **规避**: 使用率低，满意度低。目前最好避免使用的技术。  
> **待定**: 使用率高，满意度低。如果您正在使用这些技术，请重新评估它们。  

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6QtLLiaSheJPwonAE96iaiapXjGUVAqHvcy21mCeiaSoSGwpZysTuMZNR7yw/640?wx_fmt=png)

### CSS 方法论

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6QpPoWRw4s9HnQtJAHloBpmyiapic4gtQ2zHf4udRCqNO1U5UgKNYxej7Q/640?wx_fmt=png)

#### BEM：模块化命名规范

BEM（Block 块、Element 元素、Modifier 修饰符）是一种规范化的类名命名约定。

> There are only two hard problems in Computer Science: cache invalidation and naming things — _Phil Karlton_

**🙋** **BEM** **解决了什么问题？**

*   CSS 没有作用域，class 同名时会造成 “样式污染”
    
*   CSS 文件较长时，不统一的命名使得结构比较混乱
    
*   不规范的命名使得样式难以定位到对应的 HTML 元素，不同开发者进行开发时容易写出重复的代码
    

BEM 的优势：**模块化**、**结构化**、**可重用性**

**🙋** **BEM** **规范？**

**Block 块：** 一个独立且有意义的实体，任何 DOM 元素都可以是块 **Element 元素**：块的子元素，依附于块存在。如：列表中的某一项、卡片的标题、选择器中的选择项等 **Modifier 修饰符：** 表示块或元素的外观、状态或行为。如：是否点击、是否禁用等**命名规范**：

.block__element--modifier

> block 和 element 用双下划线__链接  
> element 和 modifier 用双中划线 -- 链接  
> block、element、modifier 包含多个单词时，用一个中划线 - 链接  
> 不推荐 element 嵌套，如果需要嵌套则说明该从中抽一个组件出来了  

**🌰例子**

【Element-UI 源码】https://github.com/ElemeFE/element/blob/dev/packages/table/src/table.vue

#### Atomic CSS：原子化 CSS

**🙋什么是原子化？**

> **原子**（atom），是指化学反应不可再分的基本微粒

在 ACSS 中，将每个仅有单一 CSS 规则的、不可再拆分的 CSS 类称为 CSS 原子。HTML 的样式由多个 CSS 原子组合而成，以内联的形式写在 HTML 中。

*   优势：无需维护 CSS 文件，在 HTML 中内联 “所见即所得”；移动 / 删除 HTML 元素时，其样式也能随之移动，无需额外的更新成本。
    
*   缺点：结构和样式强耦合，不利于大型项目维护，容易产生很多重复代码。
    

**🌰例子**

```
<!--- 使用ACSS语法创建一个子元素水平垂直居中的容器 --->
<div class="D(f) Jc(c) Ac(c) ">Flex容器</div>

<!--- 原子化CSS思想的代码 --->
<div class></div>
```

### CSS 预 / 后处理器

为了给 CSS 提供更强大的功能（嵌套、变量、运算等）、使其更易于维护，诞生了 Sass、Less、Stylus 等 CSS 预处理器。开发者可以使用这些工具提供的更便捷的语法和特性进行开发，预处理器则负责将代码编译为 CSS，从而达到提供样式的目的。

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6QA2xRibjT8t64cXSdjUtE93IJIadibiaBltmHYsAkCsRnuXfMDSoguAlVQ/640?wx_fmt=png)

#### Sass (Scss) & Less

Sass: 语法 | Sass 中文网 [3]、语言特性 | Less 中文网 [4]

**🙋预处理器的优越性？**

*   将 CSS 从声明语言转换成一门编程语言
    
*   可嵌套的语法增加了样式文件的可读性和可维护性
    
*   变量与混合特性能够减少很多重复的样式声明
    

**🙋Sass .vs. Less？**

*   Less 更像 CSS，易于上手，能够从 CSS 平滑过渡；Sass 的缩进语法接受度因人而异，Sass3.0 中提出了兼容 CSS 的 Scss，用户可以选择使用 Sass 或 Scss。
    
*   当项目 CSS 中需要涉及复杂逻辑时，Sass/Scss 更适合，Sass 提供了更强大、更接近编程语言的 @function、@if/@else、@while 等语法；当项目的样式复杂度不高时，选 Sass 或 Less 都可以。（下面是一个 Less 和 Scss 语法对比例子🌰）
    

```
// Less

.mixin( @count )when( @count > 0 ){
    background-color: black;
}
.mixin( @count )when( @count <= 0 ){
    background-color: white;
}
.tag {
    .mixin(100);
}

// Scss
@function checkCount($count) {
    @if $count > 0 {
        return black;
    }
    @else {
        return white;
    }
}
.tag {
    background-color: checkCount(100);
}
```

*   Sass 提供了命令行语法，当用户使用 Dart 或 Ruby 时可以通过命令行来对 Sass 进行解析（将 Sass 编译为 CSS）等操作，为开发者提供了扩展空间。基于 Sass 良好的可扩展性，诞生了诸如 Compass 等 Sass 框架。
    

#### PostCSS

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6Qhp7uGgIYeFRoneBLqibeKYL1yB0eysbqqnGzUXfPaagGz4f14iaTYicXQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ndgH50E7pIrrH3AA4RmrMR1sja8ciaT6Q7cKQjgRnrNScOubnDiauY8kEb6QONliaEbVJEu4XQ1rrG1tMTdg3ypmw/640?wx_fmt=png)

**🙋定义？**

狭义的 PostCSS：“A tool for transforming CSS with JavaScript”，将 CSS 转换为 JS 代码的工具，提供了将 CSS 代码解析为 AST 的能力，并向开发者暴露出能够修改 CSS 代码的 JavaScript API。

广义的 PostCSS：基于 PostCSS 工具提供的 API 开发出的一系列插件，能够对用户编写的 CSS 代码进行处理，这些插件也可以称之为后处理器。

**🙋有什么较为流行的 PostCSS 应用场景？**

*   autoprefixer 插件：打包时自动添加浏览器前缀属性
    

```
div { display: flex }
// 自动转换为👇

div{
  display : -webkit-box;
  display : -webkit-flex;
  display : -moz-box;
  display : -ms-flexbox;
  display : flex;
}
```

*   通过颜色值的识别与替换来切换深 / 浅色模式 前端站点一键支持暗色模式 [5]
    
*   stylelint 插件：CSS 代码检查工具
    
*   postcss-utilities[6]：为开发者提供 CSS 简写方式（原子化）
    

```
.cfx {
    @util clearfix;
}

// 等价于👇
.cfx:after {
    content: '';
    display: block;
    clear: both;
}



.rounded-top {
    @util border-top-radius(4px);
}

// 等价于👇
.rounded-top {
    border-top-left-radius: 4px;
    border-top-right-radius: 4px;
}
```

### CSS in JS

#### **styled-components** **/Emotion**

*   高可复用性：从组件的层面对 CSS 进行封装，适应 “组件化” 的前端时代，代码中不再需要维护 CSS 文件。
    
*   高灵活性：使用 styled-components 能够很轻松地找到某个组件关联的样式；移动 / 删除 HTML 元素时，其样式也能随之移动 / 删除。
    
*   解决了 CSS“全局污染” 的痛点：styled-components 在编译时为样式生成唯一的 class name，开发者不必再担心 class name 重复。
    

```
// 创建一个 Title 组件,它将渲染一个附加了样式的 <h1> 标签
const Title = styled.h1`
  font-size: 1.5em;
  text-align: center;
  color: palevioletred;
`;

// 创建一个 Wrapper 组件,它将渲染一个附加了样式的 <section> 标签，并读取wrapperColor属性作为背景色
const Wrapper = styled.section`
  padding: 4em;
  background: ${props => props.wrapperColor || "palevioletred"};
`;

// 就像使用常规 React 组件一样使用 Title 和 Wrapper
render(
  <Wrapper wrapperColor="red">
    <Title>
      Hello World!
    </Title>
  </Wrapper>
);
```

🙋**styled-components 的适用场景**

*   使用 React 或 React Native 进行开发时
    
*   中小型项目；或基于组件库开发、自定义样式复杂度不高的大型项目
    
*   页面中元素样式无需变化或变化较少时
    

总结
--

综上，CSS 作为一门前端必备的基础技能，具有许多原生的痛点。近年来，全球的开发者也在源源不断地提出不同的优化方案，我们在日常关注 React、NodeJS、性能优化等热门前端话题的时候，也不要忘了好好写 CSS 代码呀～

Reference
---------

《CSS 权威指南 第四版》

NEC : 更好的 CSS 样式解决方案 [7]

Code Guide by @AlloyTeam[8]

编码规范 by @mdo[9]

The State of CSS 2020[10]

BEM — Block Element Modifier[11]

ACSS[12]

Sass: 语法 | Sass 中文网 [13]

语言特性 | Less 中文网 [14]

hengg/styled-components-docs-zh[15]

### 参考资料

[1]

Bootstrap property order for Stylelint: _https://github.com/twbs/stylelint-config-twbs-bootstrap/blob/master/css/index.js_

[2]

The State of CSS 2020: 技术: _https://2020.stateofcss.com/zh-Hans/technologies/_

[3]

Sass: 语法 | Sass 中文网: _https://www.sasscss.com/documentation/syntax_

[4]

语言特性 | Less 中文网: _http://lesscss.cn/features/#features-overview-feature_

[5]

前端站点一键支持暗色模式: _https://tech.bytedance.net/articles/6950540123276738573_

[6]

postcss-utilities: _https://github.com/ismamz/postcss-utilities_

[7]

NEC : 更好的 CSS 样式解决方案: _http://nec.netease.com/_

[8]

Code Guide by @AlloyTeam: _http://alloyteam.github.io/CodeGuide/#css-indentation_

[9]

编码规范 by @mdo: _https://codeguide.bootcss.com/#css-nesting_

[10]

The State of CSS 2020: _https://2020.stateofcss.com/zh-Hans/_

[11]

BEM — Block Element Modifier: _http://getbem.com/naming/_

[12]

ACSS: _https://acss.io/_

[13]

Sass: 语法 | Sass 中文网: _https://www.sasscss.com/documentation/syntax_

[14]

语言特性 | Less 中文网: _http://lesscss.cn/features/#features-overview-feature_

[15]

hengg/styled-components-docs-zh: _https://github.com/hengg/styled-components-docs-zh/blob/master/Basics.md_