> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkyMTI1OTQxMA==&mid=2247507517&idx=3&sn=520a351f0d0b5b78658f8aae4e261157&chksm=c184c2c0f6f34bd64f8aa4de29c1ce63431a22c78b1ac41a79d270708d259b3c26ef40abdafa&mpshare=1&scene=1&srcid=1028zudpwaE2EnVLudbS0MVv&sharer_sharetime=1666899022842&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

> 学习了。

今天推荐的这个项目是「PythonWeb实战」，也是入门级别的实战项目，首个爬虫学习的基础实战课程。

这个是专门给留言的一位读者 准备的，也适合其它需要的小伙伴（在这里提醒大家，爬虫现在是需要必备的，我作为多年开发程序员，为了工作也是各种项目都做，各种技术都要了解学习）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/AGrBlNzG3DP9kiaHtToSl26ZteQdkLl6sIQVSgG4nhTlYd2dDF0icSxibTZpWa4ZCQ4q9G1zhGMq1yt2nY3oUUD5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

用python开发web，需要一步一个脚印，不光是python，黑板报君分享的所有的开源项目都是一步一个脚印做起来的。  

1.简单的client和server

2.加强版本的client和server

3.初步MVC的server

4.简单的`sqlite`和`MongoDB`数据库的练习MongoDB和sqlite

5.简单的2种爬虫(静态和动态)，以及一个jsonapi的小例子Spider

6.用Flask改写的server_flask

简单说说初步MVC的`server`，在文件夹`server_normal`中。

包含的功能：

**1.用户管理**

```
route_dict = {
    '/': route_index,欢迎界面。有1个login链接
    '/login': route_login,登陆界面，登陆成功该界面刷新一些信息，不跳转。有2个链接分别去该用户的todo界面和tweet界面，有2个链接分别是数据api
    '/register': route_register,注册界面，注册成功该界面刷新一些信息，不跳转。
    '/out': route_out,退出登陆
    '/messages': route_message,演示表单提交的页面，显示所有message
    '/profile': login_required(route_profile),该用户的id name password
    '/admin/users': login_required(admin),id为1的admin用户可以看所有用户id name password
    '/admin/user/update': login_required(admin_update),id为1的admin用户可以更改所有用户password
}

```

**2.****`Todo`操作**

> `index`界面，分别用`http`页面刷新方式和`ajax`方式显示。可对`todo`进行`CRUD`，也可以更改`todo`状态。

```
route_dict = {
    '/todo/index': login_required(index),
    '/todo/add': login_required(add),
    '/todo/edit': login_required(edit),
    '/todo/update': login_required(update),
    '/todo/delete': login_required(delete),
    '/todo/status_switch': login_required(switch),
}

```

> **api接口**

```
route_dict = {
    '/ajax/todo/index': login_required(index),
    '/ajax/todo/add': login_required(add),
    '/ajax/todo/delete': login_required(delete),
    '/ajax/todo/update': login_required(update),
    '/ajax/todo/status_switch': login_required(switch),
}

```

**3.****`Tweet`和`comment`操作**

`index`界面，分别用`http`页面刷新方式和`ajax`方式显示。可对`tweet`和`comment`进行CRUD 除了使用`ajax`api的`comment`不会根据`user_id`改变外， `http`的`tweet`和`comment`以及`ajax`api的`tweet`可以根据`user_id`显示，并有用户验证功能 验证规则是:自己只能删除自己的东西(`tweet`和`comment`)

```
route_dict = {
    '/tweet/index': login_required(index),
    '/tweet/delete': login_required(delete),
    '/tweet/edit': login_required(edit),
    '/tweet/update': login_required(update),
    '/tweet/add': login_required(add),
    '/tweet/new': login_required(new),
    '/comment/add': login_required(comment_add),
    '/comment/delete': login_required(comment_delete),
    '/comment/edit': login_required(comment_edit),
    '/comment/update': login_required(comment_update),
}

```

> **api接口**

```
route_dict = {
    '/ajax/tweet/index': login_required(index),
    '/ajax/tweet/add': login_required(add),
    '/ajax/tweet/delete': login_required(delete),
    '/ajax/tweet/update': login_required(update),
    '/ajax/comment/index': login_required(comment_index),
    '/ajax/comment/add': login_required(comment_add),
    '/ajax/comment/delete': login_required(comment_delete),
    '/ajax/comment/update': login_required(comment_update),
}

```

**4.简单的****`Cookie`****和Session功能**

相关技术  

1.  前端用到了`html`，`ajax`和`jinja`模板渲染
    
2.  后端未使用任何框架。基于`socket`手工打造以及`Flask`版本
    
3.  数据存储有`txt`接口和`MongoDB`接口