---
title: django错误合集
categories:
  - 编程
abbrlink: 7f55f388
tags: 
  - django
date: 2019-12-29 20:40:29
---
# 错误类型总结
由于mysql与django在配置过程中会存在很多的错误，这里总结一下
## Mysql在Centos中权限不足的错误
```text
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
```
原因是/var/lib/mysql的访问权限不足
```text
1.改变权限 chown -R mysql:mysql /var/lib/mysql   
2. 启动服务器 /etc/init.d/mysql start (会自动生成mysql.sock文件)  
3.重新启动mysql服务  /etc/init.d/mysql start
```
## 密码类型错误
```text
1045, "Access denied for user 'root'@'122.224.83.xxx  
```
原因 密码错误，注意密码不能是字符串
```text
GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```
## django setting 中的配置
```text
DATABASES = {
    'default': {

        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'itemids',  # 数据库名字
        'USER': 'root',    # 数据库登录名
        'PASSWORD': '12345x', # 数据库密码
        'HOST': '106.14.195.xxx', 数据库IP
        'POST': 3306, # 端口
    }
}
```
## 引用pymysql
由于MySQLdb只能使用在Python2中，在python3中已经停止了维护，所以这里我们引用pymyql的库
```python
pipenv install pymysql
# 在与setting.py同级别的__init__.py中使用
import pymysql
pymysql.install_as_MySQLdb()
```
## pymysql版本不符合
报错内容:django.core.exceptions.ImproperlyConfigured: mysqlclient 1.3.3 or newer is required; you have 0.7.11
修改Pipenv\Lib\site-packages\django\db\backends\mysql\base.py
注释下面内容
if version < (1, 3, 3):
    raise ImproperlyConfigured("mysqlclient 1.3.3 or newer is required; you have %s" % Database.__version__)

(在Centos中，pipenv install 所生成的包 会在pipenv shell 激活环境的时候出现)


由于上面注释，还会出现的报错内容为
File "C:\Users\Administrator\PycharmProjects\untitled1\venv\lib\site-packages\django\db\backends\mysql\operations.py", line 146, in last_executed_query
将operations.py中的decode改成encode

## mysql不能插入中文
报错内容： django.db.utils.InternalError: (1366, "Incorrect string value: '\\xE5\\xAE\\x9A\\xE6\\x97\\xB6...' for column 'name' at row 1")
解决方法: https://blog.csdn.net/tzh476/article/details/52644271
删除之前的库，创建一个新的数据库，使用utf8mb64   之前默认创建的都是latin1
不一样要删除之前的库，修改数据库的属性即可

## mysql 长度的问题
django.db.utils.InternalError: (1071, 'Specified key was too long; max key length is 767 bytes')
https://blog.csdn.net/ljfphp/article/details/80406907
https://www.orcode.com/question/407126_k280b8.html

解决方法1: 升级Mysql5.6-->>Mysql5.7
centos7 下  对mysql的操作 https://blog.csdn.net/xufengzhu/article/details/81110982
https://blog.51cto.com/lisea/1941616

解决方法2 : 先使用utf8格式


## django migrate的问题
出现下面这个问题，基本上就是你的model设置错误了。有些字段类型需要固定的属性
如integerField字段需要default这个选项，添加完之后就Ok了
```bash
Please select a fix:
 1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
 2) Quit, and let me add a default in models.p 
```