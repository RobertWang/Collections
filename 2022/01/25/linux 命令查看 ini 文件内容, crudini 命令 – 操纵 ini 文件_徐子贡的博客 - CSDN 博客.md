> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_33431149/article/details/116552195?utm_medium=distribute.pc_feed_404.none-task-blog-2~default~BlogCommendFromBaidu~Rate-6.control404&depth_1-utm_source=distribute.pc_feed_404.none-task-blog-2~default~BlogCommendFromBaidu~Rate-6.control40)

crudini 是 Pádraig Brady 用 [Python](https://so.csdn.net/so/search?q=Python&spm=1001.2101.3001.7020) 开发的、用来对配置文件 (即 ini 文件) 进行编辑的工具。crud 是 4 个单词的首字母简写，即 create、read、update 和 delete，中文译为 “增删改查”。这个是数据的最常见的 4 类操作方法。有些软件的配置文件采用的是 ini 格式，如 php.ini。这样的配置文件往往会成若干个段落。段落以[default] 之类的格式标识。具体的配置条目则为 “datadir=/var/lib/data” 形式。

语法格式： crudini [参数] [文件]

常用参数：

--format=FMT 为 --get 使用，选择输出格式。格式有 sh,ini,lines

--inplace 锁定并写入文件, 比默认的替换有更少的限制

--list 为 --set 和 --del, 更新一个列表 (集合) 的值

--list-sep=STR 使用自定义的字符代替默认的逗号

--output=FILE 将输出写入文件。’-“表示标准输出”

--verbose 在错误输出上指出是否进行了更改

参考实例：

config_file 代表要操作的文件名，section 表示变量所在的部分，如以下配置文件：

[DEFAULT]

user = admin

passwd = admin

port = 8088

[URL]

client = 127.0.0.1:8088

admin = 127.0.0.1:8080

section 则表示了以上配置文件中的 [DEFAULT] 和 [URL] ，在命令中不需要加中括号 []，param 则如 user ，passwd，client 等。

添加或更新一个变量：

[root@linuxcool ~]# crudini --set config_file section parameter value

更新一个已存在的变量：

[root@linuxcool ~]# crudini --set --existing config_file section parameter value

删除一个变量：

[root@linuxcool ~]# crudini --del config_file section parameter

删除一个段：

[root@linuxcool ~]# crudini --del config_file section

获取一个值：

[root@linuxcool ~]# crudini --get config_file section parameter

获取一个不在段里面的值：

[root@linuxcool ~]# crudini --get config_file '' parameter

获取一个段：

[root@linuxcool ~]# crudini --get config_file section

将 linuxcool.ini 配置文件合并到 linuxprobe.ini 中：

[root@linuxcool ~]# crudini --merge linuxprobe.ini < linuxcool.ini