> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1461215&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) 番茄 av  自己做的学校一期小项目  
模拟 ATM 机操作  
使用 jar 包连接数据库  
[Java] _纯文本查看_ _复制代码_

```
private static Connection conn;//数据库连接对象
    //返回可用的数据库连接
    public static Connection getConnection(){
        try{
            if(conn!=null&&conn.isClosed()==false){
                return conn;
            }
            Class.forName("com.mysql.jdbc.Driver");
            conn=DriverManager.getConnection("jdbc:mysql://localhost:3306/ATM"
                    +"?useUnicode=true&characterEncoding=utf-8","/*数据库用户名*/","数据库密码");
            return conn;
        }catch(Exception ex){
            System.out.println("BaseDao.getConnection():"+ex.getMessage());
            return null;
        }
    }
```

然后添加数据库的增删改查操作  
我是用的是 PreparedStatement  
[Asm] _纯文本查看_ _复制代码_

```
//执行查询语句
    public static ResultSet selectSQL(String sql,String[]p){
        try{
            PreparedStatement ps=getConnection().prepareStatement(sql);
            if(p!=null){
                for(int i=0;i
```

其实关于 ATM 机的重要问题是存取钱和互相转账时数据库的同步问题  
下面就是在 Java 中使用事务  
首先在设计数据库时用无符号整型数 UNSIGNED 来设计余额  
这样在余额不足，或发生异常时可以回滚事务  
先关闭自动提交  
下面是模拟转账时的代码  
[Asm] _纯文本查看_ _复制代码_

```
//转账
    Scanner input=new Scanner (System.in);
    public void zhuan(){
        Connection conn=null;
        PreparedStatement ps=null;
        PreparedStatement ps1=null;
        try {
 
            conn=DriverManager.getConnection("jdbc:mysql://localhost:3306/ATM"
                    ,"root","root");//链接数据库
            conn.setAutoCommit(false);//关闭自动提交,改为手动提交。
            //接受用户输入的收款账号和金额
            System.out.println("请输入收款账户:");
            String x=input.next();
            System.out.println("请输入转账金额:");
            int y=input.nextInt();
            String n=Integer.toString(y);///强制转换，把整形转换为字符串
            //sql语句用来给收款账户添加金额
            String sql="update `ku` set `yu`=`yu`+? where `id`=?";
            //String [] p={n,x};
            ps=conn.prepareStatement(sql);
            ps.setString(1, n);
            ps.setString(2, x);
            ps.executeUpdate();
            //sql语句用来给出钱账户减出金额
            String sql1="update `ku` set `yu`=`yu`-? where `id`=?";
            ps1=conn.prepareStatement(sql1);
            ps1.setString(1, n);
            ps1.setString(2, qwe.z);
            ps1.executeUpdate();
            conn.commit();//提交事务
            System.out.println("转账成功");
        }catch(Exception ee){
            try {//回滚事务
                conn.rollback();
                System.out.println("账户余额不足");
            } catch (SQLException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }finally{
            try {
                ps1.close();
                ps.close();
                conn.close();
            } catch (SQLException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
```

Java+MySQL 的源代：（java 源码是 jjj 文件夹下的 src）  
链接：[一键提取](https://pan.baidu.com/s/11zN7aAJ2q3xb8JltxPWuXg#u2j2)  
提取码：u2j2  
复制这段内容后打开百度网盘手机 App，操作更方便哦