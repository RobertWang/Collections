> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/yywan1314520/article/details/51566924)

1. 在每次 push 的时候，都要输入用户名和密码，很是麻烦原因是使用了 https 方式 push，在 termail 里边输入 **git remote -v**
------------------------------------------------------------------------------------

可以看到形如一下的返回结果
-------------

origin [https://github.com/wei0long/AugmentedReality.git](https://github.com/wei0long/AugmentedReality.git) (fetch)  
origin [https://github.com/wei0long/AugmentedReality.git](https://github.com/wei0long/AugmentedReality.git) (push)

下面把它换成 ssh 方式的。
---------------

*   git remote rm origin
*   git remote add origin git@github.com:wei0long/AugmentedReality.git
*   git push origin

2. 可能这样还不行，还应该添加 SSH 公匙。**ssh-keygen -t rsa -C “email”**,email 是你注册在 github 上的邮箱。生成的 C:\Users\Administrator.ssh 在这个目录下
----------------------------------------------------------------------------------------------------------------------

![](https://img-blog.csdn.net/20160602153255525)

将 id_rsa.pub 放到如下所示的目录下，sshkey 的名字可以随便取。
----------------------------------------

![](https://img-blog.csdn.net/20160602153228940)

3. 接着 push 会出现下面的问题。
--------------------

![](https://img-blog.csdn.net/20160602153639665)

这里根据提示选择新特性
-----------

git config –global push.default simple
--------------------------------------

git push –set-upstream origin master
------------------------------------

4. 以后就不用输入用户名和密码就可以 push 上去了，如果好用不在下面**顶**一下说不过去，是吧。
----------------------------------------------------

5. 如果还不行，可以看下面几种方式
------------------

### 设置记住密码（默认 **15 分钟**）：

### git config –global credential.helper cache

### 如果想自己设置时间，可以这样做：

### git config credential.helper ‘cache –timeout=3600’

### 这样就设置一个小时之后失效

### **长期存储密码**：

### git config –global credential.helper store