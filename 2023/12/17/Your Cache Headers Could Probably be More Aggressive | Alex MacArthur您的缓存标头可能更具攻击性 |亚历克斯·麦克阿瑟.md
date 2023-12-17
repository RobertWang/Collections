> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [macarthur.me](https://macarthur.me/posts/more-aggressive-cache-headers)

> It's common for modern hosts to cache static assets in a flexible, but not most optimal way. Let's ex......现代主机通常以灵活但不是最佳方式缓存静态资产。让我们举例说明......

It's common for modern hosts to cache static assets in a flexible, but not most optimal way. Let's explore why that is and what we can do to push cache performance (for some assets) even further.  
现代主机通常以灵活但不是最佳方式缓存静态资产。让我们来探讨一下为什么会这样，以及我们可以做些什么来进一步提高缓存性能（对于某些资产）。

We've got it real good when it comes to standing up websites these days (especially static ones). Modern hosts like Vercel and Netlify take care of a _lot_ out of the box, shielding us from the meticulous, complicated stuff.  
如今，在站立网站（尤其是静态网站）方面，我们做得非常好。像 Vercel 和 Netlify 这样的现代主机开箱即用，让我们免受细致、复杂的东西的影响。

Caching is one of them. To accommodate the widest range of users, many providers will cache static assets with a `Cache-Control` header of `public`, `max-age=0`, `must-revalidate`. Translation:  
缓存就是其中之一。为了适应最广泛的用户，许多提供商将缓存 `Cache-Control` 标头为 `public` 、 `max-age=0` 、 `must-revalidate` 的静态资产。译本：

> Cache this thing, but immediately let it go stale and ask the origin server if there's a fresh copy to download next time around.  
> 缓存这个东西，但立即让它过时，并询问源服务器下次是否有新的副本可以下载。

It's a smart default. On each subsequent visit to a page, the browser will _always_ check in with the origin server (or global CDN) for the latest version of the asset, but it'll only actually perform a full download _if those assets have changed._ If everything's the same, you'll get a `304 Not Modified` response back, and the browser's"stale" version will be used. Like this:  
这是一个智能默认值。在每次后续访问页面时，浏览器将始终向源服务器（或全局 CDN）签入最新版本的资产，但只有在这些资产发生更改时，它才会实际执行完整下载。如果一切都一样，你会得到一个 `304 Not Modified` 回复，并且会使用浏览器的“陈旧”版本。喜欢这个：

![](https://picperf.io/https://cms.macarthur.me/content/images/2023/09/image-4.png)non-initial page views with default caching  
使用默认缓存的非初始页面视图

Even though it's performing a distinct GET request to perform that check, if it returns with 304, it'll end up being far smaller & faster than full fetch with a 200. For example, here are a couple of actual requests for a font file from my site. The one that came back with a 304 had a significantly smaller footprint.  
即使它执行一个不同的 GET 请求来执行该检查，如果它返回 304，它最终将比使用 200 的完全提取小得多且更快。例如，以下是从我的网站对字体文件发出的几个实际请求。带着 304 回来的那个占地面积要小得多。

![](https://picperf.io/https://cms.macarthur.me/content/images/2023/08/image-6.png)screenshot comparing the duration of a 304 and a 200 response  
比较 304 和 200 响应持续时间的屏幕截图

So, while it may sound counterintuitive to immediately let something cached go stale, it's a good balance between performance and flexibility. It'll save some headaches too. You won't end up with someone being served outdated CSS or a previous deployment's JavaScript bundle. Ship as much as you want, and your users will get the latest. If you'd like to dig more into this specific caching strategy, both [Netlify](https://www.netlify.com/blog/2017/02/23/better-living-through-caching/?ref=alex-macarthur) and [Vercel](https://vercel.com/docs/edge-network/caching?ref=alex-macarthur#static-files-caching) have some information on the philosophy behind it.  
因此，虽然立即让缓存的内容过时听起来可能违反直觉，但它在性能和灵活性之间取得了很好的平衡。它也会省去一些麻烦。您最终不会向某人提供过时的 CSS 或以前部署的 JavaScript 包。随心所欲地发货，您的用户将获得最新信息。如果您想更深入地了解这种特定的缓存策略，Netlify 和 Vercel 都有一些关于其背后的理念的信息。

Even so, if you consider yourself a performance scrutineer, it should bother you that you're leaving some speed on the table by sticking with these defaults. Your caching _could be a little more aggressive_, and in my opinion, it's a no-brainer for particular types of assets.  
即便如此，如果您认为自己是性能检查员，那么坚持使用这些默认值会给您带来一些速度，这应该会让您感到困扰。您的缓存可能会更激进一些，在我看来，对于特定类型的资产来说，这是不费吹灰之力的。

[Some Things Never Change  
有些事情永远不会改变](#some-things-never-change)
------------------------------------------------------------------

For a typical content site, most of the assets served via URL aren't dynamic. The same pile of CSS will be used for every visitor. Same story for fonts, individual images, and JavaScript bundles. Certain things just aren't designed to change based on who made the request or when it was performed, and it's technically wasteful to perform an extra HTTP request (albeit a small one) when you're likely to get back 304 anyway.  
对于典型的内容网站，通过网址提供的大多数素材资源都不是动态的。每个访问者都将使用相同的 CSS 堆。字体、单个图像和 JavaScript 包也是如此。某些事情不会根据谁发出请求或何时执行而改变，而且从技术上讲，当你可能得到 304 时，执行额外的 HTTP 请求（尽管很小）是浪费的。

You can virtually eliminate those unnecessary requests by using a cache header like this instead.  
实际上，您可以通过使用这样的缓存标头来消除这些不必要的请求。

`public, max-age=31560000, immutable`

Here's what it means:  这意味着：

*   `public` - the asset can be stored in any cache between (and including) the browser and origin server  
    `public` - 资产可以存储在浏览器和源服务器之间的任何缓存中（包括浏览器和源服务器）
*   `max-age=31560000` - the cache doesn't have to consider it"stale" until a full year has passed  
    `max-age=31560000` - 在一整年过去之前，缓存不必将其视为“过时”
*   `immutable` - the browser is explicitly instructed to NOT reach out to origin/CDN just to check if something newer is available (no more revalidation requests)  
    `immutable` - 浏览器被明确指示不要仅仅为了检查是否有更新的东西可用而联系 origin/CDN（不再有重新验证请求）

With that policy in place, after a page has been visited for the first time, each asset is loaded straight from the cache, and the flow ends up looking more like this:  
实施该策略后，首次访问页面后，将直接从缓存中加载每个资产，最终流看起来更像这样：

![](https://picperf.io/https://cms.macarthur.me/content/images/2023/09/image-10.png)non-initial page views with smarter caching  
具有更智能缓存的非初始页面浏览量

As long as the browser's still got a cached copy of the asset (identified by unique URL), it'll be a full year before it ever checks origin again. That makes page performance marginally better, and you can feel a little better about yourself as an performance-minded engineer.  
只要浏览器仍然拥有资产的缓存副本（由唯一 URL 标识），它就会在整整一年的时间里再次检查来源。这使得页面性能略有提高，并且您可以对自己作为注重性能的工程师感觉更好一些。

[Nothing Lasts Forever 没有什么是永恒的](#nothing-lasts-forever)
--------------------------------------------------------

Ok. It's foolishly optimistic to say you'll never need to refresh assets before a year crawls by. You'll update a logo, refresh your site design, or swap out your fonts. It'll inevitably happen.  
还行。说在一年过去之前永远不需要刷新资产是愚蠢的乐观。您将更新徽标、刷新网站设计或更换字体。这将不可避免地发生。

But serving fresh assets in those scenarios is a problem easy to solve with an age-old cache-busting tactic: **fingerprinting**. Every time an asset's URL changes (it gets a new"fingerprint"), it'll force the browser and any intermediary cache to treat it as _a completely different asset_. The URL serves as the cache key, and when it changes, it gets a new identity.  
但是，在这些场景中提供新资产是一个很容易解决的问题，使用一种古老的缓存破坏策略：指纹识别。每当资产的 URL 发生变化（它获得一个新的“指纹”）时，它都会强制浏览器和任何中间缓存将其视为完全不同的资产。URL 用作缓存键，当它更改时，它会获得一个新标识。

Most frameworks and site builders already do this for you out-of-the-box, by the way, so it's likely that you'll need to do nothing to benefit from it (at least for some asset types). For example, my site's on Astro. On every build, each static asset is given a very unique name:  
顺便说一句，大多数框架和网站构建器已经为您开箱即用，因此您可能不需要做任何事情即可从中受益（至少对于某些资产类型）。例如，我的网站在 Astro 上。在每次构建中，每个静态资产都会被赋予一个非常独特的名称：

![](https://picperf.io/https://cms.macarthur.me/content/images/2023/09/image.png)

And for the resources that aren't auto-fingerprinted, you'll get the same benefit using a different file name. For example:  
对于未自动指纹识别的资源，使用不同的文件名将获得相同的好处。例如：

```
- <img src="./logo.svg" alt="site logo" />
+ <img src="./logo-v2.svg" alt="site logo" />

```

All of this, by the way, is a good reason _not_ to use overly aggressive caching on your page's HTML itself. The URL of your home page will likely never change, and so it's just not practical to cache it for a year with no revalidation. That's the advantage static assets have over an HTML document. The cache keys of static resources only matter to the HTML that references them. As long as the code's pointed to the correct versions, it doesn't matter what they're named or how frequently they're changed.  
顺便说一句，所有这些都是不要在页面的 HTML 本身上使用过于激进的缓存的一个很好的理由。主页的 URL 可能永远不会更改，因此在不重新验证的情况下将其缓存一年是不切实际的。这就是静态资源相对于 HTML 文档的优势。静态资源的缓存键仅对引用它们的 HTML 重要。只要代码指向正确的版本，它们的名称或更改频率都无关紧要。

[How do I do it?  
我该怎么做？](#how-do-i-do-it)
-------------------------------------------

Implementing this highly depends on how you're hosting your site, but before you do, get clear on which types of assets you'd like to cache more aggressively. For my own site, that's every `.js`, `.css`, and `.woff2` file (my images are already routed through [PicPerf](https://www.picperf.dev/?ref=alex-macarthur), so I'm good there). That list is probably similar for you too.  
实现这一点很大程度上取决于您托管网站的方式，但在此之前，请明确您希望更积极地缓存哪些类型的资产。对于我自己的网站，这是每个 `.js` 、 `.css` 和 `.woff2` 文件（我的图像已经通过 PicPerf 路由，所以我在那里做得很好）。该列表可能对您来说也类似。

### [Customizing Headers on Vercel  
在 Vercel 上自定义标头](#customizing-headers-on-vercel)

If you're on Vercel, you can update your `vercel.json` file to [set specific response headers](https://vercel.com/docs/edge-network/caching?ref=alex-macarthur#using-vercel.json-and-next.config.js) on the assets you target. I use this:  
如果你在 Vercel 上，你可以更新你的 `vercel.json` 文件，为你的目标资产设置特定的响应标头。我用这个：

```
{
	"headers": [
    	{
			"source": "/(.+\\.js|.+\\.css|.+\\.woff2)",
			"headers": [
				{
					"key": "Cache-Control",
					"value": "public, max-age=31560000, immutable"
				}
			]
		}
	]
}

```

Be careful writing that pattern, [by the way](https://vercel.com/docs/errors/error-list?ref=alex-macarthur#invalid-route-source-pattern). Vercel follows the [path-to-regex](https://github.com/pillarjs/path-to-regexp/tree/v6.1.0?ref=alex-macarthur) syntax – not `RegExp`. That's caused a moment or two of extreme frustration for me.  
顺便说一句，要小心写这个模式。Vercel 遵循 path-to-regex 语法 – 而不是 `RegExp` .这让我感到极度沮丧。

### [Customizing Headers on Netlify  
在 Netlify 上自定义标头](#customizing-headers-on-netlify)

Netlify has its own ways to [customize headers](https://docs.netlify.com/routing/headers/?ref=alex-macarthur) using a `_headers` or `netlify.toml` file. Here's the same setup using a `netlify.toml`:  
Netlify 有自己的方法来使用 `_headers` or `netlify.toml` 文件自定义标头。这是使用 `netlify.toml` ：

```
[[headers]]
  for = "/*.(css|js|woff2)"
  [headers.values]
  Cache-Control = "public, max-age=31536000, immutable"

```

### [Using Cloudflare 使用 Cloudflare](#using-cloudflare)

If you're on a provider that doesn't permit customizing response headers so easily, you're not out of luck. You can set up a Cloudflare account to act as a reverse proxy and set the response headers using a [modification rule](https://developers.cloudflare.com/rules/transform/response-header-modification/create-dashboard/?ref=alex-macarthur):  
如果您使用的提供商不允许如此轻松地自定义响应标头，那么您并非不走运。您可以将 Cloudflare 帐户设置为充当反向代理，并使用修改规则设置响应标头：

![](https://picperf.io/https://cms.macarthur.me/content/images/2023/09/image-1.png)

Or, if you'd like to do it in a more interesting way, intercept the request and modify the response with a Cloudflare Worker. Use something like [itty-router](https://github.com/kwhitley/itty-router?ref=alex-macarthur) and it'll amount to less than 30 lines of code:  
或者，如果您想以更有趣的方式执行此操作，请使用 Cloudflare Worker 拦截请求并修改响应。使用类似 itty-router 的东西，它相当于不到 30 行代码：

```
import { Router, IRequest } from "itty-router";

const router = Router();

router.get("/*.(css|js|woff2)", async (request: IRequest) => {
  const response = await fetch(request);

	return new Response(response.body, {
		status: response.status,
		statusText: response.statusText,
		headers: {
			...response.headers,
			"cache-control": "public, max-age=31560000, immutable",
		},
	});
});

export default {
	async fetch(
		request: Request,
		env: {},
        context: ExecutionContext
	): Promise<Response> {
	context.passThroughOnException();

	return router.handle(request, env, context).then((response) => response);
	},
};

```

This, by the way, is the same approach used to cache every image optimized by [PicPerf.dev](http://picperf.dev/?ref=alex-macarthur). I love it.  
顺便说一句，这与用于缓存由 PicPerf.dev 优化的每个图像的方法相同。我喜欢。

[Spend more time in the Network Tab.  
在“网络”选项卡中花费更多时间。](#spend-more-time-in-the-network-tab)
---------------------------------------------------------------------------------------------

The only reason I started thinking about this was because I was curious about the network activity behind any given page load on my own site. It struck me as too much, considering my site's fairly simple and doesn't have any ads or other network-chatty things. It was fun, I learned a lot, and my site came out a little quicker in the process.  
我开始考虑这个问题的唯一原因是因为我对我自己网站上任何给定页面加载背后的网络活动感到好奇。考虑到我的网站相当简单，没有任何广告或其他网络聊天的东西，这让我印象深刻。这很有趣，我学到了很多东西，而且我的网站在这个过程中出现得更快一些。

Dive into those tools yourself, and maybe you'll emerge with a similar tip of your own. If you do, I'd love to hear it.  
自己深入研究这些工具，也许你会有自己的类似技巧。如果你这样做，我很想听听。

Name    Email Address    Password   Submit  [Log In or Register](#) [Log In or Register](#) ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==) Anonymous 

*   [Log Out](#)

.jc-Editor{position:relative}.jc-Editor-tabs{transform:translate(8px,4px);gap:8px;display:flex;list-style:none;padding:0;margin:0}.jc-Editor-tab{border:2px solid transparent;border-bottom:0;background:var(--jc-editor-background);border-radius:var(--jc-border-radius)}.jc-Editor-tab.is-selected{border-color:var(--jc-gray)}.jc-Editor-tabButton{background:0;border:0;font-size:inherit;padding:1rem;cursor:pointer}.jc-Comment *{font:inherit}.jc-Comment-actions{display:inline-block;position:relative}.jc-Comment-actions:before{content:"";display:block;position:absolute;width:2rem;height:2px;background:var(--jc-gray);top:50%;left:calc(100% + 20px);transform:translateY(-50%);left:auto}.jc-Comment-actionList{list-style:none;margin-left:3rem}.jc-Comment-actionButton{color:var(--jc-gray-dark);font-size:.85em}.jc-Comment-replyForm{margin-top:1rem}.jc-Comment-details{display:flex;align-items:center;color:var(--jc-gray-dark);margin-bottom:15px}.jc-Comment-content{padding-bottom:12px}.jc-Comment-name{margin:0;font-weight:700}.jc-Comment-date,.jc-Comment-statusMessage{line-height:normal;color:var(--jc-gray-dark);display:block}.jc-Comment-date{font-size:.9em}.jc-Comment-statusMessage{margin:0;padding:0 0 0 .5rem;font-style:italic}.jc-Comment-meta{opacity:.6;display:flex;align-items:center;padding:0 .045rem 0 .8rem}.jc-Comment-anchor{font-weight:700;text-decoration:none;color:inherit;font-weight:400}.jc-Comment-anchor:visited{color:inherit}.jc-Comment-content>*+*{margin-top:var(--jc-paragraph-spacing)}.jc-Comment-content img{max-width:100%}.jc-CommentBox{position:relative;--jc-editor-padding: 8px}.jc-CommentBox--reply{margin-top:1rem}.jc-CommentBox-loadingDots{position:absolute;top:50%;left:50%;transform:translate(-50%);z-index:1}.jc-CommentBox-loadingDots svg{fill:var(--jc-loading-color)}.jc-CommentBox-preview{padding:var(--jc-editor-padding);border:2px solid var(--jc-gray);border-radius:var(--jc-border-radius)}.jc-CommentBox-preview>*{padding-top:0;margin-top:0}.jc-CommentBox-preview>*+*{margin-top:var(--jc-paragraph-spacing)}.jc-CommentBox-preview img{max-width:100%}.jc-CommentBox-previewLoader{text-align:center}.jc-CommentBox-form.is-disabled,.jc-CommentBox-form.is-loading input,.jc-CommentBox-form.is-loading textarea{opacity:.5;pointer-events:none}.jc-CommentBox-form.is-loading input:focus,.jc-CommentBox-form.is-loading textarea:focus{outline:none}.jc-CommentBox-heading{margin-top:1rem}.jc-CommentBox-belowCommentWrapper{text-align:right;padding-top:5px;font-size:.65em;display:flex;justify-content:flex-end}.jc-CommentBox-belowCommentWrapper a,.jc-CommentBox-belowCommentWrapper button{background:none;padding:0;border:0;font-family:inherit;text-decoration:none;color:inherit}.jc-CommentBox-attribution{opacity:.5}.jc-CommentBox-inputs.is-submitting{opacity:.15;pointer-events:none}.jc-CommentBox-inputWrapper{display:grid;grid-template-columns:1fr;grid-gap:1rem;margin-bottom:30px}@media screen and (min-width: 800px){.jc-CommentBox-inputWrapper{grid-template-columns:1fr 1fr}}.jc-CommentBox input,.jc-CommentBox textarea{border:2px solid var(--jc-gray);border-radius:var(--jc-border-radius);padding:var(--jc-editor-padding);width:auto}.jc-CommentBox textarea{min-height:100px}.jc-CommentBox-textarea{position:relative;background:var(--jc-editor-background)}.jc-CommentBox-textarea,.jc-CommentBox-buttonWrapper{grid-column:1/-1}.jc-CommentBox-label{display:flex;flex-direction:column}.jc-CommentBox-label input{margin-top:3px}.jc-CommentBox-buttonWrapper{display:flex;align-items:center}.jc-CommentBox-submitWrapper{display:flex;justify-content:space-between;grid-column:1/-1;margin-top:.75rem}.jc-Comment-replyForm .jc-CommentBox-contactInput{display:flex}.jc-CommentList{overflow:hidden}.jc-CommentList--replies{padding:2rem 0 0 1rem}.jc-CommentList--replies.stop-padding{padding-left:0}@media screen and (min-width: 800px){.jc-CommentList--replies{padding:2rem 0 0 2rem}}.jc-CommentList-count{position:relative;display:inline-block;margin-bottom:1rem}.jc-CommentList-count:after{content:"";display:block;position:absolute;width:100vw;height:2px;background:var(--jc-gray);top:50%;left:calc(100% + 20px);transform:translateY(-50%)}.jc-CommentList-list{list-style:none;padding:0;margin:0;display:flex;flex-direction:column;gap:2rem}.jc-CommentList-item{position:relative}.jc-Message{margin-bottom:1rem;margin-top:1rem;border-radius:var(--jc-border-radius);padding:.75rem 1rem}.jc-Message p{margin:0;color:inherit}.jc-Message--error{background:var(--jc-red);color:var(--jc-alert-color)}.jc-Message--success{background:var(--jc-green);color:var(--jc-alert-color)}.jc-Auth{position:relative;cursor:pointer}.jc-Auth a{color:var(--jc-gray-dark);text-decoration:none}.jc-Auth-caret{width:20px;height:20px;transform:rotate(180deg)}.jc-Auth-caret.is-open{transform:none}.jc-Auth-container{display:flex;align-items:center;gap:5px}.jc-Auth-actions{width:-moz-max-content;width:max-content;box-sizing:border-box;margin:0;list-style:none;border:2px solid var(--jc-gray);border-radius:var(--jc-border-radius);position:absolute;right:0;z-index:100;background:white;min-width:100%;padding:.5rem;top:calc(100% + .5rem)}.jc-Auth-actions a{display:block;padding:.5rem 1rem;text-decoration:none;color:var(--jc-gray-dark);border-radius:var(--jc-border-radius);transition:background-color .2s ease-in-out}.jc-Auth-actions a:hover{background-color:var(--jc-gray)}.jc-Avatar{width:40px;height:40px;border-radius:50%;overflow:hidden}.jc-Avatar-img{-o-object-fit:cover;object-fit:cover;width:100%;height:100%}[jc-cloak]{display:none!important}.jc-Shell{--jc-red: #ba274a;--jc-green: #5bba6f;--jc-gray: #dfdede;--jc-gray-dark: #5c5c5c;--jc-border-radius: 5px;--jc-loading-color: var(--jc-gray);--jc-max-width: 600px;--jc-paragraph-spacing: 16px;--jc-alert-color: white;--jc-editor-background: white;margin:2rem 0}.jc-Shell-container{margin:0 auto;max-width:var(--jc-max-width)}.jc-Shell .jc-Shell{margin:0}.jc-Shell .is-hidden{display:none}.jc-Password{position:absolute;z-index:-1;opacity:0}