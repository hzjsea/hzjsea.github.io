---
title: python使用pyjwt模块
categories:
  - python
tags:
  - python
abbrlink: 94348b80
date: 2020-03-21 18:01:32
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [python使用pyjwt模块](#python使用pyjwt模块)

<!-- /code_chunk_output -->
<!-- more -->

# python使用pyjwt模块
https://pyjwt.readthedocs.io/en/latest/api.html

安装pyjwt模块
```python
pip install pyjwt
```


案例
```py
import  jwt

Payload = {
    "username":"hzj"
}

salt = "oooooo"

# 默认加密
token = jwt.encode(
    payload=Payload,
    key=salt,
    algorithm="HS256"
)

# 加盐解密
token = jwt.decode(token,key=salt,algorithm="HS256")
print(token)

token = jwt.encode(
    payload=Payload,
    key=salt,
    algorithm="HS256"
)
# 不加盐解密
res = jwt.decode(token,verify=False)
print(res)
```

payload中固定的参数 ，有官方指定的key，程序在解密的时候会根据key的Value判断是否合法。这些key有

- “exp”: 过期时间
- “nbf”: 表示当前时间在nbf里的时间之前，则Token不被接受
- “iss”: token签发者
- “aud”: 接收者
- “iat”: 发行时间
我们一般设置 过期时间，发行时间，接收者。我们来分别解释这些key