> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/380793959)

这篇文章主要介绍了 wget 下载整个网站 (整个子目录) 或特定目录, 需要的朋友可以参考下。

### **使用 wget 命令下载父目录下的整个子目录**

使用 wget 命令下载父目录下的整个子目录，命令如下：

> wget -r --level=0 -E --ignore-length -x -k -p -erobots=off -np -N [http://www.remote.com/remote/presentation/dir](http://www.remote.com/remote/presentation/dir)

将会下载远程服务器的整个文件夹下到你电脑当前文件目录下。

### **如何使用 wget 下载一个目录下的所有文件**

> wget -r -np -nH -R index.html [http://url/including/files/you/want/to/download/](http://url/including/files/you/want/to/download/)

各个参数的含义：

-r : 遍历所有子目录  
-np : 不到上一层子目录去  
-nH : 不要将文件保存到主机名文件夹  
-R index.html : 不下载 index.html 文件

### **wget 下载整个网站或特定目录**

需要下载某个目录下面的所有文件。命令如下

> wget -c -r -np -k -L -p [http://www.xxx.org/pub/path/](http://www.xxx.org/pub/path/)

在下载时。有用到外部域名的图片或连接。如果需要同时下载就要用 - H 参数。

> wget -np -nH -r --span-hosts [http://www.xxx.org/pub/path/](http://www.xxx.org/pub/path/)

-c 断点续传  
-r 递归下载，下载指定网页某一目录下（包括子目录）的所有文件  
-nd 递归下载时不创建一层一层的目录，把所有的文件下载到当前目录  
-np 递归下载时不搜索上层目录，如 wget -c -r [http://www.xxx.org/pub/path/](http://www.xxx.org/pub/path/)  
没有加参数 - np，就会同时下载 path 的上一级目录 pub 下的其它文件  
-k 将绝对链接转为相对链接，下载整个站点后脱机浏览网页，最好加上这个参数  
-L 递归时不进入其它主机，如 wget -c -r [http://www.xxx.org/](http://www.xxx.org/)  
如果网站内有一个这样的链接：  
[http://www.yyy.org](http://www.yyy.org)，不加参数 - L，就会像大火烧山一样，会递归下载 [http://www.yyy.org](http://www.yyy.org) 网站  
-p 下载网页所需的所有文件，如图片等  
-A 指定要下载的文件样式列表，多个样式用逗号分隔  
-i 后面跟一个文件，文件内指明要下载的 URL

还有其他的用法，我从网上搜索的，也一并写上来，方便以后自己使用。

**wget 的常见用法**

wget 的使用格式

Usage: wget [OPTION]… [URL]…

* 用 wget 做站点镜像:  
wget -r -p -np -k [http://dsec.pku.edu.cn/~usr_name/](http://dsec.pku.edu.cn/~usr_name/)  
# 或者  
wget -m [http://www.tldp.org/LDP/abs/html/](http://www.tldp.org/LDP/abs/html/)

* 在不稳定的网络上下载一个部分下载的文件，以及在空闲时段下载

wget -t 0 -w 31 -c [http://dsec.pku.edu.cn/BBC.avi](http://dsec.pku.edu.cn/BBC.avi) -o down.log &

# 或者从 filelist 读入要下载的文件列表

wget -t 0 -w 31 -c -B ftp://dsec.pku.edu.cn/linuxsoft -i filelist.txt -o  
down.log &

上面的代码还可以用来在网络比较空闲的时段进行下载。我的用法是: 在 mozilla 中将不方便当时下载的 URL 链接拷贝到内存中然后粘贴到文件 filelist.txt 中，在晚上要出去系统前执行上面代码的第二条。

* 使用代理下载  
wget -Y on -p -k [https://sourceforge.net/projects/wvware/](https://sourceforge.net/projects/wvware/)

代理可以在环境变量或 wgetrc 文件中设定

# 在环境变量中设定代理  
export PROXY=[http://211.90.168.94:8080/](http://211.90.168.94:8080/)  
# 在~/.wgetrc 中设定代理  
http_proxy = [http://proxy.yoyodyne.com:18023/](http://proxy.yoyodyne.com:18023/)  
ftp_proxy = [http://proxy.yoyodyne.com:18023/](http://proxy.yoyodyne.com:18023/)

wget 各种选项分类列表

* 启动

-V, –version 显示 wget 的版本后退出  
-h, –help 打印语法帮助  
-b, –background 启动后转入后台执行  
-e, –execute=COMMAND  
执行 `.wgetrc'格式的命令，wgetrc 格式参见 / etc/wgetrc 或~/.wgetrc

* 记录和输入文件

-o, –output-file=FILE 把记录写到 FILE 文件中  
-a, –append-output=FILE 把记录追加到 FILE 文件中  
-d, –debug 打印调试输出  
-q, –quiet 安静模式 (没有输出)  
-v, –verbose 冗长模式 (这是缺省设置)  
-nv, –non-verbose 关掉冗长模式，但不是安静模式  
-i, –input-file=FILE 下载在 FILE 文件中出现的 URLs  
-F, –force-html 把输入文件当作 HTML 格式文件对待  
-B, –base=URL 将 URL 作为在 - F -i 参数指定的文件中出现的相对链接的前缀  
–sslcertfile=FILE 可选客户端证书  
–sslcertkey=KEYFILE 可选客户端证书的 KEYFILE  
–egd-file=FILE 指定 EGD socket 的文件名

* 下载

–bind-address=ADDRESS  
指定本地使用地址 (主机名或 IP，当本地有多个 IP 或名字时使用)  
-t, –tries=NUMBER 设定最大尝试链接次数 (0 表示无限制).  
-O –output-document=FILE 把文档写到 FILE 文件中  
-nc, –no-clobber 不要覆盖存在的文件或使用.# 前缀  
-c, –continue 接着下载没下载完的文件  
–progress=TYPE 设定进程条标记  
-N, –timestamping 不要重新下载文件除非比本地文件新  
-S, –server-response 打印服务器的回应  
–spider 不下载任何东西  
-T, –timeout=SECONDS 设定响应超时的秒数  
-w, –wait=SECONDS 两次尝试之间间隔 SECONDS 秒  
–waitretry=SECONDS 在重新链接之间等待 1…SECONDS 秒  
–random-wait 在下载之间等待 0…2*WAIT 秒  
-Y, –proxy=on/off 打开或关闭代理  
-Q, –quota=NUMBER 设置下载的容量限制  
–limit-rate=RATE 限定下载输率

* 目录

-nd –no-directories 不创建目录  
-x, –force-directories 强制创建目录  
-nH, –no-host-directories 不创建主机目录  
-P, –directory-prefix=PREFIX 将文件保存到目录 PREFIX/…  
–cut-dirs=NUMBER 忽略 NUMBER 层远程目录

* HTTP 选项

–http-user=USER 设定 HTTP 用户名为 USER.  
–http-passwd=PASS 设定 http 密码为 PASS.  
-C, –cache=on/off 允许 / 不允许服务器端的数据缓存 (一般情况下允许).  
-E, –html-extension 将所有 text/html 文档以. html 扩展名保存  
–ignore-length 忽略 `Content-Length'头域  
–header=STRING 在 headers 中插入字符串 STRING  
–proxy-user=USER 设定代理的用户名为 USER  
–proxy-passwd=PASS 设定代理的密码为 PASS  
–referer=URL 在 HTTP 请求中包含 `Referer: URL'头  
-s, –save-headers 保存 HTTP 头到文件  
-U, –user-agent=AGENT 设定代理的名称为 AGENT 而不是 Wget/VERSION.  
–no-http-keep-alive 关闭 HTTP 活动链接 (永远链接).  
–cookies=off 不使用 cookies.  
–load-cookies=FILE 在开始会话前从文件 FILE 中加载 cookie  
–save-cookies=FILE 在会话结束后将 cookies 保存到 FILE 文件中

* FTP 选项

-nr, –dont-remove-listing 不移走 `.listing'文件  
-g, –glob=on/off 打开或关闭文件名的 globbing 机制  
–passive-ftp 使用被动传输模式 (缺省值).  
–active-ftp 使用主动传输模式  
–retr-symlinks 在递归的时候，将链接指向文件 (而不是目录)

* 递归下载

-r, –recursive 递归下载－－慎用!  
-l, –level=NUMBER 最大递归深度 (inf 或 0 代表无穷).  
–delete-after 在现在完毕后局部删除文件  
-k, –convert-links 转换非相对链接为相对链接  
-K, –backup-converted 在转换文件 X 之前，将之备份为 X.orig  
-m, –mirror 等价于 -r -N -l inf -nr.  
-p, –page-requisites 下载显示 HTML 文件的所有图片

* 递归下载中的包含和不包含 (accept/reject)

-A, –accept=LIST 分号分隔的被接受扩展名的列表  
-R, –reject=LIST 分号分隔的不被接受的扩展名的列表  
-D, –domains=LIST 分号分隔的被接受域的列表  
–exclude-domains=LIST 分号分隔的不被接受的域的列表  
–follow-ftp 跟踪 HTML 文档中的 FTP 链接  
–follow-tags=LIST 分号分隔的被跟踪的 HTML 标签的列表  
-G, –ignore-tags=LIST 分号分隔的被忽略的 HTML 标签的列表  
-H, –span-hosts 当递归时转到外部主机  
-L, –relative 仅仅跟踪相对链接  
-I, –include-directories=LIST 允许目录的列表  
-X, –exclude-directories=LIST 不被包含目录的列表  
-np, –no-parent 不要追溯到父目录