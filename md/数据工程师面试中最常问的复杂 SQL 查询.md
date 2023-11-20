> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIwOTU0ODQxNA==&mid=2247484239&idx=1&sn=4413ad2b5dd1c3d1604a3585dc758e3b&chksm=9773665ea004ef48587d261634285abcb9b01492f2771b0f91f17b247a329d2a623f7a8712e8&scene=21#wechat_redirect)

一、示例数据表
-------

#### 1.1 假设我们有一张示例数据表 “**employees**”，包含以下列：

*   **id**：每个员工的唯一标识符
    
*   **name**：员工姓名
    
*   **gender**：员工性别
    
*   **salary**：员工工资
    
*   **department**：员工所在部门
    

#### 1.2 下面是创建`employees`表并插入样本数据的 MySQL 脚本：

```
CREATE TABLE employees
(
    id         INT AUTO_INCREMENT PRIMARY KEY,
    name       VARCHAR(50) NOT NULL,
    gender     VARCHAR(10) NOT NULL,
    salary     INT         NOT NULL,
    department VARCHAR(50) NOT NULL
);
INSERT INTO employees (name, gender, salary, department)
VALUES ('Ramesh Gupta', 'Male', 55000, 'Sales'),
       ('Priya Sharma', 'Female', 65000, 'Marketing'),
       ('Sanjay Singh', 'Male', 75000, 'Sales'),
       ('Anjali Verma', 'Female', 45000, 'Finance'),
       ('Rajesh Sharma', 'Male', 80000, 'Marketing'),
       ('Smita Patel', 'Female', 60000, 'HR'),
       ('Vikram Yadav', 'Male', 90000, 'Sales'),
       ('Neha Sharma', 'Female', 55000, 'Marketing'),
       ('Rahul Singh', 'Male', 70000, 'Finance'),
       ('Sonali Gupta', 'Female', 50000, 'Sales');

```

```
Employees 表

```

二、查询
----

#### 2.1 查询每个部门中男女职工的平均薪水

#### 2.2 查询每个部门中薪水最高的员工姓名及薪水

```
输出：

```

#### 2.3 查询每个部门中比所在部门平均薪水高的员工信息

**实现方案**：

```
SELECT name, salary, department
FROM employees
WHERE salary > (SELECT AVG(salary)
                FROM employees AS e2
                WHERE e2.department = employees.department);

```

```
输出：

```

![](https://mmbiz.qpic.cn/mmbiz_png/LDBRqwNJEPiaayPT5x34zsKSPZMTHQ6l6eA9VhwbmibyPKw0G0mQ2sdVAM84ibLhI8O8ficoYxZGQyib7flgj4cZicicA/640?wx_fmt=png)

#### 2.4 查询每个部门中薪水前三名的员工信息

**实现方案**：

```
SELECT e.department, e.name, e.salary
FROM employees e
WHERE (SELECT COUNT(*)
       FROM employees
       WHERE department = e.department
         AND salary > e.salary) < 3;

```

```
输出：

```

![](https://mmbiz.qpic.cn/mmbiz_png/LDBRqwNJEPiaayPT5x34zsKSPZMTHQ6l6iaJUxNxibbyc196WHBic02KcuPlToxt0P3oXh1B0L6G7C839WwEPsd9Ew/640?wx_fmt=png)

#### 2.5 查询每个部门中比所在部门平均薪水高的员工姓名（该问题与第三个问题一样，只不过这里使用连接实现）

**实现方案**：

```
SELECT e.name
FROM employees e
         JOIN (SELECT department, AVG(salary) AS avg_salary
               FROM employees
               GROUP BY department) AS dept_avg
              ON e.department = dept_avg.department
WHERE e.salary > dept_avg.avg_salary;

```

```
输出：

```

![](https://mmbiz.qpic.cn/mmbiz_png/LDBRqwNJEPiaayPT5x34zsKSPZMTHQ6l6L22V0Hq4e0d9phRciaelEIedIsdJBdgPvqebibHNFkLSD2tTIoWIQyVQ/640?wx_fmt=png)

#### 2.6 查询每个部门中最高薪水的员工信息

**实现方案**：

```
WITH max_salary AS (SELECT department, MAX(salary) AS highest_salary
                    FROM employees
                    GROUP BY department)
SELECT m.department, e.name, e.salary
FROM employees e
         JOIN max_salary m
              ON e.department = m.department AND e.salary = m.highest_salary;

```

**输出**：

![](https://mmbiz.qpic.cn/mmbiz_png/LDBRqwNJEPiaayPT5x34zsKSPZMTHQ6l6Ma3dgYfOJhHv3BuNzUu6ZSXK8LMxDrf0dLhKnG9us5x8E04wRhrP0A/640?wx_fmt=png)

该实现需要 MySQL 8.0 及以上版本才可运行，因为`WITH...AS...`子句只有 8.0 及以上版本才支持。