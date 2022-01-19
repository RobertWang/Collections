> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [nolanlawson.com](https://nolanlawson.com/2021/12/17/introducing-fuite-a-tool-for-finding-memory-leaks-in-web-apps/)

> Debugging memory leaks in web apps is hard. The tooling exists, but it’s complicated, cumbersome, and......

Debugging memory leaks in web apps is hard. The tooling exists, but it’s complicated, cumbersome, and often doesn’t answer the simple question: _Why is my app leaking memory?_

Because of this, I’d wager that most web developers are not actively monitoring for memory leaks. And of course, if you’re not testing something, it’s easy for bugs to slip through.

When I first started looking into memory leaks, I assumed it was a rare thing. How could JavaScript – a language with an automatic garbage collector – be a big source of memory leaks? But the more I learned, the more I suspected that memory leaks were actually quite common in Single Page Apps (SPAs) – it’s just that nobody is testing for it!

Since most web developers aren’t fiddling with the [Chrome memory tools](https://developer.chrome.com/docs/devtools/memory-problems/) for the fun of it, they probably won’t notice a leak until the browser tab crashes with an Out Of Memory error, or the page slows down, or someone happens to open up the Task Manager and notice that a website is using many megabytes (or even gigabytes!) of memory. But at that point, it’s gotten bad enough that there may be multiple leaks on the same page.

[I’ve written about memory leaks](https://nolanlawson.com/2020/02/19/fixing-memory-leaks-in-web-applications/) in the past, but my advice basically boils down to: “Use the Chrome DevTools, follow these dozen tedious steps, and then _maybe_ you can figure out why your page is leaking.” This is not a great developer experience, and I’m sure many readers just shook their heads in despair and moved on. It would be much better if a tool could find memory leaks automatically.

That’s why I wrote [`fuite`](https://github.com/nolanlawson/fuite) (French for “leak”). `fuite` is a CLI tool that you can point at any URL, and it will analyze the page for memory leaks:

```
npx fuite https://example.com
```

That’s it! By default, it assumes that the site is a client-rendered SPA, and it will crawl the page for internal links (such as `/about` or `/contact`). Then, for each link, it runs the following steps:

1.  Click the link
2.  Press the browser back button
3.  Repeat to see if memory grows

If `fuite` finds any leaks, it will show which objects are suspected of causing the leak:

```
Test         : Go to /foo and back
Memory change: +10 MB
Leak detected: Yes

Leaking objects:

| Object            | # added | Retained size increase |
| ----------------- | ------- | ---------------------- |
| HTMLIFrameElement | 1       | +10 MB                 |

Leaking event listeners:

| Event        | # added | Nodes  |
| ------------ | ------- | ------ |
| beforeunload | 2       | Window |

Leaking DOM nodes:

DOM size grew by 6 node(s)
```

To do this, `fuite` uses the basic strategy outlined in my blog post. It will launch Chrome, run some scenario _n_ number of times (7 by default) and see if any objects are leaking a multiple of _n_ times (7, 14, 21, etc.).

`fuite` will also analyze any Arrays, Objects, Maps, Sets, event listeners, and the overall DOM to see if any of those are leaking. For instance, if an Array grows by exactly 7 after 7 iterations, then it’s probably leaking.

Testing real-world websites
---------------------------

Somewhat surprisingly, the “basic” scenario of clicking internal links and pressing the back button is enough to find memory leaks in many SPAs. I tested `fuite` against the home pages for 10 popular frontend frameworks, and found leaks in all of them:

<table><thead><tr><th>Site</th><th>Leak detected</th><th>Internal links</th><th>Average growth</th><th>Max growth</th></tr></thead><tbody><tr><td>Site 1</td><td>yes</td><td>8</td><td>27.2 kB</td><td>43 kB</td></tr><tr><td>Site 2</td><td>yes</td><td>10</td><td>50.4 kB</td><td>78.9 kB</td></tr><tr><td>Site 3</td><td>yes</td><td>27</td><td>98.8 kB</td><td>135 kB</td></tr><tr><td>Site 4</td><td>yes</td><td>8</td><td>180 kB</td><td>212 kB</td></tr><tr><td>Site 5</td><td>yes</td><td>13</td><td>266 kB</td><td>1.07 MB</td></tr><tr><td>Site 6</td><td>yes</td><td>8</td><td>638 kB</td><td>1.15 MB</td></tr><tr><td>Site 7</td><td>yes</td><td>7</td><td>1.37 MB</td><td>2.25 MB</td></tr><tr><td>Site 8</td><td>yes</td><td>15</td><td>3.49 MB</td><td>4.28 MB</td></tr><tr><td>Site 9</td><td>yes</td><td>43</td><td>5.57 MB</td><td>7.37 MB</td></tr><tr><td>Site 10</td><td>yes</td><td>16</td><td>14.9 MB</td><td>186 MB</td></tr></tbody></table>

In this case, “internal links” refers to the number of internal links tested, “average growth” refers to the average memory growth for every link (i.e. clicking it and then pressing the back button), and “max growth” refers to whichever internal link was leaking the most. Note that these numbers don’t include one-time setup costs, as `fuite` does one preflight iteration before the normal 7 iterations.

To confirm these results yourself, you can use the Chrome DevTools [Memory tab](https://developer.chrome.com/docs/devtools/memory-problems/#discover_detached_dom_tree_memory_leaks_with_heap_snapshots). Here is a screenshot of the worst-performing site from my set, where I click a link, press the back button, take a heap snapshot, and repeat:

![](https://nolanwlawson.files.wordpress.com/2021/12/screenshot-memory.png?w=465)

On this particular site, memory grows by about 6 MB every time you click a link and go back.

To avoid naming and shaming, I haven’t listed the actual websites. The point is just to show a representative sample of some popular SPAs – the authors of those websites are free to run `fuite` themselves and track down these leaks. (Please do!)

Caveats
-------

Note, though, that not every leak in an SPA is an egregious problem that needs to be addressed. SPAs need to, for example, maintain the focus and scroll state to properly support accessibility, which means that there may be some small metadata that is stored for every page navigation. `fuite` will dutifully report such leaks (because they are leaks), but it’s up to the developer to decide if a tiny leak is worth chasing or not.

Some memory growth may also be due to browser-internal changes (such as JITing), which the web page can’t really control. So the memory growth numbers are an imperfect measure of what you stand to gain by fixing leaks – it could very well be that a few kBs of growth are unavoidable. (Although `fuite` tries to ignore browser-internal growth, and will only say “leaks detected” if there is actionable advice for the web developer.)

In rare cases, some memory growth may also be due to outright browser bugs. While analyzing the sites above, I actually found one (Site #4) that seems to be suffering from [this Chrome bug](https://crbug.com/1213045) due to `<img loading="lazy">` not being unloaded. Unfortunately it’d be hard for `fuite` to detect browser bugs, so if you’re mystified by a leak, it’s good to cross-check against other browsers!

Also note that it’s almost impossible for a Multi-Page App (MPA) to leak, because the browser clears memory on every page navigation. (Assuming no [browser bugs](https://stackoverflow.com/a/1886359), of course.) During my testing, I found two frontend frameworks whose home pages were MPAs, and unsurprisingly, `fuite` couldn’t find any leaks in them. These were excluded from the results above.

Memory leaks are more of a concern for SPAs, where memory isn’t cleared automatically on each navigation. `fuite` is primarily designed for SPAs, although you can run it on MPAs too.

`fuite` currently only measures the JavaScript heap memory in the main frame of the page, so cross-origin iframes, Web Workers, and Service Workers are not measured. Something like [`performance.measureUserAgentSpecificMemory()`](https://chromestatus.com/feature/5685965186138112) would be more accurate, but it’s only available in [cross-origin isolated](https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated) contexts, so it’s not practical for a general-purpose tool right now.

Other memory leak scenarios
---------------------------

The “crawl for internal links” scenario is just the default one – you can also build your own. `fuite` is built on top of [Puppeteer](https://pptr.dev/), so for whatever scenario you want to test, you essentially just need to write a Puppeteer script to tell the browser what to do. Some common scenarios you might test are:

*   Open a modal dialog and then close it
*   Hover over an element to show a tooltip, then mouse away to dismiss it
*   Scroll through an infinite-loading list, then navigate away and back
*   Etc.

In each of these scenarios, you would _expect_ memory to be the same before and after. But of course, it’s not always so simple with web apps! You may be surprised how many of your dialogs and tooltips are harboring memory leaks.

To analyze leaks, `fuite` captures [heap snapshot files](https://developer.chrome.com/docs/devtools/memory-problems/heap-snapshots/), which you can load in the Chrome DevTools to inspect. It also has a `--debug` mode that you can use for more fine-grained analysis: stepping through the test as it’s running, debugging the browser in real-time, analyzing the leaking objects, etc.

Under the hood, `fuite` is a fairly basic tool, and I won’t claim that it can do 100% of the work of fixing memory leaks. There is still the human component of figuring out why your objects were allocated and retained, and then finding a reasonable fix. But my goal is to automate ~95% of the work, so that it actually becomes _achievable_ to fix memory leaks in web apps.

You can find `fuite` [on GitHub](https://github.com/nolanlawson/fuite). Happy leak hunting!

_**Update:** I made [a video tutorial](https://youtu.be/H0BHL2lo89M) showing how to debug memory leaks with `fuite`._