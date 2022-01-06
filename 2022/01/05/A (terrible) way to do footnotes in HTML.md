> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [shkspr.mobi](https://shkspr.mobi/blog/2020/12/a-terrible-way-to-do-footnotes-in-html/)

> I’ve been thinking about better ways to display footnotes in eBooks. So this is my horrible and hacky......

I've been thinking about [better ways to display footnotes in eBooks](https://shkspr.mobi/blog/2020/07/usability-of-footnotes/).

So this is my horrible and hacky way to display inline footnotes in pure HTML and CSS. No Javascript.

Here's how it works:

```
The most cited work in history, for example, is a 1951 paper
   <details>
      <summary>1</summary> 
      Lowry, O. H., Rosebrough, N. J., Farr, A. L. & Randall, R. J. J. Biol. Chem. 193, 265–275 (1951).
   </details>
describing an assay to determine the amount of protein in a solution.
```

I've use `<details>` and `<summary>` to make a collapsible element.

Tap this to learn more about `<details>`

This is a magic element! The HTML Details Element (`<details>`) creates a disclosure widget in which information is visible only when the widget is toggled into an "open" state. A summary or label can be provided using the `<summary>` element.  
[Learn more about it on MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details).  

The problem here is that `<details>` is a [_block level_ element](https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements). By default, starting a `<details>` element will display on a new line.

It also doesn't look much like a footnote yet.

Luckily, I bodged some CSS together to make it inline:

```
details, summary {
  display: inline;
  vertical-align: super;
  font-size: 0.75em;
}
summary {
  cursor: pointer;
}
details[open] {
  display: contents;
}
details[open]::before {
  content: " [";
}
details[open]::after {
  content: "]";
}
```

*   Using `display:inline;` turns these into inline elements - so a newline isn't started.
*   The alignment and font size make it _look_ like a normal footnote.
*   The `cursor: pointer;` just makes it slightly easier for mouse users to see that the element is interactive.
*   When the `<details>` is in the open state, the content will be [displayed as a direct descendant of the parent element](https://caniuse.com/css-display-contents).
*   Add some square brackets to make it more obvious where the footnote begins and ends.

I _could_ have used the [`<sup>` element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/sup) to make the whole note superscript. But that produces [invalid HTML](https://html.spec.whatwg.org/multipage/interactive-elements.html#the-details-element).

The `<details>` element can contain _any_ HTML. So I can add a [fully semantic citation](https://shkspr.mobi/blog/2020/11/introducing-doi2ht-ml-the-simple-semantic-citation-server/).

[Problems](#problems)
---------------------

The "normal" way of doing footnotes in an ePub is to [place them at the end of a section](https://kb.daisy.org/publishing/docs/html/notes.html). Footnote links like fn1 are just hyperlinks. They take the reader to that section of the document. So eReaders will display the footnote's contents on the same page as the footnote's link:  
![](https://shkspr.mobi/blog/wp-content/uploads/2020/07/A-very-long-footnote.png.webp)

That makes it easy for a reader to search through them all, or just read them in order.

Because my style are inline, that makes it hard to see or search for them.

And because my style are interactive elements, they place a physical burden on the user to interact with them.

The main problem is with the rendering engines used by eReaders. I tried a few, but none of them displayed `<details>` as an interactive element.

I'd love to know how it renders on Kindle, Kobo, and other eReaders. Here's the HTML of the test if you'd like to experiment with it and send me a screenshot.

```
<!doctype html>
<html lang="en-gb">
<head><title>Interactive Footnote Test</title>
<style>
details, summary {
  display: inline;
  vertical-align: super;
  font-size: 0.75em;
}
summary {
  cursor: pointer;
}
details[open] {
  display: contents;
}
details[open]::before {
  content: " [";
}
details[open]::after {
  content: "]";
}</style>
</head>
<body>
The most cited work in history, for example, is a 1951 paper<details><summary>1</summary> Lowry, O. H., Rosebrough, N. J., Farr, A. L. & Randall, R. J. J. Biol. Chem. 193, 265–275 (1951).</details> describing an assay to determine the amount of protein in a solution.
<br><br>
Sunlight poured like molten gold<details><summary>2</summary> Not precisely, of course. Trees didn’t burst into flame, people didn’t suddenly become very rich and extremely dead, and the seas didn’t flash into steam. A better simile, in fact, would be ‘not like molten gold.’</details> across the sleeping landscape.
<br><br>
My blog was recently featured in an academic paper<details><summary>3</summary><span itemscope itemtype="http://schema.org/ScholarlyArticle"><span itemprop="citation"><span itemprop="author" itemscope itemtype="http://schema.org/Person"><span itemprop="name"><span itemprop="familyName">Eishita</span><span>, </span><span itemprop="givenName">Farjana Z.</span></span></span><span> & </span><span itemprop="author" itemscope itemtype="http://schema.org/Person"><span itemprop="name"><span itemprop="familyName">Stanley</span><span>, </span><span itemprop="givenName">Kevin G.</span></span></span><span> & </span><span itemprop="author" itemscope itemtype="http://schema.org/Person"><span itemprop="name"><span itemprop="familyName">Esquivel</span><span>, </span><span itemprop="givenName">Alain</span></span></span> <q><cite itemprop="headline">Quantifying the differential impact of sensor noise in augmented reality gaming input</cite></q> <span>(</span><time itemprop="datePublished" datetime="2015">2015</time><span>)</span> <span itemprop="publisher" itemscope itemtype="http://schema.org/Organization"><span itemprop="name">Institute of Electrical and Electronics Engineers (IEEE)</span></span><span>.</span> DOI: <a itemprop="url" href="https://doi.org/10.1109/gem.2015.7377202">https://doi.org/10.1109/gem.2015.7377202</a></span></span></details> which pleased me greatly.
</body>
</html>
```
