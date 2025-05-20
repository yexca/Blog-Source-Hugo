---
slug: 8
title: 'SQL 语句的一些语法细节 (SQL SERVER 语句)'
date: '2021-11-08T11:51:18+08:00'
author: hiyoung
# layout: post
# permalink: /archives/8
views:
    - '183'
categories:
    - 技术学习
tags:
    - 数据库
---

> 该文章由 [Hiyoung](https://blog.hiyoung.icu/) 编写

## 1.SQL ORDER BY 关键字

ORDER BY 关键字用于对结果集按照一个列或者多个列进行排序。

ORDER BY 关键字默认按照升序对记录进行排序。如果需要按照降序对记录进行排序，您可以使用 DESC 关键字

### SQL ORDER BY 语法

```sql
SELECT *column\_name*,*column\_name*  
FROM *table\_name*  
ORDER BY *column\_name*1,*column\_name*2 ASC|DESC;
```

–ASC 表示升序，DESC 表示降序

–使用 order by 语句时应放在所有语句的最后使用，并且排序多个列时先排 `column\_name1` 再 `column\_name2…`

## 2.删除所有数据（delete 和 drop table）

您可以在不删除表的情况下，删除表中所有的行。这意味着表结构、属性、索引将保持不变：`DELETE FROM table\_name`;

或

`DELETE * FROM table\_name;`

**注释：**在删除记录时要格外小心！因为您不能重来！

### DROP TABLE 语句

DROP TABLE 语句用于删除表。`DROP TABLE table\_name`

**注释：**与 delete 不同的是 drop table 会删除表数据和结果，也是不可逆的！

- - - - - -

### DROP DATABASE 语句

DROP DATABASE 语句用于删除数据库。`DROP DATABASE database\_name`

- - - - - -

### TRUNCATE TABLE 语句

如果我们仅仅需要删除表内的数据，但并不删除表本身，那么我们该如何做呢？

请使用 TRUNCATE TABLE 语句：`TRUNCATE TABLE table\_name`

## 3.SQL JOIN

SQL join 用于把来自两个或多个表的行结合起来。

下图展示了 LEFT JOIN、RIGHT JOIN、INNER JOIN、OUTER JOIN 相关的 7 种用法。

- **INNER JOIN：如果表中有至少一个匹配，则返回行 (INNER JOIN 与 JOIN 是相同的)**
- **LEFT JOIN：即使右表中没有匹配，也从左表返回所有的行**
- **RIGHT JOIN：即使左表中没有匹配，也从右表返回所有的行**
- **FULL JOIN：只要其中一个表中存在匹配，则返回行**

![sql-join](https://www.runoob.com/wp-content/uploads/2019/01/sql-join.png)

注释：SQL 中的 join 语句其实对应数据库理论中的连接概念，left join、right join 和 inner join 对应自然连接，full join 对应笛卡尔积

## 4.SQL 约束 (Constraints)

```sql
CREATE TABLE table_name
(
column_name1 data_type(size) constraint_name,
column_name2 data_type(size) constraint_name,
column_name3 data_type(size) constraint_name,
....
);
```

- **NOT NULL** – 指示某列不能存储 NULL 值。
- **UNIQUE** – 保证某列的每行必须有唯一的值。（一个表可以有多个UNIQUE约束但只能有一个 primary key，primary key 自动包含 unique 约束）
- **PRIMARY KEY** – NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。（主键）
- **FOREIGN KEY** – 保证一个表中的数据匹配另一个表中的值的参照完整性。（外键）
- **CHECK** – 保证列中的值符合指定的条件。
- **DEFAULT** – 规定没有给列赋值时的默认值。

## 5.AUTO INCREMENT 字段

我们通常希望在每次插入新记录时，自动地创建主键字段的值。

我们可以在表中创建一个 auto-increment 字段。

下面的 SQL 语句把 “Persons” 表中的 “ID” 列定义为 auto-increment 主键字段：CREATE TABLE Persons

```sql
(
ID int IDENTITY(1,1) PRIMARY KEY,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

在上面的实例中，IDENTITY 的开始值是 1，每条新记录递增 1。

**提示：**要规定 “ID” 列以 10 起始且递增 5，请把 identity 改为 IDENTITY(10,5)。

要在 “Persons” 表中插入新记录，我们不必为 “ID” 列规定值（会自动添加一个唯一的值）：

```sql
INSERT INTO Persons (FirstName,LastName)
VALUES ('Lars','Monsen')
```

上面的 SQL 语句会在 “Persons” 表中插入一条新记录。”ID” 列会被赋予一个唯一的值。”FirstName” 列会被设置为 “Lars”，”LastName” 列会被设置为 “Monsen”。

## 6.触发器

参见：[SqlServer 基础之(触发器) – wangchuang2017 – 博客园](https://www.cnblogs.com/wangprince2017/p/7827091.html#:~:text=%E8%A7%A6%E5%8F%91%E5%99%A8%EF%BC%88trigger%EF%BC%89%E6%98%AFSQL%20server%20%E6%8F%90%E4%BE%9B%E7%BB%99%E7%A8%8B%E5%BA%8F%E5%91%98%E5%92%8C%E6%95%B0%E6%8D%AE%E5%88%86%E6%9E%90%E5%91%98%E6%9D%A5%E4%BF%9D%E8%AF%81%E6%95%B0%E6%8D%AE%E5%AE%8C%E6%95%B4%E6%80%A7%E7%9A%84%E4%B8%80%E7%A7%8D%E6%96%B9%E6%B3%95%EF%BC%8C%E5%AE%83%E6%98%AF%E4%B8%8E%E8%A1%A8%E4%BA%8B%E4%BB%B6%E7%9B%B8%E5%85%B3%E7%9A%84%E7%89%B9%E6%AE%8A%E7%9A%84%E5%AD%98%E5%82%A8%E8%BF%87%E7%A8%8B%EF%BC%8C%E5%AE%83%E7%9A%84%E6%89%A7%E8%A1%8C%E4%B8%8D%E6%98%AF%E7%94%B1%E7%A8%8B%E5%BA%8F%E8%B0%83%E7%94%A8%EF%BC%8C%E4%B9%9F%E4%B8%8D%E6%98%AF%E6%89%8B%E5%B7%A5%E5%90%AF%E5%8A%A8%EF%BC%8C%E8%80%8C%E6%98%AF%E7%94%B1%E4%BA%8B%E4%BB%B6%E6%9D%A5%E8%A7%A6%E5%8F%91%EF%BC%8C%E5%BD%93%E5%AF%B9%E4%B8%80%E4%B8%AA%E8%A1%A8%E8%BF%9B%E8%A1%8C%E6%93%8D%E4%BD%9C%EF%BC%88,insert%EF%BC%8Cdelete%EF%BC%8C%20update%EF%BC%89%E6%97%B6%E5%B0%B1%E4%BC%9A%E6%BF%80%E6%B4%BB%E5%AE%83%E6%89%A7%E8%A1%8C%E3%80%82%20%E8%A7%A6%E5%8F%91%E5%99%A8%E7%BB%8F%E5%B8%B8%E7%94%A8%E4%BA%8E%E5%8A%A0%E5%BC%BA%E6%95%B0%E6%8D%AE%E7%9A%84%E5%AE%8C%E6%95%B4%E6%80%A7%E7%BA%A6%E6%9D%9F%E5%92%8C%E4%B8%9A%E5%8A%A1%E8%A7%84%E5%88%99%E7%AD%89%E3%80%82)
