---
title: fastapi完整示例
categories:
  - fastapi
tags:
  - fastapi
abbrlink: a7e8896e
date: 2020-04-02 13:41:17
---

learn fastapi
<!-- more -->


# fastapi完整实例

fastapi官方提供了一个实例，实例中做到的增查用户等功能
使用一下内容
- fastapi做后端api
- sqlalchemy做orm工具
- sqllite做数据库 ，更推荐使用postgresql

https://fastapi.tiangolo.com/tutorial/sql-databases/#migrations


# 项目目录

```py
➜  project git:(master) ✗ tree .
.
├── __init__.py
├── crud.py           # database action
├── database.py       # connect database  sqlalchemy for sqllite3  (device update setting)
├── main.py           # main
├── models.py         # database for sqlalchemy 类似于django中的models.py
├── schemas.py        # show 
└── test.db           # sqllite3
``` 


## 自己对目录结构重新整理了一下
自己整理了一下目录，像他给出的案例，里面放太多model后期不好维护

```py
(py_env) ➜  project git:(master) ✗ tree .        
.
├── app                                 # -> app 多个app组成  里面放一些crud 以及model
│   ├── Item
│   │   ├── __pycache__
│   │   │   └── model.cpython-37.pyc
│   │   ├── crud.py
│   │   └── model.py
│   ├── __init__.py
│   ├── schemas.py                      # schemas这个文件一直不知道怎么处理，它相当于是对sqlalchemy设计出来的model做了一个映射把
│   └── user
│       ├── crud.py
│       └── models.py
├── main.py
├── routers                             # 路由设计，这里是官方给出的设计
│   ├── __init__.py
│   ├── items.py
│   └── users.py
├── setting.py
└── test.db
```

上面的内容会传到github上，后面记录地址


## 关于APIrouter的使用
他其实对第一层路径相同的情况下做了优化

这个是一个user的请求路径，但是这样会重复user的这个第一层路径
```py
from fastapi import APIRouter
from app.user import crud
from app import  schemas


from typing import  List
from fastapi import  Depends, HTTPException


from sqlalchemy.orm import  Session
from setting import  SessionLocal


def get_db():
    try:
        db = SessionLocal()
        yield db
    finally:
        db.close()

router = APIRouter()

@router.post("/users/", response_model=schemas.User)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    db_user = crud.get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return crud.create_user(db=db, user=user)




@router.get("/users/", response_model=List[schemas.User])
def read_users(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    users = crud.get_users(db, skip=skip, limit=limit)
    return users


@router.get("/users/{user_id}", response_model=schemas.User)
def read_user(user_id: int, db: Session = Depends(get_db)):
    db_user = crud.get_user(db, user_id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return db_user
```

可以在路由中修改第一层路由路径
```py
app.include_router(
    users.router,
    prefix="/users",
)

```