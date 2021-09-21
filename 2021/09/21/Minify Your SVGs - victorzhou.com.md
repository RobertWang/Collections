> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [victorzhou.com](https://victorzhou.com/blog/minify-svgs/)

> How I optimize SVGs for this blog and why you probably should, too.

I use a lot of [SVG](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics)s in my blog posts. They’re great for simple diagrams or illustrations, like this one:

![](https://victorzhou.com/media/nn-series/network.svg) From my [Neural Networks From Scratch](https://victorzhou.com/series/neural-networks-from-scratch/) Series.

I use [Inkscape](https://inkscape.org/), a free and open-source vector graphics editor, to make my SVGs. In the beginning, I just saved my SVGs using the default Inkscape format, something called _Inkscape SVG_. That turned out to be not ideal…

Let’s use this SVG of a circle as an example:

![](https://victorzhou.com/media/svg-post/circle.svg)

Here’s the _Inkscape SVG_ markup for that laughably-simple icon:

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>


<svg
   xmlns:dc="http://purl.org/dc/elements/1.1/"
   xmlns:cc="http://creativecommons.org/ns#"
   xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
   xmlns:svg="http://www.w3.org/2000/svg"
   xmlns="http://www.w3.org/2000/svg"
   xmlns:sodipodi="http://sodipodi.sourceforge.net/DTD/sodipodi-0.dtd"
   xmlns:inkscape="http://www.inkscape.org/namespaces/inkscape"
   width="24"
   height="24"
   viewBox="0 0 24 24"
  
   version="1.1"
   inkscape:version="0.91 r13725"
   sodipodi:doc>
  <defs
     />
  <sodipodi:namedview
    
     pagecolor="#ffffff"
     bordercolor="#666666"
     borderopacity="1.0"
     inkscape:pageopacity="0.0"
     inkscape:pageshadow="2"
     inkscape:zoom="33.458333"
     inkscape:cx="12"
     inkscape:cy="12"
     inkscape:document-units="px"
     inkscape:current-layer="layer1"
     showgrid="false"
     units="px"
     inkscape:window-width="1680"
     inkscape:window-height="1005"
     inkscape:window-x="0"
     inkscape:window-y="1"
     inkscape:window-maximized="1" />
  <metadata
    >
    <rdf:RDF>
      <cc:Work
         rdf:about="">
        <dc:format>image/svg+xml</dc:format>
        <dc:type
           rdf:resource="http://purl.org/dc/dcmitype/StillImage" />
        <dc:title></dc:title>
      </cc:Work>
    </rdf:RDF>
  </metadata>
  <g
     inkscape:label="Layer 1"
     inkscape:groupmode="layer"
    >
    <circle
      
      
       cx="12"
       cy="12"
       r="12" />
  </g>
</svg>
```

Wow. That’s **2 KB** of markup for basically nothing.

Eventually (read: after an embarrassingly long time 🤷), I figured out that Inkscape had an _Optimized SVG_ output format. This was much more reasonable - using Inkscape’s default settings, the _Optimized SVG_ markup for our circle is:

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<svg xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#" xmlns="http://www.w3.org/2000/svg" height="24" width="24" version="1.1" xmlns:cc="http://creativecommons.org/ns#" xmlns:dc="http://purl.org/dc/elements/1.1/" viewBox="0 0 24 24">
 <g>
  <circle cx="12" cy="12" r="12"/>
 </g>
</svg>
```

Still, though, that’s **387 bytes** just to draw a 24x24 circle. Surely this isn’t the end of the road, right…?

Of course not. Look, I figured out how to save files in the Optimized SVG format doesn’t warrant its own blog post.

The final evolution of our circle SVG is a result of passing it through [svgo](https://github.com/svg/svgo), a popular Node.js tool specifically for optimizing SVGs:

```
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24"><circle cx="12" cy="12" r="12"/></svg>
```

**102 bytes**! That’s much more like it.

Here’s how all 3 versions of our circle SVG look when rendered:

image/svg+xml

The _Inkscape SVG_ version: 2 KB

The _Optimized SVG_ version: 387 bytes

The final version: 102 bytes

Yup. They’re all just plain black circles, but the third one takes up **20x** less space than the first one. **Minify your SVGs**!

[](#how-i-minify-svgs)How I Minify SVGs
---------------------------------------

Sure, I _could_ just manually run [svgo](https://github.com/svg/svgo) on any SVGs I wanted to use, but what I _really_ wanted was a way to optimize SVGs at **build time** because:

*   Who wants to have to remember to _manually_ optimize every SVG?!
*   I wanted to keep using the _Inkscape SVG_ format, which retains some useful metadata (e.g. to preserve your session the next time you open the file).

This blog is built on [Gatsby.js](https://www.gatsbyjs.org/) (it’s open-source on [Github](https://github.com/vzhou842/victorzhou.com) if you’re curious), so I wrote a simple plugin called [gatsby-plugin-optimize-svgs](https://github.com/vzhou842/gatsby-plugin-optimize-svgs) to run `svgo` on any SVGs present in the build output. It’s trivial to install:

```
$ npm install gatsby-plugin-optimize-svgs
```

```
module.exports = {
  plugins: [
    
    'gatsby-plugin-optimize-svgs',  ],
};
```

Here’s what my results with `gatsby-plugin-optimize-svgs` were:

 ![](https://victorzhou.com/static/ffaba449937444056c76f8a1b5b06ed2/39600/results.png) 

**62 SVGs minified, reducing the total size from 459322 bytes to 208897 bytes, a reduction of 54.5%**! That’s a total of 250 KB, or 4 KB per SVG. Keep in mind that all of my SVGs were already saved in the _Optimized SVG_ format - **these savings were on top of _already optimized SVGs_**. If you haven’t thought about minifying your SVGs before, chances are you’d see much more drastic results.

[](#now-its-your-turn)Now It’s Your Turn
----------------------------------------

Go check your sites. Do they serve any SVGs? Make sure they’re minified!