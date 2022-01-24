---
title: fastapi中更新数据
categories:
  - python
tags:
  - python
abbrlink: 58b178db
date: 2020-03-20 14:03:58
---

learn python
<!-- more -->

# fastapi中利用json_code更新数据
更新数据使用到的http方法是put ，大致就是利用jsonable_encoder来序列化我们传过去的dict数据，然后交互数据库更新就Ok了

## 使用put来更新

```python
from typing import  List
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


# 创建模型类
class Item(BaseModel):
    name: str = None
    desction: str = None
    price: float = None
    tax : float = None
    tags: List[str] = []





# 模拟数据库中的数据
items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


# 传入参数
@app.get("/")
async def root():
    return {"desc":"hello world"}

# 传入参数
@app.get("/items/{item_id}",response_model=Item)
async def read_item(item_id: str):
    ## 这里只是模拟了数据库中的内容，实际上这里要多一步数据库提取数据的过程
    return items[item_id]



# 更新参数
@app.put("/items/{item_id}",response_model=Item)
async def update_item(item_id:str,item:Item):
    update_item_encode = jsonable_encoder(item)
    items[item_id] = update_item_encode
    return update_item_encode
```

这里response_model有什么用，暂时还不知道，官方文档是这样写的，后面还有带研究


## 使用Patch部分更新

```python

from typing import  List
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()


# 创建模型类
class Item(BaseModel):
    name: str = None
    desction: str = None
    price: float = None
    tax : float = None
    tags: List[str] = []




# 模拟传入的数据
items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


# 传入参数
@app.get("/")
async def root():
    return {"desc":"hello world"}




# 获取参数
@app.get("/items/{item_id}",response_model=Item)
async def read_item(item_id: str):
    return  items[item_id]

# @ 更新部分内容
@app.patch("/items/{item_id}",response_model=Item)
async  def update_item(item_id: str,item: Item):
    # 从数据库中拿出要修改的数据
    store_data = items[item_id]
    print(store_data)
    # 返回一个Item的对象
    store_model = Item(**store_data)
    print(store_model)
    print(type(store_model))


    # 生成新的字典并返回
    update_data = item.dict(exclude_unset=True)
    print(update_data)
    ## 更新部分数据
    updated_item = store_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return items[item_id]



```