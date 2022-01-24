---
title: django内容总结
categories:
  - 编程
abbrlink: 7e91b378
tags: 
  - django
date: 2019-12-29 20:40:44
---
# django内容总结

## 在后台对某些固定字段设置颜色

```py
# models.py
from django.urls.html import format_html
class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    color_code = models.CharField(max_length=6)


    def colored_name(self):

        return format_html(

            '<span style="color: #{};">{} {}</span>',

            self.color_code,

            self.first_name,

            self.last_name,

        )
```
### admin.py
class PersonAdmin(admin.ModelAdmin):
    list_display = ('first_name', 'last_name', 'colored_name')

## DjangoAdmin
> Django自带后台管理模板是其一大特色，但作为后台管理很多地方需要我们去改进，当然也可以使用一些其他的后台模板

- 推荐模板1: xadmin xadmin在几年前是为django定制的后台管理模板，但是作者已经停止维护，但是在其分支中依旧有部分人在维护。
    具体使用跳转到我的掘金https://juejin.im/post/5afd267d51882542714ff756
- 推荐模板2： django simple-ui 在github上的星数并不高，但是是基于饿了么element-ui所写的django后台模板，我还未使用
- 使用本身自带的admin模板
- 使用其他兼容多语言的后台模板

url :https://my.oschina.net/zhiyonghe/blog/1532030


-------------------


### 自带的Admin模板
> 这里介绍一下自带的admin模板。

1. admin界面汉化
在setting.py文件中设置语言
```python
# setting.py

LANGUAGE_CODE = 'zh-hans'
IME_ZONE = 'Asia/Shanghai'
```

2. Model注册
在Model.py中创建完类后，需要在admin.py中完成注册，才能在后台模板中显示，方法有二

```python
# admin.py
# 方法一
from django.contrib import admin 
from blog.models import Blog

class BlogAdmin(admin.ModelAdmin):
    list_display=('xx','xx','xx')  # 要显示的字段
    list_per_page = 50  # 每页最多显示50条
    ordering = ("-xxx")  # 设置排序，-表示降序排序
    list_editable = ['','',''] # 设置可编辑字段
    list_display_links = ('','',) # 默认情况下只能点击第一个字段进入编辑，设置多个字段可点击进入编辑
    list_filter = ('','','') # 筛选器
    search_fields  = ('','','',) # 搜索字段
    date_hierarchy = 'go_time' # 详细时间筛选

admin.site.register(Blog,BlogAdmin)

# 方法二
from djang.contrib import admin
from blog.models import Blog

@admin.register(Blog)
class BlogAdmin(admin.ModelAdmin):
    list_display=('','','')
```
------
## Django for ajax
django 前后台传递消息内容的方法有很多种,这里我们来讲一下ajax for django

```python
# 路由地址
# 项目路径下
path('',include('GraphPro.urls'))
# 应用路径下
path('echartsdemo/',demoView.as_view(),name="demo"),
# 视图层
class  demoView(View):
    def get(self,request):
        if request.GET.get('name') == 'hzj':
            return HttpResponse("ssdsdsd")
        else:
            print(2)
        return render(request,'demo.html')
    def post(self,request):
        pass
# 模板层
<body>
    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
    <div id="main" style="width: 1000px;height:1000px;" ></div>

    <input type="text" id="name" value="hzj">
    <span id="result"></span>
    <button class="button" id="click_func">点击</button>
    {{ data }}


    <script type="text/javascript">

         $(document).ready(function () {
            var name = $('#name').val();
            var data = {"name":name};
            $("#click_func").click(function () {
                $.get(
                    //请求的url
                    '{% url 'demoecharts' %}',
                    data,
                    function (ret) {
                        console.log(ret)
                    }
                )
            })
        })


    </script>
</body>
```


进入页面直接请求后台数据
```html

    <script type="text/javascript">

         $(document).ready(function () {
             var data = {"name":"hzj"};
             $.get(
                //请求的url
                '{% url 'demo' %}',
                data,
                function (ret) {
                    console.log(ret)
                }
             )
        })


    </script>
```

异步加载数据
```html
 <script type="text/javascript">

         $(document).ready(function () {
             var data = {"name":"hzj"};
             $.get(
                //请求的url
                '{% url 'demo' %}',
                data,
             ).done(function(ret){
                 console.log(ret)
             })
        })


    </script>

```
-----
> 介绍Mysql在Django中的引用
由于Django在初始化项目后，所使用的数据库是sqlite3,但为了扩展，我们这里计划使用Mysql,配置步骤如下

## 配置
```python
1.在服务器上安装Mysql数据库(Centos版)

2.在django的setting.py中引用
```

--------------------------------------------

## 错误类型总结
由于mysql与django在配置过程中会存在很多的错误，这里总结一下
#### Mysql在Centos中权限不足的错误
```text
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
```
原因是/var/lib/mysql的访问权限不足
```text
1.改变权限 chown -R mysql:mysql /var/lib/mysql   
2. 启动服务器 /etc/init.d/mysql start (会自动生成mysql.sock文件)  
3.重新启动mysql服务  /etc/init.d/mysql start
```
#### 密码类型错误
```text
1045, "Access denied for user 'root'@'122.224.83.xxx  
```
原因 密码错误，注意密码不能是字符串
```text
GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```
#### django setting 中的配置
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
#### 引用pymysql
由于MySQLdb只能使用在Python2中，在python3中已经停止了维护，所以这里我们引用pymyql的库
```python
pipenv install pymysql
# 在与setting.py同级别的__init__.py中使用
import pymysql
pymysql.install_as_MySQLdb()
```
#### pymysql版本不符合
报错内容:django.core.exceptions.ImproperlyConfigured: mysqlclient 1.3.3 or newer is required; you have 0.7.11
修改Pipenv\Lib\site-packages\django\db\backends\mysql\base.py
注释下面内容
if version < (1, 3, 3):
    raise ImproperlyConfigured("mysqlclient 1.3.3 or newer is required; you have %s" % Database.__version__)

(在Centos中，pipenv install 所生成的包 会在pipenv shell 激活环境的时候出现)


由于上面注释，还会出现的报错内容为
File "C:\Users\Administrator\PycharmProjects\untitled1\venv\lib\site-packages\django\db\backends\mysql\operations.py", line 146, in last_executed_query
将operations.py中的decode改成encode

#### mysql不能插入中文
报错内容： django.db.utils.InternalError: (1366, "Incorrect string value: '\\xE5\\xAE\\x9A\\xE6\\x97\\xB6...' for column 'name' at row 1")
解决方法: https://blog.csdn.net/tzh476/article/details/52644271
删除之前的库，创建一个新的数据库，使用utf8mb64   之前默认创建的都是latin1


#### mysql 长度的问题
django.db.utils.InternalError: (1071, 'Specified key was too long; max key length is 767 bytes')
https://blog.csdn.net/ljfphp/article/details/80406907
https://www.orcode.com/question/407126_k280b8.html

解决方法1: 升级Mysql5.6-->>Mysql5.7
centos7 下  对mysql的操作 https://blog.csdn.net/xufengzhu/article/details/81110982
https://blog.51cto.com/lisea/1941616

解决方法2 : 先使用utf8格式

----> 介绍Mysql在Django中的引用
由于Django在初始化项目后，所使用的数据库是sqlite3,但为了扩展，我们这里计划使用Mysql,配置步骤如下

## 配置
```python
1.在服务器上安装Mysql数据库(Centos版)

2.在django的setting.py中引用
```

--------------------------------------------

## 错误类型总结
由于mysql与django在配置过程中会存在很多的错误，这里总结一下
#### Mysql在Centos中权限不足的错误
```text
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
```
原因是/var/lib/mysql的访问权限不足
```text
1.改变权限 chown -R mysql:mysql /var/lib/mysql   
2. 启动服务器 /etc/init.d/mysql start (会自动生成mysql.sock文件)  
3.重新启动mysql服务  /etc/init.d/mysql start
```
#### 密码类型错误
```text
1045, "Access denied for user 'root'@'122.224.83.xxx  
```
原因 密码错误，注意密码不能是字符串
```text
GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```
#### django setting 中的配置
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
#### 引用pymysql
由于MySQLdb只能使用在Python2中，在python3中已经停止了维护，所以这里我们引用pymyql的库
```python
pipenv install pymysql
# 在与setting.py同级别的__init__.py中使用
import pymysql
pymysql.install_as_MySQLdb()
```
#### pymysql版本不符合
报错内容:django.core.exceptions.ImproperlyConfigured: mysqlclient 1.3.3 or newer is required; you have 0.7.11
修改Pipenv\Lib\site-packages\django\db\backends\mysql\base.py
注释下面内容
if version < (1, 3, 3):
    raise ImproperlyConfigured("mysqlclient 1.3.3 or newer is required; you have %s" % Database.__version__)

(在Centos中，pipenv install 所生成的包 会在pipenv shell 激活环境的时候出现)


由于上面注释，还会出现的报错内容为
File "C:\Users\Administrator\PycharmProjects\untitled1\venv\lib\site-packages\django\db\backends\mysql\operations.py", line 146, in last_executed_query
将operations.py中的decode改成encode

#### mysql不能插入中文
报错内容： django.db.utils.InternalError: (1366, "Incorrect string value: '\\xE5\\xAE\\x9A\\xE6\\x97\\xB6...' for column 'name' at row 1")
解决方法: https://blog.csdn.net/tzh476/article/details/52644271
删除之前的库，创建一个新的数据库，使用utf8mb64   之前默认创建的都是latin1


#### mysql 长度的问题
django.db.utils.InternalError: (1071, 'Specified key was too long; max key length is 767 bytes')
https://blog.csdn.net/ljfphp/article/details/80406907
https://www.orcode.com/question/407126_k280b8.html

解决方法1: 升级Mysql5.6-->>Mysql5.7
centos7 下  对mysql的操作 https://blog.csdn.net/xufengzhu/article/details/81110982
https://blog.51cto.com/lisea/1941616

解决方法2 : 先使用utf8格式