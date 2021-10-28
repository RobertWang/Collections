> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/e8ee93f88cbd)

git 回退到上个版本

git reset --hard HEAD^

回退到前 3 次提交之前，以此类推，回退到 n 次提交之前

git reset --hard HEAD~3  
退到 / 进到 指定 commit 的 sha 码

git reset --hard dde8c25694f34acf8971f0782b1a676f39bf0a46  
强推到远程

git push origin HEAD --force  
[https://www.cnblogs.com/spring87/p/7867435.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fspring87%2Fp%2F7867435.html)

————————————————  
版权声明：本文为 CSDN 博主「fareast_mzh」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。  
原文链接：[https://blog.csdn.net/fareast_mzh/article/details/93709734](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Ffareast_mzh%2Farticle%2Fdetails%2F93709734)