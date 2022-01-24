---
title: mysql基础
categories:
  - mysql
tags:
  - mysql
abbrlink: 8e40d0ad
date: 2020-02-23 15:58:49
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [mysql基础](#mysql基础)
  - [准备mysql](#准备mysql)
  - [mysql常用数据类型](#mysql常用数据类型)
    - [数值类型](#数值类型)
    - [时间类型](#时间类型)
    - [字符串类型](#字符串类型)
    - [验证数据类型 TINYINT](#验证数据类型-tinyint)
  - [基本语法](#基本语法)
  - [建表约束](#建表约束)
    - [主键约束](#主键约束)
    - [联合主键](#联合主键)
    - [自增约束](#自增约束)
    - [使用alter修改表属性](#使用alter修改表属性)
    - [唯一约束](#唯一约束)
    - [非空约束](#非空约束)
    - [默认约束](#默认约束)
    - [外键约束](#外键约束)
  - [数据库的三大设计范式](#数据库的三大设计范式)
  - [查询](#查询)
    - [准备数据](#准备数据)
    - [基础查询](#基础查询)
    - [分组查询](#分组查询)
    - [多表查询](#多表查询)
    - [子查询](#子查询)
    - [YEAR 函数与带 IN 关键字查询](#year-函数与带-in-关键字查询)
    - [多层嵌套子查询](#多层嵌套子查询)
    - [UNION 和 NOTIN 的使用](#union-和-notin-的使用)
    - [ANY 表示至少一个 - DESC ( 降序 )](#any-表示至少一个-desc-降序)
    - [表示所有的 ALL](#表示所有的-all)
    - [条件加组筛选](#条件加组筛选)
    - [NOTLIKE 模糊查询取反](#notlike-模糊查询取反)
    - [YEAR 与 NOW 函数](#year-与-now-函数)
    - [MAX 与 MIN 函数](#max-与-min-函数)

<!-- /code_chunk_output -->


learn mysql 📚
<!-- more -->



# mysql基础

## 准备mysql

## mysql常用数据类型

mysql 大致可以分为三类：
- 数值
- 日期/时间
- 字符串(字符)类型

https://www.runoob.com/mysql/mysql-data-types.html 

### 数值类型
类型	大小	范围（有符号）	范围（无符号）	用途
TINYINT	1 字节	(-128，127)	(0，255)	小整数值
SMALLINT	2 字节	(-32 768，32 767)	(0，65 535)	大整数值
MEDIUMINT	3 字节	(-8 388 608，8 388 607)	(0，16 777 215)	大整数值
INT或INTEGER	4 字节	(-2 147 483 648，2 147 483 647)	(0，4 294 967 295)	大整数值
BIGINT	8 字节	(-9,223,372,036,854,775,808，9 223 372 036 854 775 807)	(0，18 446 744 073 709 551 615)	极大整数值
FLOAT	4 字节	(-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38)	0，(1.175 494 351 E-38，3.402 823 466 E+38)	单精度
浮点数值
DOUBLE	8 字节	(-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)	0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)	双精度
浮点数值
DECIMAL	对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2	依赖于M和D的值	依赖于M和D的值	小数值

### 时间类型
类型	大小        范围	格式	用途
        (字节)	
DATE	3	1000-01-01/9999-12-31	YYYY-MM-DD	日期值
TIME	3	'-838:59:59'/'838:59:59'	HH:MM:SS	时间值或持续时间
YEAR	1	1901/2155	YYYY	年份值
DATETIME	8	1000-01-01 00:00:00/9999-12-31 23:59:59	YYYY-MM-DD HH:MM:SS	混合日期和时间值
TIMESTAMP	4	
1970-01-01 00:00:00/2038

结束时间是第 2147483647 秒，北京时间 2038-1-19 11:14:07，格林尼治时间 2038年1月19日 凌晨 03:14:07

YYYYMMDD HHMMSS	混合日期和时间值，时间

### 字符串类型
类型	大小	用途
CHAR	0-255字节	定长字符串
VARCHAR	0-65535 字节	变长字符串
TINYBLOB	0-255字节	不超过 255 个字符的二进制字符串
TINYTEXT	0-255字节	短文本字符串
BLOB	0-65 535字节	二进制形式的长文本数据
TEXT	0-65 535字节	长文本数据
MEDIUMBLOB	0-16 777 215字节	二进制形式的中等长度文本数据
MEDIUMTEXT	0-16 777 215字节	中等长度文本数据
LONGBLOB	0-4 294 967 295字节	二进制形式的极大文本数据
LONGTEXT	0-4 294 967 295字节	极大文本数据

### 验证数据类型 TINYINT
创建表
```sql 
create table testType(
    -> number TINYINT
    -> ); 
```
查看表
```sql
desc testType;
+--------+------------+------+-----+---------+-------+
| Field  | Type       | Null | Key | Default | Extra |
+--------+------------+------+-----+---------+-------+
| number | tinyint(4) | YES  |     | NULL    |       |
+--------+------------+------+-----+---------+-------+
1 row in set (0.00 sec) 
```
插入数据
```sql
 insert into testType values (127);
Query OK, 1 row affected (0.00 sec)

insert into testType  values (128);
ERROR 1264 (22003): Out of range value for column 'number' at row 1
```
在插入128的时候报错，因为已经超越了要求的值


## 基本语法
```sql 
-- 显示所有数据库
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

-- 创建数据库
CREATE DATABASE test;
Query OK, 1 row affected (0.00 sec)

-- 切换数据库
use  test;
Database changed

-- 显示数据库中的所有表
show tables;
Empty set (0.00 sec)

-- 创建数据表
mysq> CREATE TABLE pet(
    name VARCHAR(20),
    owner VARCHAR(20),
    species VARCHAR(20),
    sex CHAR(1),
    birth DATE,
    death DATE
);
Query OK, 0 rows affected (0.03 sec)

-- 查看数据表结构
desc pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | NULL    |       |
| owner   | varchar(20) | YES  |     | NULL    |       |
| species | varchar(20) | YES  |     | NULL    |       |
| sex     | char(1)     | YES  |     | NULL    |       |
| birth   | date        | YES  |     | NULL    |       |
| death   | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
6 rows in set (0.00 sec)

-- 查询表
select * from pet;
Empty set (0.00 sec)

-- 插入数据
INSERT INTO pet 
VALUES ('puffball', 'Diane', 'hamster', 'f', '1990-03-30', NULL);
Query OK, 1 row affected (0.00 sec)

-- 修改数据
UPDATE pet 
SET name = 'squirrel' 
where owner = 'Diane';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

-- 删除数据
DELETE FROM pet 
where name = 'squirrel';

-- 删除表
DROP TABLE pet;
```




## 建表约束

### 主键约束

他能够唯一确定一张表中的每一条记录，也就是我们通过给某个字段添加约束，就可以使得该字段不重复且不为空
创建有主键约束的表
```sql 
create table user(
    id int primary key,
    name varchar(25)
);
```
查看表结构
```sql 
desc user;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| name  | varchar(25) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```
这里key上有一个Primary的就是主键
插入两条值
```sql
insert into user 
values(1,'张三'); 

insert into user
values(1,'李四');

insert into user values(1,'李四');
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'
```
报错，主键重复 


### 联合主键
只要联合的主键值加起来不重复就可以出现
```sql
create table user2(
    id int,
    name varchar(20),
    password varchar(20),
    primary key (id,name)
);
```
插入值
```sql
insert into user2 values (1,'王五','hzjqq');
insert into user2 values (2,'王五','hzjqq'); -- 可以插入 
insert into user2 values (1,'王五','hzjqq'); -- 无法插入 联合组一摸一样
```



### 自增约束

创建表
```sql 
create table user3 (
    id int primary key auto_increment,
    name varchar(20)
);
```
插入值
```sql
insert into user3 (name) values("hzj");
insert into user3 (name) values("zh");
insert into user3 (name) values("qqq");
```


### 使用alter修改表属性
创建表
```sql 
create table user4(
    id int ,
    name varchar(20)
);
```
使用Alter修改表属性来添加主键
```sql
alter table user4 add primary key (id);
```
使用alter修改表属性来添加表头
```sql
alter table user4 add column sex varchar(20);
```
使用alter修改表属性来删除主键
```sql 
alter table user4 drop primary key ;
```
使用alter修改表属性来修改主键 （名字+type  id int）
```sql
alter table user4 
modify id int primary key;
```
### 唯一约束
约束修饰的字段的值不可以重复

<font color='red '>主键约束唯一且不允许为空，但是唯一约束保证唯一允许为空</font>  

```sql
create table user5(
    id int,
    name varchar(20)
);
```
修改主键添加唯一主键约束
```sql
alter table user5 add unique(name);
-- 或者
alter table user5 modify name varchar(20) unique;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(20) | YES  | UNI | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```
插入值
```sql
insert into  user5 value (1,NULL); -- success
insert into user5 value (1,Null); -- success 
insert into user5 value (2,NULL);  -- success
insert into user5 value (2,'hzj'); -- faild
insert into user5 value (2,'hzj'); -- faild

select * from user5;
+------+------+
| id   | name |
+------+------+
|    1 | NULL |
|    1 | NULL |
|    2 | NULL |
+------+------+
```
<font color='red'>之前说过unique会保证唯一性，但是允许为空，对于空的字段，则不保证唯一性，可以插入</font>

<font color='blue'>创建唯一约束的另外的方法</font>

```sql
create table user7(
    id int,
    name varchar(20) unique,
    age int
)
```

<font color='blue'>组合键不重复即可</font>

unique(id,name) 表示两个键在一起不重复就可以通过

<font color='blue'>删除唯一约束</font>

```sql
-- index  指定是删除唯一约束的命令
-- user5 指定表
-- name 指定表项
alter table use5 drop index name
```

### 非空约束
修饰的字段不能为空  NULL
创建表
```sql
create table user9(
    id int,
    name varchar(10) NOT NULL
);
```
查看表
```sql
mysql> desc user9;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(10) | NO   |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

### 默认约束
约束某个字段的默认值
```sql
create table user10(
    id int,
    name varchar(20),
    age int Default 10
);
```
查看表
```sql
mysql> desc user10;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| age   | int(11)     | YES  |     | 10      |       |
+-------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```
移除默认约束
```sql
alter table user10 modify age int;
```

### 外键约束
表与表之间的联系

创建教室表
```sql
create table classes(
    id int primary key,
    name varchar(20)
);

创建学生表
create table students(
    id int primary key,
    name varchar(20),
    age int NOT Null,
    class_id int ,
    foreign key(class_id) references classes(id) 
);
```
插入数据
```sql
insert into classes values(1,"one");
insert into classes values(2,"two");
insert into classes values(3,"there");
insert into classes values(4,"four");
```
插入学生表
```sql
insert into students values(1,"hzj",1);
insert into students values(2,"hzj",2);
insert into students values(3,"hzj",3);
insert into students values(5,"hzj",5); -- 报错，因为没有5班
-- ERROR 1136 (21S01): Column count doesn't match value count at row 1
insert into students values(4,"hzj",4);
```

主表classes 中没有的数据值，在副表中，是不可以使用的
主表中的记录被副表引用时，是不可以被删除的。只有将引用的副表删除了以后，才能删除主表



## 数据库的三大设计范式
- 1NF
只要字段值还可以继续拆分，就不满足第一范式。
范式设计得越详细，对某些实际操作可能会更好，但并非都有好处，需要对项目的实际情况进行设定
举例: 创建一个表
```sql
create table selinfo (
    id int primary key  auto_increment,
    name varchar(20),
    address varchar(20)
);
```
插入数据
```sql
insert into selinfo values('hzj','杭州市萧山区xx街道');
```
这样是不行的，因为address还可以再分，因此可以设计成为
```sql
create table selinfo (
    id int primary key auto_increment,
    name varchar(20),
    city varchar(20),
    address varchar(20)
);
```
- 2NF
在满足第一范式的前提下，其他列都必须完全依赖于主键列。如果出现不完全依赖，只可能发生在联合主键的情况下：
举例,单表创建一个订单表
```sql
-- 订单表
CREATE TABLE myorder (
    product_id INT,
    customer_id INT,
    product_name VARCHAR(20),
    customer_name VARCHAR(20),
    PRIMARY KEY (product_id, customer_id)
);
```
实际上，在这张订单表中，product_name 只依赖于 product_id ，customer_name 只依赖于 customer_id 。也就是说，product_name 和 customer_id 是没用关系的，customer_name 和 product_id 也是没有关系的。

创建多张表
```sql
-- 主表
CREATE TABLE myorder (
    order_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT
);
-- 产品表
CREATE TABLE product (
    id INT PRIMARY KEY,
    name VARCHAR(20)
);
-- 用户表
CREATE TABLE customer (
    id INT PRIMARY KEY,
    name VARCHAR(20)
);
```
<font color='red'>也就是说一个主键要对应一段信息</font>
拆分之后，myorder 表中的 product_id 和 customer_id 完全依赖于 order_id 主键，而 product 和 customer 表中的其他字段又完全依赖于主键。满足了第二范式的设计！

- 3NF
在满足第二范式的前提下，除了主键列之外，其他列之间不能有传递依赖关系。
```sql
CREATE TABLE myorder (
    order_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT,
    customer_phone VARCHAR(15)
);
```
表中的 customer_phone 有可能依赖于 order_id 、 customer_id 两列，也就不满足了第三范式的设计：其他列之间不能有传递依赖关系。
```sql
CREATE TABLE myorder (
    order_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT
);

CREATE TABLE customer (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    phone VARCHAR(15)
)
```
修改后就不存在其他列之间的传递依赖关系，其他列都只依赖于主键列，满足了第三范式的设计！

## 查询

### 准备数据
```sql
-- 创建数据库
CREATE DATABASE select_test character set utf8 collate utf8_general_ci;

-- 切换数据库
USE select_test1;

-- 创建学生表
CREATE TABLE student (
    no VARCHAR(20) PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    sex VARCHAR(10) NOT NULL,
    birthday DATE, -- 生日
    class VARCHAR(20) -- 所在班级
);

-- 创建教师表
CREATE TABLE teacher (
    no VARCHAR(20) PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    sex VARCHAR(10) NOT NULL,
    birthday DATE,
    profession VARCHAR(20) NOT NULL, -- 职称
    department VARCHAR(20) NOT NULL -- 部门
);

-- 创建课程表
CREATE TABLE course (
    no VARCHAR(20) PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    t_no VARCHAR(20) NOT NULL, -- 教师编号
    -- 表示该 tno 来自于 teacher 表中的 no 字段值
    FOREIGN KEY(t_no) REFERENCES teacher(no) 
);

-- 成绩表
CREATE TABLE score (
    s_no VARCHAR(20) NOT NULL, -- 学生编号
    c_no VARCHAR(20) NOT NULL, -- 课程号
    degree DECIMAL,	-- 成绩
    -- 表示该 s_no, c_no 分别来自于 student, course 表中的 no 字段值
    FOREIGN KEY(s_no) REFERENCES student(no),	
    FOREIGN KEY(c_no) REFERENCES course(no),
    -- 设置 s_no, c_no 为联合主键
    PRIMARY KEY(s_no, c_no)
);

-- 查看所有表
SHOW TABLES;

-- 添加学生表数据
INSERT INTO student VALUES('101', '曾华', '男', '1977-09-01', '95033');
INSERT INTO student VALUES('102', '匡明', '男', '1975-10-02', '95031');
INSERT INTO student VALUES('103', '王丽', '女', '1976-01-23', '95033');
INSERT INTO student VALUES('104', '李军', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('105', '王芳', '女', '1975-02-10', '95031');
INSERT INTO student VALUES('106', '陆军', '男', '1974-06-03', '95031');
INSERT INTO student VALUES('107', '王尼玛', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('108', '张全蛋', '男', '1975-02-10', '95031');
INSERT INTO student VALUES('109', '赵铁柱', '男', '1974-06-03', '95031');

-- 添加教师表数据
INSERT INTO teacher VALUES('804', '李诚', '男', '1958-12-02', '副教授', '计算机系');
INSERT INTO teacher VALUES('856', '张旭', '男', '1969-03-12', '讲师', '电子工程系');
INSERT INTO teacher VALUES('825', '王萍', '女', '1972-05-05', '助教', '计算机系');
INSERT INTO teacher VALUES('831', '刘冰', '女', '1977-08-14', '助教', '电子工程系');

-- 添加课程表数据
INSERT INTO course VALUES('3-105', '计算机导论', '825');
INSERT INTO course VALUES('3-245', '操作系统', '804');
INSERT INTO course VALUES('6-166', '数字电路', '856');
INSERT INTO course VALUES('9-888', '高等数学', '831');

-- 添加添加成绩表数据
INSERT INTO score VALUES('103', '3-105', '92');
INSERT INTO score VALUES('103', '3-245', '86');
INSERT INTO score VALUES('103', '6-166', '85');
INSERT INTO score VALUES('105', '3-105', '88');
INSERT INTO score VALUES('105', '3-245', '75');
INSERT INTO score VALUES('105', '6-166', '79');
INSERT INTO score VALUES('109', '3-105', '76');
INSERT INTO score VALUES('109', '3-245', '68');
INSERT INTO score VALUES('109', '6-166', '81');

-- 查看表结构
mysql> SELECT * FROM course;
+-------+-----------------+------+
| no    | name            | t_no |
+-------+-----------------+------+
| 3-105 | 计算机导论      | 825  |
| 3-245 | 操作系统        | 804  |
| 6-166 | 数字电路        | 856  |
| 9-888 | 高等数学        | 831  |
+-------+-----------------+------+
4 rows in set (0.00 sec)

mysql> SELECT * FROM score;
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 103  | 3-245 |     86 |
| 103  | 6-166 |     85 |
| 105  | 3-105 |     88 |
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+
9 rows in set (0.00 sec)
mysql> SELECT * FROM student;
+-----+-----------+-----+------------+-------+
| no  | name      | sex | birthday   | class |
+-----+-----------+-----+------------+-------+
| 101 | 曾华      | 男  | 1977-09-01 | 95033 |
| 102 | 匡明      | 男  | 1975-10-02 | 95031 |
| 103 | 王丽      | 女  | 1976-01-23 | 95033 |
| 104 | 李军      | 男  | 1976-02-20 | 95033 |
| 105 | 王芳      | 女  | 1975-02-10 | 95031 |
| 106 | 陆军      | 男  | 1974-06-03 | 95031 |
| 107 | 王尼玛    | 男  | 1976-02-20 | 95033 |
| 108 | 张全蛋    | 男  | 1975-02-10 | 95031 |
| 109 | 赵铁柱    | 男  | 1974-06-03 | 95031 |
+-----+-----------+-----+------------+-------+
9 rows in set (0.00 sec)

mysql> SELECT * FROM teacher;
+-----+--------+-----+------------+------------+-----------------+
| no  | name   | sex | birthday   | profession | department      |
+-----+--------+-----+------------+------------+-----------------+
| 804 | 李诚   | 男  | 1958-12-02 | 副教授     | 计算机系        |
| 825 | 王萍   | 女  | 1972-05-05 | 助教       | 计算机系        |
| 831 | 刘冰   | 女  | 1977-08-14 | 助教       | 电子工程系      |
| 856 | 张旭   | 男  | 1969-03-12 | 讲师       | 电子工程系      |
+-----+--------+-----+------------+------------+-----------------+
4 rows in set (0.00 sec)

```

### 基础查询
```sql
-- 查询 student 表的所有行
SELECT * FROM student;

-- 查询 student 表中的 name、sex 和 class 字段的所有行
SELECT name, sex, class FROM student;

-- 查询 teacher 表中不重复的 department 列
-- department: 去重查询
SELECT DISTINCT department FROM teacher;

-- 查询 score 表中成绩在60-80之间的所有行（区间查询和运算符查询）
-- BETWEEN xx AND xx: 查询区间, AND 表示 "并且"
SELECT * FROM score WHERE degree BETWEEN 60 AND 80;
SELECT * FROM score WHERE degree > 60 AND degree < 80;

-- 查询 score 表中成绩为 85, 86 或 88 的行
-- IN: 查询规定中的多个值
SELECT * FROM score WHERE degree IN (85, 86, 88);

-- 查询 student 表中 '95031' 班或性别为 '女' 的所有行
-- or: 表示或者关系
SELECT * FROM student WHERE class = '95031' or sex = '女';

-- 以 class 降序的方式查询 student 表的所有行
-- DESC: 降序，从高到低
-- ASC（默认）: 升序，从低到高
SELECT * FROM student ORDER BY class DESC;
SELECT * FROM student ORDER BY class ASC;

-- 以 c_no 升序、degree 降序查询 score 表的所有行
SELECT * FROM score ORDER BY c_no ASC, degree DESC;

-- 查询 "95031" 班的学生人数
-- COUNT: 统计
SELECT COUNT(*) FROM student WHERE class = '95031';

-- 查询 score 表中的最高分的学生学号和课程编号（子查询或排序查询）。
-- (SELECT MAX(degree) FROM score): 子查询，算出最高分
SELECT s_no, c_no 
FROM score 
WHERE degree = (
    SELECT MAX(degree) 
    FROM score
);

--  排序查询
-- LIMIT r, n: 表示从第r行开始，查询n条数据
SELECT s_no, c_no, degree 
FROM score 
ORDER BY degree 
DESC LIMIT 0, 1;

```
### 分组查询
1. 查询每门课的平均成绩
```sql
-- AVG: 平均值
SELECT AVG(degree) FROM score WHERE c_no = '3-105';
SELECT AVG(degree) FROM score WHERE c_no = '3-245';
SELECT AVG(degree) FROM score WHERE c_no = '6-166';

-- GROUP BY: 分组查询
SELECT c_no, AVG(degree) FROM score GROUP BY c_no;

-- 计算每一门课程拥有的报名人数，这里以c_no分组，其他列默认选第一条数据的内容
mysql> select c_no,s_no,degree, COUNT(c_no) from score group by c_no;
+-------+------+--------+-------------+
| c_no  | s_no | degree | COUNT(c_no) |
+-------+------+--------+-------------+
| 3-105 | 103  |     92 |           3 |
| 3-245 | 103  |     86 |           3 |
| 6-166 | 103  |     85 |           3 |
+-------+------+--------+-------------+
3 rows in set (0.00 sec)
```


2. 查询 score 表中至少有 2 名学生选修，并以 3 开头的课程的平均分数。
简答的分析一下，首先是求平均数  avg(degree)
其次是以课程为分组  group by c_no
然后是 2名以上学生参加 3开头的课程
```sql
mysql> select avg(degree) from score group by c_no having c_no like '3%' and count(c_no)>2;
+-------------+
| avg(degree) |
+-------------+
|     85.3333 |
|     76.3333 |
+-------------+
```


### 多表查询
1. 查询所有学生的 name，以及该学生在 score 表中对应的 c_no 和 degree 。
```sql
-- 查看所有学生
mysql> select * from student;
+-----+-----------+-----+------------+-------+
| no  | name      | sex | birthday   | class |
+-----+-----------+-----+------------+-------+
| 101 | 曾华      | 男  | 1977-09-01 | 95033 |
| 102 | 匡明      | 男  | 1975-10-02 | 95031 |
| 103 | 王丽      | 女  | 1976-01-23 | 95033 |
| 104 | 李军      | 男  | 1976-02-20 | 95033 |
| 105 | 王芳      | 女  | 1975-02-10 | 95031 |
| 106 | 陆军      | 男  | 1974-06-03 | 95031 |
| 107 | 王尼玛    | 男  | 1976-02-20 | 95033 |
| 108 | 张全蛋    | 男  | 1975-02-10 | 95031 |
| 109 | 赵铁柱    | 男  | 1974-06-03 | 95031 |
+-----+-----------+-----+------------+-------+
9 rows in set (0.00 sec)
-- 查询所有成绩
mysql> select * from score;
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 103  | 3-245 |     86 |
| 103  | 6-166 |     85 |
| 105  | 3-105 |     88 |
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+
9 rows in set (0.00 sec)
```
其中这里no与score中的s_no做了对应,首先做一个聚合，8*8一共会出现81条数据
然后在做条件筛选
```sql
mysql > select s_no,no,name from student,score ;
...
81 rows in set (0.00 sec)
mysql> select name,c_no,degree  from student,score where student.no = score.s_no ;
+-----------+-------+--------+
| name      | c_no  | degree |
+-----------+-------+--------+
| 王丽      | 3-105 |     92 |
| 王丽      | 3-245 |     86 |
| 王丽      | 6-166 |     85 |
| 王芳      | 3-105 |     88 |
| 王芳      | 3-245 |     75 |
| 王芳      | 6-166 |     79 |
| 赵铁柱    | 3-105 |     76 |
| 赵铁柱    | 3-245 |     68 |
| 赵铁柱    | 6-166 |     81 |
+-----------+-------+--------+
```
2. 查询所有学生的 no 、课程名称 ( course 表中的 name ) 和成绩 ( score 表中的 degree ) 列。
```sql
mysql> select * from course;
+-------+-----------------+------+
| no    | name            | t_no |
+-------+-----------------+------+
| 3-105 | 计算机导论      | 825  |
| 3-245 | 操作系统        | 804  |
| 6-166 | 数字电路        | 856  |
| 9-888 | 高等数学        | 831  |
+-------+-----------------+------+
4 rows in set (0.01 sec)
mysql> select * from score;
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 103  | 3-245 |     86 |
| 103  | 6-166 |     85 |
| 105  | 3-105 |     88 |
| 105  | 3-245 |     75 |
| 105  | 6-166 |     79 |
| 109  | 3-105 |     76 |
| 109  | 3-245 |     68 |
| 109  | 6-166 |     81 |
+------+-------+--------+
9 rows in set (0.00 sec)
```
```sql
-- 增加一个查询字段 name，分别从 score、course 这两个表中查询。
-- as 表示取一个该字段的别名。
SELECT s_no, name as c_name, degree FROM score, course
WHERE score.c_no = course.no;
+------+-----------------+--------+
| s_no | c_name          | degree |
+------+-----------------+--------+
| 103  | 计算机导论      |     92 |
| 105  | 计算机导论      |     88 |
| 109  | 计算机导论      |     76 |
| 103  | 操作系统        |     86 |
| 105  | 操作系统        |     75 |
| 109  | 操作系统        |     68 |
| 103  | 数字电路        |     85 |
| 105  | 数字电路        |     79 |
| 109  | 数字电路        |     81 |
+------+-----------------+--------+
```
3. 查询所有学生的 name 、课程名 ( course 表中的 name ) 和 degree 。
由于name字段名在student表中以及course表中都出现过了，因此使用 "表名.字段名 as 别名" 代替。
```sql
mysql> SELECT student.name as s_name, course.name as c_name, degree
    -> FROM student, score, course
    -> WHERE student.NO = score.s_no
    -> AND score.c_no = course.no;
+-----------+-----------------+--------+
| s_name    | c_name          | degree |
+-----------+-----------------+--------+
| 王丽      | 计算机导论      |     92 |
| 王芳      | 计算机导论      |     88 |
| 赵铁柱    | 计算机导论      |     76 |
| 王丽      | 操作系统        |     86 |
| 王芳      | 操作系统        |     75 |
| 赵铁柱    | 操作系统        |     68 |
| 王丽      | 数字电路        |     85 |
| 王芳      | 数字电路        |     79 |
| 赵铁柱    | 数字电路        |     81 |
+-----------+-----------------+--------+
9 rows in set (0.00 sec)
```

### 子查询
1. 查询 95031 班学生每门课程的平均成绩
```sql
SELECT c_no, AVG(degree) FROM score
WHERE s_no IN (SELECT no FROM student WHERE class = '95031')
GROUP BY c_no;
+-------+-------------+
| c_no  | AVG(degree) |
+-------+-------------+
| 3-105 |     82.0000 |
| 3-245 |     71.5000 |
| 6-166 |     80.0000 |
+-------+-------------+
```

2. 首先筛选出课堂号为 3-105 ，在找出所有成绩高于 109 号同学的的行。
```sql
mysql> select * from score where c_no = 3-105 and  degree >(select degree from score where s_no = 109 and c_no = '3-105');
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 105  | 3-105 |     88 |
+------+-------+--------+
2 rows in set (0.00 sec)
```

3. 查询所有成绩高于109号同学的3-105课程成绩记录
```sql
-- 不限制课程号，只要成绩大于109号同学的3-105课程成绩就可以。
SELECT * FROM score
WHERE degree > (SELECT degree FROM score WHERE s_no = '109' AND c_no = '3-105');
```

### YEAR 函数与带 IN 关键字查询
1. 查询所有和 101 、108 号学生同年出生的 no 、name 、birthday 列。
```sql
SELECT no, name, birthday FROM student
WHERE YEAR(birthday) IN (SELECT YEAR(birthday) FROM student WHERE no IN (101, 108));
```

### 多层嵌套子查询
1. 查询 '张旭' 教师任课的学生成绩表。
先查询老师的id 然后通过反查课程id  再通过课程id反查成绩
```sql
mysql> SELECT * FROM score WHERE c_no = (
    ->     SELECT no FROM course WHERE t_no = (
    ->         SELECT no FROM teacher WHERE NAME = '张旭'
    ->     )
    -> );
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 6-166 |     85 |
| 105  | 6-166 |     79 |
| 109  | 6-166 |     81 |
+------+-------+--------+
3 rows in set (0.00 sec)
```
2. 查询某选修课程多于5个同学的教师姓名
```sql
SELECT name FROM teacher WHERE no IN (
    -- 最终条件
    SELECT t_no FROM course WHERE no IN (
        SELECT c_no FROM score GROUP BY c_no HAVING COUNT(*) > 5
    )
);
```
3. 查询 “计算机系” 课程的成绩表。
```sql
-- 根据筛选出来的课程号查询成绩表
SELECT * FROM score WHERE c_no IN (
    SELECT no FROM course WHERE t_no IN (
        SELECT no FROM teacher WHERE department = '计算机系'
    )
);
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-245 |     86 |
| 105  | 3-245 |     75 |
| 109  | 3-245 |     68 |
| 101  | 3-105 |     90 |
| 102  | 3-105 |     91 |
| 103  | 3-105 |     92 |
| 104  | 3-105 |     89 |
| 105  | 3-105 |     88 |
| 109  | 3-105 |     76 |
+------+-------+--------+
```

### UNION 和 NOTIN 的使用
1. 查询 计算机系 与 电子工程系 中的不同职称的教师。
```sql
-- NOT: 代表逻辑非
SELECT * FROM teacher WHERE department = '计算机系' AND profession NOT IN (
    SELECT profession FROM teacher WHERE department = '电子工程系'
)
-- 合并两个集
UNION
SELECT * FROM teacher WHERE department = '电子工程系' AND profession NOT IN (
    SELECT profession FROM teacher WHERE department = '计算机系'
);
```



### ANY 表示至少一个 - DESC ( 降序 )
1. 查询课程 3-105 且成绩 至少 高于 3-245 的 score 表。
```sql
-- ANY: 符合SQL语句中的任意条件。
-- 也就是说，在 3-105 成绩中，只要有一个大于从 3-245 筛选出来的任意行就符合条件，
-- 最后根据降序查询结果。
SELECT * FROM score WHERE c_no = '3-105' AND degree > ANY(
    SELECT degree FROM score WHERE c_no = '3-245'
) ORDER BY degree DESC;
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 105  | 3-105 |     88 |
| 109  | 3-105 |     76 |
+------+-------+--------+
```


### 表示所有的 ALL
1. 查询课程 3-105 且成绩高于 3-245 的 score 表。
```sql
-- 只需对上一道题稍作修改。
-- ALL: 符合SQL语句中的所有条件。
-- 也就是说，在 3-105 每一行成绩中，都要大于从 3-245 筛选出来全部行才算符合条件。
SELECT * FROM score WHERE c_no = '3-105' AND degree > ALL(
    SELECT degree FROM score WHERE c_no = '3-245'
);
+------+-------+--------+
| s_no | c_no  | degree |
+------+-------+--------+
| 103  | 3-105 |     92 |
| 105  | 3-105 |     88 |
+------+-------+--------+
2 rows in set (0.00 sec)
```
### 条件加组筛选
1. 查询 student 表中至少有 2 名男生的 class 。
```sql
SELECT class FROM student WHERE sex = '男' GROUP BY class HAVING COUNT(*) > 1;
```

### NOTLIKE 模糊查询取反
查询 student 表中不姓 "王" 的同学记录。
```sql
-- NOT: 取反
-- LIKE: 模糊查询
mysql> SELECT * FROM student WHERE name NOT LIKE '王%';
+-----+-----------+-----+------------+-------+
| no  | name      | sex | birthday   | class |
+-----+-----------+-----+------------+-------+
| 101 | 曾华      | 男  | 1977-09-01 | 95033 |
| 102 | 匡明      | 男  | 1975-10-02 | 95031 |
| 104 | 李军      | 男  | 1976-02-20 | 95033 |
| 106 | 陆军      | 男  | 1974-06-03 | 95031 |
| 108 | 张全蛋    | 男  | 1975-02-10 | 95031 |
| 109 | 赵铁柱    | 男  | 1974-06-03 | 95031 |
| 110 | 张飞      | 男  | 1974-06-03 | 95038 |
+-----+-----------+-----+------------+-------+
```


### YEAR 与 NOW 函数
查询 student 表中每个学生的姓名和年龄。
```sql
-- 使用函数 YEAR(NOW()) 计算出当前年份，减去出生年份后得出年龄。
SELECT name, YEAR(NOW()) - YEAR(birthday) as age FROM student;
+-----------+------+
| name      | age  |
+-----------+------+
| 曾华      |   42 |
| 匡明      |   44 |
| 王丽      |   43 |
| 李军      |   43 |
| 王芳      |   44 |
| 陆军      |   45 |
| 王尼玛    |   43 |
| 张全蛋    |   44 |
| 赵铁柱    |   45 |
| 张飞      |   45 |
+-----------+------+
```


### MAX 与 MIN 函数
查询 student 表中最大和最小的 birthday 值。
```sql
SELECT MAX(birthday), MIN(birthday) FROM student;
+---------------+---------------+
| MAX(birthday) | MIN(birthday) |
+---------------+---------------+
| 1977-09-01    | 1974-06-03    |
+---------------+---------------+

```

https://github.com/hjzCy/sql_node/blob/master/mysql/MySQL%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0.md
https://www.bilibili.com/video/av39807944?p=23

![2020-02-24-16-12-39](http://noback.upyun.com/2020-02-24-16-12-39.png)