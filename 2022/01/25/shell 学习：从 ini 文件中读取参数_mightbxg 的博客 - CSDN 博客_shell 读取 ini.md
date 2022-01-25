> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/mightbxg/article/details/79121775?utm_medium=distribute.pc_feed_404.none-task-blog-2~default~BlogCommendFromBaidu~Rate-4.control404&depth_1-utm_source=distribute.pc_feed_404.none-task-blog-2~default~BlogCommendFromBaidu~Rate-4.control40)

打算编写一个 shell 脚本来自动备份网站，需要从配置文件中读取一些参数，比如数据库名称、用户名和密码等。我分析了一下 wdcp 自带的数据库备份脚本 mysqlbackup.sh，里面仅仅从文件 mrpw.conf 中读取了数据库密码，只用到了 cat 而已：

```
mrpw=`cat /www/wdlinux/wdcp/conf/mrpw.conf`
```

这个方法虽然简易，但总不能每一个参数都用一个文件来保存吧。现在很多软件的配置参数都是用 MS ini 文件来保存的，使用方便而且可读性非常好，所以我在网上找了一个 [shell 读取 ini 文件的方法](http://www.jb51.net/article/60854.htm "Shell实现读取ini格式配置文件方法")，本文则是对该方法的学习总结。

这个方法主要知识点是 awk 命令以及正则表达式，相关资料：

*   [shell awk 入门](http://www.cnblogs.com/zhuyp1015/archive/2012/07/11/2586985.html "shell awk 入门")
*   [正则表达式 - 语法](http://www.runoob.com/regexp/regexp-syntax.html "正则表达式 - 语法")
*   [awk 正则表达式、正则运算符详细介绍](http://www.cnblogs.com/chengmo/archive/2010/10/11/1847772.html "awk 正则表达式、正则运算符详细介绍")

MS ini 文件格式可以参照[百度百科](https://baike.baidu.com/item/INI/9212321?fr=aladdin "MS ini 文件格式")，大致结构为：

```
[section1]
name1 = value1
name2 = value2

[section2]
name3 = value3
```

中括号里面是节，name 是键名，value 是键值

实现读取 ini 文件的函数代码看起来比较简单：

```
function readINI()
{
 FILENAME=$1; SECTION=$2; KEY=$3
 RESULT=`awk -F '=' '/\['$SECTION'\]/{a=1}a==1&&$1~/'$KEY'/{print $2;exit}' $FILENAME`
 echo $RESULT
}
```

调用方法（从名为 Filename 的文件中读取 Section 节中的 Key 键值保存到变量 Value 中）：

```
Value=$(readINI Filename Section Key)
```

**解析：**  
readINI() 函数接收三个参数分别传给 FILENAME、SECTION 和 KEY，然后调用 awk 命令在文件中查找指定键值。这里最重要的就是 awk 命令：

```
awk -F '=' '/\['$SECTION'\]/{a=1}a==1&&$1~/'$KEY'/{print $2;exit}' $FILENAME
```

1.  `-F '='` 表示用 “=” 作为分割符（多余的空格同样会被忽略）；
2.  `/\['$SECTION'\]/` 和 `/'$KEY'/` 是正则表达式，用于匹配节名和键名，注意在 awk 正则表达式中参数要用单引号括起来；
3.  `/\['$SECTION'\]/{a=1}` 表示逐行搜索到目标节时用变量 a 标记下来；
4.  `a==1&&$1~/'$KEY'/{print $2;exit}` 表示当目标节已找到（a==1）并且当前行第一个参数匹配键名时，打印第二个参数（键值）并退出，这里 **exit** 是为了防止后面其它节有相同的键名。

**注意：**  
该方法的一个缺陷是，如果节名和键名都存在但目标键并不在目标节下时会返回错误的搜索结果，因此在写 ini 文件的时候需要注意。