> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jb51.net](https://www.jb51.net/article/168244.htm)

> 在本篇文章里小编给大家整理的是关于 PHP Primary script unknown 终极解决方法以及相关知识点，有需要的朋友们参考学习下。

相信很多配置 php 环境的都遇到过这个恼人的问题：

*   浏览器访问 php 文件，返回来 File not found
*   查看 / var/log/nginx/error.log ，有 “Primary script unknown”，类似如下：

<table><tbody><tr><td><p>1</p><p>2</p></td><td><p><code>2019/01/03 10:24:02 [error] 11931#11931: *260 FastCGI sent in stderr: </code><code>"Primary script unknown"</code> <code>while</code> <code>reading response header from upstream,</code></p><p><code>client: 1.2.3.4, server: localhost, request: </code><code>"GET /index.php HTTP/1.1"</code><code>, upstream: </code><code>"<a href="fastcgi://127.0.0.1:9000">fastcgi://127.0.0.1:9000</a>"</code><code>, host: &lt;a href=</code><code>"<a href="http://www.example.com/">http://www.example.com/</a>"</code> <code>rel=</code><code>"external nofollow"</code><code>&gt;www.example.com&lt;/a&gt;</code></p></td></tr></tbody></table>

原因只有两个，一个是 php-fpm 找不到 php 文件，一个是 php-fpm 没有权限读取和执行文件。

**1. 找不到文件问题**  

nginx 的站点配置文件 php 段要这样：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p></td><td><p><code># pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000</code></p><p><code>&nbsp;&nbsp;</code><code>location ~ \.php$ {&nbsp;&nbsp;&nbsp; #root 路径配置必须要有，而且必须要写对（别笑，真的能写错）</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; /usr/share/nginx/html;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>fastcgi_pass&nbsp; 127.0.0.1:9000;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>fastcgi_index index.php;&nbsp;&nbsp;&nbsp; #SCRIPT_FILENAME用</code><code>$document_root</code><code>，而不是具体路径</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>fastcgi_param SCRIPT_FILENAME </code><code>$document_root</code><code>$fastcgi_script_name</code><code>;</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>include</code>&nbsp;&nbsp;&nbsp; <code>fastcgi_params;</code></p><p><code>&nbsp;&nbsp;</code><code>}</code></p></td></tr></tbody></table>

**2. 权限问题**  

也是坑最多的。

1) 进程用户  

nginx.conf 里的 user 配置要跟 php-fpm.d/www.conf 一致，比如都用 nginx，或者自定义用户 phpuser（再来句废话，这个用户需要提前建好）。  

nginx.conf ：

<table><tbody><tr><td><p>1</p><p>2</p></td><td><p><code>user phpuser;</code></p><p><code>worker_processes auto;</code></p></td></tr></tbody></table>

php-fpm.d/www.conf ：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p></td><td><p><code>; Unix user/group of processes</code></p><p><code>; Note: The user is mandatory. If the group is not set, the </code><code>default</code> <code>user's group</code></p><p><code>;&nbsp;&nbsp;&nbsp; will be used.</code></p><p><code>user = phpuser</code></p><p><code>group = phpuser</code></p></td></tr></tbody></table>

nginx 和 php-fpm 进程 / 监听信息：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p></td><td><p><code>root&nbsp;&nbsp; 19107 0.0 0.1 207644 5852 ?&nbsp;&nbsp;&nbsp; Ss&nbsp; 1月02&nbsp; 0:03 php-fpm: master process (/usr/local/etc/php-fpm.conf)</code></p><p><code>phpuser 19108 0.0 0.1 207644 7108 ?&nbsp;&nbsp;&nbsp; S&nbsp; 1月02&nbsp; 0:00 php-fpm: pool www</code></p><p><code>phpuser 19109 0.0 0.1 207644 7112 ?&nbsp;&nbsp;&nbsp; S&nbsp; 1月02&nbsp; 0:00 php-fpm: pool www</code></p><p><code>root&nbsp;&nbsp; 24676 0.0 0.0 56660 1024 ?&nbsp;&nbsp;&nbsp; Ss&nbsp; 13:08&nbsp; 0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf</code></p><p><code>phpuser 24677 0.0 0.7 84680 29976 ?&nbsp;&nbsp;&nbsp; S&nbsp; 13:08&nbsp; 0:00 nginx: worker process</code></p><p><code>phpuser 24678 0.0 0.7 84324 29236 ?&nbsp;&nbsp;&nbsp; S&nbsp; 13:08&nbsp; 0:00 nginx: worker process</code></p><p><code>tcp&nbsp;&nbsp;&nbsp; 0&nbsp;&nbsp; 0 127.0.0.1:9000&nbsp;&nbsp;&nbsp;&nbsp; 0.0.0.0:*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LISTEN&nbsp;&nbsp; 19107/php-fpm: mast</code></p><p><code>tcp&nbsp;&nbsp;&nbsp; 0&nbsp;&nbsp; 0 0.0.0.0:80&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 0.0.0.0:*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LISTEN&nbsp;&nbsp; 24676/nginx: master</code></p><p><code>tcp6&nbsp;&nbsp;&nbsp; 0&nbsp;&nbsp; 0 :::80&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; :::*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LISTEN&nbsp;&nbsp; 24676/nginx: master</code></p></td></tr></tbody></table>

如果修改了 nginx 运行用户还必须要改些目录权限：

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p></td><td><p><code>chown</code> <code>-R phpuser:phpuser /</code><code>var</code><code>/log/nginx</code></p><p><code>chown</code> <code>-R phpuser:phpuser /</code><code>var</code><code>/cache/nginx</code></p><p><code>chown</code> <code>-R phpuser:phpuser /usr/share/nginx/html</code></p></td></tr></tbody></table>

还有 / etc/logrotate.d/nginx，create 640 nginx adm 这行要改：

2) 目录和文件权限  

php 文件不必非得设为 777，让人怪担心的，只要是 nginx 和 php-fpm 运行用户可读写执行即可，一般可以 770 。  

php 文件目录和文件样例：

<table><tbody><tr><td><p>1</p><p>2</p></td><td><p><code>drwxrwx--- 6 phpuser phpuser 4.0K 2019-01-03 13:09 /usr/share/nginx/html</code></p><p><code>-rwxrwx--- 1 phpuser phpuser 40&nbsp; 2019-01-03 13:09 /usr/share/nginx/html/phpinfo.php</code></p></td></tr></tbody></table>

这里有个深坑，对于使用其他目录放置 php 文件的很可能中招，就是 /path/to/phpfiles 的每一层目录都要允许 phpuser 访问，缺一层就会 Permission denied。

本例，/usr/share/nginx/html 之上的每一层目录，所有者都是 root，都有 o+rx ，即所有人都有读取和执行权限（读取和执行权限是目录访问的根本），因此 phpuser 可以访问到 html 目录。

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p></td><td><p><code>drwxr-xr-x. 13 root root&nbsp;&nbsp;&nbsp; 155 2018-07-10 15:42 /usr</code></p><p><code>drwxr-xr-x. 86 root root&nbsp;&nbsp;&nbsp; 4.0K 2018-12-17 07:33 /usr/share/</code></p><p><code>drwxr-xr-x&nbsp; 4 root root&nbsp;&nbsp;&nbsp;&nbsp; 40 2018-12-17 08:06 /usr/share/nginx/</code></p><p><code>drwxrwx---&nbsp; 6 phpuser phpuser 4.0K 2019-01-03 13:11 /usr/share/nginx/html/</code></p></td></tr></tbody></table>

测试方法：

<table><tbody><tr><td><p>1</p></td><td><p><code>sudo -u phpuser ls -l /usr/share/nginx/html/</code></p></td></tr></tbody></table>

3) SELINUX  

nginx/apache 网页文件的 selinux 上下文，如果更换目录需要配上。（在 Cenots7+php7.3 上测试，没有 selinux 上下文时，静态文件 404，而 php 文件反倒没有遇到问题，没有深究）

<table><tbody><tr><td><p>1</p><p>2</p></td><td><p><code># ll -dZ /usr/share/nginx/html</code></p><p><code>drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /usr/share/nginx/html</code></p></td></tr></tbody></table>

配置 selinux 上下文：

<table><tbody><tr><td><p>1</p></td><td><p><code>chcon -R -t httpd_sys_content_t /path/to/phpfiles</code></p></td></tr></tbody></table>

或者干脆关闭 selinux（需要重启服务器）  

/etc/selinux/config ：

**3. 最后**

<table><tbody><tr><td><p>1</p></td><td><p><code>echo</code> <code>"&lt;p align='center'&gt;Good Luck :)&lt;/p&gt;&lt;?php phpinfo(); ?&gt;"</code> <code>&gt; /usr/share/nginx/html/phpinfo.php</code></p></td></tr></tbody></table>

以上就是 PHP Primary script unknown 终极解决方法的全部知识点内容，感谢大家对脚本之家的支持。