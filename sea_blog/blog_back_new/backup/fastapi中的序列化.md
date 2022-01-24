---
title: python中的序列化
categories:
  - python
tags:
  - python
abbrlink: 3d479224
date: 2020-03-20 14:03:58
---

learn python
<!-- more -->

# fastapi中的序列化
在正常情况下，如果框架没有提供任何的的序列化工具的话，那就要使用json自己去写序列化，而且如果数据库中有时间等内容的话，还要自定义json的解析，才能正常解析时间
在fastapi中获得他提供了一个jsonable_encoder的功能来解决

```python
from fastapi import  FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel


from datetime import  datetime


"""
    desc: 介绍在fastapi中如何通过jsonable_encoder来序列化json数据
"""

## 创建一个虚拟的数据库，假设是数据库中返回的json对象
get_db = {}

## 创建一个数据类型
class Item(BaseModel):
    title : str
    timestamp = datetime
    desc: str = None


app = FastAPI()

@app.put("/items/{id}")
def update_item(id: str,item: Item):
    # 将要更新的json传入函数
    # 序列化json对象
    # 更新json对象
    json_code = jsonable_encoder(item)
    fake_db[id] = json_code
```


在这个示例中，它将 Pydantic 模型转换为 dict，将日期时间转换为 str。