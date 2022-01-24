---
title: fastapi依赖注入
categories:
  - python
tags:
  - python
abbrlink: 316bf58e
date: 2020-03-20 14:03:58
---

learn python
<!-- more -->


# fastapi依赖注入

fastapi中的依赖注入，是为了减少同类型代码的出现，将重复动作归为一类，你可以把它看成简陋的接口实现
在fastapi中，他被称之为依赖

```python
from typing import  List
from fastapi import FastAPI
from pydantic import BaseModel
from fastapi import  Depends



app = FastAPI()


# 创建模型类
class Item(BaseModel):
    name: str = None
    desction: str = None
    price: float = None
    tax : float = None
    tags: List[str] = []

"""
    依赖注入
"""
# 创建一个需要重复的动作
# 定义了一个动作，返回输入的三个参数
async def repeat_action(q: str = None, skip: int = 0,limit: int = 100):
    return {"q":q,"skip":skip,"limit":limit}




# 传入参数
@app.get("/")
async def root():
    return {"desc":"hello world"}

# 执行相同的动作
@app.get("/items/")
async def read_item(commons: dict = Depends(repeat_action)):
    return commons
@app.get("/uesrs/")
async def read_item(commons: dict = Depends(repeat_action)):
    return commons
```

另外 依赖的使用没有强制要求使用async也可以使用普通的def
> 您可以在常规的 def 路径操作函数中使用 async def 声明依赖关系，或者在 async def 路径操作函数中使用 def 声明依赖关系，等等。


## 高级的依赖注入
之前的依赖注入只是返回了一个dict字典，并不能对其中内容做一些处理，高级的依赖注入支持返回一个类，在类中可以完成对参数的操作
```python

from typing import  List
from fastapi import FastAPI
from pydantic import BaseModel
from fastapi import  Depends

app = FastAPI()


## 创建一个类
class repeat_action(object):
    def __init__(self,
                 name: str,
                 age: int = None,):
        self.name = name
        if age is None:
            self.age = age
        else:
            self.age = age + 1

@app.get("/")
async def root():
    return {"hello world":"helloworld"}
@app.get("/age/")
async def read_info(commons: repeat_action=Depends(repeat_action)):
# async  def read_info(commons: repeat_action=Depends()) 你还可以这样写
    res = {}
    if commons.age is not None:
        res = None
        return {"is birth":res}
    else:
        res = {"res":"{} is not birth,age is {}".format(commons.name,commons.age)}
        return {"is birth":res}
```


## 子级依赖注入
既然依赖已经可以用类去申明，那他也有子级别依赖注入
```python
from typing import  List
from fastapi import FastAPI
from pydantic import BaseModel
from fastapi import  Depends

app = FastAPI()

## 创建子级依赖
async  def sub_action(q: str =None):
    q = "my type is sub action for {}".format(q)
    return q

## 创建一个类
class repeat_action(object):
    def __init__(self,
                 name: str,
                 age: int = None,
                 desc: str = None,):
        self.name = name
        if age is None:
            self.age = age
        else:
            self.age = age + 1
        self.desc = Depends(sub_action(desc));
@app.get("/age/")
async def read_info(commons: repeat_action=Depends(repeat_action)):
    res = {}
    if commons.age is not None:
        res = None
        return {"is birth":res}
    else:
        res = {"res":"{} is not birth,age is {}".format(commons.name,commons.age)}
        # return {"is birth":res}
        if commons.desc is not None:
            res['desc'] = commons.desc
        else:
            return {"is birth ":res}

```

> 这里官方文档写了一个use_cache=False是什么用作 后面在解决

补充，fastapi中，如果你在一个相同的请求路径中多次使用相同的依赖项，fastapi只会使用一次依赖注入 因为他会把第一次依赖注入的结果放到缓存中，
中如果需要使用多次调用，可以在调用的过程中使用`use_cache=False`
```python
async def needy_dependency(fresh_value: str = Depends(get_value, use_cache=False)):
    return {"fresh_value": fresh_value}
```

## 依赖注入的其他使用方法
有时候只需要在Url调用的时候执行以下依赖注入的方法就可以了，不需要返回值，这时候就可以用到`Depends` 方法与Depends相同，但是不能返回结果
传入多个依赖注入
```python
@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```
