---
title: jwtForFastapi
categories:
  - fastapi
tags:
  - fastapi
abbrlink: 37e22cfa
date: 2020-03-20 13:51:06
---

learn jwt
<!-- more -->


# jwtForFastapi


## 生成验证
```py
from fastapi import Depends, FastAPI
from fastapi.security import OAuth2PasswordBearer

from pydantic import BaseModel
from typing import Optional

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/token")


@app.get("/items/")
async def read_items(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

## 创建验证模块
