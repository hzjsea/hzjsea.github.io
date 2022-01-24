---
title: python使用passlib模块
categories:
  - python
tags:
  - python
abbrlink: d6289d7f
date: 2020-03-30 11:30:14
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [python使用passlib模块](#python使用passlib模块)
- [passlib实现加密解密过程](#passlib实现加密解密过程)

<!-- /code_chunk_output -->
<!-- more -->


# python使用passlib模块

文档地址: https://passlib.readthedocs.io/en/stable/

python中的passlib模块主要是用来加密解密的，它提供了30多种密码哈希算法的跨平台实现，以及用于管理现有密码哈希的框架。从验证/ etc / shadow中的哈希值到为多用户应用程序提供完整强度的密码哈希值，它可用于多种任务


```python
>>> # import the hash algorithm
>>> from passlib.hash import pbkdf2_sha256

>>> # generate new salt, and hash a password
>>> hash = pbkdf2_sha256.hash("toomanysecrets")
>>> hash
'$pbkdf2-sha256$29000$N2YMIWQsBWBMae09x1jrPQ$1t8iyB2A.WF/Z5JZv.lfCIhXXN33N23OSgQYThBYRfk'

>>> # verifying the password
>>> pbkdf2_sha256.verify("toomanysecrets", hash)
True
>>> pbkdf2_sha256.verify("joshua", hash)
False
```
这里面传入的值密码应该是unicode类型的


passlib库主要是集成了各种的加密和解密算法

# passlib实现加密解密过程
```py
from passlib.context import CryptContext

# 定义算法
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# 对密码进行hash加密
def gen_hashpasswd(newpassword):
    hashpasswd = pwd_context.hash(newpassword)
    return hashpasswd

# 检验哈希
def check_hash(newpassword,hashpasswd):
    return pwd_context.verify(newpassword,hashpasswd)

if __name__ == '__main__':
    newpassword = "scret"
    hashpasswd = gen_hashpasswd(newpassword)

    flag = check_hash("scret","$2b$12$y.NjTvOEkuJtLtr7.dMXguRjiAcuqNb8bOkJIf80v/knno/vF6mGy")
    print(flag)


```