---
title: jwt原理与实战应用
categories:
  - jwt
tags:
  - jwt
abbrlink: e8b3a2c2
date: 2020-01-20 13:07:46
---



learn jwt
<!-- more -->
# jwt原理


- 基于传统的token认证
用户登陆，服务端给返回token,并将token保存在服务端，以后用户访问时，需要携带token，服务端获取token后，再去数据库中获取token认证校验

- jwt形式的token认证
用户登录，服务端给用户返回一个token(服务端不保存)，以后用户再来访问，需要携带token，服务端获取token后，再做token认证

<font color='优点'>不用在服务端保存token</font>


## jwt实现过程

- 用户提交用户名和密码给服务器，如果登录成功，使用jwt对账号密码进行二次加密并且生成一个token值，将token返回给用户，简单的来说这个token
- 用户每次登录的时候，都必须输入用户名和密码，在登录过程中，程序依旧对他进行二次


## jwt-token组成
由三段字符串组成，并且用.连接

### 加密过程
- 第一段字符串 header 内部包含算法/token类型.
json 转换成字符串，然后做base64url加密(base64加密)
```bash
{
    "alg": "HS256", # 算法
    "typ": "JWT", # 类型
} 
```

- 第二段字符串payload 自定义值
json转换成字符串，然后做base64url加密(base64加密)
```bash
{
    "id": "12343242",
    "name": "hzj",
    "exp": 323212423 # 超时时间
} 
```

- 第三段字符串:
拼接1,2部分内容，并对其进行HS256加密 + 加盐(加随机字符串)
对HS256加密后的密文再做base64url加密

返回给用户，用户下次访问带着jwt-token来访问

### 解密过程

用户第二次访问，带着jwt-token来访问，后端需要对token进行验证

- 获取token，根据.对token进行切割
- 对第二段进行base64url解密，并获取payload信息
```bash
{
    "id": "1234343",
    "name": "hzj",
    "exp": 432423 # 超时时间
} 
```
- 第三步: 把第一二段拼接，再次HS256加密后对base64url解密后的内容进行比较(认证通过)
在

## jwt实现原理
- 底层使用pyjwt
- drf封装库使用的是django-estframework-jwt


### pyjwt使用
```bash
# 安装
pip install pyjwt
```
生成token
```py
# 配置
import  jwt
import datetime
from django.conf import  settings
# 生成token
def create_token(payload,timeout=1):

    # 随机符号，这里写了该项目的secret_key
    salt = settings.SECRET_KEY

    # 构造header
    headers = {
        'typ': 'jwt',
        'alg': 'HS256',
    }

    # 构造payload
    payload = {
        'user_id': "ud",
        'username': "username",
        'exp': datetime.datetime.utcnow() + datetime.timedelta(minutes=timeout)
    }

    token = jwt.encode(payload=payload, key=salt, algorithm="HS256", headers=headers).decode('utf-8')

    return token
```
验证token
```py
from rest_framework.authentication import BaseAuthentication
import  this
import jwt
import  datetime
from  rest_framework.response import  Response
from django.conf import  settings
from rest_framework import exceptions
salt = settings.SECRET_KEY



class JwtQueryParamsAuthentication(BaseAuthentication):

    def authenticate(self, request):
        # 获取token 并判断token的合法性
        token = request.query_params.get("token")


        # 1。切割
        # 2. 解密第二段/判断过期
        # 3. 验证第三段合法性

        playload = None
        msg = None

        try:
            payload = jwt.decode(token,salt,True)
        except exceptions:
            msg = 'token已经失效'
        except jwt.DecodeError:
            msg = "token认证失败"
        except jwt.InvalidTokenError:
            msg = "非法的token"
        if not playload:
            pass
        else:
            return (playload,msg)

```
views编写
```py
# views编写 
from api.extensions.auth import JwtQueryParamsAuthentication
class ProOrderView(ApiView):
    authentication_classes = [JwtQueryParamsAuthentication,] 
```

### djangorestframework-jwt使用
djangorestframework-jwt本质是调用pyjwt实现的
```bash
# 安装
pip install djangorestframework-jwt

```



## 地址
jwt认证
http://yangjianhua.me/2020/01/django-drf-8/

https://shuke163.github.io/2019/02/24/Django-REST-framework-API%E8%AE%A4%E8%AF%81-%E5%8C%85%E5%90%ABJWT%E8%AE%A4%E8%AF%81/
https://www.cnblogs.com/ruhai/p/11311852.html
https://www.cnblogs.com/chichung/p/9967325.html