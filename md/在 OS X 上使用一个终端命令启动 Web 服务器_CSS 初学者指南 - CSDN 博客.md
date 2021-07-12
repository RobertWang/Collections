> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/cunqu9743/article/details/106999420)

我本周已经搜索了 3 次，所以我认为最好确保有它的副本。

Python 2：

```
python -m SimpleHTTPServer 8000
```

导航到终端中的项目目录，然后执行该命令。 然后 http：// localhost：8000 将服务器存储在该目录中（例如，它是 index.html 文件）。

Python 3：

```
python3 -m http.server --cgi 8080
```

PHP：

```
php -S localhost:2222
```

npm：

```
npm i -g serve
serve
```

> 翻译自: [https://css-tricks.com/snippets/html/start-a-web-server-with-one-terminal-command-on-os-x/](https://css-tricks.com/snippets/html/start-a-web-server-with-one-terminal-command-on-os-x/)