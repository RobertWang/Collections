> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [muxueqz.top](http://muxueqz.top/a-small-static-site-generator.html)

> 原来写一个静态博客如此简单！

众所周知，随着 Github Pages 这样的服务越来越流行，现在像 Hexo、Hugo、Pelican 这样的静态博客越来越多， 像我以前就是用`Pelican`的，但因为`Pelican`的依赖比较多（其实是想自己造轮子）， 自从见过`Nim`就一直很想自己写一个静态博客，但总是觉得比较麻烦，

直到看到 [Writing a small static site generator](https://blog.thea.codes/a-small-static-site-generator/) ， 才发现原来写一个静态博客竟如此简单。

废话不多说，那我们就开始动手做吧！

准备工作
----

*   安装好 Nim 编译器，如果你用 Debian GNU/Linux，`apt install nim`即可
*   安装依赖

```
nimble install markdown
nimble install nwt
```

收集 markdown 列表
--------------

静态博客大多是使用 markdown 这样的源格式来编写文章，然后输出成 HTML，

因为最近几年写 markdown 比较多，这里就只支持 markdown。

这段代码比较简单，遍历`srcs`目录中的`*.md`文件，然后交给`md_processor`去解析

```
proc write_posts(): seq[JsonNode] = 
  var
    post: JsonNode
  for file in walkDirRec "./srcs/":
    if file.endsWith ".md":
      echo file
      post = md_processor(file)
      write_post(post)
      result.add post
```

解析 markdown 源
-------------

因为不像 Python 里有好用的`frontmatter`，所以我们这里用`pegs`来解析 markdown 中的元信息（如 Title、Tags 等）。

这里的`markdown`解析是用 [nim-markdown](https://github.com/soasme/nim-markdown) ，这在准备工作中已经安装好了

```
proc md_processor(file_path: string): JsonNode = 
  var
    file_meta = splitFile(file_path)
    head = true
    post = initTable[string, string]()
    matches: array[0..1, string]
    src = ""
  for line in file_path.open().lines:

    if head and line.match(peg"\s* {\w+} \s* ':' \s* {.+}", matches):
      post[matches[0]] = matches[1]
    elif head and line.strip.len == 0:
      discard
    else:
      head = false
    if head == false:
      src.add line & "\n"
  if not post.contains"Slug":
    post["Slug"] = file_meta.name
  if "Category" in post:
    if "Tags" in post:
      post["Tags"].add "," & post["Category"]
    else:
      post["Tags"] = post["Category"]
  post["content"] = markdown(src)
  result = %* post
```

生成博客文章
------

现在我们已经完成了对 markdown 的解析，接着我们把它输出到 HTML 中供浏览器查看，

在这里，我选择了类似`jinja2`的`nwt`模板引擎，得以兼容 Thea 模板的大部分内容。

这是一个简化后的模板样例：

```
{% block title %}{{title}}{% endblock %}

{% block content %}
<article>
  <aside>
    <time>{{Date}}</time>
    Tags: {{tag_links}}
    · <a href="/">view all posts</a>
  </aside>
  <h1>{{title}}</h1>
  <content>
    {{content}}
  </content>
</article>
{% endblock %}
```

接着，我们在 Nim 中根据这个模板去输出 HTML，我把 HTML 输出到`./public`这个目录：

```
proc write_post(post: JsonNode)=
    var
      p: string
      new_post = initTable[string, string]()
    new_post["Tags"] = ""
    for k, v in post:
      new_post[k] = v.getStr
    new_post["tag_links"] = ""
    if "Tags" in post:
      for tag in post["Tags"].getStr.split(","):
        p = """
        <a href="/tags/$1.html">
        $1
        </a>
        """ % tag.strip()
        new_post["tag_links"].add p
    var json_post = %* new_post
    var content = templates.renderTemplate("post.html", json_post)

    writeFile("public/" & post["Slug"].getStr & ".html", content)
```

生成博客首页索引
--------

现在我们已经完成了对博客文章的解析和生成，接下来我们去更新一下首页吧！

首页的模板如下：

```
{% block content %}
<nav>
  <h1>My blog posts</h1>
  <dl class="posts-list">
    {{content}}
  </dl>
</nav>
{% endblock %}
```

在 Nim 中去生成`index.html`（因为 nwt 不支持`for`循环，所以只好在 Nim 中去循环了）：

```
proc write_index(posts: seq[JsonNode]) =
    var
      seq_post : seq[string]
      tags = initCountTable[string]()
      p, summary, tag_cloud: string

    for key, post in posts:
      if "Summary" in post:
        summary = post["Summary"].getStr
      else:
        summary = ""
      p = """
    <a href="/$1.html">
      <dt>$2</dt>
      <dd>
        <time>$3</time>
      </dd>
    </a>
    <div class="summary">
    $4
    </div>
    """ % [
          post["Slug"].getStr,
          post["Title"].getStr,
          post["Date"].getStr,
          summary,
          ]
      seq_post.add p
      if "Tags" in post:
        for tag in post["Tags"].getStr.split(","):
          tags.inc tag.strip()

    for tag, count in tags:
      p = """
      <a href="/tags/$1.html">
      $1
      </a>
      """ % tag
      tag_cloud.add p
    seq_post.add tag_cloud

    var index_post = %* {
        "content": seq_post.join("\n")
      }
    var content = templates.renderTemplate("index.html", index_post)

    writeFile("public/" & "index.html", content)
```

最后的工作，写个`main`函数
----------------

```
proc main()= 
  var
    posts = write_posts()
  posts = sort_posts(posts)
  write_index(posts)
```

开始编译吧！
------

```
nim c -r build.nim
```

完整代码
----

[kun](https://github.com/muxueqz/kun)

感谢 Thea ，在我想用 Nim 编写自己的静态博客时， 看到了 [Writing a small static site generator](https://blog.thea.codes/a-small-static-site-generator/) ， 惊叹竟然可以如此简洁，所以此篇文章基本也是和她一样的思路，只是实现的语言换成了 Nim