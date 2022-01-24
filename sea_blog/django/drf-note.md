---
title: drf-note
categories:
  - 编程
  - django
abbrlink: 437192d0
date: 2019-12-29 20:48:11
---
## drf
如何使用django-rest-framework

1.首先下载
```python
pip3 install django-rest-framework
```
2.应用
在setting中添加
```python
INSTALLED_APPS = (
    ...
    'rest_framework',
)
```
3.在drf中使用登陆登出的功能
```python
urlpatterns = [
    ...
    url(r'^api-auth/', include('rest_framework.urls'))
]
```


---------

## 序列化
创建model 与 django类似
创建serializer 序列化model数据，以json的形式展示  （serializer modelserialzier）
创建view  与django类似

使用httpie查看

pip install httpie
http http://localhost:8000/snippets/

