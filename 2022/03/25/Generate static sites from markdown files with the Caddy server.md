> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.thomaspuppe.de](https://blog.thomaspuppe.de/static-sites-from-markdown-with-caddy-server)

> Caddy can not only serve files fast and safely. It can also generate static files from markdown, so y......

#webdevelopment 10. April 2018

Lately, I am experimenting with different forms of static-site generation. While investigating the server [Caddy](https://caddyserver.com/) for _serving_ pages, I found that it can also _generate_ these pages from markdown.

> ⚠️ Update 2022: This article is from 2018 and refers to Caddy version 1. User [camkego on HN](https://news.ycombinator.com/item?id=30145237#30146034) provides links to the current documentation of Caddy V2 markdown usage:
> 
> *   [https://caddyserver.com/docs/caddyfile/directives/templates](https://caddyserver.com/docs/caddyfile/directives/templates)
> *   [https://github.com/caddyserver/website/blob/master/src/docs/index.html](https://github.com/caddyserver/website/blob/master/src/docs/index.html)
> *   [https://caddy.community/t/markdown-support-in-v2/6984](https://caddy.community/t/markdown-support-in-v2/6984)

Render markdown as HTML
-----------------------

Basically it is enough to activate markdown in your [Caddyfile](https://caddyserver.com/docs/caddyfile):

```
localhost:8000
markdown /
```

Now start your server with `caddy -conf ../path/to/Caddyfile` and visit pages `http://localhost:8000/one.md`. You will see that pages written in markdown (which you have to create, of course) ...

```
# one.md:
---
title: My First Post
---

What The Fuck. Cool!

## This is h2

Lorem Ipsum youknow.
```

... are rendered as HTML:

```
# http://localhost:8000/one.md
<!DOCTYPE html>
<html>
    <head>
        <title>My First Post</title>
        <meta charset="utf-8">
    </head>
    <body>
        <p>What The Fuck. Cool!</p>
        <h2>This is h2</h2>
        <p>Lorem Ipsum youknow.</p>
    </body>
</html>
```

Ta dah! A website with title-tag and content.

Add CSS and JS
--------------

For some cases like a programming journal this might be enough. But you might want at least some CSS with it. You can do this by simply adding the path to a CSS file into the markdown-block of your caddyfile:

```
localhost:8000
markdown / {
    css /styles.css
}
```

which results into the line

```
<link rel="stylesheet" href="/styles.css">
```

in the head of your rendered HTML file. This also works for JavaScript (`js /script.js`), which is also inserted into the head of your HTML file.

Templating
----------

But you want to load JS asyncronously and deferred at the end of the body-tag, don't you? And you want to add some more HTML, like a custom header or something. You can do that – by creating a template file and using that in the caddyfile:

```
localhost:8000
markdown / {
    template ../template.html
}
```

Note: the template path is not relative to the caddyfile, but relative to the folder where you run your `caddy -conf path/to/Caddyfile` command.

Note 2: You need to restart the caddy server after changing the caddyfile.

And: we do not need to specify the CSS and JS here, because we can do it directly inside out HTML template file:

```
<!DOCTYPE html>
<html>
    <head>
        <title>{{.Doc.title}}</title>
        <link rel="stylesheet" media="screen" href="/styles.css">
    </head>
    <body>
        <head>
            <h1>{{.Doc.title}}</h1>
        </head>
        <article>
            {{.Doc.body}}
        </article>
        <script async defer src="/script.js"></script>
    </body>
</html>
```

which renders this:

```
<!DOCTYPE html>
<html>
    <head>
        <title>My First Post</title>
        <link rel="stylesheet" media="screen" href="/styles.css">
    </head>
    <body>
        <head>
            <h1>My First Post</h1>
        </head>
        <article>
            <p>What The Fuck. Cool!</p>
            <h2>This is h2</h2>
            <p>Lorem Ipsum youknow.</p>
        </article>
        <script async defer src="/script.js"></script>
    </body>
</html>
```

Template variables
------------------

You can see that the variable `{{.Doc.title}}` holds the content of the line `title: My First Post` in your markdown file. That kind of declaration in markdown (an other) files is called "frontmatter" and can be written in YAML or JSON format.

Even better: it can hold different contents ...

```
---
title: My First Post
author: Thomas Puppe
author_url: https://www.thomaspuppe.de
date: April 10th 2018
---
```

... which then can be used inside your template file:

```
<a href="{{.Doc.author_url}}">{{.Doc.author}}</a> on {{.Doc.date}}</footer>
```

Caddy even offers some own template variables like `{{.Cookie "cookiename"}}` or `{{.RandomString 100 10000}}`, sanitizing functions and control statements. A [complete list of template actions](https://caddyserver.com/docs/template-actions) can be found in the Caddy docs.

This is pretty cool and you can get very far with that.

Clean URLs
----------

But one more thing: The URL `http://localhost:8000/one.md` is neither pretty nor common. Something like `http://localhost:8000/my-first-post` would be nicer. The slug "my-first-post" is simply the name of your file, of course. But what about the extension? You can get rid of that by making use of Caddys [ext](https://caddyserver.com/docs/ext) directive:

```
localhost:8000

markdown / {
    ext .md .html
    template ../template.html
}
```

This means that caddy tries to match every request, which is not served by an existing file, is tested against files with these extensions. So if you request `http://localhost:8000/my-first-post`, Caddy tries to find a `my-first-post` file, and if that does not exist, it tries to find `my-first-port.md`, which we just have created. `.html` would be another fallback, and you can define as many as you like.

I also tried the other way around: I put markdown files without any extension into the folder, and wanted Caddy to render them as HTML, but that did not work.

Special templates per file
--------------------------

You might want render diffent posts inside different templates. Maybe there are special posts which have a sidebar, or you create some landing pages with a lot of beautiful custom HTML and only some copy text from markdown. You can introduce diffent templates to Caddy be listing them in the caddyfile.

```
markdown / {
    template ../template.html
    template special ../special.html
}
```

In the frontmatter part of your your markdown file, you can specify the template just like this: `template: index`. Then, this page will use the special.html template for rendering. If no template is given in the markdown file, the default `template.html`, which is defined without a name in the Caddyfile, will be used.

If your templates share a common set of HTML tags, you should be able to use the import directive:

```
{{.Include "path/to/file.html"}}  // no arguments
{{.Include "path/to/file.html" "arg1" 2 "value 3"}}  // with arguments
```

(This is written in the [template action docs](https://caddyserver.com/docs/template-actions), but did not work for me. Maybe because of a strange combination of relative paths?)

File listings
-------------

On index pages, it would be handy to hava an automatic listing of subpages: the home page of a blog, the members page of your sports club. Caddy can even do that:

```
<ul>
    {{.Files "sub/folder"}}
        <li>
            <a href="{{$.StripExt .Name}}">{{$.StripExt .Name}}</a>
        </li>
    {{end}}
</ul>
```

You "just" have to filter files like ".DS_Store", asset folders, the index file itself and so on. But, for your convenience, Caddy at least provides [control statements](https://caddyserver.com/docs/template-actions) like `{{if not .IsDir }}` and you can strip away extensions (`{{$.StripExt .Name}}`).

According to the docs, you can even list a subset of "tagged" files by, but I have not explored that.

External data sources
---------------------

It would be cool to store the markdown files externally (think GitHub), and have Caddy fetch them. I think this should be possible, because Caddy can be used as a proxy server.

I tried the proxy directive (`proxy /test https://raw.githubusercontent.com/thomaspuppe/blog.thomaspuppe.de/`), but got stuck with SSL handshake errors.

Other shortcomings
------------------

While Caddy gets you pretty far with its templating and markdown rendering functions, there are a few things which keep me from using it for this blog.

*   I have not figured out sorting of the file listing. Caddy does no alphabetic sorting, it looks rather random.
*   I want to have more control over content rendering, and
*   No feed generation.
*   No pagination of listing pages.

Wrap-up
-------

Caddys built-in features for static-site generation do a good job for putting together a simple and fast static site. With templating, logical conditions and string manipulation you get pretty far with few lines of code. If you reach the limits of what Caddy can do, you won't have wasted much time. So it is worth a try.