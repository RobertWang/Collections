> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.joshwcomeau.com](https://www.joshwcomeau.com/css/custom-css-reset/)

> I have a set of baseline CSS styles that come with me from project to project. In the past, I'd use a......

Whenever I start a new project, the first order of business is to sand down some of the rough edges in the CSS language. I do this with a functional set of custom baseline styles.

For a long time, I used Eric Meyer's famous [CSS Reset](https://meyerweb.com/eric/tools/css/reset). It's a solid chunk of CSS, but it's a bit long in the tooth at this point; it hasn't been updated in more than a decade, and _a lot_ has changed since then!

Recently, I've been using my own custom CSS reset. It includes all of the little tricks I've discovered to improve both the user experience and the CSS authoring experience.

Like other CSS resets, it's unopinionated when it comes to design / cosmetics. You can use this reset for any project, no matter the aesthetic you're going for.

In this tutorial, we'll go on a tour of my custom CSS reset. We'll dig into each rule, and you'll learn what it does and why you might want to use it!

Without further ado, here it is:

```
css

*, *::before, *::after {

  box-sizing: border-box;

}

* {

  margin: 0;

}

html, body {

  height: 100%;

}

body {

  line-height: 1.5;

  -webkit-font-smoothing: antialiased;

}

img, picture, video, canvas, svg {

  display: block;

  max-width: 100%;

}

input, button, textarea, select {

  font: inherit;

}

p, h1, h2, h3, h4, h5, h6 {

  overflow-wrap: break-word;

}

#root, #__next {

  isolation: isolate;

}
```

It's relatively short, but there's _a lot of stuff_ packed into this small stylesheet. Let's get into it!

Pop quiz! Measuring by the visible pink border, how wide is the `.box` element in the following scenario, assuming no other CSS has been applied?

html

```
<style>

  .parent {

    width: 200px;

  }

  .box {

    width: 100%;

    border: 2px solid hotpink;

    padding: 20px;

  }

</style>

<div>

  <div></div>

</div>
```

Our `.box` element has `width: 100%`. Because its parent is 200px wide, that 100% will resolve to 200px.

But _where does it apply that 200px width?_ By default, it will apply that size to the _content box_.

If you're not familiar, the “content box” is the rectangle in the box model that actually holds the content, inside the border and the padding:

![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNTY1LjY2NjY2NjY2NjY2NjYiIGhlaWdodD0iMjA5LjY2NjY2NjY2NjY2NjY2IiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZlcnNpb249IjEuMSIvPg==)![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

The `width: 100%` declaration will set the `.box`'s content-box to 200px. The padding will add an extra 40px (20px on each side). The border adds a final 4px (2px on each side). When we do the math, the visible pink rectangle will be 244px wide.

When we try and cram a 244px box into a 200px-wide parent, it overflows:

### Code Playground

```
<style>

  .parent {

    width: 200px;

    border: 2px solid black;

  }

  .box {

    width: 100%;

    border: 2px solid hotpink;

    padding: 20px;

  }

</style>

<div>

  <div></div>

</div>
```

This behavior is weird, right? Fortunately, we can change it, by setting the following rule:

```
css

*, *::before, *::after {

  box-sizing: border-box;

}
```

With this rule applied, percentages will resolve based on the _border-box_. In the example above, our pink box would be 200px, and the inner content-box would shrink down to 156px (200px - 40px - 4px).

**This is a must-have rule, in my opinion.** It makes CSS _significantly_ nicer to work with.

We apply it to all elements and pseudo-elements using the wildcard selector (`*`). Contrary to popular belief, this is [not bad for performance](https://www.paulirish.com/2012/box-sizing-border-box-ftw/).

### 

Link to this heading

2. Remove default margin

Browsers make common-sense assumptions around margin. For example, an `h1` will include more margin by default than a paragraph.

These assumptions are reasonable within the context of a word-processing document, but they might not be accurate for a modern web application.

Margin is a [tricky devil](https://www.joshwcomeau.com/css/rules-of-margin-collapse/), and more often than not, I find myself wishing elements didn't have any by default. So I've decided to remove it all. 🔥

If/when I do want to add some margin to specific tags, I can do so in my custom project styles. The wildcard selector (`*`) has extremely low specificity, so it'll be easy to override this rule.

### 

Link to this heading

3. Percentage-based heights

```
css

html, body {

  height: 100%;

}
```

Have you ever tried to use a percentage-based height in CSS, only to discover that it seems to have no effect?

Here's an example:

### Code Playground

```
<style>

  main {

    outline: solid red;

    min-height: 100%;

  }

</style>

<main>

  <p>Hello World</p>

</main>
```

The `main` element has `height: 100%`, but the element doesn't grow at all!

This doesn't work because in Flow layout (the primary layout mode in CSS), `height` and `width` operate on fundamentally different principles. The width of an element is calculated based on its parent, but the _height_ of an element is calculated based on its _children_.

This is a complicated topic, and it's way beyond the scope of this article. I plan on writing a blog post about it, but in the meantime, you can learn all about it in my CSS course, [CSS for JavaScript Developers](https://css-for-js.dev/).

As a quick demo, here we see that our `main` element can grow when we apply this rule:

### Code Playground

```
<style>

  html, body {

    height: 100%;

  }

  main {

    outline: solid red;

    min-height: 100%;

  }

</style>

<main>

  <p>Hello World</p>

</main>
```

If you're using a JS framework like React, you might also wish to add a third selector to this rule: the root-level element used by the framework.

For example, in my Next.js projects, I update the rule as follows:

```
css

html, body, #__next {

  height: 100%;

}
```

### 

Link to this heading

4 Tweaking line-height

```
css

body {

  line-height: 1.5;

}
```

`line-height` controls the vertical spacing between each line of text in a paragraph. The default value varies between browsers, but it tends to be around 1.2.

This unitless number is a ratio based on the font size. It functions just like the `em` unit. With a `line-height` of 1.2, each line will be 20% larger than the element's font size.

Here's the problem: for those who are dyslexic, these lines are packed too closely together, making it harder to read. The WCAG criteria states that [line-height should be at least 1.5](https://www.w3.org/WAI/WCAG21/Understanding/text-spacing.html).

Now, this number does tend to produce quite-large lines on headings and other elements with large type:

### Code Playground

```
<style>

  * {

    line-height: 1.5;

  }

</style>

<p>

  This paragraph has a 1.5x ratio

  for line-height, and it feels

  pretty good, right? I think

  this text is legible and pleasant.

</p>

<h1>

  But it's a bit much on headings!

</h1>
```

You may wish to override this value on headings. My understanding is that the WCAG criteria is meant for "body" text, not headings.

```
css

body {

  -webkit-font-smoothing: antialiased;

}
```

Alright, so this one is a bit controversial.

On MacOS computers, the browser will use “subpixel antialiasing” by default. This is a technique that aims to make text easier to read, by leveraging the R/G/B lights within each pixel.

In the past, this was seen as an accessibility win, because it improved text contrast. You may have read a popular blog post, [Stop “Fixing” Font Smoothing](https://usabilitypost.com/2012/11/05/stop-fixing-font-smoothing/), that advocates _against_ switching to “antialiased”.

Here's the problem: that article was written in 2012, before the era of high-DPI “retina” displays. Today's pixels are much smaller, invisible to the naked eye.

The physical arrangement of pixel LEDs has changed as well. If you look at a modern monitor under a microscope, you won't see an orderly grid of R/G/B lines anymore.

In MacOS Mojave, released in 2018, **Apple disabled subpixel antialiasing across the operating system**. I'm guessing they realized that it was doing more harm than good on modern hardware.

Confusingly, MacOS browsers like Chrome and Safari still use subpixel antialiasing by default. We need to explicitly turn it off, by setting `-webkit-font-smoothing` to `antialiased`.

Here's the difference:

### Antialiasing

![](https://www.joshwcomeau.com/images/custom-css-reset/subpixel.png)

### Subpixel Antialiasing

![](https://www.joshwcomeau.com/images/custom-css-reset/antialiased.png)

MacOS is the only operating system to use subpixel-antialiasing, and so this rule has no effect on Windows, Linux, or mobile devices. If you're on a MacOS computer, you can experiment with a live render:

### Code Playground

```
<p>

  This paragraph uses the default subpixel antialiasing.

</p>

<p>

  This paragraph does not use subpixel antialiasing.

</p>
```

### 

Link to this heading

6. Sensible media defaults

```
css

img, picture, video, canvas, svg {

  display: block;

  max-width: 100%;

}
```

So here's a weird thing: images are considered"inline" elements. This implies that they should be used in the middle of paragraphs, like `<em>` or `<strong>`.

This doesn't jive with how I use images most of the time. Typically, I treat images the same way I treat paragraphs or headers or sidebars; they're layout elements.

If we try to use an inline element in our layout, though, weird things happen. If you've ever had a mysterious 4px gap that wasn't margin or padding or border, it was probably the “inline magic space” that browsers add with `line-height`.

By setting `display: block` on all images by default, we sidestep a whole category of funky issues.

I also set `max-width: 100%`. This is done to keep large images from overflowing, if they're placed in a container that isn't wide enough to contain them.

Most block-level elements will automatically grow/shrink to fit their parent, but media elements like `<img>` are special: they're known as _replaced elements_, and they don't follow the same rules.

If an image has a "native" size of 800×600, the `<img>` element will also be 800px wide, even if we plop it into a 500px-wide parent.

This rule will prevent that image from growing beyond its container, which feels like much more sensible default behavior to me.

### 

Link to this heading

7. Inherit fonts for form controls

```
css

input, button, textarea, select {

  font: inherit;

}
```

Here's another weird thing: by default, buttons and inputs don't inherit typographical styles from their parents. Instead, they have their own _weird_ styles.

For example, `<textarea>` will use the system-default monospace font. Text inputs will use the system-default sans-serif font. And both will choose a microscopically-small font size (13.333px in Chrome).

As you might imagine, it's very hard to read 13px text on a mobile device. When we focus an input with a small font size, the browser will automatically zoom in, so that the text is easier to read.

Unfortunately, this is not a good experience:

If we want to avoid this auto-zoom behavior, the inputs need to have a font-size of at least 1rem / 16px. Here's one way to address the issue:

```
css

input, button, textarea, select {

  font-size: 1rem;

}
```

This fixes the auto-zoom issue, but it's a band-aid. Let's address the root cause instead: form inputs shouldn't have their own typographical styles!

```
css

input, button, textarea, select {

  font: inherit;

}
```

`font` is a rarely-used shorthand that sets a bunch of font-related properties, like `font-size`, `font-weight`, and `font-family`. By setting it to `inherit`, we instruct these elements to match the typography in their surrounding environment.

As long as we don't choose an obnoxiously small font size for our body text, this solves all of our issues at once. 🎉

```
css

p, h1, h2, h3, h4, h5, h6 {

  overflow-wrap: break-word;

}
```

In CSS, text will automatically line-wrap if there isn't enough space to fit all of the characters on a single line.

By default, the algorithm will look for “soft wrap” opportunities; these are the characters that the algorithm can split on. In English, the only soft wrap opportunities are whitespace and hyphens, but this varies from language to language.

If a line doesn't have any soft wrap opportunities, and it doesn't fit, it will cause the text to overflow:

### Code Playground

```
<div>

  <p>

    This is a narrow column of text, with a very long word: antidisestablishmentarianism.

  </p>

  <p>

    The same problem happens with URLs: https://www.somewebsite.com/articles/a1b2c3

  </p>

</div>
```

This can cause some annoying layout issues. Here, it adds a horizontal scrollbar. In other situations, it might cause text to overlap other elements, or slip behind an image/video.

The `overflow-wrap` property lets us tweak the line-wrapping algorithm, and give it permission to use hard wraps when no soft wrap opportunties can be found:

### Code Playground

```
<style>

  p {

    overflow-wrap: break-word;

  }

</style>

<div>

  <p>

    This is a narrow column of text, with a very long word: antidisestablishmentarianism.

  </p>

  <p>

    The same problem happens with URLs: https://www.somewebsite.com/articles/a1b2c3

  </p>

</div>
```

Neither solution is perfect, but at least hard wrapping won't mess with the layout!

Thanks to Sophie Alpert for [suggesting a similar rule](https://twitter.com/sophiebits/status/1462921205359386628)! She suggests applying it to _all_ elements, which is probably a good idea, but not something I've personally tested.

You can also try adding the `hyphens` property:

```
css

p {

  overflow-wrap: break-word;

  hyphens: auto;

}
```

`hyphens: auto` uses hyphens (in languages that support them) to indicate hard wraps. It also makes hard wraps much more common.

It can be worthwhile if you have very-narrow columns of text, but it can also be a bit distracting. I chose not to include it in the reset, but it's worth experimenting with!

### 

Link to this heading

9. Root stacking context

```
css

#root, #__next {

  isolation: isolate;

}
```

This last one is optional. It's generally only needed if you use a JS framework like React.

As we saw in [“What The Heck, z-index??”](https://www.joshwcomeau.com/css/stacking-contexts/), the `isolation` property allows us to create a new stacking context without needing to set a `z-index`.

This is beneficial since it allows us to guarantee that certain high-priority elements (modals, dropdowns, tooltips) will always show up above the other elements in our application. No weird stacking context bugs, no z-index arms race.

You should tweak the selector to match your framework. We want to select the top-level element that your application is rendered within. For example, create-react-app uses a `<div>`, so the correct selector is `#root`.

Link to this heading

Our finished product
--------------------------------------------

Here's the CSS reset again, in a condensed copy-friendly format:

```
css

*, *::before, *::after {

  box-sizing: border-box;

}

* {

  margin: 0;

}

html, body {

  height: 100%;

}

body {

  line-height: 1.5;

  -webkit-font-smoothing: antialiased;

}

img, picture, video, canvas, svg {

  display: block;

  max-width: 100%;

}

input, button, textarea, select {

  font: inherit;

}

p, h1, h2, h3, h4, h5, h6 {

  overflow-wrap: break-word;

}

#root, #__next {

  isolation: isolate;

}
```

Feel free to copy/paste this into your own projects! It's released without any restrictions, into the public domain (though if you wanted to keep the link to this blog post, I'd appreciate it!).

I chose not to release this CSS reset as an NPM package because I feel like _you should own your reset_. Bring this into your project, and tweak it over time as you learn new things or discover new tricks. You can always make your own NPM package to facilitate reuse across your projects, if you want. Just keep in mind: **you own this code, and it should grow along with you.**

Thanks to Andy Bell for sharing his [Modern CSS Reset](https://piccalil.li/blog/a-modern-css-reset/). It helped tune some of my thinking, and inspired this blog post!

My CSS reset is quite short (only 11 declarations!), and yet I've managed to spend an entire blog post talking about them. And honestly, there's _so much more_ I want to say! We only scratched the surface in a bunch of places.

CSS is a deceptively complex language. Unless you pop the hood and learn what's _really_ going on under there, the language will always feel a bit unpredictable and inconsistent. When your mental model is incomplete, you're bound to run into some problems.

If you take a bit of time to learn how the language _really_ works, though, everything becomes so much more intuitive and predictable. I love writing CSS these days!

For the past year and a half, I've been focused on helping JS developers change their relationship with CSS. I recently launched a comprehensive, interactive online course called **[CSS for JavaScript Developers](https://css-for-js.dev/)**.

[![](https://www.joshwcomeau.com/images/the-importance-of-learning-css/css-for-js-banner.png)](https://css-for-js.dev/)

If you wish you were one of those people who liked/understood CSS, I made this course specifically for you! You can learn all about it on the course homepage:  
[https://css-for-js.dev](https://css-for-js.dev/)