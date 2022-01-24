---
title: fastapi安全
categories:
  - python
tags:
  - python
abbrlink: 5b8e78df
date: 2020-03-20 14:03:58
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [fastapi安全](#fastapi安全)
  - [OAuth2验证方式](#oauth2验证方式)
  - [OpenId Connect登录](#openid-connect登录)
  - [OpenAPI](#openapi)
  - [fastapi使用](#fastapi使用)
  - [创建一个登陆检验的功能](#创建一个登陆检验的功能)
  - [使用jwt完成token验证](#使用jwt完成token验证)

<!-- /code_chunk_output -->

learn python
<!-- more -->

# fastapi安全
既然是web框架，安全问题是必须要设计的，主要使用在用户登录和后期权限操作上


## OAuth2验证方式
Oauth2是一个协议规范，定义了几种处理身份验证和授权的方法,也就是给一些应用接口做的协议规范，这样开发者只需要遵守这个规范就能够轻松的完成
多种厂商的app接口，比如qq登录 微信登录 邮箱登录等等
Oauth2没有指定如何加密通信，它希望您的应用程序使用 HTTPS 提供服务

## OpenId Connect登录
是另一个基于 OAuth2的规范。它只是扩展了 OAuth2，指定了一些在 OAuth2中相对模糊的东西，试图使它更具互操作性。


## OpenAPI
Openapi (以前称为 Swagger)是构建 api 的开源标准(现在是 Linux 基金会的一部分)。
Fastapi是基于 OpenAPI 的。



## fastapi使用
安装库
```shell script
pip install python-multipart
```
创建OAuth2PasswordBearer
```python
from fastapi import  FastAPI,Depends
from fastapi.security import  OAuth2PasswordBearer
app = FastAPI()

# 创建一个token生成地址
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/token")

@app.get("/items/")
async def read_item(token: str = Depends(oauth2_scheme)):
    return {"token":token}
```

![2020-03-29-00-00-34](http://noback.upyun.com/2020-03-29-00-00-34.png)
启动项目，`http://127.0.0.1:8000/docs` 右上角会出现一个锁，这个就是验证
不过我们还没有给这个授权锁添加验证的功能，

## 创建一个登陆检验的功能
```python
from fastapi import  FastAPI,Depends,HTTPException,status
from fastapi.security import  OAuth2PasswordBearer,OAuth2PasswordRequestForm
from pydantic import  BaseModel
from typing import  Optional
app = FastAPI()

"""
    //大致逻辑
    1. 创建一个地址用来生成token，地址生成token由OAuth2PasswordBearer控制，会在右上角生成一把锁
    2. 创建用户模型以及模拟用户数据fake_user_db
    3. 对用户输入的密码做二次加密的过程，因为数据库后面检验的是你二次加密后的内容，并不是你输入的密码 UserInDB
    4. 
"""


# 创建一个token生成地址
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/token")

# 创建一个用户模型
class User(BaseModel):
    username :str
    email: Optional[str] = None
    full_name: Optional[str] = None
    disabled: Optional[bool] = False


class UserInDB(User):
    hashed_password: str

# 创建虚拟数据 模拟数据库
fake_user_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "fakehashedsecret",
        "disabled": False,
    },
    "alice": {
        "username": "alice",
        "full_name": "Alice Wonderson",
        "email": "alice@example.com",
        "hashed_password": "fakehashedsecret2",
        "disabled": True,
    },
    "hzj": {
        "username": "hzj",
        "full_name": "hezejun",
        "email": "hzj@example.com",
        "hashed_password": "fakehashedhzj",
        "disabled": True,
    },
}


# 模拟密码加密过程
def fake_hash_password(password: str):
    return "fakehashed" + password

# 获取用户
def get_user(db,username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)
    else:
        return None


def fake_decode_token(token):
    # 查询用户是否在数据库中，如果存在则返回user对象,如果没有则返回一个None
    user = get_user(fake_user_db,username=token)
    return user


async def get_current_user(token: str = Depends(oauth2_scheme)):
    user = fake_decode_token(token)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return user

async def get_current_active_user(current_user: User = Depends(get_current_user())):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user



# 返回token
"""
    0. 验证的数据表单由OAuth2PasswordRequestForm获得
    1.查看数据库中有没有这个用户对象
    2. 有则返回一个实例化的user对象，并且插入加密后的token
    3. 生成token进行校验，如果不相同，则返回错误
    
"""
@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    # 传入对象为username 比如我传入的username=hzj
    user_dict = fake_user_db.get(form_data.username)
    if not user_dict:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user = UserInDB(**user_dict)
    print(user)
    # 返回的内容
    # username='hzj' email='hzj@example.com' full_name='hezejun' disabled=True hashed_password='fakehashedhzj'
    # 对输入的密码进行手动二次加密，再与数据库中的密码进行比较，存在则返回username
    hashed_password = fake_hash_password(form_data.password)
    if not hashed_password == user.hashed_password:
        raise HTTPException(status_code=400, detail="Incorrect username or password")

    return {"access_token": user.username, "token_type": "bearer"}

# @app.get("/items/")
# async def read_item(token: str = Depends(oauth2_scheme)):
#    return {"token":token}


@app.get("/users/me")
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user


```

## 使用jwt完成token验证
jwt => json web
jwt的生成方式如上所示，无非他的二次加密中加入了各种各样的算法，并且还加入了令牌的过期时间，所以更加的眼睛
python调用jwt要使用到pyjwt这个库,另外我们还要用到passlib来处理密码哈希
```bash
pip install pyjwt
pip install passlib
```


