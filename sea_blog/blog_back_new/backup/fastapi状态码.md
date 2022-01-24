---
title: fastapi状态码
categories:
  - fastapi
tags:
  - fastapi
abbrlink: 92c8b31c
date: 2020-04-01 13:41:30
---

learn fastapi
<!-- more -->


# fastapi状态码

fastapi中返回状态码的两种形式
```py

from fastapi import FastAPI,status
app = FastAPI()
@app.get("/",status_code = status.HTTP_200_OK)
async  def root():
    return {"hello":"world"}

```
```py
from fastapi import FastAPI,status
app = FastAPI()
@app.get("/",status_code = 200)
async  def root():
    return {"hello":"world"}
```


