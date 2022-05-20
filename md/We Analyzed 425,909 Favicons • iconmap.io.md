> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [iconmap.io](https://iconmap.io/blog)

> This is the analysis for our gigantic icon map. Send feedback to help AT gurge.com.

This is the analysis for our [gigantic icon map](https://iconmap.io/). Send feedback to help AT gurge.com.

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/montage_2400x1600.jpg)

this is only 20,000 favicons

Websites crawled: 100,000

Favicons downloaded: 425,909

The humble favicon was messily birthed with the pernicious IE5 release. Since that fateful day, browsers have slowly expanded favicon technology to encompass many wildly differing and lightly documented use cases. Here in 2021 favicons are found primarily in browser tabs, home screens, and Google search results, but they continue to pop up in the strangest places.

Recently my team was tasked with building a favicon fetcher. As a warmup, I looked to see how Chrome handles favicon loading. Do you know that the favicon loader in Chrome is many thousands of lines of code? Why is it so complicated?

We realized we knew very little about the favicon ecosystem. Eventually we decided to fetch the [Tranco top 100,000 websites](https://tranco-list.eu/) and analyze their favicons. We checked each home page for favicons, Apple touch icons, and manifest icons. We also examined fallback locations like `/favicon.ico`. Here’s a quick table to catch you up:

<table><thead><tr><th>where</th><th>what</th></tr></thead><tbody><tr><td>/favicon.ico</td><td>fallback icon file</td></tr><tr><td>/apple-touch-icon.png</td><td>fallback Apple touch icon</td></tr><tr><td>/apple-touch-icon-precomposed.png (obsolete)</td><td>same, but “precomposed”</td></tr><tr><td>link rel=”icon”</td><td>link to an icon file from HTML</td></tr><tr><td>link rel=“apple-touch-icon”</td><td>link to an Apple touch icon from HTML</td></tr><tr><td>link rel=“apple-touch-icon-precomposed” (obsolete)</td><td>same, but “precomposed”</td></tr><tr><td>icons from link rel=“manifest” JSON</td><td>for PWAs</td></tr></tbody></table>

Table of Contents
-----------------

*   [Before We Begin - Favicon Best Practices](#before-we-begin---favicon-best-practices)
*   [Speed, File Size & Resolution](#speed-file-size--resolution)
*   [ICO Files](#ico-files)
*   [Image Formats](#image-formats)
*   [Icon HTML](#icon-html)
*   [Dominant Color](#dominant-color)
*   [Methodology](#methodology)
*   [About Us](#about-us)
*   [Further Reading](#further-reading)

Before We Begin - Favicon Best Practices
----------------------------------------

There are many excellent, modern articles that explain favicon best practices in painstaking detail. This is not one of them. Instead, I invite you to read Masa Kudamatsu’s obsessively researched [How to Favicon in 2021](https://dev.to/masakudamatsu/favicon-nightmare-how-to-maintain-sanity-3al7).

Speed, File Size & Resolution
-----------------------------

The average favicon network request takes 130ms, at least from our speedy cloud instance. The top 1,000 websites do a bit better, averaging 80ms. The median favicon file is 1.9k bytes. Not everyone is so considerate, though. [Discord’s favicon](https://discord.com/assets/847541504914fd33810e70a0ea73177e.ico) is a 280k monster. Now that the popular chat app is worth $15B, they should follow Microsoft’s grudging advice from 2007 and turn that 256x256 into a PNG. I bet that would cut down on their Cloudflare bill.

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/discord.png)

The big winner is the mesmerizing [favicon from eventhorizontelescope.org](https://eventhorizontelescope.org/files/log_2048.png), clocking in at a hefty 7MB. When I downloaded this favicon, the density of my machine increased and nearly collapsed into a black hole.

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/eventhorizontelescope_org.jpg)

When we examine all icon files (including apple touch and manifest icons), we find that many websites have higher resolutions. 52.5% of websites have at least one icon that’s 128x128 or greater. 18.3% of websites have one that’s 256x256 or greater.

It’s possible to go overboard, though. Lufthansa has an apple touch icon that’s 7087x5197. The NFL favicon is a bruising 2000x2000.

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/nfl.png)

Hat tip to VSCode, the GOAT icon that fits into a svelte 112 bytes:

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/vscode.png)

BTW, 2% of the favicons we examined are not square. Here’s a weird 40x31 specimen from Zendesk:

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/zendesk.png)

Also see the incredibly lazy Apple touch icon from Huge Domains, which simply redirects to their logo. Guys, the three people who added your website to their iOS home screens are not happy:

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/hugedomains.jpg)

ICO Files
---------

The ICO file format exhibits surprising staying power. Predating PNG by over a decade, I’ve come to love ICO files despite their many foibles. Foibles which include the [lack of a magic number](https://github.com/ImageMagick/ImageMagick/issues/4230), but I digress.

56% of ICO files contain exactly one image, which isn’t shocking. You know what’s shocking? The [hola.org favicon](https://hola.org/favicon.ico), which contains 20 images. A 12x12, 13x13, 14x14, 15x15, 16x16, 17x17, 18x18, 19x19, 20x20, 21x21, 22x22, 23x23…

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/hola_org.jpg)

Check out this startling ICO with 64 images, all roughly 16x16. I suspect a bug.

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/rajbhasha_net.jpg)

The original goal of the survey was to identify trends in ICO file resolutions. I am pleased to present the definitive answer. What percentage of ICO files contain images of various resolutions?

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/ICO%20File%20Has%20This%20Size.svg)

The most common combinations are 16x16, 16x16+32x32+48x48, 16x16+32x32, and 32x32 in that order. These combinations account for 79% of all ICO files.

Image Formats
-------------

The vast majority of the favicons offered up by websites are PNG. 71.6% of `<link rel=”icon”>` images are PNG. 21.1% of `/favicon.ico` files are secretly PNGs, including [Reddit’s](https://www.reddit.com/favicon.ico). Strangely, only 96.1% of Apple touch icons are PNG. Presumably the other 4% are broken.

Some websites are deeply confused about formats. For example, the [uts.edu.au PNG icon](https://www.uts.edu.au/themes/uts_theme/favicon/favicon-16x16.png) is in fact a Photoshop PSD file. A few stalwarts cling to decrepit BMP files, too lazy to upgrade.

SVG is on the move, held back by lack of support in Safari. 1.1% of `<link rel=”icon”>` files are SVG, increasing to 2.5% for the top 1,000 websites. This is definitely the way to go. Also, 9% of all websites have an SVG mask-icon for Safari.

Unfortunately, the ballooning favicon ecosystem has led to a commensurate rise in the number of icons. For example, [Seeking Alpha](https://seekingalpha.com/) serves up 26 identical favicon images of varying size. Helpful tip - browsers can resize just as well as Photoshop these days:

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/seekingalpha_com.jpg) ![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/seekingalpha_src.png)

Not to be outdone, [appmaker.xyz](https://appmaker.xyz/) serves up 141 nearly identical favicons for your enjoyment:

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/appmaker_xyz.jpg)

Icon HTML
---------

You know, it’s tough to write a web browser. Your browser is expected to display divs, log every click for ad targeting, and also render favicons. To render favicons you rely on the HTML supplied by web pages. Unfortunately, web developers (like me) are incompetent. Not just the little guys, even top tech companies can’t get their HTML right.

Youtube says their icon is 144x144, but it’s 145x145.

```
<link rel="icon" href=".../favicon_144x144.png" sizes="144x144" />

$ identify https://www.youtube.com/s/desktop/3f03b58f/img/favicon_144x144.png
favicon_144x144.png PNG 145x145 145x145+0+0 8-bit sRGB 1664B 0.000u 0:00.000
```

Twitter says their favicon is an image/x-icon, but it’s a PNG.

```
<link rel="shortcut icon" href=".../twitter.ico" type="image/x-icon" />

$ identify https://abs.twimg.com/favicons/twitter.ico twitter.ico PNG 32x32
32x32+0+0 8-bit sRGB 912B 0.000u 0:00.000
```

The loosely worded [favicon section of the HTML living standard](https://html.spec.whatwg.org/multipage/links.html#rel-icon) makes it clear that the `sizes=xxx` and `types=xxx` attributes are just hints. Hints that the browser is free to ignore. In fact, I recommend that browsers ignore these hints because they are wrong much of the time. We calculated a 6.7% error rate for these attributes, where the size is not found in the image or the type does not match the file format.

74% of websites have a `<link rel=”icon”>`. 43% of websites have an apple touch icon. 13% still have the crusty precomposed variant. The winner for shortest favicon URL goes to [https://exe.io/fv.ico](https://exe.io/fv.ico), while the longest URLs are at facebookblueprint.com. Those bad boys are pushing 800 characters.

0.2% of pages had a favicon with a data url. This portly 403 from [telia.se](https://telia.se/) has a tasteful 140k favicon embedded in a data url.

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/140k.jpg)

Dominant Color
--------------

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/montage_top_sites.jpg)

We did a hacky image analysis with ImageMagick to survey favicon colors. Here is the dominant color breakdown across our favicons. This isn’t very accurate, unfortunately. I suspect that many multicolored favicons are getting lumped in with purple.

![](https://d3fdzqvmwexz0m.cloudfront.net/images/blog/survey/Dominant%20Icon%20Color.svg)

Methodology
-----------

We used Go to fetch and normalize favicons. HTTP responses were cached on disk with [gohttpdisk](https://github.com/gurgeous/gohttpdisk). The crawler outputs a CSV. We used Ruby to analyze the results since Go is terrible at data wrangling. We looked for favicons, Apple touch icons, and manifest icons including fallback paths like `/favicon.ico`. For each domain in the [Tranco top 100,000](https://tranco-list.eu/), we tried both `https://domain.com` and `https://www.domain.com` before giving up. ImageMagick was used for image processing, as always.

About Us
--------

This analysis and the gigantic icon map were created by Adam, Nathan & Patrick. Enjoy!

Further Reading
---------------

*   [wikipedia Favicon entry](https://en.wikipedia.org/wiki/Favicon) & [HTML living standard](https://html.spec.whatwg.org/multipage/links.html#rel-icon)
*   [How to Favicon in 2021](https://dev.to/masakudamatsu/favicon-nightmare-how-to-maintain-sanity-3al7) by Masa Kudamatsu
*   tools like [RealFaviconGenerator](https://realfavicongenerator.net/) and [besticon](https://github.com/mat/besticon)
*   favicon APIs from [Google](https://www.google.com/s2/favicons?sz=64&domain_url=netflix.com), [DuckDuckGo](https://icons.duckduckgo.com/ip3/www.netflix.com.ico), [Icon Horse](https://icon.horse/)