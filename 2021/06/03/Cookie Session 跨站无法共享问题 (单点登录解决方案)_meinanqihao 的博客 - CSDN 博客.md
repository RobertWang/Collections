> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/meinanqihao/article/details/89310321)

至于什么是单点登录，举个例子，如果你登录了 msn messenger，访问 hotmail 邮件就不用在此登录。  
一般单点登录都需要有一个独立的登录站点, 一般具有独立的域名，专门的进行注册，登录，注销等操作

我们为了讨论方便，把这个登录站点叫做站点 P，设其 Url 为 [http://passport.yizhu2000.com](http://passport.yizhu2000.com/)，需要提供服务的站点设为 A 和 B，跨站点单点登录是指你在 A 网站进行登录后，使用 B 网站的服务就不需要再登录

  
从技术角度讲单点登录分为：

*   跨子域单点登录
*   完全跨单点域登录

跨子域单点登录
-------

所谓跨子域登录，A，B 站点和 P 站点位于同一个域下面，比如 A 站点为 http://blog.yizhu2000.com     B 站点为 [http://forum.yizhu2000.com](http://forum.yizhu2000.com/), 他们和登录站点 P 的关系可以看到，都是属于同一个父域，yizhu2000.com, 不同的是子域不同，一个为 blog，一个为 forum，一个是 passport

我们先看看最常用的非跨站点普通登录的情况，一般登录验证通过后，一般会将你的用户名和一些用户信息，通过某一密钥进行加密，写在本地，也就是一个加密的 cookie，我们把这个 cookie 叫做 -- 票（ticket）。

需要判断用户是否登录的页面，需要读取这个 ticket，并从其中解密出用户信息，如果 ticket 不存在，或者无法解密，意味着用户没有登录，或者登录信息不正确，这时就要跳转到登录页面进行登录，在这里加密的作用有两个，一是防止用户信息被不怀好意者看到，二是保证 ticket 不会被伪造，后者其实更为重要，加密后，各个应用需要采用与加密同样的密钥进行解密，如果不知道密钥，就不能伪造出 ticket，

（注：加密和解密的密钥有可能不同，取决于采用什么加密算法，如果是对称加密，则为同一密钥，如果是非对称，就不同了，一般用私钥加密，公钥解密，但是无论怎样，密钥都只有内部知道，这样伪造者既无法伪造也无法解密 ticket）

跨子域的单点登录，和上述普通登录的过程没有什么不同，唯一不同的是写 cookie 时，由于登录站点 P 和应用 A 处于不同的子域，P 站写入的 cookie 的域为 passport.yizhu2000.net，而 A 站点为 forum.yizhu2000.net，A 在判断用户登录时无法读到 P 站点的 ticket

解决方法非常简单，当 Login 完成后 P 站点写 ticket 的时候，只需把 cookie 的域设为他们共同的父域，yizhu2000.net 就可以了：cookie.domain="yizhu2000.net"，A 站点自然就可以读到这个 ticket 了

完全跨单点域登录
--------

完全跨域登录，是指 A，B 站点和 P 站点没有共同的父域，比如 A 站点为 forum.yizhu1999.net,B 站点为 blog.yizhu1998.net，大家可以参考微软旗下的几个站点 [http://www.live.com](http://www.live.com/)，[www.hotmail.com](http://www.hotmail.com/), 这两个站点就没有共同的父域，而仍然可以共用登录，怎样才能实现呢？请参考下图，由于这种情况 ticket 比较复杂，我们暂时把 P 站点创建的的 ticket 叫做 P-ticket，而 A 站点创建的 ticket 叫 A-ticket，B 的为 B-ticket

![](https://img-blog.csdn.net/20170712193001821?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3RvcHBz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

由于站点 A（forum.yizhu1999.com）不能读取到由站点 P（passport.yizhu2000.com）创建的加密 ticket，所以当用户访问 A 站点上需要登录才能访问的资源时，A 站点会首先查看是否有 A-ticket，如果没有，证明用户没有在 A 站点登录过，不过并不保证用户没有在 B 站点登录，（重复一下，既然是单点登录，当然无论你在 A，B 任意一个站点登录过，另外一个站点都要可以访问），请求会被重定向到 p 站点的验证页面，验证页面读取 P-ticket，如果没有，或者解密不成功，就需要重定向登录页面，登录页面完成登录后，写一个加密 cookie，也就是 P-ticket，并且重定向到 A 站点的登录处理页，并把加密的用户信息作为参数传递给这个页面，这个页面接收登录页的用户信息，解密后也要写一个 cookie，也就是 A-ticket，今后用户再次访问 A 站点上需要登录权限才能访问的资源时，只需要检查这个 A-cookie 是否存在就可以了。

当用户访问 B 站点时，会重复上面的过程，监测到没有 B-ticket，就会重定向到 P 站点的验证页面，去检查 P-ticket，如果没有，就登录，有则返回 B 的登录处理页面写 B-ticket。

注销的时候需要删除 P-ticket 和 A-ticket。

怎么删除 cookie：本来以为这个不是问题，不过还是有朋友问道，简单的说其实是创建一个和你要删除的 cookie 同名的 cookie，并把 cookie 的 expire 设为当前时间之前的某个时间，不过在跨子域的删除 cookie 时有一点要注意：必须要把 cookie 的域设置为父域，在本文中为 yizhu2000.com。

为了保证各个环节的传输的安全性，最好使用 https 连接。