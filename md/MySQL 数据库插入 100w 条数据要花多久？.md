> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzkzNzIzNzczMA==&mid=2247488614&idx=1&sn=308c0a0eebb840b5bf58b2c2503d5d87&chksm=c293d9ebf5e450fd5ef07fdb59efaab3ca4cc60f57e55ba3e80a6cb141211f4e6467ca06b50f&mpshare=1&scene=1&srcid=0609QvBp2eHjzNSwRNn01aIs&sharer_sharetime=1623232289875&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

**后端实验室**

关注后端技术栈：分布式、容器、Kubernetes、微服务、云原生、Redis、数据库、消息队列、负载均衡、存储、安全性、高可用、高性能、ElasticSearch、监控等领域，分享技术、资讯、经验、坚持干货！

241 篇原创内容

公众号

1、多线程插入 (单表)  
2、多线程插入 (多表)  
3、预处理 SQL  
4、多值插入 SQL  
5、事务 (N 条提交一次)

### **多线程插入 (单表)**

**问：为何对同一个表的插入多线程会比单线程快？同一时间对一个表的写操作不应该是独占的吗？**

答：在数据里做插入操作的时候，整体时间的分配是这样的：

1、多链接耗时 （30%） 

2、多发送 query 到服务器 （20%） 

3、多解析 query （20%） 

4、多插入操作 （10% * 词条数目） 

5、多插入 index （10% * Index 的数目）

6、多关闭链接 （10%）

从这里可以看出来，真正耗时的不是操作，而是链接，解析的过程。

MySQL 插入数据在写阶段是独占的，但是插入一条数据仍然需要解析、计算、最后才进行写处理，比如要给每一条记录分配自增 id，校验主键唯一键属性，或者其他一些逻辑处理，都是需要计算的，所以说多线程能够提高效率。

### **多线程插入 (多表)**

分区分表后使用多线程插入。

### **预处理 SQL**

普通 SQL：即使用 Statement 接口执行 SQL

预处理 SQL：即使用 PreparedStatement 接口执行 SQL

使用 PreparedStatement 接口允许数据库预编译 SQL 语句，以后只需传入参数，避免了数据库每次都编译 SQL 语句，因此性能更好。

### **多值插入 SQL**

普通插入 SQL：INSERT INTO TBL_TEST (id) VALUES(1)

多值插入 SQL：INSERT INTO TBL_TEST (id) VALUES (1), (2), (3)

使用多值插入 SQL，SQL 语句的总长度减少，即减少了网络 IO，同时也降低了连接次数，数据库一次 SQL 解析，能够插入多条数据。

### **事务 (N 条提交一次)**

在一个事务中提交大量 INSERT 语句可以提高性能。

1、将表的存储引擎修改为 myisam

2、将 sql 拼接成字符串，每 1000 条左右提交事务。

*   执行多条 SQL 语句，实现数据库事务。 
    
*   mysql 数据库 
    
*   多条 SQL 语句
    

```
public void ExecuteSqlTran(List<string> SQLStringList)
{
    using (MySqlConnection conn = new MySqlConnection(connectionString))
    {
        if (DBVariable.flag)
        {
            conn.Open();
            MySqlCommand cmd = new MySqlCommand();
            cmd.Connection = conn;
            MySqlTransaction tx = conn.BeginTransaction();
            cmd.Transaction = tx;
            try
            {
                for (int n = 0; n < SQLStringList.Count; n++)
                {
                    string strsql = SQLStringList[n].ToString();
                    if (strsql.Trim().Length > 1)
                    {
                        cmd.CommandText = strsql;
                        cmd.ExecuteNonQuery();
                    }
                    if (n > 0 && (n % 1000 == 0 || n == SQLStringList.Count - 1))
                    {
                        tx.Commit();
                        tx = conn.BeginTransaction();
                    }
                }
            }
            catch (System.Data.SqlClient.SqlException E)
            {
                tx.Rollback();
                throw new Exception(E.Message);
            }
        }
    }
}

```

10w 条数据大概用时 10s！

**参考资料：**

*   https://www.cnblogs.com/aicro/p/3851434.html
    
*   http://blog.jobbole.com/29432