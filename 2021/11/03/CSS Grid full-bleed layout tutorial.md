> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.joshwcomeau.com](https://www.joshwcomeau.com/css/full-bleed/)

> Certain layouts are surprisingly dastardly. On the modern web, one of the most common layouts is also......

Back in the day, there was a gold-standard website layout that everyone strived to create, but that was notoriously difficult to get right: the .

It doesn't seem like it would be so tricky, right? But this was an era before flexbox existed; our tools for the job were tables and floats, and neither were really up to this task. It was technically possible, but some shenanigans were required.

Once flexbox achieved mainstream browser support, this layout went from "holy grail" to "fountain drink"; it was everywhere, because it offered a great user experience, and was within reach for all developers.

As the web has evolved, I've discovered a new aspirational layout. It offers a _fantastic_ user experience, especially for long-form text content like news articles or documentation. But, like its predecessor, it's been deceptively hard to achieve; most implementations require obscure hacks or counterintuitive tricks.

I recently discovered an elegant solution to this problem using CSS Grid. In this post, we'll learn how it works!

Have you ever tried to read Wikipedia on a very large screen? It looks like this:

![](https://www.joshwcomeau.com/images/full-bleed/wikipedia.png)

Those paragraphs are so wide! Wikipedia doesn't constrain the container width at all. This leads to lines that are hundreds of characters in length.

It's hard for our eyes to wrap around when we reach the end of a line. If you're like me, you wind up using your mouse to assist:

In addition to the line-wrapping concern, it's just generally hard to read lines of text that are so wide; it fatigues the eye.

[Research has shown](https://readings.design/PDF/the_elements_of_typographic_style.pdf) that the ideal line length is about 65 characters. Anywhere between 45 and 85 is generally seen as acceptable, in the context of a roman alphabet. Reading is a [complex process](https://www.smashingmagazine.com/2014/09/balancing-line-length-font-size-responsive-web-design/), and we should strive to make it as easy as possible.

The standard solution to this problem is to create a single fixed-width column in the center of the page. You've seen this layout everywhere: online magazines, documentation, news sites, and blogs. You're looking at it right now, on this site!

![](https://www.joshwcomeau.com/images/full-bleed/this-website.png)

There's a complicating factor, however—_not all content should be constrained_. We should allow images, videos, and custom widgets to break free and fill the available width:

![](https://www.joshwcomeau.com/images/full-bleed/meerkat.jpg)

This meerkat used as an example of a child that breaks free of constraints. Photo By Sean Paul Kinnear.

The common term for this kind of thing is “full-bleed”. It's a term borrowed from the publishing world; when something is printed full-bleed, it extends to the very edge of the paper.

This new requirement makes the problem considerably more tricky. It's relatively easy to constrain _all_ children, but CSS doesn't really have a mechanism to selectively constrain _some_ children.

Let's start at the end, with our solution:

These styles are assigned to markup in this shape:

There's a lot to unpack here, so let's go through it step by step.

If you're not familiar with CSS Grid, this might seem like a lot of random characters and keywords. Never fear! All will be explained.

`grid-template-columns` is a property that lets us define the shape of our grid. By providing 3 discrete values, we're indicating that we want 3 columns.

The values define the width of each column. The first column is `1fr`, same as the last column. The `fr` unit is a flexible unit that fills available space. It's similar in principle to `flex-grow`; it's a ratio of how much of the free space the column should consume.

Our center column is a fixed width. We use the `min` helper to pick whichever value winds up being smaller. On large screens, it will take up `65ch` width. On smaller screens, where there isn't enough horizontal space for 65 characters, it is clamped to 100% of the available container width.

(If you use Sass, the `min` keyword won't work properly, because it's already a helper in the preprocessor. [View a workaround](https://github.com/sass/node-sass/issues/2815#issuecomment-575926329).)

Here's what this looks like, in practice:

### 

Link to this heading

Assigned children column

We've defined a 3-column grid, and now it's time to assign children to it.

By default, children will be slotted into the first available grid cell. We want to override this default behaviour though; all children should sit in the center column, leaving the first and third columns empty.

In CSS Grid, columns are 1-indexed, so `2` is a reference to the middle column.

The asterisk (*) is a wildcard; it'll match elements of all types. We're saying that every child should be assigned to that second middle column. Each new child will create a new row, like so:

We've seen how our grid can constrain elements of all types, but what about when we want a child to break free and fill the available width?

That's where this fella comes in:

This special `.full-bleed` class allows a specific child to bust out of that column, and span all 3 columns. `1 / 4` is a start/end syntax; we're saying the element should start on Column 1 (inclusive) and span all the way to Column 4 (exclusive).

The trick is that each child becomes its own grid row, and each child can fill as much of that row as it wants. Most elements will only ever occupy that center column, but some will instead span all 3.

On smaller screen-sizes, we want to add a bit of padding, so that our text isn't right at the edge of the display.

We can accomplish this in a number of ways, including straight-up using `padding`! But since this is an article about CSS Grid, let's check out the grid-based solution:

`grid-column-gap` is a nifty property which lets us add gutters between columns. In this case, I've hardcoded a value of 32px.

This introduces a problem, though; because of this extra space, we'll wind up with the dreaded horizontal scrollbar on mobile 😡

The reason is that _gaps are outside columns_. We're telling our column to consume 100% of the available width, **and** we're saying that the column should have 32px of space on each side of it. If our device has a 500px-wide screen, we'll wind up rendering 564px worth of stuff (500px column + 32px per side).

We can solve this problem with an absolutely magical CSS property, `calc`.

`calc` lets us mix units of different types. In this case, what we want is to clamp our center column to be 64px shorter than the full width. We can derive this value with `calc(100% - 64px)`. This will become our new minimum width for mobile devices.

Link to this heading

CSS for JavaScript Developers
-----------------------------------------------------

Let me ask you a question: how comfortable do you feel with CSS?

I know a lot of JS devs who find CSS frustrating and anxiety-inducing. They're proficient with the basics, but they can't seem to develop mastery, to be able to write CSS with confidence.

I was in this boat, and I was sick of it. So I went deep, learning about how the browser interprets and renders CSS. It took years, but the end result is that I feel _so much more confident_ with the language now.

I've distilled this knowledge into an online course called [CSS for JavaScript Developers](https://css-for-js.dev/). It's an _interactive course_, leveraging the same techniques I use on this blog to build dynamic, interactive content. I'm so confident in this idea, I left my job as a staff software engineer at Gatsby Inc. to work on it full-time 😱

CSS Grid is super powerful, and now that it's achieved [wide browser support](https://caniuse.com/css-grid), it can solve so many of our problems!

Some readers have suggested that the same effect could be accomplished with flexbox, or without the use of a wrapper at all. Unfortunately, there are some trade-offs that make those alternatives unworkable. It's beyond the scope of this article, but I wrote up a couple points in [this HackerNews comment](https://news.ycombinator.com/item?id=24691566).

The historical solution to this problem uses negative margins. It works perfectly well, but it feels a bit hacky to me. You can read more about that solution [on CSS Tricks](https://css-tricks.com/full-width-containers-limited-width-parents/).

As I focus on CSS for my [upcoming course](https://css-for-js.dev/), I'll undoubtedly have lots of additional blog posts about it! Be sure to join the newsletter if you'd like to learn more 💖