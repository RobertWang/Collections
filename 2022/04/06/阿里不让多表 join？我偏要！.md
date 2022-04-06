> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzU3NDgxMzI0Mw==&mid=2247503016&idx=2&sn=2ef4f8b0f9c6cc8a47276328b0f9e691&chksm=fd2e29fcca59a0ea665f707f8c7d4cada1d2d5d89b28c9482ef55f3ba6396c1b3d4abb15161d&mpshare=1&scene=1&srcid=0403snomkQduigXZplfBmEGV&sharer_sharetime=1649001374761&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

作者：谢斌 

来源：http://blog.itpub.net/30393770/viewspace-2650450/

阿里规范中强制要求不要多表 join，并且肥朝也给出了常见的把多表 join 转成单表操作的两个套路。那么问题来了，**阿里强制不让用，但是你偏要用，究竟会有什么后果?** 本文将用数据角度告诉你，你偏要用的话，会有什么后果，因此强烈建议跟着思路看完！

提出问题和环境准备
---------

《阿里巴巴 JAVA 开发手册》里面写超过三张表禁止 join，这是为什么？

对这个结论，你是否有怀疑呢？也不知道是哪位先哲说的不要人云亦云，今天我设计 sql，来验证这个结论。（实验没有从代码角度分析，目前达不到。可以把 mysql 当一个黑盒，使用角度来验证这个结论） 验证结论的时候，会有很多发现，各位往后看。

实验环境：vmware10+centos7.4+mysql5.7.22 ，centos7 内存 4.5G，4 核，50G 硬盘。mysql 配置为 2G，特别说明硬盘是 SSD。

我概述下我的实验：有 4 张表，student 学生表，teacher 老师表，course 课程表，sc 中间关系表，记录了学生选修课程以及分数。具体 sql 脚本，看文章结尾，我附上。中间我自己写了造数据的脚本，也在结尾。

![](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkiaX4xczdhtpjosWQ2K3eJkI16cxY4pquKuTqPiaXiaHERMf0cK5ZAocmbnbJgXdfpBymBcy6wuMXubA/640?wx_fmt=png)

实验是为解决一个问题的：查询选修 “tname553” 老师所授课程的学生中，成绩最高的学生姓名及其成绩  
。查询 sql 是：

```
select Student.Sname,course.cname,score 
    from Student,SC,Course ,Teacher 
    where Student.s_id=SC.s_id and SC.c_id=Course.c_id  and sc.t_id=teacher.t_id 
    and Teacher.Tname='tname553' 
    and SC.score=(select max(score)from SC where sc.t_id=teacher.t_Id);


```

我来分析一下这个语句：4 张表等值 join，还有一个子查询。算是比较简单的 sql 语句了（相比 ERP 动就 10 张表的哦，已经很简单了）。我 还会分解这个语句成 3 个简单的 sql：

```
 select max(score)  from SC ,Teacher where sc.t_id=teacher.t_Id and Teacher.Tname='tname553';
   select sc.t_id,sc.s_id,score   from SC ,Teacher 
   where sc.t_id=teacher.t_Id 
   and score=590  
   and Teacher.Tname='tname553';
    select Student.Sname,course.cname,score 
    from Student,SC ,course
    where Student.s_id=SC.s_id and  sc.s_id in (20769800,48525000,26280200) and course.c_id = sc.c_id;


```

我来分析下：第一句，就是查询最高分，得到最高分 590 分。第二句就是查询出最高分的学生 id，得到

```
20769800,48525000,26280200


```

。第三句就是查询出学生名字和分数。这样这 3 个语句的就可以查询出来成绩最高的学生姓名及其成绩。

接下来我会分别造数据：1 千万选课记录 (一个学生选修 2 门课), 造 500 万学生，100 万老师 (一个老师带 5 个学生，挺高端的吧)，1000 门课，。用上面查询语句查询。其中 sc 表我测试了下有索引和没有索引情况，具体见下表。再接下来，我会造 1 亿选课记录 (一个学生选修 2 门课),5000 万学生，1000 万老师，1000 门课。然后分别执行上述语句。最后我会在 oracle 数据库上执行上述语句。

测试结果
----

![](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkiaX4xczdhtpjosWQ2K3eJkIEcBc5osMtWXr4K2x3T8V35NneF8feuoicX9v8IzVEMyyY9ib3y6BkgAA/640?wx_fmt=jpeg)

  

![](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkiaX4xczdhtpjosWQ2K3eJkIiaGgxp5EI9Ub3qgD2jyWRGQryK8ibETun7uOJel9LYDW9hCXBLDCHOog/640?wx_fmt=jpeg)

敲黑板划重点
------

仔细看上表，可以发现：

1.  步骤 3.1 没有在连接键上加索引，查询很慢，说明：“多表关联查询时，保证被关联的字段需要有索引”；
    
2.  步骤 6.1,6.2,6.3，换成简单 sql，在数据量 1 亿以上， 查询时间还能勉强接受。此时说明 mysql 查询有些吃力了，但是仍然能查询出来。
    
3.  步骤 5.1，mysql 查询不出来，4 表连接，对我本机 mysql 来说，1.5 亿数据超过极限了（我调优过这个 SQL，执行计划和索引都走了，没有问题，show profile 显示在 sending data. 这个问题另外文章详谈。）
    
4.  对比 1.1 和 5.1 步骤 sql 查询，4 表连接，对我本机 mysql 来说，1.5 千万数据查询很流利，是一个 mysql 数据量流利分水岭。(这个只是现象，不太准确，需要同时计算表的容量)。
    
5.  步骤 5.1 对比 6.1,6.2,6.3，多表 join 对 mysql 来说，处理有些吃力。
    
6.  超过三张表禁止 join, 这个规则是针对 mysql 来说的。后续会看到我用同样机器，同样数据量，同样内存，可以完美计算  1.5 亿数据量 join。针对这样一个规则，对开发来说 ，需要把一些逻辑放到应用层去查询。
    

**总结：这个规则超过三张表禁止 join, 由于数据量太大的时候，mysql 根本查询不出来，导致阿里出了这样一个规定。(其实如果表数据量少，10 张表也不成问题, 你自己可以试试) 而我们公司支付系统朝着大规模高并发目标设计的，所以，遵循这个规定。在业务层面来讲，写简单 sql，把更多逻辑放到应用层，我的需求我会更了解，在应用层实现特定的 join 也容易得多。**

**肥朝小声逼逼：由 5.1 我们就知道，假如你偏要用，很可能等到天荒地老，海枯石烂，都等不到结果！**

让我们来看看 oracle 数据库的优秀表现
----------------------

![](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkiaX4xczdhtpjosWQ2K3eJkInNUOPicYw2agZRRtBs6pzrekJsyPqTmsIYibuqpCDS1PabOj3BPDYnTg/640?wx_fmt=jpeg)

看步骤 7.1，就是没有索引，join 表很多的情况下，oracle 仍然 26 秒查询出结果来。所以我会说 mysql 的 join 很弱。

看完本篇文章，另外我还附加赠送，所谓搂草打兔子。就是快速造数据。你可以自己先写脚本造数据，看看我是怎么造数据的，就知道我的技巧了。

附上部分截图
------

![](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkiaX4xczdhtpjosWQ2K3eJkIw9ujf3Kr2G2ZrScEDc0WF5C5T9ba81DhqAhxOHzYHvjhq9tdu4t8cA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkiaX4xczdhtpjosWQ2K3eJkImXNb7yxQY6smgdfL76VdNDq9Mc36JYNR8mM8yBRJE0YUiahhoXAOsVw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkiaX4xczdhtpjosWQ2K3eJkIs9BwibBicSaQwmhhbtkEsG6fkXGomerXKWHyuYIjLSzyXltZuRL7ebibw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/N34tfh8WYkiaX4xczdhtpjosWQ2K3eJkIZtiavlJNOsX9wrLFLe0xQGqDC2iadcG7bWytEvX8UK8YNFzrlN3va6Lg/640?wx_fmt=jpeg)

附上 sql 语句和造数据脚本

```
use stu;
drop table if exists student;
create table student 
  (  s_id int(11) not null auto_increment ,
     sno    int(11), 
     sname varchar(50), 
     sage  int(11), 
     ssex  varchar(8) ,
     father_id int(11),
      mather_id int(11),
      note varchar(500),
     primary key (s_id),
   unique key uk_sno (sno)
  ) engine=innodb default charset=utf8mb4;
truncate table student;
  delimiter $$
drop function if exists   insert_student_data $$
create function insert_student_data()
 returns  int deterministic
    begin
    declare  i int;
      set i=1;
      while  i<50000000 do 
      insert into student  values(i ,i, concat('name',i),i,case when floor(rand()*10)%2=0 then 'f' else 'm' end,floor(rand()*100000),floor(rand()*1000000),concat('note',i) );
      set i=i+1;
      end while;
      return 1;
    end$$
delimiter ;    
select  insert_student_data();
select count(*) from student;
use stu;
create table course 
  ( 
     c_id int(11) not null auto_increment ,
     cname varchar(50)
     note varchar(500), primary key (c_id)
  )  engine=innodb default charset=utf8mb4;
truncate table course;
  delimiter $$
drop function if exists   insert_course_data $$
create function insert_course_data()
 returns  int deterministic
    begin
    declare  i int;
      set i=1;
      while  i<=1000 do 
      insert into course  values(i , concat('course',i),floor(rand()*1000),concat('note',i) );
      set i=i+1;
      end while;
      return 1;
    end$$
delimiter ;    
select  insert_course_data();
select count(*) from course;
use stu;
drop table if exists sc;
create table sc 
  ( 
     s_id    int(11), 
     c_id    int(11), 
     t_id    int(11),
     score int(11) 
  )  engine=innodb default charset=utf8mb4;
truncate table sc;
  delimiter $$
drop function if exists   insert_sc_data $$
create function insert_sc_data()
 returns  int deterministic
    begin
    declare  i int;
      set i=1;
      while  i<=50000000 do 
      insert into sc  values( i,floor(rand()*1000),floor(rand()*10000000),floor(rand()*750)) ;
      set i=i+1;
      end while;
      return 1;
    end$$
delimiter ;    
select  insert_sc_data();
commit;
select  insert_sc_data();
commit;
create index idx_s_id  on sc(s_id)   ; 
create index idx_t_id  on sc(t_id)   ; 
create index idx_c_id  on sc(c_id)   ; 
select count(*) from sc;
use stu;
drop table if exists teacher;
create table teacher 
  ( 
    t_id  int(11) not null auto_increment ,
     tname varchar(50) ,
     note varchar(500),primary key (t_id)
  )  engine=innodb default charset=utf8mb4;

  truncate table teacher;
  delimiter $$
drop function if exists   insert_teacher_data $$
create function insert_teacher_data()
 returns  int deterministic
    begin
    declare  i int;
      set i=1;
      while  i<=10000000 do 
      insert into teacher  values(i , concat('tname',i),concat('note',i) );
      set i=i+1;
      end while;
      return 1;
    end$$
delimiter ;    
select  insert_teacher_data();
commit;
select count(*) from teacher;
这个是oracle的测试和造数据脚本
create tablespace scott_data  datafile  '/home/oracle/oracle_space/sitpay1/scott_data.dbf'  size 1024m autoextend on; 
create tablespace scott_index   datafile  '/home/oracle/oracle_space/sitpay1/scott_index.dbf'  size 64m  autoextend on; 
create temporary tablespace scott_temp  tempfile  '/home/oracle/oracle_space/sitpay1/scott_temp.dbf'  size 64m autoextend on; 
drop user  scott cascade;
create user  scott  identified by  tiger  default tablespace scott_data  temporary tablespace scott_temp  ;
grant resource,connect,dba to  scott;
drop table student;
create table student  
  (  s_id number(11) ,
     sno    number(11) , 
     sname varchar2(50), 
     sage  number(11), 
     ssex  varchar2(8) ,
     father_id number(11),
      mather_id number(11),
      note varchar2(500)
  ) nologging;
truncate table student;
create or replace procedure insert_student_data
 is 
   q number(11);
    begin 
     q:=0;
      for i in  1..50 loop 
      insert /*+append*/ into student   select rownum+q as s_id,rownum+q  as sno, concat('sutdent',rownum+q ) as sname,floor(dbms_random.value(1,100)) as sage,'f' as ssex,rownum+q  as father_id,rownum+q  as mather_id,concat('note',rownum+q ) as note from dual connect by level<=1000000;
      q:=q+1000000;
      commit;
      end loop; 
end insert_student_data;
/
call insert_student_data();
alter table student  add constraint  pk_student primary key (s_id);
commit;    
select count(*) from student;
create table course 
  ( 
     c_id number(11) primary key,
     cname varchar2(50),
     note varchar2(500) 
  )  ;
truncate table course;
 create or replace procedure insert_course_data
 is 
   q number(11);
    begin 

      for i in  1..1000 loop 
      insert /*+append*/ into course  values(i , concat('name',i),concat('note',i) );      
      end loop; 
end insert_course_data;
/
call insert_course_data();
commit;    
select count(*) from course;
create table sc 
  ( 
     s_id    number(11), 
     c_id    number(11), 
     t_id    number(11),
     score number(11) 
  ) nologging;
truncate table sc;
 create or replace procedure insert_sc_data
 is 
   q number(11);
    begin 
     q:=0;
      for i in  1..50 loop 
      insert /*+append*/ into sc   select rownum+q as s_id, floor(dbms_random.value(0,1000))  as c_id,floor(dbms_random.value(0,10000000)) t_id,floor(dbms_random.value(0,750)) as score from dual connect by level<=1000000;
      q:=q+1000000;
      commit;
      end loop; 
end insert_sc_data;
/
call insert_sc_data();
create index idx_s_id  on sc(s_id)   ; 
create index idx_t_id  on sc(t_id)   ; 
create index idx_c_id  on sc(c_id)   ; 
select count(*) from sc;
create table teacher 
  ( 
    t_id  number(11) ,
     tname varchar2(50) ,
     note varchar2(500)
  )nologging ;
    truncate table teacher;
create or replace procedure insert_teacher_data
 is 
   q number(11);
    begin 
     q:=0;
      for i in  1..10 loop 
      insert /*+append*/ into teacher   select rownum+q as t_id, concat('teacher',rownum+q ) as tname,concat('note',rownum+q ) as note from dual connect by level<=1000000;
      q:=q+1000000;
      commit;
      end loop; 
end insert_teacher_data;
/
call insert_teacher_data();
alter table teacher  add constraint  pk_teacher primary key (t_id);
select count(*) from teacher;


```