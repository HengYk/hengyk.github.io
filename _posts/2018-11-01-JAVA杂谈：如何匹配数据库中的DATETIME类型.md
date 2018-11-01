---
layout: post
title: Java SQL-DATETIME
date: '2018-10-29 21:55'
description: "java.util.Date--java.sql.Date--java.sql.Time--java.sql.Timestamp--SimpleDateFormat"
tag: SQL系列文章（JAVA-SQL-*）
---

# JAVA杂谈：如何匹配数据库中的DATETIME类型

### 1. Java.sql.Date类

#### a)	继承关系

<img src="/images/post/dateclass.png" width="600px" height="">

#### b)	日期格式

> utilDate: Thu Nov 01 11:15:35 CST 2018   type = java.util.Date

#### c)	能否直接匹配数据库的DATETIME类型

> 不能，需要经过转化，详见2、3、4。

#### d)	转化为毫秒（ms）

> utilDate.getTime(): 1541042135041   type = long

### 2. 第一种转化方式 SimpleDateFormat类

#### a)	实施细节

<img src="/images/post/stringdatetime.png" width="600px" height="">

#### b)	写入和读取结果

> 写入：formatDate: 2018-11-01 11-15-35   type = String  
> 读取：getFormatData: 2018-11-01 11:15:35.0   type = String

#### c)	数据库中的写入结果

> 2018-11-01 11:15:35

### 3. 第二种转化方式 java.sql.Date类

#### a)	实施细节

<img src="/images/post/sqldatedatetime.png" width="600px" height="">

#### b)	写入和读取结果

> 写入：sqlDate: 2018-11-01   type = java.sql.Date
> 读取：getSqlData: 2018-11-01   type = java.sql.Date

#### c)	数据库中的写入结果

> 2018-11-01 00:00:00

### 4. 第三种转化方式 Timestamp类

#### a)	实施细节

<img src="/images/post/timestampdatetime.png" width="600px" height="">

#### b)	写入和读取结果

> 写入：timestamp: 2018-11-01 11:15:35.041   type = java.sql.Timestamp  
> 读取：getTimestamp: 2018-11-01 11:15:35.0   type = java.sql.Timestamp

#### c)	数据库中的写入结果

> 2018-11-01 11:15:35

### 5. 对java.sql.Time类的转化说明

> Time类无法直接写入为DATETIME类型。

<img src="/images/post/timedatetime.png" width="600px" height="">

> 输出结果：time: 11:15:35   type = java.sql.Time  
> 执行写入报错：Data truncation: Incorrect datetime value

### 6. 总结

> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;针对上述四种写入方式，我们发现，对于数据库中的DATETIME类型，只有2、3、4三种能够成功写入，其中2和4能够保持完整的“年月日时分秒”信息，而3只会保持“年月日”信息；5虽然能够输出“时分秒”的信息，但是无法与数据库的DATETIME类型匹配。

### 7. Java数据类型与Mysql数据类型对照表

| 类型名称	| 显示长度	| 数据库类型	| JAVA类型	| JDBC类型索引(int) |
| :------: | :-------: | :-------: | :-------: | :-------------: |		
| VARCHAR	| L+N	| VARCHAR	| java.lang.String	| 12 |
| CHAR	| N	| CHAR	| java.lang.String	| 1 |
| BLOB	| L+N	| BLOB	| java.lang.byte[]	| -4 |
| TEXT	| 65535	| VARCHAR	| java.lang.String	| -1 |
| INTEGER	| 4	| INTEGER UNSIGNED	| java.lang.Long	| 4 |
| TINYINT	| 3	| TINYINT UNSIGNED	| java.lang.Integer	| -6 |
| SMALLINT	| 5	| SMALLINT UNSIGNED	| java.lang.Integer	| 5 |
| MEDIUMINT	| 8	| MEDIUMINT UNSIGNED	| java.lang.Integer	| 4 |
| BIT	| 1	| BIT	| java.lang.Boolean	| -7 |
| BIGINT	| 20	| BIGINT UNSIGNED	| java.math.BigInteger | -5 |
| FLOAT	| 4+8	| FLOAT	| java.lang.Float	| 7 |
| DOUBLE	| 22	| DOUBLE	| java.lang.Double	| 8 |
| DECIMAL	| 11	| DECIMAL	| java.math.BigDecimal	| 3 |
| BOOLEAN	| 1	| 同TINYINT |	|  |
| ID	| 11	| PK(INTEGER UNSIGNED)	| java.lang.Long	| 4 |				
| DATE	| 10	| DATE	| java.sql.Date	| 91 |
| TIME	| 8	| TIME	| java.sql.Time	| 92 |
| DATETIME	| 19	| DATETIME	| java.sql.Timestamp	| 93 |
| TIMESTAMP	| 19	| TIMESTAMP	| java.sql.Timestamp	| 93 |
| YEAR	| 4	| YEAR	| java.sql.Date	| 91 |

`注：此信息来源于互联网，并未亲自考证。`

### 8. java和mysql之间的时间日期类型传递

> 惊奇发现，其实CSDN博客上面已经有大佬做出了比较系统的总结。链接如下：https://blog.csdn.net/weinianjie1/article/details/6310770

### 9. GitHub源码
[https://github.com/HengYk/DateTimeExploring](https://github.com/HengYk/DateTimeExploring)
