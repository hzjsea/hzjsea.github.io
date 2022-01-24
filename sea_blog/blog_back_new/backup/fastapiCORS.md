---
title: fastapi CORS
categories:
  - fastapi
tags:
  - fastapi
abbrlink: b4ae913
date: 2020-04-01 13:41:30
---

learn fastapi
<!-- more -->


# 跨域请求

https://fastapi.tiangolo.com/tutorial/cors/
```py
from fastapi import  FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()


origins = [
    "http://localhost.tiangolo.com",
    "https://localhost.tiangolo.com",
    "http://localhost",
    "http://localhost:8080",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,  # 应该允许进行跨域请求的来源列表
    allow_credentials=True, # 表示跨域请求应支持cookie
    allow_methods=["*"],  # 跨域请求应允许的HTTP方法列表 默认为get * 表示所有
    allow_headers=["*"], # 指出应使浏览器可以访问的所有响应标头
)

@app.get("/")
async def main():
    return {"message": "Hello World"}
```
