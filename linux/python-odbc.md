---
title: python-odbc
date: 2021-04-26 09:48:27
categories:  [odbc]
tags: [odbc]
---


<!--more-->


# python-odbc


odbc是一个连接mssql数据库的工具，python的库是 pyodbc


mac上正常是无法使用pyodbc这个库的，需要安装一些工具，命令如下
https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/install-microsoft-odbc-driver-sql-server-macos?view=sql-server-ver15

<div style='font-size:20px;color:red'>另外，git前面两个内容的时候，会很慢 因为包有点大  brew update 会更新很多东西，看个人情况 比如说我的Py3.7就被更新成了3.9</div>


```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
brew tap microsoft/mssql-release https://github.com/Microsoft/homebrew-mssql-release
brew update
HOMEBREW_NO_ENV_FILTERING=1 ACCEPT_EULA=Y brew install msodbcsql17 mssql-tools
```

查看自己下载好的驱动是什么版本的

```bash
> python3.9
> impot pyodbc
> pyodbc.drivers() 
...
> ['ODBC Driver 17 for SQL Server']
```

<div style='font-size:20px;color:red'>错误了可以试试这个,  谢谢，我按照您的方法连接上了，但是有一点点不同，您这里用ODBC Driver 17 for SQL Server，我这样是无效的，需要把17改成13；另外brew install —no-sandbox msodbcsql@13.1.9.2 mssql-tools@14.0.6.0这里，去掉了—no-sandbox才安装上</div>


## 连接数据库

```python
import pyodbc

# print(pyodbc.drivers())

server = '10.0.6.39'
database = 'master' # 数据库名
username = 'SA' # 用户名
password = 'umQlu2jf' # 密码
cnxn = pyodbc.connect('DRIVER={ODBC Driver 17 for SQL Server};SERVER='+server+';DATABASE='+database+';UID='+username+';PWD='+ password)
cursor = cnxn.cursor()

```