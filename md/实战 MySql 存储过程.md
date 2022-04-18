> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1649843262&ver=3736&signature=*rVkrOUVgrcMkY6ieuR83cus1HudLlHZOFlXIdkoNOw0T0w02GdlEy0vp06Cz7z6Ylumu3bP09JSsggII3Ru5ZAJyXgEYK8yF01WmCrrELMsYSc1GQPkpAxsVKCNxD-J&new=1)

**一、存储过程基本用法**

**1、创建存储过程**

MySQL 中，创建存储过程的基本形式如下：

其中参数列表的形式如下：

```
[IN|OUT|INOUT] param_name type

```

其中 in 表示输入参数，out 表示输出参数，inout 表示既可以输入也可以输出；param_name 表示参数名称；type 表示参数的类型，该类型可以是 MYSQL 数据库中的任意类型。

例子：下面的语句创建一个查询 tb_user 表全部数据的存储过程

```
DROP PROCEDURE IF EXISTS sp_test;

DELIMITER //
CREATE PROCEDURE sp_test()
BEGIN
    SELECT * FROM tb_user;
END //
DELIMITER ;

```

（1）由括号包围的参数列必须总是存在。如果没有参数，也该使用一个空参数列 ()。每个参数默认都是一个 IN 参数。要指定为其它参数，可在参数名之前使用关键词 OUT 或 INOUT

（2）"DELIMITER //" 语句的作用是将 MYSQL 的结束符设置为 //，因为 MYSQL 默认的语句结束符为分号; ，存储过程中的 SQL 语句需要分号来结束，为了避免与存储过程中 SQL 语句结束符相冲突，需要使用 DELIMITER 改变存储过程的结束符，并以 "END //" 结束存储过程。存储过程定义完毕之后再使用 DELIMITER ; 恢复默认结束符。DELIMITER 也可以指定其他符号为结束符。**注意：当使用 DELIMITER 命令时，应该避免使用反斜杠（\）字符，因为反斜杠是 MYSQL 的转义字符！！！**

DELIMITER 是分割符的意思，其实就是定义了一个语句执行的结束符，类似函数 or 存储过程这样的 create 语句由于其中包含了很多的 ";"，而默认 MySQL 的结束符就是 ";"，那么当我们创建的时候就会报错，有了 DELIMITER 就可以告诉 mysql 解释器，该段命令是否已经结束了，mysql 是否可以执行了。 

默认情况下，delimiter 是分号 ";"。在命令行客户端中，如果有一行命令以分号结束， 那么回车后，mysql 将会执行该命令。如输入下面的语句 

```
mysql> select * from stu;

```

然后回车，那么 MySQL 将立即执行该语句。但有时候，不希望 MySQL 这么做。因为可能输入较多的语句，且语句中包含有分号。 默认情况下，不可能等到用户把这些语句全部输入完之后，再执行整段语句。因为 mysql 一遇到分号，它就要自动执行。 即，在语句之后为 ";" 时，mysql 解释器就要执行了。 这种情况下，就需要事先把 delimiter 换成其它符号，如 // 或 $$ 等其他符号。 这样只有当 $$ 出现之后，mysql 解释器才会执行这段语句 。记得最后一个要将结束符修改回 ";"。

**2、删除存储过程**

语法：

```
DROP PROCEDURE  IF  EXISTS  存储过程名;


```

eg

```
DROP PROCEDURE IF EXISTS proc_employee;

```

这个语句被用来移除一个存储程序。不能在一个存储过程中删除另一个存储过程，只能调用另一个存储过程。

**3、调用存储过程**

语法：

CALL 存储过程名 (参数列表);

注：

  
（1）CALL 语句是用来调用一个先前用 CREATE PROCEDURE 创建的存储过程。  
（2）CALL 语句可以用声明为 OUT 或 INOUT 参数的参数给它的调用者传回值。  
（3）存储过程名称后面必须加括号，哪怕该存储过程没有参数传递。

**二、存储过程体**

存储过程体中可以使用各种 sql 语句和过程式语句的组合，来封装数据库应用中复杂的业务逻辑和处理规则，以实现数据库应用的灵活编程。下面主要介绍几个用于构造存储过程体的常用语法元素。

**1、变量的使用**

在存储过程和函数中，可以定义和使用变量。用户可以使用 DECLARE 关键字来定义变量。然后可以为变量赋值。这些变量的作用范围是 BEGIN…END 程序段中。

```
DECLARE  var_name[,...] type  [DEFAULT value]

```

【示例】 下面定义变量 my_sql，数据类型为 INT 型，默认值为 10。代码如下：

DECLARE  my_sql  INT  DEFAULT 10 ;

**使用说明：**

局部变量只能在存储过程体的 begin…end 语句块中声明。

  
局部变量必须在存储过程体的开头处声明。

  
局部变量的作用范围仅限于声明它的 begin..end 语句块，其他语句块中的语句不可以使用它。

  
局部变量不同于用户变量，两者区别：局部变量声明时，在其前面没有使用 @符号，并且它只能在 begin..end 语句块中使用；而用户变量在声明时，会在其名称前面使用 @符号，同时已声明的用户变量存在于整个会话之中。

**（2）为变量赋值**

MySQL 中可以使用 SET 关键字来为变量赋值。SET 语句的基本语法如下：

```
SET  var_name = expr [, var_name = expr] ...

```

其中，SET 关键字是用来为变量赋值的；var_name 参数是变量的名称；expr 参数是赋值表达式。一个 SET 语句可以同时为多个变量赋值，各个变量的赋值语句之间用逗号隔开。

【示例】下面为变量 my_sql 赋值为 30。代码如下：

```
SET  my_sql = 30 ;

```

MySQL 中还可以使用 SELECT…INTO 语句为变量赋值。其基本语法如下：

```
SELECT  col_name[,…] INTO  var_name[,…] FROM  table_name WEHRE condition

```

其中，col_name 参数表示查询的字段名称；var_name 参数是变量的名称；table_name 参数指表的名称；condition 参数指查询条件。

【示例】 下面从 employee 表中查询 id 为 2 的记录，将该记录的 d_id 值赋给变量 my_sql。代码如下：

```
SELECT  d_id INTO  my_sql FROM  employee WEHRE id=2 ;

```

 **2、定义条件和处理程序**

特定条件需要特定处理。这些条件可以联系到错误，以及子程序中的一般流程控制。定义条件是事先定义程序执行过程中遇到的问题。处理程序定义了在遇到这些问题时候应当采取的处理方式，并且保证存储过程或函数在遇到警告或错误时能继续执行。这样可以增强存储程序处理问题的能力，避免程序异常停止运行。

**（1）定义条件**

MySQL 中可以使用 DECLARE 关键字来定义条件。其基本语法如下：

```
DECLARE condition_name CONDITION FOR[condition_type]
[condition_type]:
SQLSTATE[VALUE] sqlstate_value |mysql_error_code

```

condition_name：表示条件名称

condition_type：表示条件的类型

sqlstate_value 和 mysql_error_code 都可以表示 mysql 错误，sqlstate_value 为长度 5 的字符串错误代码，mysql_error_code 为数值类型错误代码，例如：ERROR1142(42000) 中，sqlstate_value 的值是 42000，mysql_error_code 的值是 1142。

这个语句指定需要特殊处理条件。他将一个名字和指定的错误条件关联起来。这个名字随后被用在定义处理程序的 DECLARE HANDLER 语句中。

【示例】定义 ERROR1148(42000) 错误，名称为 command_not_allowed。

可以用两种方法定义

**（2）定义处理程序**

MySQL 中可以使用 DECLARE 关键字来定义处理程序。其基本语法如下：

其中，handler_type 参数指明错误的处理方式，该参数有 3 个取值。这 3 个取值分别是 CONTINUE、EXIT 和 UNDO。

CONTINUE 表示遇到错误不进行处理，继续向下执行；

EXIT 表示遇到错误后马上退出；

UNDO 表示遇到错误后撤回之前的操作，MySQL 中暂时还不支持这种处理方式。

注意：通常情况下，执行过程中遇到错误应该立刻停止执行下面的语句，并且撤回前面的操作。但是，MySQL 中现在还不能支持 UNDO 操作。因此，遇到错误时最好执行 EXIT 操作。如果事先能够预测错误类型，并且进行相应的处理，那么可以执行 CONTINUE 操作。

condition_value 参数指明错误类型，该参数有 6 个取值。sqlstate_value 和 mysql_error_code 与条件定义中的是同一个意思。condition_name 是 DECLARE 定义的条件名称。SQLWARNING 表示所有以 01 开头的 sqlstate_value 值。NOT FOUND 表示所有以 02 开头的 sqlstate_value 值。SQLEXCEPTION 表示所有没有被 SQLWARNING 或 NOT FOUND 捕获的 sqlstate_value 值。sp_statement 表示一些存储过程或函数的执行语句。

【示例】下面是定义处理程序的几种方式。代码如下：

上述代码是 6 种定义处理程序的方法。

第一种方法是捕获 sqlstate_value 值。如果遇到 sqlstate_value 值为 42000，执行 CONTINUE 操作，并且输出 "CAN NOT FIND" 信息。

第二种方法是捕获 mysql_error_code 值。如果遇到 mysql_error_code 值为 1148，执行 CONTINUE 操作，并且输出 "CAN NOT FIND" 信息。

第三种方法是先定义条件，然后再调用条件。这里先定义 can_not_find 条件，遇到 1148 错误就执行 CONTINUE 操作。

第四种方法是使用 SQLWARNING。SQLWARNING 捕获所有以 01 开头的 sqlstate_value 值，然后执行 EXIT 操作，并且输出 "ERROR" 信息。

第五种方法是使用 NOT FOUND。NOT FOUND 捕获所有以 02 开头的 sqlstate_value 值，然后执行 EXIT 操作，并且输出 "CAN NOT FIND" 信息。

第六种方法是使用 SQLEXCEPTION。SQLEXCEPTION 捕获所有没有被 SQLWARNING 或 NOT FOUND 捕获的 sqlstate_value 值，然后执行 EXIT 操作，并且输出 "ERROR" 信息。

【示例】定义条件和处理程序

![](https://mmbiz.qpic.cn/mmbiz_jpg/Ka2Jc290x6ickFX7M0NT2EFN8GHBc4eYZcGTD5ZhicSKVhAdJn2mBv3lkibuJAV7PNM5qE4TiaPb0lwRSlYAIF2Aag/640?wx_fmt=jpeg)

@X 是一个用户变量，执行结果 @X 等于 3，这表明 MYSQL 执行到程序的末尾。

如果 DECLARE CONTINUE HANDLER FOR SQLSTATE '23000' SET @X2=1;，这一行不存在

第二个 INSERT 因 PRIMARY KEY 约束而失败之后，MYSQL 可能已经采取 EXIT 策略，并且 SELECT @X 可能已经返回 2。

注意：@X 表示用户变量，使用 SET 语句为其赋值，用户变量与连接有关，一个客户端定义的变量不能被其他客户端所使用，即有作用域的，该客户端退出时，客户端连接的所有变量将自动释放。

**3、光标**

MYSQL 里叫光标，SQLSERVER 里叫游标，实际上一样的。

查询语句可能查询出多条记录，在存储过程和函数中使用光标来逐条读取查询结果集中的记录。

光标的使用包括声明光标、打开光标、使用光标和关闭光标。光标必须声明在处理程序之前，并且声明在变量和条件之后。

**（1）声明光标**

MySQL 中使用 DECLARE 关键字来声明光标。其语法的基本形式如下：

```
DECLARE cursor_name CURSOR FOR select_statement ;

```

其中，cursor_name 参数表示光标的名称；select_statement 参数表示 SELECT 语句的内容，返回一个用于创建光标的结果集。

【示例】下面声明一个名为 cur_employee 的光标。代码如下：

```
DECLARE cur_employee CURSOR FOR SELECT name, age FROM employee ;

```

上面的示例中，光标的名称为 cur_employee；SELECT 语句部分是从 employee 表中查询出 name 和 age 字段的值。

**（2）打开光标**

MySQL 中使用 OPEN 关键字来打开光标。其语法的基本形式如下：

```
OPEN  cursor_name ;

```

其中，cursor_name 参数表示光标的名称。

【示例】下面打开一个名为 cur_employee 的光标，代码如下：

```
OPEN  cur_employee ;

```

**（3）使用光标**  

MySQL 中使用 FETCH 关键字来使用光标。其语法的基本形式如下：

```
FETCH cursor_name INTO var_name[,var_name…] ;

```

其中，cursor_name 参数表示光标的名称；var_name 参数表示将光标中的 SELECT 语句查询出来的信息存入该参数中。var_name 必须在声明光标之前就定义好。

【示例】下面使用一个名为 cur_employee 的光标。将查询出来的数据存入 emp_name 和 emp_age 这两个变量中，代码如下：

```
FETCH  cur_employee INTO emp_name, emp_age ;

```

上面的示例中，将光标 cur_employee 中 SELECT 语句查询出来的信息存入 emp_name 和 emp_age 中。emp_name 和 emp_age 必须在前面已经定义。

**（4）关闭光标**

MySQL 中使用 CLOSE 关键字来关闭光标。其语法的基本形式如下：

```
CLOSE  cursor_name ;

```

其中，cursor_name 参数表示光标的名称。

【示例】 下面关闭一个名为 cur_employee 的光标。代码如下：

```
CLOSE  cur_employee ;

```

上面的示例中，关闭了这个名称为 cur_employee 的光标。关闭之后就不能使用 FETCH 来使用光标了。

**注意：MYSQL 中，光标只能在存储过程和函数中使用！！** 

**4、流程控制的使用**

存储过程和函数中可以使用流程控制来控制语句的执行。

MySQL 中可以使用 IF 语句、CASE 语句、LOOP 语句、LEAVE 语句、ITERATE 语句、REPEAT 语句和 WHILE 语句来进行流程控制。

每个流程中可能包含一个单独语句，或者是使用 BEGIN...END 构造的复合语句，构造可以被嵌套。

**（1）IF 语句**

IF 语句用来进行条件判断。根据是否满足条件，将执行不同的语句。其语法的基本形式如下：

```
IF search_condition THEN statement_list
[ELSEIF search_condition THEN statement_list] ...
[ELSE statement_list]
END IF

```

其中，search_condition 参数表示条件判断语句；statement_list 参数表示不同条件的执行语句。

注意：MYSQL 还有一个 IF() 函数，他不同于这里描述的 IF 语句

下面是一个 IF 语句的示例。代码如下：

```
IF age>20 THEN SET @count1=@count1+1;
ELSEIF age=20 THEN SET @count2=@count2+1;
ELSE SET @count3=@count3+1;
END IF;

```

该示例根据 age 与 20 的大小关系来执行不同的 SET 语句。如果 age 值大于 20，那么将 count1 的值加 1；如果 age 值等于 20，那么将 count2 的值加 1；其他情况将 count3 的值加 1。IF 语句都需要使用 END IF 来结束。

**（2）CASE 语句**

CASE 语句也用来进行条件判断，其可以实现比 IF 语句更复杂的条件判断。CASE 语句的基本形式如下：

```
CASE case_value
WHEN when_value THEN statement_list
[WHEN when_value THEN statement_list] ...
[ELSE statement_list]
END CASE

```

其中，case_value 参数表示条件判断的变量；

when_value 参数表示变量的取值；

statement_list 参数表示不同 when_value 值的执行语句。

CASE 语句还有另一种形式。该形式的语法如下：

```
CASE 
WHEN search_condition THEN statement_list
[WHEN search_condition THEN statement_list] ...
[ELSE statement_list]
END CASE

```

其中，search_condition 参数表示条件判断语句；  

statement_list 参数表示不同条件的执行语句。

下面是一个 CASE 语句的示例。代码如下：

```
CASE age 
WHEN 20 THEN SET @count1=@count1+1;
ELSE SET @count2=@count2+1;
END CASE ;

```

代码也可以是下面的形式：

```
CASE 
WHEN age=20 THEN SET @count1=@count1+1;
ELSE SET @count2=@count2+1;
END CASE ;

```

本示例中，如果 age 值为 20，count1 的值加 1；否则 count2 的值加 1。CASE 语句都要使用 END CASE 结束。

**注意：这里的 CASE 语句和 “控制流程函数” 里描述的 SQL CASE 表达式的 CASE 语句有轻微不同。这里的 CASE 语句不能有 ELSE NULL 子句 并且用 END CASE 替代 END 来终止！！** 

**（3）LOOP 语句**

LOOP 语句可以使某些特定的语句重复执行，实现一个简单的循环。但是 LOOP 语句本身没有停止循环的语句，必须是遇到 LEAVE 语句等才能停止循环。

LOOP 语句的语法的基本形式如下：

```
[begin_label:] LOOP
statement_list
END LOOP [end_label]

```

其中，begin_label 参数和 end_label 参数分别表示循环开始和结束的标志，这两个标志必须相同，而且都可以省略；

statement_list 参数表示需要循环执行的语句。

下面是一个 LOOP 语句的示例。代码如下：

```
add_num: LOOP
SET @count=@count+1;
END LOOP add_num ;

```

该示例循环执行 count 加 1 的操作。因为没有跳出循环的语句，这个循环成了一个死循环。LOOP 循环都以 END LOOP 结束。

**（4）LEAVE 语句**

LEAVE 语句主要用于跳出循环控制。其语法形式如下：

```
LEAVE label

```

其中，label 参数表示循环的标志。

下面是一个 LEAVE 语句的示例。代码如下：

```
add_num: LOOP
SET @count=@count+1;
IF @count=100 THEN
LEAVE add_num ;
END LOOP add_num ;

```

该示例循环执行 count 加 1 的操作。当 count 的值等于 100 时，则 LEAVE 语句跳出循环。

**（5）ITERATE 语句**

ITERATE 语句也是用来跳出循环的语句。但是，ITERATE 语句是跳出本次循环，然后直接进入下一次循环。

ITERATE 语句只可以出现在 LOOP、REPEAT、WHILE 语句内。

ITERATE 语句的基本语法形式如下：

```
ITERATE label

```

下面是一个 ITERATE 语句的示例。代码如下：

```
add_num: LOOP 
SET @count=@count+1;
IF @count=100 THEN
LEAVE add_num ;
ELSE IF MOD(@count,3)=0 THEN
ITERATE add_num;
SELECT * FROM employee ;
END LOOP add_num ;

```

该示例循环执行 count 加 1 的操作，count 值为 100 时结束循环。如果 count 的值能够整除 3，则跳出本次循环，不再执行下面的 SELECT 语句。

说明：LEAVE 语句和 ITERATE 语句都用来跳出循环语句，但两者的功能是不一样的。

LEAVE 语句是跳出整个循环，然后执行循环后面的程序。而 ITERATE 语句是跳出本次循环，然后进入下一次循环。

使用这两个语句时一定要区分清楚。

**（6）REPEAT 语句**

REPEAT 语句是有条件控制的循环语句。当满足特定条件时，就会跳出循环语句。REPEAT 语句的基本语法形式如下：

```
[begin_label:] REPEAT
statement_list
UNTIL search_condition
END REPEAT [end_label]

```

其中，statement_list 参数表示循环的执行语句；search_condition 参数表示结束循环的条件，满足该条件时循环结束。

下面是一个 REPEAT 语句的示例。代码如下：

```
REPEAT
SET @count=@count+1;
UNTIL @count=100
END REPEAT ;

```

该示例循环执行 count 加 1 的操作，count 值为 100 时结束循环。REPEAT 循环都用 END REPEAT 结束。

**（7）WHILE 语句**

WHILE 语句也是有条件控制的循环语句。但 WHILE 语句和 REPEAT 语句是不一样的。WHILE 语句是当满足条件时，执行循环内的语句。

WHILE 语句的基本语法形式如下：

```
[begin_label:] WHILE search_condition DO 
statement_list
END WHILE [end_label]

```

其中，search_condition 参数表示循环执行的条件，满足该条件时循环执行；

statement_list 参数表示循环的执行语句。

下面是一个 WHILE 语句的示例。代码如下：

```
WHILE @count<100 DO
SET @count=@count+1;
END WHILE ;

```

该示例循环执行 count 加 1 的操作，count 值小于 100 时执行循环。

如果 count 值等于 100 了，则跳出循环。WHILE 循环需要使用 END WHILE 来结束。

**三、实例**

1、mysql 通用分页存储过程

```
DELIMITER //
DROP PROCEDURE IF EXISTS pr_pager;

CREATE PROCEDURE pr_pager(
    IN p_table_name VARCHAR(100), -- 表名称
    IN p_fields VARCHAR(500), -- 要显示的字段
    IN pagecurrent INT, -- 当前页
    IN pagesize INT, -- 每页显示的记录数
    IN p_where VARCHAR(500) CHARSET utf8, -- 查询条件
    IN p_order VARCHAR(100), -- 排序
    OUT totalcount INT                        -- 总记录数
)
BEGIN
IF pagesize <= 1 THEN
        SET pagesize = 20;
END IF;
IF pagecurrent THEN
    SET pagecurrent = 1;
END IF;

SET @startIndex = (pagecurrent-1)*pagesize;
SET @endIndex = pagesize;

SET @strsql = CONCAT('select ',p_fields,' from ',p_table_name,
CASE IFNULL(p_where,'') WHEN '' THEN '' ELSE CONCAT(' where ',p_where) END,
CASE IFNULL(p_order,'') WHEN '' THEN '' ELSE CONCAT(' order by ',p_order) END,
' limit ',@startIndex,',',@endIndex);
-- 预定义一个语句，并将它赋给stmtsql
PREPARE stmtsql FROM @strsql;
EXECUTE stmtsql;
-- 释放一个预定义语句的资源
DEALLOCATE PREPARE stmtsql;

SET @strsqlcount = CONCAT('select count(*) into @Rows_Total from ',p_table_name,
CASE IFNULL(p_where,'') WHEN '' THEN '' ELSE CONCAT(' where ',p_where) END);

PREPARE stmtsqlcount FROM @strsqlcount;
EXECUTE stmtsqlcount;
DEALLOCATE PREPARE stmtsqlcount;

SET totalcount = @Rows_Total;
-- 计算总数也可以是下面这种方法
-- SELECT COUNT(*) INTO totalcount FROM tb_user;
END //

DELIMITER ;

```

2、存储过程调用

（1）不带查询条件和排序

```
CALL pr_pager('t_user','id,username,birthday,sex,address',1,5,
NULL,NULL,@totalcount);

SELECT @totalcount;

```

![](https://mmbiz.qpic.cn/mmbiz_png/Ka2Jc290x6ickFX7M0NT2EFN8GHBc4eYZKJyibsPsM6DnFfszY3n5licKrymLyOWofIOmBjcLm9DNY8JUuewy1nCA/640?wx_fmt=png)

（2）带查询条件和排序

```
CALL pr_pager('t_user','id,username,birthday,sex,address',1,5,
'username like \'小%\'','id asc',@totalcount);

SELECT @totalcount;

```

![](https://mmbiz.qpic.cn/mmbiz_png/Ka2Jc290x6ickFX7M0NT2EFN8GHBc4eYZNGicX7SNhTJn2hPeaToWIkETVBQdsoqoJJaAI0PNDunkx41B1e84xog/640?wx_fmt=png)

链接：cnblogs.com/xiaoxi/p/6398347.html