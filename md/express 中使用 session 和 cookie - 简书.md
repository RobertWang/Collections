> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/c5ac502daaad)

cookie 和 session 的工作机制：

由于 cookie 保存在客户端，不能存放敏感信息，而 session 保存于服务器端，可用于存放敏感信息。比如对用户登录状态的保存。但是 http 是无状态的，session 如何保存用户登录状态呢，当用户登录成功之后，服务器端会将用户信息对应于一个 session 数据，并将获取这个 session 数据的钥匙发送给客户端，而这个钥匙在客户端的保存形式是保存在 cookie 中，用户再次访问当前网站的其他网页的时候，将 cookie 信息一起发送给服务器，当服务器收到 cookie 中保存的钥匙的时候，查看这个钥匙对应的 session 数据，从而判断用户是否是在登录状态的，如果已经登录，则可以直接访问，否则跳转到登录页。

而在 express 框架中，默认不支持 Session 和 Cookie，但是可以使用第三方中间件`express-session`来解决

下载：

```
npm install --save express-session


```

配置

```
var session = require('express-session'); # 引入

# 配置中间件
app.use(session({
  // 配置加密字符串，会在原有加密基础上和这个字符串拼起来去加密
  // 目的是：增加安全性，防止客户端恶意伪造
  secret: 'chen',
  resave: true,
  // 当为false,表示只有使用session，才会分配钥匙
  // 当为true，表示无论是否使用session,都会分配钥匙
  saveUninitialized: false
}))


```

使用

```
#req.session.xx = xx表示设置session（对象）
# req.session.xx 表示获取session数据
app.get('/',(req,res) => {
    # 当访问/的时候，设置当前session值，并将session钥匙回传给客户端，保存在客户端的cookie中
  req.session.uname = 'chen';
  res.render('index.html',{
    name: 'chen'
  })
})

app.get('/a',(req,res) => {
    // 当访问/a的时候，获取session数据
  console.log(req.session.uname);
  res.send('ok');
})


```