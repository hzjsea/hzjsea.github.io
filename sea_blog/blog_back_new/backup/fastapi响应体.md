---
title: fastapi响应体
categories:
  - fastapi
tags:
  - fastapi
abbrlink: 71ff961b
date: 2020-03-08 19:37:17
---

learn fastapi ⚡️
<!-- more -->

# fastapi响应体

fastapi可以指定一个响应体用来作为返回参数，呈现给用户

```py
from fastapi import FastAPI
from typing import  List
from pydantic import BaseModel


app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
    tags: List[str] = []

@app.post("/",response_model=Item)
async def root(item:Item):
    return item


```


定制不同的请求体和响应体
![2020-04-01-13-19-10](http://noback.upyun.com/2020-04-01-13-19-10.png)
```py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: str
    full_name: str = None


class UserOut(BaseModel):
    username: str
    email: str
    full_name: str = None


@app.post("/user/", response_model=UserOut)
async def create_user(*, user: UserIn):
    return user
```


过滤请求体中没有值得字段
![2020-04-01-13-34-42](http://noback.upyun.com/2020-04-01-13-34-42.png)

![2020-04-01-13-35-40](http://noback.upyun.com/2020-04-01-13-35-40.png)


```py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: str
    full_name: str = None


class UserOut(BaseModel):
    username: str
    email: str
    full_name: str = None


fake_db = {
    "hzj":{"username":"hzj","email":"1088xxx.com"},
    "qq": {"username": "hzdsj", "email": "1088xxxds.com"},
    "zz": {"username": "hzjdsds", "email": "1088xxx.dscom"},
    "j": {"username": "dsdhzj", "email": "1088xxxsds.com"},
}

@app.post("/user",response_model=UserOut)
async def create_user(*,user: UserIn):
    # 这里虽然返回的是user 但是他会根据UserOut返回response
    return user


@app.get("/user/{user_id}", response_model=UserOut,response_model_exclude_unset=True)
async def get_user(user_id :str=None ):
    if user_id in fake_db:
        return fake_db[user_id]
```

还有其他的一些过滤方法
![2020-04-01-13-36-31](http://noback.upyun.com/2020-04-01-13-36-31.png)