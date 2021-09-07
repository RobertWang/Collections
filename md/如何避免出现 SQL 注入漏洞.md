> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/mP49OCkwqSnamtop0cChdQ)

**一  前言**
---------

本文将针对开发过程中依旧经常出现的 SQL 编码缺陷，讲解其背后原理及形成原因。并以几个常见漏洞存在形式，提醒技术同学注意相关问题。最后会根据原理，提供解决或缓解方案。

**二  SQL 注入漏洞的原理、形成原因**
-----------------------

  
SQL 注入漏洞，根本上讲，是由于错把外部输入当作 SQL 代码去执行。目前最佳的解决方案就是预编译的方式。  
SQL 语句在执行过程中，需要经过以下三大基本步骤：

1.  代码语义分析
    
2.  制定执行计划
    
3.  获得返回结果
    

  
而一个 SQL 语句是由代码和数据两部分，如：

SELECT id, name, phone FROM userTable WHERE name = 'xiaoming';    
SELECT id, name, phone FROM userTable WHERE name = 是代码，'xiaoming'是数据。  
而预编译，以 Mybatis 为例，就是预先分析带有占位符的语义：  
如 SELECT id, name, phone FROM userTable WHERE id = #{name};    
然后再将数据'xiaoming'，传入到占位符。这样一来，错开来代码语义分析阶段，也就不会被误认为是代码的一部分了。  
在最早期，开发者显式使用 JDBC 来自己创建 Connection，执行 SQL 语句。这种情况下，如果将外部可控数据拼接到 SQL 语句，且没有做充分过滤的话，就会产生漏洞。这种情况在正常的业务开发过程中已经很少了，按照公司规定，无特殊情况下，必须使用 ORM 框架来执行 SQL。  
但目前部分项目中，仍会使用 JDBC 来编写一些工具脚本，如 DataMerge.java 、DatabaseClean.java，借用 JDBC 的灵活性，通过这些脚本来执行数据库批量操作。  
此类代码不应该出现在线上版本中，以免因各种情况，被外部调用。  

**三  直接使用 Mybatis**
-------------------

### 1  易错点

目前大部分的平台代码是基于 Mybatis 来处理持久层和数据库之间的交互的，Mybatis 传入数据有两种占位符 {} 和#{}。{} 和 #{}。{} 可以理解为语义分析前的字符串拼接，讲传入的参数，原封不动地传入。  

比如说

  
SELECT id, name, phone FROM userTable WHERE name = '${name}';    
传入 name=xiaoming 后，相当于  
SELECT id, name, phone FROM userTable WHERE name = 'xiaoming';    
实际应用中

  
SELECT id, name, phone FROM userTable WHERE ${col} = 'xiaoming';    
传入 col = "name"，相当于  
SELECT id, name, phone FROM userTable WHERE name = 'xiaoming';    
就像预编译原理介绍里讲的一样，使用 #{} 占位符就不存在注入问题了。但有些业务场景是不可以直接使用 #{} 的。  
比如 order by 语法中  
如果编写 SELECT id, name, phone FROM userTable ORDER BY #{}; ，执行时是会报错的。因为 order by 后的内容，是一个列名，属于代码语义的一部分。如果在语义分析部分没有确定下来，就相当于执行 SELECT id, name, phone FROM userTable ORDER BY 。肯定会有语法错误。

再比如 like 场景下

  
SELECT id, name, phone FROM userTable WHERE name like '%#{name}%';    
#{} 不会被解析，从而导致报错。  
in 语法和 between 语法都是如此，那么如何解决这类问题呢？  

### 2  正确写法

#### **order by（group by）语句中使用 ${}**

1.  使用条件判断
    
      
    

```
<select resultType="Emp" parameterType="Emp">
    select * from users where id < #{id}
    <choose>
        <when test="order == \"name\"">
            order by name
        </when>
        <when test="order != \"age\"">
            order by age
        </when>
        <otherwise>
            order by id
        </otherwise>
    </choose>
</select>
```

     2. 使用全局过滤机制，限制 order by 后的变量内容只能是数字、字母、下划线。

  
如使用正则过滤：

```
keyword = keyword.replaceAll("[^a-zA-Z0-9_\s+]", "");
```

这里需要注意，过滤需要使用白名单，不能使用黑名单，黑名单无法解决注入问题。  

#### **LIKE 语句**

由于需要 like 中的关键词需要包裹在两个 % 符号中，因此可以使用 CONCAT 函数进行拼接。

```
<select resultMap="studentMap">
    SELECT *
    FROM student
    WHERE student.stu_name
            LIKE CONCAT('%',#{stuName},'%')
</select>
```

注意不要用 CONCAT('%','${stuName}','%') ，这样仍然存在漏洞。也就是说，使用 $ 符号是不对的，使用 #符号才安全。

#### **IN 语句**

类似于 like 语句，直接使用 #{} 会报错，常见的错误写法为：

```
tenant_id in (${tenantIds})
```

正确的写法为：

```
select * from news where id in
<foreach collection="ids" item="item" open="("separator="," close=")">#{item}</foreach>
```

**四  Mybatis-generator 使用安全**
-----------------------------

  
繁重的 CRUD 代码压力下，开发者慢慢开始通过 Mybatis-generator、idea-mybatis-generator 插件、通用 Mapper、Mybatis-generator-plus 来自动生成 Mapper、POJO、Dao 等文件。  
这些工具可以自动的生成 CRUD 所需要的文件，但如果使用不当，就会自动产生 SQL 注入漏洞。我们以最常用的 org.mybatis.generator 为例，来讲解可能会出现的问题。  

### 1  动态语句支持

Mybatis-generator 提供来一些函数，帮助用户把 SQL 的各个条件连接起来，比如多个参数的 like 语法，多个参数的比较语法。为了保证使用的简洁性，需要使用将一些语义代码拼接到 SQL 语句中。而如果开发者使用不当，将外部输入也传入了 {} 占位符。就会产生漏洞。  

### 2  targetRuntime 参数配置

  
在配置 generator 时，配置文件 generator-rds.xml 中有一个 targetRuntime 属性，默认为 MyBatis3。在这种情况下，会启动 Mybatis 的动态语句支持，启动 enableSelectByExample、enableDeleteByExample、enableCountByExample 以及 enableUpdateByExample 功能。

以 enableSelectByExample 为例，会在 xml 映射文件中代入以下动态模块：

```
<sql >
    <where >
      <foreach collection="oredCriteria" item="criteria" separator="or" >
        <if test="criteria.valid" >
          <trim prefix="(" suffix=")" prefixOverrides="and" >
            <foreach collection="criteria.criteria" item="criterion" >
              <choose >
                <when test="criterion.noValue" >
                  and ${criterion.condition}
                </when>
                <when test="criterion.singleValue" >
                  and ${criterion.condition} #{criterion.value}
                </when>
                <when test="criterion.betweenValue" >
                  and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
                </when>
                <when test="criterion.listValue" >
                  and ${criterion.condition}
                  <foreach collection="criterion.value" item="listItem" open="(" close=")" separator="," >
                    #{listItem}
                  </foreach>
                </when>
              </choose>
            </foreach>
          </trim>
        </if>
      </foreach>
    </where>
  </sql>
```

开发者 include 该模块就可以添加 where 条件，但如果使用不当，就会导致 SQL 注入漏洞：

```
<select resultMap="BaseResultMap" parameterType="com.doctor.mybatisdemo.domain.userExample" >
    select
    <if test="distinct" >
      distinct
    </if>
    <include refid="Base_Column_List" />
    from user
    <if test="_parameter != null" >
      <include refid="Example_Where_Clause" />
    </if>
    <if test="orderByClause != null" >
      order by ${orderByClause}
    </if>
  </select>
```

并使用自定义的参数添加函数：

```
public Criteria addKeywordTo(String keyword) {
  StringBuilder sb = new StringBuilder();
  sb.append("(display_name like '%" + keyword + "%' or ");
  sb.append("org like '" + keyword + "%' or ");
  sb.append("status like '%" + keyword + "%' or ");
  sb.append("id like '" + keyword + "%') ");
  addCriterion(sb.toString());
  return (Criteria) this;
}
```

目的是为了实现同时对 display_name、org、status、id 的 like 操作。其中 addCriterion 是 Mybatis-generator 自带的函数：

```
protected void addCriterion(String condition) {
    if (condition == null) {
        throw new RuntimeException("Value for condition cannot be null");
    }
    criteria.add(new Criterion(condition));
}
```

这里的误区在于，addCriterion 本身提供了多个条件的支持，但开发者认为需要自己把多个条件拼接起来，一同传入 addCriterion 方法。如同案例中的代码一样，最终传入 addCriterion 的只有一个参数。从而执行 Example_Where_Clause 语句中的：

```
<when test="criterion.noValue" >
    and ${criterion.condition}
</when>
```

也就是说，开发者把自己拼接的 SQL 语句，直接代入了 ${criterion.condition} 中，从而导致了漏洞的产生。

而按照 Mybatis-generator 的文档，正确的写法应该是：

```
public void addKeywordTo(String keyword, UserExample userExample) {
  userExample.or().andDisplayNameLike("%" + keyword + "%");
  userExample.or().andOrgLike(keyword + "%");
  userExample.or().andStatusLike("%" + keyword + "%");
  userExample.or().andIdLike("%" + keyword + "%");
}
```

or 方法负责创建 Criteria，这时触发的逻辑就是

```
<when test="criterion.singleValue" >
  and ${criterion.condition} #{criterion.value}
</when>
```

${criterion.condition} 被替换为了没有单引号的 like，like 作为语义代码，在语义分析前拼接到了 SQL 语句中，而 "%" + keyword + "%" 会作为数据添加到预编译 #{criterion.value} 中去，从而避免了注入。

类似的，也提供了 In 语法的安全使用方法：

```
List<Integer> field5Values = new ArrayList<Integer>();
  field5Values.add(8);
  field5Values.add(11);
  field5Values.add(14);
  field5Values.add(22);
  example.or()
    .andField5In(field5Values);
```

Beetween 的安全使用方法：

```
example.or()
    .andField6Between(3, 7);
```

Mybatis-generator 默认生成的 order by 语句也是使用 ${} 直接进行拼接的：

```
<if test="orderByClause != null" >
      order by ${orderByClause}
    </if>
```

如果没有对传入的参数进行额外的过滤的话，就会导致注入问题。

### 3  order by

除了自己写的 SQL 语句以外，Mybatis-generator 默认生成的 order by 语句也是使用 ${} 直接进行拼接的：

```
<if test="orderByClause != null" >
      order by ${orderByClause}
    </if>
```

如果没有对传入的参数进行额外的过滤的话，就会导致注入问题。  
PS: 实际扫雷过程中发现很多语句自动生成了 order by 语法，但上层调用时，并没有传入该可选参数。这种情况应当删除多余的 order by 语法。  

### 4  其它插件

插件与插件之间的安全缺陷还不太一样，下面简单列举了常用的几种插件。  

#### **idea-mybatis-generator**

这是 IDEA 的插件，可以在开发过程中，从 IDE 的层面，自动生成 CRUD 中需要的文件。使用该插件时，也有一些默认安全隐患需要注意。  

##### 1）自定义 order by 处理

like\in\between 可以参照官方文档使用，无安全隐患。

  
但该插件没有内置的 order by 处理，需要自行编写，编写时，参考 Case2  

##### 2）默认的 IF 条件前需要判断是否为空

  
插件默认生成的语法大致如下：

  
<if test="ID != null">  
ID = #{ID} and  
当 ID 参数为 null 时，if 标签下的逻辑不会添加到 SQL 语句中，可能会导致 DOS、权限绕过等漏洞。因此，参数传入查询语句前，需要确认不为空。 

#### **com.baomidou.mybatis-plus**

1. apply 方法传参时，应当使用 {}

2. 自带的 last 方法，其原理是直接拼接到 SQL 语句的末尾，存在注入漏洞。  

**五  其它 ORM 框架**
----------------

### 1  Hibernate

  
ORM 全称为对象关系映射 (Object Relational Mapping)，简单地说，就是将数据库中的表映射为 Java 对象, 这种只有属性，没有业务逻辑的对象也叫做 POJO（Plain Ordinary Java Object）对象。  
Hibernate 是第一个被广泛使用的 ORM 框架，它通过 XML 管理数据库连接，提供全表映射模型，封装程度很高。在配置映射文件和数据库链接文件后，Hibernate 就可以通过 Session 对象进行数据库操作，开发者无需接触 SQL 语句，只需要写 HQL 语句即可。  
Hibernate 经常与 Struts、Spring 搭配使用，也就是 Java 世界的经典 SSH 框架。  
HQL 相较于 SQL，多了很多语法限制：  

1. 不能查询未做映射的表，只有当模型之间的关系明确后，才可以使用 UNION 语法。

2. 表名，列名大小写敏感。

3. 没有 *、#、-- 。

4. 没有延时函数。  

  
所以 HQL 注入利用要比 SQL 注入苦难得多。从代码审计的角度和普通 SQL 注入是一致的：  
拼接会导致注入漏洞：

```
List<Student> studentList = session.createQuery("FROM Student s WHERE s.stuId = " + stuId).list();
```

可以使用占位符和具名参数来防止 SQL 语句，其本质都是预编译。

```
List<Student> studentList = session.createQuery("FROM Student s WHERE s.stuId = :stuId").setParameter("stuId",stuId).list();
```

```
List<Student> studentList = session.createQuery("FROM Student s WHERE s.stuId = ?").setParameter(stuId).list();
```

Hibernate 在使用过程中有很多不足：  

1. 全表映射不灵活，更新时需要发送所有字段，影响程序运行效率。

2. 对复杂查询的支持很差。

3. 对存储过程的支持很差。

4. HQL 性能较差，无法根据 SQL 进行优化。  

  
在审计 Hibernate 相关注入时，可以通过全局搜索 createQuery 来快速定位 SQL 操作的位置。  

### 2  JPA

  
JPA 全称为 Java Persistence API，是 Java EE 提供的一种数据持久化的规范，允许开发者通过 XML 或注解的方式，将某个对象，持久化到数据库中。  
主要包括三方面内容：  

1. ORM 映射元数据，通过 XML 或注解，描述对象和数据表之间的对应关系。框架便可以自动将对象中的数据保存到数据库中。

  
常见的注解有：@Entity、@Table、@Column、@Transient  

2. 数据操作 API，内置接口，方便对某个数据表执行 CRUD 操作，节省开发者编写 SQL 的时间。

  
常见的方法有：entityManager.merge(T t);  

3. JPQL, 提供一种面向对象而不是面向数据库的查询语言，将程序和数据库、SQL 解耦合。

  
JPA 是一套规范，Hibernate 实现了这一 JPA 规范。  

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naL7zZhFK89LlYkJ4W20egLwMr1oiax0Qo2D4BrlKqzPzQVoUVMgdGWQpbFHwSmyH1Ds7zXDFzjswiag/640?wx_fmt=jpeg)

在 Spring 框架中，提供了简易版的 JPA 实现——spirng data jpa。按照约定好的方法命名规则写 dao 层接口，就可以在不写接口实现的情况下，实现对数据库的访问和操作。同时提供了很多除了 CRUD 之外的功能，如分页、排序、复杂查询等等。使用起来更简单，但底层仍然在使用 Hibernate 的 JPA 实现。  
  
和 HQL 注入一样，如果使用拼接的方式，将用户可控的数据代入了查询语句中，就会导致 SQL 注入。  
  
安全的查询应该使用预编译技术。  
  
Spring Data JPA 的预编译写法为：

```
String getUser = "SELECT username FROM users WHERE id = ?";
Query query = em.createNativeQuery(getUser);
query.setParameter(1, id);
String username = query.getResultList();
```

小贴士：其实 Hibernate 的出现日期比 JPA 规范要早，Hibernate 逐渐成熟之后，JavaEE 的开发团队，邀请 Hibernate 核心开发人员一起制定了 JPA 规范。之后 Spring Data JPA 按照规范做了进一步优化。除此之外，JPA 规范的实现有很多产品，比如 Eclipse 的 TopLink（OracleLink）。  

**六  总结**
---------

经过上面的介绍，尤其是围绕 Mybatis 易错点的讨论，我们可以得到以下结论：

1. 持久层组件种类繁多。

2. 开发者对工具使用的错误理解，是漏洞出现的主要原因。

3. 由于自动生成插件的动态特性，自动化发现 SQL 漏洞不能简单地使用 ${} 来寻找。必须要根据全局的持久层组件特性，来做详细的匹配规则。  

参考链接：
-----

https://www.anquanke.com/post/id/190170#h2-3  
https://www.cnblogs.com/alka1d/p/11582993.html

‍‍‍‍‍‍‍‍

**PostgreSQL 实战进阶**
-------------------

PostgreSQL 被誉为 “世界上功能最强大的开源数据库”，是以加州大学伯克利分校计算机系开发的 POSTGRES 4.2 为基础的对象关系型数据库管理系统。

PostgreSQL 支持大部分 SQL 标准并且提供了许多其他现代特性：复杂查询、外键、触发器、视图、事务完整性、MVCC。同样，PostgreSQL 可以用许多方法扩展，比如，通过增加新的数据类型、函数、操作符、聚集函数、索引。开发者可以免费使用、修改、和分发 PostgreSQL，不管是私用、商用、还是学术研究使用。 

本课程由 PostgreSQL 社区核心成员出品，带你快速从 0-1 深入 PostgreSQL 核心特性。

点击阅读原文查看课程～