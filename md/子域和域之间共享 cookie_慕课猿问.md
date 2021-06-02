> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.imooc.com](https://www.imooc.com/wenda/detail/565257)

> 子域和域之间共享 cookie 我有两个问题。我理解如果我指定域为. mydomain.com 在 Cookie 中，所有子域都可以共享 Cookie。能，会，可以 subdomain.mydomain.com...

[![](https://img4.sycdn.imooc.com/5458478b0001f01502200220-100-100.jpg)](https://www.imooc.com/u/6457903/bbs)

[拉丁的传说](https://www.imooc.com/u/6457903/bbs "拉丁的传说")

两个域`mydomain.com`和`subdomain.mydomain.com`属性中显式命名域时，才能共享 cookie。`Set-Cookie`头球。否则，cookie 的作用域仅限于请求主机。(这被称为 “纯宿主饼干”。看见[什么是主人唯一的饼干？](https://stackoverflow.com/questions/12387338/what-is-a-host-only-cookie))

例如，如果您从`subdomain.mydomain.com`:

```
Set-Cookie: name=value

```

那么 cookie 就不会被发送到`mydomain.com`.. 但是，如果您使用以下内容，则它在这两个域上都是可用的：

```
Set-Cookie: name=value; domain=mydomain.com

```

在…… 里面 [RFC 2109](http://tools.ietf.org/html/rfc2109)，没有前导点的域意味着不能在子域上使用它，而只有一个前导点 (`.mydomain.com`) 将允许跨多个子域 (但不包括顶级域，所以您所要求的在旧规范中是不可能的)。

然而，所有现代浏览器都尊重更新的规范。[RFC 6265](http://tools.ietf.org/html/rfc6265)，并将忽略任何前导点，这意味着您可以在子域和顶级域上使用 cookie。

总之，如果您设置了一个 cookie，如上面的第二个示例`mydomain.com`，它可以通过`subdomain.mydomain.com`，反之亦然。这也可以用来允许`sub1.mydomain.com`和`sub2.mydomain.com`分享饼干。

另见：

*   [WWW 与 No-www 和 cookie](http://www.phpied.com/www-vs-no-www-and-cookies/)
    
*   [cookie 测试脚本](https://scripts.cmbuckley.co.uk/cookies.php)
    
    试试看

查看完整回答

_反对_ 回复 2019-06-14

[![](https://img4.sycdn.imooc.com/5458477300014deb02200220-100-100.jpg)](https://www.imooc.com/u/6463836/bbs)

[哆啦的时光机](https://www.imooc.com/u/6463836/bbs "哆啦的时光机")

我不确定 “cmackley” 的答案是否显示了全貌。我读到的是：

> 除非 cookie 的属性另有说明，否则 Cookie 只返回到源服务器 (例如，不返回到任何子域)，并且在当前会话结束时过期 (用户代理定义的)。用户代理忽略未识别的 cookie。
> 
> [RFC 6265](https://tools.ietf.org/html/rfc6265#section-4.1.2)

也

```
8.6.  Weak Integrity

   Cookies do not provide integrity guarantees for sibling domains (and
   their subdomains).  For example, consider foo.example.com and
   bar.example.com.  The foo.example.com server can set a cookie with a
   Domain attribute of "example.com" (possibly overwriting an existing
   "example.com" cookie set by bar.example.com), and the user agent will
   include that cookie in HTTP requests to bar.example.com.  In the
   worst case, bar.example.com will be unable to distinguish this cookie
   from a cookie it set itself.  The foo.example.com server might be
   able to leverage this ability to mount an attack against
   bar.example.com.

```

对我来说，这意味着您可以保护 cookie 不被子域 / 域读取，但不能阻止将 cookie 写入其他域。因此，有人可以通过控制同一个浏览器访问的另一个子域来重写您的网站 cookie。这可能不是什么大问题。

@cmackley / 为那些像我一样在他的答案中漏掉了它的人提供的可怕的 cookie 测试站点；值得翻阅和更新 /：

*   [http://scripts.cmbuckley.co.uk/cookies.php](http://scripts.cmbuckley.co.uk/cookies.php)
    

查看完整回答

_反对_ 回复 2019-06-14

[![](https://img4.sycdn.imooc.com/5458502c00012d4a02200220-100-100.jpg)](https://www.imooc.com/u/6458195/bbs)

[蛊毒传说](https://www.imooc.com/u/6458195/bbs "蛊毒传说")

下面是一个使用 DOM Cookie API 的示例 ([https：/developer.mozilla.org/en-US/docs/web/api/document/cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie))，这样我们就可以自己看到这种行为了。

如果我们执行以下 JavaScript：

> document.cookie=“key=value”

它似乎与执行：

> document.cookie=“key=value；Domain=mydomain.com”

饼干_钥匙_在域上变得可用 (仅)_mydomain.com_.

* * *

现在，如果在 mydomain.com 上执行以下 JavaScript：

> document.cookie=“key=value；Domain=.mydomain.com”

饼干_钥匙_变得可用 _mydomain.com_ 以及 _subdomain.mydomain.com_.

* * *

最后，如果您尝试在 subdomain.mydomain.com 上执行以下操作：

> document.cookie=“key=value；Domain=.mydomain.com”

饼干_钥匙_可供 _subdomain.mydomain.com_？我对允许这样做感到有点惊讶；我认为子域能够在父域上设置 cookie 是一种违反安全的行为。

查看完整回答

_反对_ 回复 2019-06-14