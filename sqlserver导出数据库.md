---
title: sqlserver导出数据库
date: 2021-05-17 10:59:10
categories:  [sql]
tags: [sql]
---


<!--more-->


# sqlserver导出数据库


方法一：
1.新建查询然后输入如下代码，点击F5键或者点击运行按钮即可
```bash
EXEC sp_attach_db @dbname = '你的数据库名',
@filename1 = 'mdf文件路径（包缀名）',
@filename2 = 'Ldf文件路径（包缀名）'
EXEC  sp_attach_db  @dbname  =  'empmanasys',     
@filename1  =  '/var/opt/mssql/data/empmanasys.mdf',     
@filename2  =  '/var/opt/mssql/data/empmanasys_log.ldf'
```
2.如果失败需要开启权限
方法二：附加