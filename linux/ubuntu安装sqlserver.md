---
title: ubuntu安装sqlserver
date: 2021-04-25 13:53:34
categories:  [ubuntu]
tags: [ubuntu]
---


<!--more-->


# ubuntu安装sqlserver



## ubuntu安装sqlserver

> ubuntu版本要求  18.04 or 16.04
https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-ubuntu?view=sql-server-ver15



<div style='font-size:20px;color:red'>
    ubuntu add-apt-repository command not found  解决办法
</div>

```bash
Launchpad PPA Repositories是很有用的非ubuntu官方的第三方个人资源库，可以很方便地安装第三方软件。
但是在运行add-apt-repository命令时，有时会提示命令不存在，这个时候直接apt-get add-apt-repository是不可以的！
解决的方法是安装software-properties-common。输入命令：


apt-get install software-properties-common
```

> 安装流程

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
# 18.04
sudo add-apt-repository "$(wget -qO- https://packages.microsoft.com/config/ubuntu/18.04/mssql-server-2019.list)" 
sudo apt-get update
sudo apt-get install -y mssql-server
sudo /opt/mssql/bin/mssql-conf setup
systemctl status mssql-server --no-pager
```

中途会让你选择安装的版本，主要是收费与不收费的问题

![](https://noback.upyun.com/2021-04-25-13-56-12.png!)




## 使用docker 安装mssql
实体机安装mssql 可能会破坏环境，大学的时候被mssql折磨坏了，一直怕出问题，这里更推荐使用docker 安装mssql
```bash
docker pull mcr.microsoft.com/mssql/server
```



复杂密码: umQlu2jf
生成更多的复杂密码
http://tool.c7sky.com/password/


连接mssql数据库
```bash
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P umQlu2jf
```

## python 连接数据库
查看另外一篇文章

## mssql数据库操作

<div style='font-size:20px;color:red'>创建数据库</div>
```sql
USE master ;  
GO  
CREATE DATABASE Sales  
ON   
( NAME = Sales_dat,  
    FILENAME = 'C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\MSSQL\DATA\saledat.mdf',  
    SIZE = 10,  
    MAXSIZE = 50,  
    FILEGROWTH = 5 )  
LOG ON  
( NAME = Sales_log,  
    FILENAME = 'C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\MSSQL\DATA\salelog.ldf',  
    SIZE = 5MB,  
    MAXSIZE = 25MB,  
    FILEGROWTH = 5MB ) ;  
GO  
```
```sql
CREATE DATABASE Testdb
```

### <div style='font-size:20px;color:red'>数据库相关</div>

```sql
// 查看当前数据库中的所有库名
select name from sys.databases 
go

// 创建数据库
CREATE DATABASE MyDb
go

// 使用数据库
USE MyDb
go


```

### <div style='font-size:20px;color:red'>表相关</div>

```sql

// 创建表
CREATE TABLE 表名
(
    字段名,数据类型,
    字段名,数据类型,
    .....
)
go

CREATE TABLE t_user
(
    id INT primary key,
    username VARCHAR(32),
    password VARCHAR(32),
)

// 查看表结构
sp_help 表名
go

```

### 参考资料
https://blog.csdn.net/star_of_science/article/details/107026211