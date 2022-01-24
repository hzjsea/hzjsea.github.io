---
title: swagger-ui使用
categories:
  - code
tags: 
  - vue
  - code
abbrlink: a2d15b23
date: 2020-01-13 11:29:18
---

<!--more-->
# SwaggerUi使用



## django搭建swagger-ui
django2.0之前使用django-rest-swagger
django2.0之后使用drf-yasg
这里主要记录一些使用drf-yasg安装的过程

## Qucik-Start
```python
# 安装
yum install drf-yasg

# setting设置
INSTALLED_APPS  = [
    ... 
   ' drf_yasg '，
    ... 
]
# urls.py设置
...
from rest_framework import permissions
from drf_yasg.views import get_schema_view
from drf_yasg import openapi

...

schema_view = get_schema_view(
   openapi.Info(
      title="Snippets API",
      default_version='v1',
      description="Test description",
      terms_of_service="https://www.google.com/policies/terms/",
      contact=openapi.Contact(email="contact@snippets.local"),
      license=openapi.License(name="BSD License"),
   ),
   public=True,
   permission_classes=(permissions.AllowAny,),
)

urlpatterns = [
   url(r'^swagger(?P<format>\.json|\.yaml)$', schema_view.without_ui(cache_timeout=0), name='schema-json'),
   url(r'^swagger/$', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
   url(r'^redoc/$', schema_view.with_ui('redoc', cache_timeout=0), name='schema-redoc'),
   ...
]
```
生成地址
- A JSON view of your API specification at /swagger.json
- A YAML view of your API specification at /swagger.yaml
- A swagger-ui view of your API specification at /swagger/
- A ReDoc view of your API specification at /redoc/

## 自定义其他验证请求方法
```bash
# setting.py
SWAGGER_SETTINGS = {
    'SECURITY_DEFINITIONS': {
        # 默认验证
        'basic': {
            'type': 'basic'
        },
        # 在header中以Authorization验证TOKEN
        'Bearer': {
            'type': 'apiKey',
            'name': 'Authorization',
            'in': 'header'
        }
    }
} 
```
以密码的形式输出，则为basic
另外在UI界面输入,需要加入Bearer标识
![2020-02-12-15-55-50](http://img.noback.top/2020-02-12-15-55-50.png)

## 网址
官方网址
https://swagger.io/tools/swagger-ui/
drf-yasg gitbook文档
https://drf-yasg.readthedocs.io/en/stable/
drf-yasg gitbook地址
https://github.com/axnsan12/drf-yasg