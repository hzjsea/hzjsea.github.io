---
title: fastapi自建文件分享服务器
categories:
  - fastapi
tags:
  - fastapi
abbrlink: 34e8397f
date: 2020-05-05 13:25:25
---



<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [fastapi自建文件分享服务器](#fastapi自建文件分享服务器)
  - [旧版使用http服务开启文件分享](#旧版使用http服务开启文件分享)
  - [新版使用fastapi搭建文件分享服务器](#新版使用fastapi搭建文件分享服务器)

<!-- /code_chunk_output -->

<!-- more -->


# fastapi自建文件分享服务器


## 旧版使用http服务开启文件分享
```bash
pip install httpserver
python3 -m http.server
```

但是这样做并不安全，并且只能下载一个文件，如果文件比较大，在下载的过程中其他人无法下载其他文件。
当前文件夹下面的所有文件都会出现在网页上，容易导致敏感数据泄露。如果你只想让别人下载其中一个文件，你需要单独给这个文件创建一个文件夹，并在这个文件夹里面执行命令。
这个简单的网络服务不稳定。

## 新版使用fastapi搭建文件分享服务器
fastapi 基于starlette 开发。而 starlette里面有一个返回类型叫做FileResponse。使用它，可以非常方便地返回文件。我们来看看代码。
```bash
pip install uvicorn 
pip install fastapi
pip install aiofiles
```


```python
import os
from fastapi import FastAPI
from starlette.responses import FileResponse


@app.get('/record/{filename}')
def get_record(filename: str):
    path = os.path.join('output', filename)
    ifnot os.path.exists(path):
        return {'success': False, 'msg': '文件不存在！'}
    response = FileResponse(path)
    return response
```
这样，你想分享具体哪个文件，你就构造一个 URL：http://ip:8888/record/文件名发给别人。别人直接访问这个 URL 就能来下载对应的文件了。只要对方不知道其他文件的文件名，就无法看到或者下载其他文件。