---
title: fastapi
categories:
  - fastapi
tags:
  - fastapi
abbrlink: bec9d697
date: 2020-05-09 10:24:17
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Fastapi介绍](#fastapi介绍)
  - [fastapi与其他python-web框架的区别](#fastapi与其他python-web框架的区别)
  - [fastapi 官方定位](#fastapi-官方定位)
  - [Framework Benchmarks](#framework-benchmarks)
  - [start demo](#start-demo)
  - [fastapi服务于API](#fastapi服务于api)
    - [设置根目录](#设置根目录)
    - [对路径参数进行限制](#对路径参数进行限制)
    - [对查询参数做限制](#对查询参数做限制)
    - [设置请求体](#设置请求体)
    - [请求体嵌套](#请求体嵌套)
    - [设置响应体](#设置响应体)
    - [自定义响应码](#自定义响应码)
    - [依赖注入](#依赖注入)
    - [fastapi login demo](#fastapi-login-demo)
  - [数据库以及orm的选择](#数据库以及orm的选择)
    - [sqlalchemy实例](#sqlalchemy实例)
    - [tortoise-orm实例](#tortoise-orm实例)
  - [部署](#部署)
    - [dockerfile](#dockerfile)
    - [单机本地部署](#单机本地部署)
    - [supervisor项目托管](#supervisor项目托管)
    - [gunicorn部署asgi服务](#gunicorn部署asgi服务)
    - [部署完整示例](#部署完整示例)

<!-- /code_chunk_output -->


<!-- more -->

# Fastapi介绍
![fastapi](https://fastapi.tiangolo.com/img/logo-margin/logo-teal.png)

Documentation: https://fastapi.tiangolo.com

Source Code: https://github.com/tiangolo/fastapi

## fastapi与其他python-web框架的区别
在fastapi之前，python的web框架使用的是django falsk tornado 三种web框架。
- django 自带admin ,快速构建  但是太重，虽然很多都封装好了，如果是mvc形式的开发，这样的确蛮合适的，但是restful风格设计，则django就显得有一些笨重
- flask 快速构建，自由度高。因为他十分轻盈，插件即插即用，很适合用来做restful风格的设计
- tornado python web框架和异步网络库，它执行非阻塞I/O ,
没有对REST API的内置支持，但是用户可以手动实现REST API
- fastapi 快速构建，异步IO,自带Swagger作为API文档，不用后续去内嵌Swagger-Ui 
在我用来，我感觉fastapi是一个专门为restful风格设计的web框架，全面服务于api形式web后端

fastapi的设计还是很符合restful的，用到的技术也都是很多新技术,同时也没有抛弃之前一些比较好用的内容，包括类型注释、依赖注入，websocket，swaggerui等等，以及其他的一些注释，比如GraphQL

## fastapi 官方定位
在fastapi 官方文档中，可以看到官方对fastapi做的一些定位
- 快速：非常高的性能，看齐的NodeJS和go（感谢Starlette和Pydantic) 
- 快速编码：将功能开发速度提高约200％至300％*。
- 错误更少：减少约40％的人为错误（开发人员）。*  (fastapi内置很多python高版本的语法 比如类型注释，typing库等等，因此他被要求的python版本为3.6+)
- 简易：旨在易于使用和学习。减少阅读文档的时间。
- 功能完善: 自带Swagger作为API文档

## Framework Benchmarks
https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=fortune
![2020-06-03-21-21-22](http://noback.upyun.com/2020-06-03-21-21-22.png)
![2020-06-03-21-21-47](http://noback.upyun.com/2020-06-03-21-21-47.png)
![2020-06-03-21-23-11](http://noback.upyun.com/2020-06-03-21-23-11.png)
![2020-06-03-21-23-24](http://noback.upyun.com/2020-06-03-21-23-24.png)
可以看出在高并发下的排名情况，当然web框架如果单纯是性能之间的比较是没用的，还要看业务上的需求和使用场景。最适合的才是最好的，这里只介绍fastapi的一些用法和特性.



## start demo
```py
pip install fastapi  # fastapi 库
pip install uvicorn  # Asgi server
```
```py
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}

# 启动
uvicorn main:app 
# 支持自定义host和端口
uvicorn main:app  --host 0.0.0.0 --port 8080
# 支持热更新
uvicorn main:app --reload --host 0.0.0.0 --port 8080
```
同样的fastpai支持异步请求
```py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```
![2020-05-11-10-53-49](http://noback.upyun.com/2020-05-11-10-53-49.png)
![2020-05-11-10-56-24](http://noback.upyun.com/2020-05-11-10-56-24.png)
项目浏览地址
http://127.0.0.1:8000
交互式文档地址
http://127.0.0.1:8000/docs
redoc地址
http://127.0.0.1:8000/redoc




## fastapi服务于API
在我使用的情况中，我的感受是，fastapi主要受众群体应该是web-api开发者，对于前后端不分离的项目，也就是mvc设计模式，fastapi很明显是不受用的，并且他也没有提供这方面的解决方案。
### 设置根目录
```py
# main.py
# 设置路由
from fastapi import FastAPI
import users
app = FastAPI()
app.include_router(
    users.router,
    prefix="/fastapi/play/v1/users", # 路由前缀
    tags=['users'] # 路由接口类别
)

# routers/users.py
from fastapi import FastAPI,APIRouter
from datetime import datetime,timedelta
router = APIRouter()
@router.get("/get/users/")
async def get_users():
    return {
        "desc":"Return to user list"
    }
```
![2020-06-03-21-35-53](http://noback.upyun.com/2020-06-03-21-35-53.png)



### 对路径参数进行限制
```py
# 根据名字获取列表
@router.get("/get/user/{username}")
async def get_user_by_username(username :str):
    """
        - username: 用户名
    """
    return {
       "desc":"this username is "+ username 
    }

```
![2020-06-03-21-39-37](http://noback.upyun.com/2020-06-03-21-39-37.png)

上面只是实现了一个简单的路径参数查找
使用枚举做限制

```py
from enum import Enum
class type_enum(str,Enum):
    woman="alice"
    man = "hzj"
    dog = "hhh"
# 路径参数+ 枚举
@router.get("/username/{username_type}")
async def get_user_by_type(username_type : str):
    result = ""
    if username_type == type_enum.alice:
        result = "ok, i am {}".format(type_enum.woman)
    elif username_type == type_enum.hzj:
        result = "ok , i am {}".format(type_enum.man)
    else:
        result = "None"
    return {
        "desc":result
    }
```

### 对查询参数做限制
```py
test_json = [
    {
        "id":1,
        "name":"hzj"
    },
    {
        "id":2,
        "name":"app"
    },
    {
        "id":3,
        "name":"小敏"
    },
    {
        "id":4,
        "name":"小红"
    },
    {
        "id":5,
        "name":"晓晓"
    },
    {
        "id":6,
        "name":"cy"
    },
]
import json
# 对查询参数做限制
@router.get("/friends/")
async def get_friends_by_id(id :int):
    for item in test_json:
        if item['id'] == id:
            return item
        else:
            return {
                "desc": "no this id"
            }
```
```py
import json
# 对查询参数做限制
@router.get("/friends/")
# 设置为None的时候，默认不可以不填
async def get_friends_by_id(id :int=None):
    for item in test_json:
        if item['id'] == id:
            return item
        else:
            return {
                "desc": "no this id"
            }
```

<font color='red'>对查询参数做进一步的限制</font>

```py
@router.get("/friends/"{id})
async def get_friends_by_id(id :int= Query(None, gt=0)):
    for item in test_json:
        if item['id'] == id:
            return item
        else:
            return {
                "desc": "no this id"
            }

# 多参数请求查询
from typing import List
@router.get("/items/")
async def read_items(q: List[str] = Query(["foo", "bar"])):
    query_items = {"q": q}
    return query_items
```


### 设置请求体
```py
# 设置请求实体

from pydantic import BaseModel,Field
class requestModel(BaseModel):
    name :str
    age : int = Field(..., gt=0, description="The age must be greater than zero")
    desc: str


@router.post("/post/UserInfo/")
async def post_UserInfo(item: requestModel):
    return item

```
![2020-06-03-22-19-53](http://noback.upyun.com/2020-06-03-22-19-53.png)

### 请求体嵌套
```py
from pydantic import BaseModel,Field

class levelInfoModel(BaseModel):
    id:int = None
    info: str = None

class ClassInfo(BaseModel):
    id: int = None
    name: str = Field(..., max_length=20, min_length=10,
                      description="The necessary fields")
    desc: str = Field(None, max_length=30, min_length=10)
    levelInfo: List[levelInfoModel]

    class Config:
        schema_extra = {
            "example": {
                "id": 1,
                "name": "Foo",
                "desc": "A very nice Item",
                "levelInfo": [{
                    "id": 1,
                    "info": "一级"
                }]
            }
        }

@router.post("/info/")
async def get_classInfo(item:ClassInfo):
    return item
```
![2020-06-03-22-29-20](http://noback.upyun.com/2020-06-03-22-29-20.png)

### 设置响应体
```py
test_json = {
    "code": 0,
    "status": "success",
    "data":[
        {
        "id": 1,
        "age": 2,
        "name":"hzj"
    }
    ]
}
@router.get("/get/UserInfo",response_model=responseModel)
async def get_UserInfo(id :int):
    return test_json

```

### 自定义响应码
```py

@router.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}

from fastapi import FastAPI, status


@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}
```

### 依赖注入
```py
from fastapi import Depends, FastAPI

async def common_parameters(q: str = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@router.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons

@router.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```
支持多层嵌套依赖注入

### fastapi login demo
```python
# 安装环境
mkdir fastapi-demo && cd fastapi-demo
virtualenv env
source env/bin/active

# 下载项目
git clone https://github.com/hzjsea/BaseFastapi 
cd BaseFastapi/ 
pip install -r requirements.txt
# 开启项目
uvicorn main:app --reload 
# uvicorn main:app --host 0.0.0.0 --port 80 --reload
```


## 数据库以及orm的选择
上面的案例都没有用到数据库，数据库的选择现在无非就是mysql PostgreSQL redis
但是对于orm来说fastapi有好几种选择
- sqlalchemy 但是不支持异步，不过貌似可以扩展成异步
- tortoise-orm 类django-orm的异步orm，不过正在起步过程中，有些功能还没有完成。

### sqlalchemy实例
```py
from typing import List

import databases
import sqlalchemy
from fastapi import FastAPI
from pydantic import BaseModel

# SQLAlchemy specific code, as with any other app
DATABASE_URL = "sqlite:///./test.db"
# DATABASE_URL = "postgresql://user:password@postgresserver/db"

database = databases.Database(DATABASE_URL)

metadata = sqlalchemy.MetaData()

notes = sqlalchemy.Table(
    "notes",
    metadata,
    sqlalchemy.Column("id", sqlalchemy.Integer, primary_key=True),
    sqlalchemy.Column("text", sqlalchemy.String),
    sqlalchemy.Column("completed", sqlalchemy.Boolean),
)


engine = sqlalchemy.create_engine(
    DATABASE_URL, connect_args={"check_same_thread": False}
)
metadata.create_all(engine)


class NoteIn(BaseModel):
    text: str
    completed: bool


class Note(BaseModel):
    id: int
    text: str
    completed: bool


app = FastAPI()


@app.on_event("startup")
async def startup():
    await database.connect()


@app.on_event("shutdown")
async def shutdown():
    await database.disconnect()


@app.get("/notes/", response_model=List[Note])
async def read_notes():
    query = notes.select()
    return await database.fetch_all(query)


@app.post("/notes/", response_model=Note)
async def create_note(note: NoteIn):
    query = notes.insert().values(text=note.text, completed=note.completed)
    last_record_id = await database.execute(query)
    return {**note.dict(), "id": last_record_id}
```


### tortoise-orm实例
```bash
pip install tortoise-orm
```
```py

# main.py
from tortoise.contrib.fastapi import HTTPNotFoundError, register_tortois
# 创建的数据表
models = [
    "app.Users.models",
    "app.Face.models",
    "app.Roster.models",
    "app.Statistical.models",
    "app.pay.models"
]

register_tortoise(
    app,
    db_url="mysql://username:password@ip:port/yydb",
    modules={"models": models},
    generate_schemas=True,
    add_exception_handlers=True,
)

# models.py
from tortoise import fields,models
from tortoise.contrib.pydantic import pydantic_queryset_creator
from pydantic import BaseModel
class RosterGroupTable(models.Model):
    id = fields.IntField(pk=True)
    phone = fields.CharField(max_length=20,blank=True,null=True)
    name = fields.CharField(max_length=20)

    class Meta:
        db = "RosterGroupTable"

class RosterTabel(models.Model):
    id = fields.IntField(pk=True)
    phone = fields.CharField(max_length=20,blank=True,null=True)
    name = fields.CharField(max_length=20)
    group_id = fields.ForeignKeyField(model_name='models.RosterGroupTable',on_delete=fields.CASCADE,related_name="events",blank=True,null=True)

    class Meta:
        db = "RosterTabel"

RosterGroupTable_desc = pydantic_queryset_creator(RosterGroupTable)
RosterTabel_desc = pydantic_queryset_creator(RosterTabel)



# roster.py
@router.post("/roster_statistics/add")
async def roster_add_statics(*,item:RosterItem,token :str):
    res = await RosterTabel.filter(id=item['memberId']).first()
    if res:
        await StatisticalRosterTable.create(
            phone = current_user.Phone,
            date = item['date'],
            time = item['time'],
            data = item['data'],
            name_id_id = item['memberId'],
            temp_type = item['tm_type'],
            today_num = item['todayNum'],
            group_id_id = res.group_id_id,
            name = res.name
        )
    else:
        return rp_faildMessage(msg="名单不存在",code=-1)
    return rp_successMessage(msg="名单创建成功",code=0)
```

## 部署
### dockerfile
```dockerfile
#================================================================================
#      基于python3.7的创建fastapi的dockerfile文件
#      fastapi + python3.7 + guvicorn
#================================================================================

FROM python:3.7
LABEL version="v1" \
    description="fastapi" \
    maintainer="hzjsea" \
    using="fastapi & python3.7 office image & gunicorn"
WORKDIR /root/code
COPY  . .
RUN pip install -r requirements.txt
EXPOSE 8889
CMD ["gunicorn","-c","guvicorn.conf","main:app"]

```

### 单机本地部署
```bash
#!/usr/bin/bash
# 测试用脚本部署

# 开启的端口
port=80
host=0.0.0.0

# 删除旧的文件
rm -rf /root/hzj/fastapi_play
# 拷贝新的文件
cp -r /root/fastapi_play/ /root/hzj/
# 切换workdir
cd /root/hzj/fastapi_play/
# 开启python虚拟环境
source /root/hzj/pro_env_all/venv/bin/activate
# 安装依赖
pip install -r requirements.txt
# 关闭之前运行的进程  
lsof -i:$port | awk '{getline; print $2 }' | xargs -t -I {} kill -9 {}
# 开启新进程
nohup uvicorn --host $host  --port $port  main:app --reload  &
echo "success"
```

### supervisor项目托管
```bash
[program:webserver]
directory=/root/hzj/fastapi_play
command=/root/hzj/pro_env_all/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8888
autostart = true
```

### gunicorn部署asgi服务
```bash
# gunicorn.conf
并行工作进程数
workers = 4
# 指定每个工作者的线程数
threads = 2
# 监听内网端口5000
bind = '0.0.0.0:8889'
# 设置守护进程,将进程交给supervisor管理
daemon = 'false'
# 工作模式协程
worker_class = 'uvicorn.workers.UvicornWorker'
# 设置最大并发量
worker_connections = 2000


#运行
gunicorn -c gunicorn.conf
```

### 部署完整示例
fastapi官方提供了一个前后端分离项目完整示例
https://github.com/tiangolo/full-stack-fastapi-postgresql

