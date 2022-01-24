---
title: django-jwt 
categories: 
   - django
   - jwt
tags:
   - jwt
abbrlink: 8d17cdf0
date: 2019-12-29 20:48:40
---


Introducing drf
how to use it 

<!--more-->


## 使用drf-jwt

```python
# 安装jwt
source /bin/active
pip install djangorestframework-jwt 
# 配置setting.py
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ),
}

# 配置url.py
from rest_framework_jwt.views import obtain_jwt_token
urlpatterns = [
    '',
    # ...

    url(r'^api-token-auth/', obtain_jwt_token),
]

# 测试
curl -X POST -d "username=admin&password=password123" http://localhost:8000/api-token-auth/
# 返回json
curl -X POST -H "Content-Type: application/json" -d '{"username":"admin","password":"password123"}' http://localhost:8000/api-token-auth/
# 定义验证方式
curl -H "Authorization: JWT <your_token>" http://localhost:8000/protected-url/ 
```
![2020-01-13-17-45-18](http://img.noback.top/2020-01-13-17-45-18.png)

   
```python
//settings.py
# jwt token 有效期
import datetime
JWT_AUTH = {
    # jwt 验证码有效期
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=365),
    # jwt 验证头部
    'JWT_AUTH_HEADER_PREFIX': 'Bearer',
}
```

### django-jwt 自定义用户认证

```bash
//settings.py
AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'guardian.backends.ObjectPermissionBackend',
    'users.views.CustomBackend',
)

JWT_AUTH = {
		'JWT_AUTH_HEADER_PREFIX': 'JWT',
		'JWT_EXPIRATION_DELTA': datetime.timedelta(days=7)
}
//users views.py
from django.db.models import Q
from django.contrib.auth.backends import ModelBackend
from django.contrib.auth import get_user_model

User = get_user_model()
# Create your views here.

class CustomBackend(ModelBackend):
    """
    自定义用户验证
    """
    def authenticate(self, request, username=None, password=None, **kwargs):
        try:
            user = User.objects.get(Q(username=username) | Q(mobile=username))
            if user.check_password(password):
                return user
        except Exception as e:
            return None 
```




https://zhuanlan.zhihu.com/p/30524397
https://jpadilla.github.io/django-rest-framework-jwt/#security

## vue中使用token验证
https://blog.csdn.net/baiqiangdoudou/article/details/100174505

```bash
curl -H "Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoxNTc0OTMwMjYyLCJlbWFpbCI6IjEwOTc2OTAyNjhAcXEuY29tIn0.cxzZhCOStN_w9OpVcq7rO-bcA_vWA172DtQaNdeGF3A" http://localhost:8000/switch/ 
```

<font color='red'>为什么要用token 与其他的差别</font>