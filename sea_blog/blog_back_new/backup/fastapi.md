---
title: fastapi基础
categories:
  - fastapi
tags:
  - fastapi
abbrlink: e2882394
date: 2020-03-08 19:37:17
---

learn fastapi ⚡️
<!-- more -->
# 使用fastapi框架

## start
```py 
# 引入包
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class itemmodel(BaseModel):
    name: str
    price: float
    is_offer: bool = None




# 定义根目录 方法为get 返回helloworld
@app.get("/")
def get_first():
    return {
        "hello": "world"
    }


# 定义耳机目录以及输入参数
# 定义方法，请求参数类型
@app.get("/items/{item_id}")
def get_second(item_id: int, itemmodel: itemmodel):
    return {
        "item_id": item_id,
        "item": itemmodel.name
    }

# 请求put
@app.put("/items/{item_id}")
def update_item(item_id: int, itemmodel: itemmodel):
    return {
        "item_name": itemmodel.name,
        "item_id": item_id
    }

```
运行项目
这里我所创建的py文件是server.py,所以启动方式是这样的 --reload的作用是，当代码变更的时候,服务器会重新开启
```py 
uvicorn server:app --reload
```
![2020-03-08-00-46-06](http://noback.upyun.com/2020-03-08-00-46-06.png)


<font color='red'>交互式api文档,fastapi具有良好的api文档，自带swagger接口文档</font>

访问地址 http://127.0.0.1:8000/docs
自定义api文档地址  http://127.0.0.1:8000/redoc
你可以在更开放的端口上看到他  http://127.0.0.1:8000/openapi.json


## study
几种常用的http请求
- get 请求
- put 更新
- delete 删除
- post 添加 
增删改查

### 路径参数  Path
```py
from fastapi import FastAPI
app = FastAPI()
@app.get("/")
# 使用异步请求
async def root():
    return {
        "message":"hello world"
    }

# 路径参数
@app.get('/items/{item_id}')
async def get_single_item(item_id):
    return {
        "item_id" : item_id 
    }

# 路径参数+ 参数类型限制
@app.get('/items/{item_str}')
async def get_str_item(item_str:str):
    return {
        "item_str": item_str
    }

# 参数类型限制表示，你在url中输入的内容是string类型的，他会自动帮你转换其他类型的，比如说这里的 string - > int 
@app.get('/items/{item_number}')
async def get_number_item(item_number:int):
    return {
        "item_number": item_number
    }


# 路径参数的默认值
@app.get('/items/{item_number}')
async def get_number_item(item_number: int=1):
    message = "这是默认的"
    if item_number == 1:
        message = "你输出了默认的参数值 1"
    else:
        message = "你输出了其他的参数{}".format(item_number)

    return {
        "message":message

    }


```

#### 路径参数的顺序
fastapi的路径参数如果仅仅只有上面的内容，那么设计是不合理的，比如当你创建如下两个路径
```py

from fastapi import  FastAPI

app  = FastAPI()

static_url = "http://127.0.0.1:8000{0}"

@app.get("/")
async def root():
    return {
        "path1":static_url.format("/name/me"),
        "path":static_url.format("/name/{you can input something}")
    }
@app.get("/name/me")
async def return_name():
    return {"name":"this name is me "}

@app.get("/name/{name}")
async def return_name(name:str):
    return {"name":"you  input name is {}".format(name)}
```
这里在定制了两个路径参数其中第一个`/name/me` 返回this name is me  第二个`/name{name}` 返回 you input name is {name}
而fastapi的执行顺序是从上到下的，如果我们把第二个接口放到第一位，当我们输入`me`的时候，程序无法判断我们使用的是哪个接口

<font color='red'>因此，一般有实例的api需要放在最前面</font>


### 枚举 预定义值

<font color='red'>类似于选项框类的，需要预定义值得时候，可以使用枚举</font>

```py
from fastapi import FastAPI
from enum import Enum

app = FastAPI()

class PeopleModel(str,Enum):
    name = 'hzj',
    age = 12,
    text = 'like PeopleModel'

@app.get('/items/{model_name}')
def get_people(model_name: PeopleModel):
    if model_name == PeopleModel.name:
        return {"model_name":model_name,"input_message":'show PeopleModel.name'}
    if model_name == PeopleModel.age:
        return {"model_name":model_name,"input_message":'show PeopleModel.age'}
    if model_name == PeopleModel.like:
        return {"model_name":model_name,"input_message":'show PeopleModel.like'}
```
在python3.4之后其实就已经支持枚举类型了，和平常的枚举使用情况一样，没什么差别

### 查询参数 Query
和路径参数不同的是，我们可以在方法中输入请求方法中没有出现过的参数，这些参数会被解释成查询参数
```py
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

# 这里可以使用查询参数
# 请求形式 http://127.0.0.1:8000/items/?skip=0&limit=1
# or http://127.0.0.1:8000/items/
# 步进的形式，对列表做切割
@app.get('/items/')
async def read_item(skip: int = 1,limit: int = 2):
    return fake_items_db[skip:skip+limit]

# 这里只有路径参数
# 请求形式 http://127.0.0.1:8000/items/1/2
@app.get('/items/{skip}/{limit}')
async def read_item(skip: int ,limit: int):
    return fake_items_db[skip:skip+limit]
```

### 查询参数中的可选与必选
在定义查询参数的时候，我们定义三种查询参数的形式
1. 可选参数,即默认为None
2. 必选参数,即传入参数，且初始不为None
3. 必选参数+默认值
```py
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: 03_queryParameters.py
@time: 2020/3/15 12:13 上午
'''

from fastapi import  FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"speak":"helloworld"}

#无查询参数
@app.get("/name/")
async def get_name():
    return {"name":"hzj"}

# 查询可选参数,默认为None
@app.get("/name/")
async def  get_age(age :int=None ):
    message = "his age is {}"
    if age is None:
        message = "he dont have age"
    else:
        message = message.format(age)
    return {
        "message":"his name is and {}".format(message)
    }

#查询必选参数
@app.get("/name/")
async def  get_age_one(age :int ):
    return {
        "message":"his name is  and his age is {0}".format(age)
    }

# 查询必须参数默认
@app.get("/name/")

async def  get_age_one(age :int=20 ):
    return {
        "message":"his name is and his age is {0}".format(age)
    }
```
无查询参数时候，只是简单的get路径参数就可以了
![2020-03-31-10-16-27](http://noback.upyun.com/2020-03-31-10-16-27.png)

有查询参数的时候，如果有默认值None或者默认为其他内容，则为可选查询参数 
![2020-03-31-10-19-48](http://noback.upyun.com/2020-03-31-10-19-48.png)

有查询参数，没有默认值，只定义了一个类型，则为必选参数
![2020-03-31-10-20-32](http://noback.upyun.com/2020-03-31-10-20-32.png)




### 请求体
除了get方法 fastapi还可以创建实例，请求体可以对应数据库里面查询出来的内容，以json的形式出现
```python
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: 04_request_body.py
@time: 2020/3/15 12:30 上午
'''

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None
app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item
```

### 查询参数和字符串验证
Query可以对查询参数做值上面的限定，当你查询某一个参数的时候，如果这个参数没有满足Query中的限制内容就无法查询

如下有这么一个程序，输入名字查询结果
```py 
from fastapi import FastAPI

app = FastAPI()

name_server = [
    {"name":"hzj"},
    {"name":"xxx"},
    {"name":"yyy"},
    {"name":"qqq"},
    {"name":"fff"},
]
@app.get('/user/')
def get_user(name: str):
    for name_s in name_server:
        if name in name_s.values():
            return name_s
        else:
            return {"msg":"不存在你要查询的内容"}

# url  http://127.0.0.1:8000/user/?name=hzj
# res {name: "hzj"}
```
但是现在我们要把name这个查询参数做字符上的限制
```py
from fastapi import FastAPI
from fastapi import Query
app = FastAPI()

name_server = [
    {"name":"hzj"},
    {"name":"xxx"},
    {"name":"yyy"},
    {"name":"qqq"},
    {"name":"fff"},
]

@app.get('/user/')
def get_user(name: str = Query(None,max_length=3,min_length=1,)):
    for name_s in name_server:
        if name in name_s.values():
            return name_s
        else:
            return {"msg":"不存在你要查询的内容"}

## 其中None为查询参数name 的默认值
```

<font color='red'>比如现在我们用上面的代码中设置请求的查询参数最大长度是1个字符，使用swagger再次查询hzj这个名字是否存在于name_server中的时候，会出现报错</font>

![2020-03-31-10-30-41](http://noback.upyun.com/2020-03-31-10-30-41.png)

Query的约束和后面的Path约束的参数一样，他们来自同一个类，参数为
```python
  alias: str = None,
    title: str = None,
    description: str = None,
    gt: float = None,
    ge: float = None,
    lt: float = None,
    le: float = None,
    min_length: int = None,
    max_length: int = None,
    regex: str = None,
    deprecated: bool = None,
    **extra: Any,
```


### None与...以及default
虽然个人觉得这个...的设定有点奇葩，但是后来仔细看了一下文档，发现是有用处的
之前我们讲过在查询参数中的三种传入形式
- 在查询的时候，默认查询参数为None或者其他内容，只要有默认值   可选查询参数
- 在查询的时候，默认查询参数为某一类型  无默认值   必选查询参数
- 无查询参数

这是之前总结的

<font color='red'>现在要为这个查询参数在格式上或者说插入值上做一些限定，用的是Query这个关键字，但同样的在插入的形式上也要做限定，这个时候就要用到...这个关键字了</font>

- 在查询的时候，默认查询参数为None或者其他内容，只要有默认值，  可选查询参数
```py
@app.get('/user/')
def get_user(name: str = Query(default="hzj",max_length=1,min_length=1,)):
```
- 在查询的时候，默认查询参数为某一类型 无默认值  必选查询参数
```py
@app.get('/user/')
def get_user(name: str = Query(...,max_length=1,min_length=1,)):
```

<font color='red'>这些关键字在Path路径查询参数中同样适用</font>


### 路径参数和数值验证
在路径参数中，同样也可以像Query一样做参数上的约束
```py
from fastapi import FastAPI, Path, Query
app = FastAPI()
@app.get("/items/{item_id}")
async def read_items(
    item_id: int = Path(..., title="The ID of the item to get"),
    q: str = Query(None, alias="item-query"),
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```
Path用于限制路径参数 ...表示该参数是必填的

如果你想要让q也称为必填的参数，你可以这样写
```python
from fastapi import FastAPI, Path, Query
app = FastAPI()
@app.get("/items/{item_id}")
async def read_items(
    item_id: int = Path(..., title="The ID of the item to get"),
    q: str = Query(..., alias="item-query"),
):
    results = {"item_id": item_id}
    results.update({"q": q})
    return results
```
这和我们之前写的必须查询参数是一样的
```python
    q: str = Query(..., alias="item-query"),
    # or
    q: str 
```
但是这里我们用到了Query可以做更多的东西

当然你也可以这样写
```python
from fastapi import FastAPI, Path, Query
app = FastAPI()
@app.get("/items/{item_id}")
async def read_items(
    q: str,item_id: int = Path(..., title="The ID of the item to get"),
):
    results = {"item_id": item_id}
    results.update({"q": q})
    return results
```

<font color='red'>数值验证</font>



```py
from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(..., title="The ID of the item to get", ge=0, le=1000),
    q: str,
    size: float = Query(..., gt=0, lt=10.5)
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```


### *
这边这个`*`做了什么动作

<font color='red'>文档里面说这个*没有做任何的动作。只是声明了后面所有传入的参数都为键值对的形式传入</font>

> 如果要声明的q查询参数不带Query默认值，也不带任何默认值，并且要item_id使用设置路径参数Path，并且使用不同的顺序，则Python对此有一些特殊的语法。
> 通过*，作为函数的第一个参数。
> Python不会对此做任何事情*，但是它将知道以下所有参数都应称为关键字参数（键-值对），也称为kwargs。即使它们没有默认值。

之前在vscode中试着把`*`去掉，但是没什么反应，后面尝试Pycharm实例一下，发现如果去掉`*`,就会报一个无法识别的错误
![2020-03-31-13-05-30](http://noback.upyun.com/2020-03-31-13-05-30.png)


### 模型类
类比django中的django-model 在fastapi中我们还缺少model类，于是我们借用了Pydatic中的模型类，来完成模型的实现
```py 
from fastapi import FastAPI, Path
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None


@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int = Path(..., title="The ID of the item to get", ge=0, le=1000),
    q: str = None,
    item: Item = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if item:
        results.update({"item": item})
    return results
```
return values
![2020-03-08-15-13-05](http://noback.upyun.com/2020-03-08-15-13-05.png)

```py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 创建模型类
class Student(BaseModel):
    name: str = 'hzj' # 默认值,你开心
    age: int = None
    like: str = "to do list"


class User(BaseModel):
    username: str
    full_name: str = None

@app.get('/')
async def root():
    return {"hello":"hello world"}

# 更新用户
# 传入参数为{user-id}
@app.put('/user/{user_id}')
async def update_user_items(
    user_id: int,
    S_item: Student,
    user: User,
):
    res = {"user_id":user_id}
    if S_item or user:
        res.update({"S_item":S_item,"user":user})
    elif user:
        res.update({"user":user})
    elif S_item:
        res.update({"S_item":S_item})
    else:
        return {"status":404}
    return res
```

### 使用Body
Body的用法和Query Path的用法一样

<font color='red'>这个关键字使用的方式是，将查询参数直接映射到request body中，而不是变成一个查询条件用于查询</font>

```python

from fastapi import Body, FastAPI,Path
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None


class User(BaseModel):
    username: str
    full_name: str = None


@app.put("/items/{item_id}")
async def update_item(
    *, item_id: int = Path(...,description="test filed",), item: Item, user: User, importance: int = Body(...,description="test_body")
):
    importance = 1000000
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
```
![2020-03-31-11-31-38](http://noback.upyun.com/2020-03-31-11-31-38.png)
没有出现在查询条件中，而是直接出现在了request body中

把Body更换成Query 对查询参数做限制
```py
from fastapi import Body, FastAPI,Path,Query
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None


class User(BaseModel):
    username: str
    full_name: str = None


@app.put("/items/{item_id}")
async def update_item(
    *, item_id: int = Path(...,description="test filed",), item: Item, user: User, importance: int = Query(...,description="test_body")
):
    importance = 100000000000000
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
```
![2020-03-31-11-33-41](http://noback.upyun.com/2020-03-31-11-33-41.png)

<font color='red'>另外Body还有一些扩展的功能</font> 
比如写一个例子
```py
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None


@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int,
    item: Item = Body(
        ...,
        example={
            "name": "Foo",
            "description": "A very nice Item",
            "price": 35.4,
            "tax": 3.2,
        },
    )
):
    results = {"item_id": item_id, "item": item}
    return results
```

#### 规定模型类中的字段
pass












### 使用 Field
Field可以在定义模型类的时候，对模型类的中内容做限制
使用方法和Query以及Path一样，但是这个是在pydantic

之前定义模型类 ，如下
```py
#!/usr/bin/env python3
# encoding: utf-8
'''
@author: hzj
@file: test4.py
@time: 2020/3/31 5:39 下午
'''


from fastapi import  FastAPI
from pydantic import BaseModel,Field

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float = None
    tax: float = None
    test_string: str = Field(None,title="The description of the item",max_length=1)
    # test_example: str = Field(...,example="xxx")
    test_example: str = Field(None,example="xxx")


@app.get("/")
async  def root():
    return {"hello":"world"}

@app.post("/items")
async def post_items(item: Item):
    return item
```

对字段上做限制，比如只允许test_string最长字符串为1
报错内容
![2020-04-01-13-00-01](http://noback.upyun.com/2020-04-01-13-00-01.png)

> example可以对这个字段指定一个例子，但不是默认值



## 暂定
## 静态资源
安装aiofiles库文件
```py
pip install aiofiles

# 挂在静态文件到程序中
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")
```