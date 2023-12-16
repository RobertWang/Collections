> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [jakelazaroff.com](https://jakelazaroff.com/words/web-components-will-outlive-your-javascript-framework/)

> If we're building things that we want to work in five or ten or even 20 years, we need to avoid depen......如果我们要建造的东西，我们想在5年、10年甚至20年内工作，我们需要避免......

If you’re anything like me, when you’re starting a project, there’s a paralyzing period of indecision while you try to figure out how to build it. In the JavaScript world, that usually boils down to picking a framework. Do you go with Ol’ Reliable, a.k.a. React? Something slimmer and trendier, like Svelte or Solid? How about kicking it old school with a server-side framework and HTMX?  
如果你和我一样，当你开始一个项目时，当你试图弄清楚如何构建它时，会有一段犹豫不决的瘫痪期。在 JavaScript 世界中，这通常归结为选择一个框架。你选择Ol'Reliable，又名React吗？更纤薄、更时尚的东西，比如 Svelte 或 Solid？用服务器端框架和 HTMX 来踢它怎么样？

When I was writing my [CRDT blog post series](https://jakelazaroff.com/words/an-interactive-intro-to-crdts/), I knew I wanted to include interactive demos to illustrate the concepts. Here’s an example: a toy collaborative pixel art editor.  
当我写我的 CRDT 博客文章系列时，我知道我想包括交互式演示来说明这些概念。下面是一个示例：玩具协作像素艺术编辑器。

Even though I’ve written before — and still believe — that [React is a good default option](https://jakelazaroff.com/words/no-one-ever-got-fired-for-choosing-react/), the constraints of a project should determine the technology decisions. In this case, I chose to use vanilla JS web components. I want to talk about why.  
尽管我之前写过——并且仍然相信——React 是一个很好的默认选项，但项目的约束应该决定技术决策。在这种情况下，我选择使用普通的 JS Web 组件。我想谈谈为什么。

There was one guiding principle for this project: although they happened to be built with HTML, CSS and JS, these examples were **content, not code**. In other words, they’d be handled more or less the same as any image or video I would include in my blog posts. They should be portable to any place in which I can render HTML.  
这个项目有一个指导原则：尽管它们恰好是用 HTML、CSS 和 JS 构建的，但这些示例是内容，而不是代码。换句话说，它们的处理方式或多或少与我在博客文章中包含的任何图像或视频相同。它们应该可以移植到我可以呈现 HTML 的任何地方。

As of 2023, this blog is built with Astro. Before that, it was built with [my own static site generator](https://jake.museum/jakelazaroff-v5/). Before that, [Hugo](https://jake.museum/jakenyc-v3/); before that, [a custom CMS written in PHP](https://jake.museum/jakelazaroff-blog/); before that, [Tumblr](https://jake.museum/hexnut-v5/), [Movable Type](https://jake.museum/hexnut-v4/) and [WordPress](https://jake.museum/mlingojones/) — and I’m sure I’m missing some in between. I really like Astro, but it’s reasonable to assume that this website won’t run on it forever.  
截至 2023 年，这个博客是用 Astro 构建的。在此之前，它是用我自己的静态站点生成器构建的。在此之前，雨果;在此之前，用PHP编写的自定义CMS;在此之前，Tumblr、Movable Type 和 WordPress——我敢肯定我错过了介于两者之间的一些。我真的很喜欢 Astro，但可以合理地假设这个网站不会永远运行在上面。

One thing that has made these migrations easier in recent years is keeping all my content in plain text files written in Markdown. Rather than dealing with the invariably convoluted process of moving my content between systems — exporting it from one, importing it into another, fixing any incompatibilities, maybe removing some things that I can’t find a way to port over — I drop my Markdown files into the new website and it mostly Just Works.  
近年来，使这些迁移更容易的一件事是将我的所有内容保存在用 Markdown 编写的纯文本文件中。我没有处理在系统之间移动我的内容的复杂过程——从一个系统导出，导入到另一个系统，修复任何不兼容问题，也许删除一些我找不到移植方法的东西——我把我的 Markdown 文件放到新网站中，它主要是 Just Works。

Most website generators have a way to include more complex markup within your content, and Astro is no different. The MDX integration allows you to render Astro components within your Markdown files. Those components have access to all the niceties of the Astro build system: you can write HTML, CSS and JS within one file, and Astro will automagically extract and optimize everything for you. It will scope CSS selectors and compile TypeScript and let you conditionally render markup and do all sorts of other fancy stuff.  
大多数网站生成器都有办法在你的内容中包含更复杂的标记，Astro 也不例外。MDX 集成允许你在 Markdown 文件中渲染 Astro 组件。这些组件可以访问 Astro 构建系统的所有细节：你可以在一个文件中编写 HTML、CSS 和 JS，Astro 会自动为你提取和优化所有内容。它将限定 CSS 选择器的范围并编译 TypeScript，并允许你有条件地渲染标记并执行各种其他花哨的事情。

The drawback, of course, is that it all only works inside Astro. In order to switch to a different site generator, I’d have to rewrite those components. I might need to split up the HTML, CSS and JS, or configure a new build system, or find a new way to scope styles. So Astro-specific features were off limits — no matter how convenient.  
当然，缺点是它只能在 Astro 中工作。为了切换到不同的站点生成器，我必须重写这些组件。我可能需要拆分 HTML、CSS 和 JS，或者配置一个新的构建系统，或者找到一种新的方法来限定样式的范围。因此，Astro 特定的功能是禁止的——无论多么方便。

But Markdown has a secret weapon: [you can write HTML inside of it](https://daringfireball.net/projects/markdown/syntax#html)! That means any fancy interactive diagrams I wanted to add would be just as portable as my the rest of my Markdown as long as I could express them as plain HTML tags.  
但是 Markdown 有一个秘密武器：你可以在里面写 HTML！这意味着我想添加的任何花哨的交互式图表都与我的 Markdown 的其余部分一样可移植，只要我可以将它们表示为纯 HTML 标签。

[Web components](https://developer.mozilla.org/en-US/docs/Web/API/Web_Components) hit that nail square on the head. They’re a set of W3C standards for building reusable HTML elements. You use them by writing a class for a custom element, registering a tag name and using it in your markup. Here’s how I embedded that pixel art editor before:  
Web 组件一针见血。它们是一组用于构建可重用 HTML 元素的 W3C 标准。您可以通过为自定义元素编写类、注册标记名称并在标记中使用它来使用它们。以下是我之前嵌入像素艺术编辑器的方式：

```
<pixelart-demo></pixelart-demo>

```

That’s the honest-to-goodness HTML I have in the Markdown for this post. That’s it! There’s no special setup; I don’t have to remember to put specific elements on the page before calling a function or load a bunch of extra resources.[1](#user-content-fn-cheating) Of course, I do need to keep the JS files around and link to them with a `<script>` tag. But that goes for any media: there needs to be **some** way to reference it from within textual content. With web components, once the script is loaded, the tag name gets registered and works anywhere on the page — even if the markup is present before the JavaScript runs.  
这就是我在这篇文章的 Markdown 中诚实至善的 HTML。 就是这样！没有特殊设置;我不必记得在调用函数或加载一堆额外资源之前将特定元素放在页面上。 [1](#user-content-fn-cheating) 当然，我确实需要保留 JS 文件并使用 `<script>` 标签链接到它们。但这适用于任何媒体：需要有某种方式从文本内容中引用它。使用 Web 组件时，一旦加载脚本，标记名称就会被注册并在页面上的任何位置工作，即使标记在 JavaScript 运行之前就存在。

Web components encapsulate all their HTML, CSS and JS within a single file, with no build system necessary. Having all the code for a component in one place significantly reduces my mental overhead, and I continue to be a huge fan of single-file components for their developer experience. While web components aren’t **quite** as nice to write as their Astro or Svelte counterparts, they’re still super convenient.[2](#user-content-fn-frameworks)  
Web 组件将其所有 HTML、CSS 和 JS 封装在一个文件中，无需构建系统。将组件的所有代码放在一个地方可以大大减少我的精神开销，并且我仍然非常喜欢单文件组件，因为它们具有开发人员体验。虽然 Web 组件不像 Astro 或 Svelte 那样好听，但它们仍然非常方便。 [2](#user-content-fn-frameworks)

In case you’re not familiar with web components, here’s the code for that `<pixelart-demo>` component above:[3](#user-content-fn-abridged)  
如果您不熟悉 Web 组件，下面是上面该 `<pixelart-demo>` 组件的代码： [3](#user-content-fn-abridged)

```
import PixelEditor from "./PixelEditor.js";

class PixelArtDemo extends HTMLElement {
  constructor() {
    super();

    this.shadow = this.attachShadow({ mode: "closed" });
    this.render();

    const resolution = Number(this.getAttribute("resolution")) || 100;
    const size = { w: resolution, h: resolution };

    const alice = new PixelEditor(this.shadow.querySelector("#alice"), size);
    const bob = new PixelEditor(this.shadow.querySelector("#bob"), size);

    alice.debug = bob.debug = this.hasAttribute("debug");
  }

  render() {
    this.shadow.innerHTML = `
      <div class="wrapper">
        <canvas class="canvas" id="alice"></canvas>
        <canvas class="canvas" id="bob"></canvas>
        <input class="color" type="color" value="#000000" />
      </div>

      <style>
        .wrapper {
          display: grid;
          grid-template-columns: 1fr 1fr;
          grid-template-rows: 1fr auto;
          gap: 1rem;
          margin: 2rem 0 3rem;
        }

        .canvas {
          grid-row: 1;
          width: 100%;
          aspect-ratio: 1 / 1;
          border: 0.25rem solid #eeeeee;
          border-radius: 0.25rem;
          cursor: crosshair;
        }

        .color {
          grid-column: 1 / span 2;
        }
      </style>
    `;
  }
}

customElements.define("pixelart-demo", PixelArtDemo);

```

Everything is nicely contained within this one file. There is that one import at the top, but it’s an [ES module import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) — it doesn’t rely on any sort of build system. As long as I keep all the files together, the browser will sort everything out.  
一切都很好地包含在这个文件中。顶部有一个导入，但它是一个 ES 模块导入——它不依赖于任何类型的构建系统。只要我把所有文件放在一起，浏览器就会把所有东西都整理出来。

Another nice thing about Web components is shadow DOM, which isolates the component from the surrounding page. I think shadow DOM is often awkward when you want to share styles between your components and the rest of your app, but it’s perfect when you do truly want everything to be isolated. Just like images and videos, these components will look and act the same no matter where they’re used.  
Web 组件的另一个好处是影子 DOM，它将组件与周围的页面隔离开来。我认为当你想在组件和应用程序的其余部分之间共享样式时，影子 DOM 通常是很尴尬的，但当你真的希望一切都被隔离时，它是完美的。就像图像和视频一样，无论在哪里使用，这些组件的外观和行为都是一样的。

Sorry — they’re not **just** like images and videos. Web components can expose attributes that allow you to configure them from the outside. You can think of them as native props. Voilà:  
对不起，它们不仅仅是图像和视频。Web 组件可以公开允许您从外部配置它们的属性。你可以把它们看作是原生道具。瞧：

Two input ranges with different accent colors. In this case, I’m just setting a CSS variable, which is one of the few things allowed into the shadow DOM:  
具有不同主题色的两个输入范围。在本例中，我只是设置了一个 CSS 变量，这是允许进入影子 DOM 的少数内容之一：

```
<range-slider style="--accent: #0085F2"></range-slider>

```

Here’s a more complex example:  
下面是一个更复杂的示例：

And here’s the markup. It uses attributes to alter the component’s behavior, setting the resolution to 20 and showing debug information on every pixel:  
这是标记。它使用属性来改变组件的行为，将分辨率设置为 20 并在每个像素上显示调试信息：

```
<pixelart-demo debug resolution="20"></pixelart-demo>

```

If you were wondering what those calls to `getAttribute` and `hasAttribute` were doing in the web component class, now you know. This was particularly useful when reusing the same component for different stages of a tutorial, allowing me to enable certain features as the tutorial progressed.  
如果您想知道这些对 Web 组件类的调用和 `hasAttribute` 在 Web 组件类中执行的 `getAttribute` 操作，现在您知道了。这在教程的不同阶段重用同一组件时特别有用，允许我随着教程的进行启用某些功能。

The other part of the equation was using vanilla JS. There are frameworks that compile to web components — most notably Lit (although I’d call it more of a library) but also Stencil, Svelte, and probably others. I’m sure they’re all wonderful tools that would have made my life easier in a lot of ways. But frameworks are dependencies, and dependencies have a bunch of tradeoffs. In this case, the tradeoff I’m most worried about is maintenance.[4](#user-content-fn-vendoring)  
等式的另一部分是使用vanilla JS。有一些框架可以编译成 Web 组件——最著名的是 Lit（尽管我更愿意称它为库），但也有 Stencil、Svelte 和其他框架。我相信它们都是很棒的工具，可以在很多方面让我的生活更轻松。但是框架是依赖关系，而依赖关系需要权衡取舍。在这种情况下，我最担心的权衡是维护。 [4](#user-content-fn-vendoring)

That goes for TypeScript, too. By my count, the last 15 versions of TypeScript have had breaking changes — many of them new features that I was happy to have, even though I had to change my code to accommodate them. But as much as I love TypeScript, it’s not a native substrate of the web. It’s still a dependency.  
TypeScript 也是如此。据我统计，TypeScript 的最后 15 个版本都有重大变化——其中许多是我很高兴拥有的新功能，尽管我不得不更改我的代码来适应它们。但是，尽管我很喜欢 TypeScript，但它并不是 Web 的原生基础。它仍然是一个依赖关系。

There’s a cost to using dependencies. New versions are released, APIs change, and it takes time and effort to make sure your own code remains compatible with them. And the cost accumulates over time. It would be one thing if I planned to continually work on this code; it’s usually simple enough to migrate from one version of a depenency to the next. But I’m not planning to ever really touch this code again unless I absolutely need to. And if I **do** ever need to touch this code, I **really** don’t want to go through multiple years’ worth of updates all at once.  
使用依赖项是有成本的。新版本发布，API 发生变化，需要花费时间和精力来确保您自己的代码与它们保持兼容。成本会随着时间的推移而累积。如果我打算继续处理此代码，那将是一回事;从Depenency的一个版本迁移到下一个版本通常很简单。但是除非我绝对需要，否则我不打算再次真正接触此代码。如果我确实需要接触这些代码，我真的不想一次经历多年的更新。

I [learned that lesson the hard way](https://jakelazaroff.com/words/preserving-the-web/) when I built my online museum, wiping the cobwebs off of code saved on laptops that hadn’t been turned on in a full decade. The more dependencies a website had, the more difficult it was to restore.  
当我建立我的在线博物馆时，我以艰难的方式吸取了这一教训，清除了保存在笔记本电脑上的代码上的蜘蛛网，这些代码已经整整十年没有打开过了。网站的依赖关系越多，恢复的难度就越大。

I’ve been building on the web for almost 20 years. That’s long enough to witness the birth, rise and fall of jQuery. Node.js was created, forked into io.js and merged back into Node. Backbone burst onto the scene and was quickly replaced with AngularJS, which was replaced with React, which has been around for only half that time and has **still** gone through like five different ways to write components.  
我已经在网络上构建了将近 20 年。这足以见证jQuery的诞生、兴起和衰落。Node.js 被创建，分叉到 io.js 中，然后合并回 Node。Backbone突然出现，并很快被AngularJS取代，后者被React取代，React只存在了一半，并且仍然经历了五种不同的组件编写方式。

But as the ecosystem around it swirled, the web platform itself remained remarkably stable — largely because the stewards of the standards painstakingly ensured that no new change would break existing websites.[5](#user-content-fn-smooshgate) The [original Space Jam website](https://www.spacejam.com/1996/) from 1996 is famously still up, and renders perfectly in modern browsers. So does the [first version of the website you’re reading now](https://jake.museum/jakelazaroff-v1/), made when I was a freshman in college 15 years ago. Hell, the [first website ever created](http://info.cern.ch/hypertext/WWW/TheProject.html) — built closer to the formation of the Beatles [6](#user-content-fn-moonlanding) than to today! — still works, in all its barebones hypertext glory.  
但是，随着围绕它的生态系统的旋转，网络平台本身仍然保持着非常稳定的状态——这主要是因为标准的管理者煞费苦心地确保没有新的变化会破坏现有的网站。 [5](#user-content-fn-smooshgate) 1996 年的原始 Space Jam 网站仍然在运行，并且在现代浏览器中完美呈现。你现在正在阅读的网站的第一个版本也是如此，它是15年前我还是大学新生时制作的。地狱，有史以来第一个网站——比今天更接近披头士乐队 [6](#user-content-fn-moonlanding) 的成立！— 仍然有效，在其所有准系统超文本荣耀中。

If we want that sort of longevity, we need to avoid dependencies that we don’t control and stick to standards that we know won’t break. If we want our work to be accessible in five or ten or even 20 years, we need to use the web with no layers in between. For all its warts, the web has become the most resilient, portable, future-proof computing platform we’ve ever created — at least, if we build with that in mind.  
如果我们想要这种长寿，我们需要避免我们无法控制的依赖关系，并坚持我们知道不会破坏的标准。如果我们希望我们的工作在5年、10年甚至20年内都可以访问，我们需要使用中间没有层的网络。尽管存在种种疣，但网络已经成为我们有史以来最有弹性、最便携、最面向未来的计算平台——至少，如果我们在构建时考虑到这一点的话。