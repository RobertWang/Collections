> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u010168781/article/details/103291193)

### 目录

*   *   *   *   [一、参考网址](#_1)
            *   [二、详解](#_5)
            *   *   [1、查看、设置 sqlite 限制命令. limit](#1sqlitelimit_6)
                *   [2、SQLite 中的限制汇总](#2SQLite_22)
                *   *   [1）字符串或 BLOB 的最大长度](#1BLOB_23)
                    *   [2）最大列数](#2_34)
                    *   [3）SQL 语句的最大长度](#3SQL_47)
                    *   [4）联接中的最大表数](#4_51)
                    *   [5）表达式树的最大深度](#5_53)
                    *   [6）函数的最大参数个数](#6_58)
                    *   [7）复合 SELECT 语句中的最大项数](#7SELECT_62)
                    *   [8）LIKE 或 GLOB 模式的最大长度](#8LIKEGLOB_67)
                    *   [9）单个 SQL 语句中的最大占位符个数](#9SQL_72)
                    *   [10）触发递归的最大深度](#10_76)
                    *   [11）数据库关联的最大数](#11_82)
                    *   [12）数据库文件中的最大页数](#12_87)
                    *   [13）表中的最大行数](#13_94)
                    *   [14）数据库大小限制](#14_97)
                    *   [15）表个数限制](#15_99)
                *   [3、运行时限制类别](#3_102)

#### 一、参考网址

[SQLite](https://so.csdn.net/so/search?q=SQLite&spm=1001.2101.3001.7020) 中的限制（官网）：https://sqlite.org/limits.html  
运行时限制类别（官网）：https://sqlite.org/c3ref/c_limit_attached.html#sqlitelimitcolumn

#### 二、详解

##### 1、查看、设置 sqlite 限制命令. limit

```
sqlite> .limit	// 显示或设置数据库限制信息：SQLITE_LIMIT
              length 1000000000	// 字符串或BLOB的最大长度10亿，一行的最大长度
          sql_length 1000000000	// sql语句最大长度
              column 2000	// 列数，可以在编译时才可以将最大列出改为32767
          expr_depth 1000	// 表达式树的最大深度，SQLite将表达式解析到树中进行处理。
     compound_select 500	// 复合SELECT语句中的最大术语数
             vdbe_op 25000	// 虚拟机程序中用于实现SQL语句的最大指令数
        function_arg 127	// 一个函数的最大参数个数
            attached 10		// ATTACH语句，附加数据库最大值为125
 like_pattern_length 50000	// LIKE模式匹配算法或GLOB模式的最大长度
     variable_number 250000	// 任何参数的索引号
       trigger_depth 1000	// 触发递归的最大深度
      worker_threads 0		// 可以启动的辅助工作线程的最大数量

```

##### 2、SQLite 中的限制汇总

###### 1）字符串或 BLOB 的最大长度

注：BLOB 是 sqlite 的一种类型，用于存储二进制。  
限制 SQLite 中字符串、BLOB 类型值、数据库一行的最大字节数，默认值为 10 亿，最大为 2147483647；  
在编译时，通过 SQLITE_MAX_LENGTH 来设置；

```
-DSQLITE_MAX_LENGTH = 123456789

```

在运行时，通过 sqlite3_limit(db, SQLITE_LIMIT_LENGTH, size) 来降低该值；

官方建议：最好将最大字符串长度和 blob 长度减小到几百万。  
在 SQLite 的 INSERT 和 SELECT 处理的一部分期间，数据库中每一行的全部内容被编码为单个 BLOB。因此，SQLITE_MAX_LENGTH 参数还确定一行中的最大字节数。

###### 2）最大列数

最大列数用于限制一下项的上限：

```
表中的列数
索引中的列数
视图中的列数
UPDATE语句的SET子句中的术语数
SELECT语句的结果集中的列数
GROUP BY或ORDER BY子句中的术语数
INSERT语句中的值数

```

默认值为 2000，最大为 32767；  
在编译时，通过 SQLITE_MAX_COLUMN 来设置，  
在运行时，通过 sqlite3_limit(db, SQLITE_LIMIT_COLUMN,size）来降低最大列数。

###### 3）SQL 语句的最大长度

对 SQL 语句的最大字节数的限制，默认值为 1000000，最大为 SQLITE_MAX_LENGTH 和 1073741824 中较小的一个。  
在编译时，通过 SQLITE_MAX_SQL_LENGTH 来设置；  
在运行时，通过 sqlite3_limit(db, SQLITE_LIMIT_SQL_LENGTH, size) 来降低该值。

###### 4）联接中的最大表数

SQLite 不支持包含超过 64 个表的联接。不可更改

###### 5）表达式树的最大深度

限制 SQL 语句表达式的复杂程度；  
编译时通过 SQLITE_MAX_EXPR_DEPTH 参数设置，默认是 1000；  
如果 SQLITE_MAX_EXPR_DEPTH 为正，可以在运行时通过 sqlite3_limit(db,SQLITE_LIMIT_EXPR_DEPTH,size) 来降低；  
如果 SQLITE_MAX_EXPR_DEPTH 为 0，则不受限制，上面的接口无效。

###### 6）函数的最大参数个数

限制函数的参数个数，默认值是 100，最大为 127；  
在编译时，通过 SQLITE_MAX_FUNCTION_ARG 来设置最大值；  
在运行时，通过 sqlite3_limit(db,SQLITE_LIMIT_FUNCTION_ARG,size) 来降低该值。

###### 7）复合 SELECT 语句中的最大项数

复合 SELECT 语句是通过运算符 UNION，UNION ALL，EXCEPT 或 INTERSECT 连接的两个或多个 SELECT 语句。每个单独的 SELECT 语句称为 “项”。  
默认值为 500；官方不建议再增大；  
在编译时，通过 SQLITE_MAX_COMPOUND_SELECT 来设置最大值；  
在运行时，通过 qlite3_limit(db,SQLITE_LIMIT_COMPOUND_SELECT,size) 来降低该值。

###### 8）LIKE 或 GLOB 模式的最大长度

LIKE、GLOB 运算符：模式匹配比较，类似正则表达式；  
限制 LIKE 或 GLOB 模式的表达式的字符长度；默认值为 50000，官方不建议再增大；  
在编译时，通过 SQLITE_MAX_LIKE_PATTERN_LENGTH 来设置最大值；  
在运行时，通过 sqlite3_limit(db,SQLITE_LIMIT_LIKE_PATTERN_LENGTH,size) 来降低该值。

###### 9）单个 SQL 语句中的最大占位符个数

限制 C 或 C++ SQL 接口中使用的占位符的个数；默认为 999，最大值 10 亿；  
在编译时，通过 SQLITE_MAX_VARIABLE_NUMBER 来设置最大值；  
在运行时，通过 sqlite3_limit(db,SQLITE_LIMIT_VARIABLE_NUMBER,size) 来降低该值。

###### 10）触发递归的最大深度

SQLite 触发器（Trigger）是数据库的回调函数，它会在指定的数据库事件发生时自动执行 / 调用。  
限制回调函数递归的深度。  
默认值是 1000.；  
在编译时，通过 SQLITE_MAX_TRIGGER_DEPTH 来设置最大值；  
在运行时，无法设置；

###### 11）数据库关联的最大数

使用 ATTACH 可以将多个数据库关联到一起，这样在操作时，就像操作一个数据库。  
限制关联数据库的个数，默认值 10，最大 125；  
在编译时，通过 SQLITE_MAX_ATTACHED 来设置最大值；  
在运行时，通过 sqlite3_limit(db,SQLITE_LIMIT_ATTACHED,size) 来降低该值。

###### 12）数据库文件中的最大页数

防止单个数据库文件过大；数据库文件大小由页数和单个页大小决定；  
默认值为 1073741823，最大 2147483646  
在编译时，通过 SQLITE_MAX_PAGE_COUNT 来设置；  
在运行时，通过 PRAGMA schema.max_page_count 来查询和设置该值；  
在插入新数据时，页数将要超过该值，会返回 SQLITE_FULL。  
如果页数最大为 2147483646，页大小为 65536 时，数据库大小约为 140 TB。

###### 13）表中的最大行数

表中的理论最大行数为 2^ 64（18446744073709551616 或大约 1.8e+19）。由于将首先达到 140 TB 的最大数据库大小，因此无法达到此限制。一个 140 TB 的数据库最多可以容纳大约 1e+13 行。  
无法改变该值。

###### 14）数据库大小限制

页数最大为 2147483646，页大小为 65536 时，数据库大小约为 140 TB

###### 15）表个数限制

保存每个表的描述信息需要一页，因此该表的数量不会超过页数。  
初始化数据库时，需要扫描所有表的描述信息、并保存在内存中。因此数据库连接启动时间和初始内存使用量和表的数量成正比。

##### 3、运行时限制类别

```
#define SQLITE_LIMIT_LENGTH                    0
#define SQLITE_LIMIT_SQL_LENGTH                1
#define SQLITE_LIMIT_COLUMN                    2
#define SQLITE_LIMIT_EXPR_DEPTH                3
#define SQLITE_LIMIT_COMPOUND_SELECT           4
#define SQLITE_LIMIT_VDBE_OP                   5
#define SQLITE_LIMIT_FUNCTION_ARG              6
#define SQLITE_LIMIT_ATTACHED                  7
#define SQLITE_LIMIT_LIKE_PATTERN_LENGTH       8
#define SQLITE_LIMIT_VARIABLE_NUMBER           9
#define SQLITE_LIMIT_TRIGGER_DEPTH            10
#define SQLITE_LIMIT_WORKER_THREADS           11

```

这些常量定义了各种性能限制，可以在运行时使用 sqlite3_limit 降低这些限制

```
int sqlite3_limit(sqlite3*, int id, int newVal);

```

id 取值如下：  
SQLITE_LIMIT_LENGTH：任何字符串或 BLOB 或表行的最大大小，以字节为单位。

SQLITE_LIMIT_SQL_LENGTH：SQL 语句的最大长度，以字节为单位。

SQLITE_LIMIT_COLUMN：表定义或 SELECT 结果集中的最大列数，或索引或 ORDER BY 或 GROUP BY 子句中的最大列数。

SQLITE_LIMIT_EXPR_DEPTH：任何表达式上分析树的最大深度。

SQLITE_LIMIT_COMPOUND_SELECT：复合 SELECT 语句中的最大术语数。

SQLITE_LIMIT_VDBE_OP：虚拟机程序中用于实现 SQL 语句的最大指令数。如果 sqlite3_prepare_v2（）或等效方法试图在单个准备好的语句中为多个操作码分配空间，则将返回 SQLITE_NOMEM 错误。

SQLITE_LIMIT_FUNCTION_ARG：一个函数的最大参数个数。

SQLITE_LIMIT_ATTACHED：附加数据库的最大数量。

SQLITE_LIMIT_LIKE_PATTERN_LENGTH：LIKE 或 GLOB 运算符的 pattern 参数的最大长度。

SQLITE_LIMIT_VARIABLE_NUMBER：SQL 语句中任何参数的最大索引号。

SQLITE_LIMIT_TRIGGER_DEPTH：触发器的最大递归深度。

SQLITE_LIMIT_WORKER_THREADS：一个准备好的语句可以启动的辅助工作线程的最大数量 。