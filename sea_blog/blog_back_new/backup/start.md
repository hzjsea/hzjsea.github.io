---
title: sqlalchemy使用方法
categories:
  - python
tags:
  - sqlalchemy
  - orm
abbrlink: 56b71958
date: 2020-03-20 10:37:09
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [sqlalchemy](#sqlalchemy)
  - [什么是sqlalchemy](#什么是sqlalchemy)
  - [一个完整的实例](#一个完整的实例)
  - [获取数据操作](#获取数据操作)
    - [给获取的字段名修改名称](#给获取的字段名修改名称)
    - [类似sql的一些查询操作](#类似sql的一些查询操作)

<!-- /code_chunk_output -->
<!-- more -->

# sqlalchemy

## 什么是sqlalchemy
sqlalchemy是一个python的orm库，用于控制数据库


## 一个完整的实例
从创建数据库到添加第一条数据的整个实例过程

`创建数据库表结构`
```py
# 创建数据库实例，
# echo表示在之后对数据库的操作中是否会显示提示内容
engine = create_engine("sqlite:///memory.db",echo=True)
# 创建基本类 这个基本类会映射到数据库中的表
Base = declarative_base()

# 所有的表都要继承Base这个类
class User(Base):
    # 数据库中的表名字
    __tablename__ = 'users' # 表的名字

    # 定义各字段
    # 定义方式和django中的类似
    # Sequence('user_id_seq')这个只是针对Oracle来添加的，用来申明主键标识符，其他数据库可以删除
    # id = Column(Integer,Sequence('user_id_seq'), primary_key=True)
    id = Column(Integer, primary_key=True)
    name = Column(String(50))# 定义长度
    fullname = Column(String)
    password = Column(String)

   # 面向用户，最终显示
    # def __str__(self):
        # return self.id
    
    # 面向调试，输出所有
    def __repr__(self):
        return "<User(name='%s', fullname='%s', password='%s')>" % (
                                self.name, self.fullname, self.password)

# 上诉代码会创建的表结构为
# Table('users', MetaData(bind=None),
#             Column('id', Integer(), table=<users>, primary_key=True, nullable=False),
#             Column('name', String(), table=<users>),
#             Column('fullname', String(), table=<users>),
#             Column('nickname', String(), table=<users>), schema=None)

# 创建表
Base.metadata.create_all(engine)
```

`插入数据`
```py
# 建立连接
Session = sessionmaker()
# 绑定数据库
Session.configure(bind=engine)
# 实例化一个session
session = Session()


# session = Session.configure(bind=engine) 这样写是不行的
```

`创建并提交数据`
```py
# 创建一个数据实例
user_data = User(name="hzj",fullname="hzj",password="zzz")
# 添加实例到session中,这里只是缓存到了session这个实例中
session.add(user_data)
# 提交缓存在session中的所有实例
session.commit()

user_data1 = User(name="hzj",fullname="hzj",password="qqq")
session.add(user_data1)
session.commit()
```

`批量添加数据`
```py
user_data_list = [
   User(name="hzj",fullname="hzj",password="zzz"),
   User(name="hzj",fullname="hzj",password="zzz"),
   User(name="hzj",fullname="hzj",password="zzz"),
   User(name="hzj",fullname="hzj",password="zzz"),
   User(name="hzj",fullname="hzj",password="zzz")
]

session.add_all(user_data_list)
session.commit()
```

注释:
```py
for i in range(1,7):
    session.add(data)
    session.commit()
```
别想着这样去加数据 没用2333

![2020-03-20-10-50-34](http://noback.upyun.com/2020-03-20-10-50-34.png)


`获取数据`
```py
# 获取数据
get_data = session.query(User).filter_by(name="hzj").first()
print(get_data)
```


## 获取数据操作
```py
# 获取所有内容，并按照id排序
get_data_list = session.query(User).filter_by(name="hzj").order_by(User.id)
print(get_data_list)
for i in get_data_list:
    print("by_id:{}".format(i))
# by_id:<User(name='hzj', fullname='hzj', password='zzz')>
# by_id:<User(name='hzj', fullname='hzj', password='zzz')>


instance_list = session.query(User)
for i in instance_list:
    print("User:{}".format(i))
# User:<User(name='hzj', fullname='hzj', password='zzz')>
# User:<User(name='hzj', fullname='hzj', password='zzz')>
# User:<User(name='hzj', fullname='hzj', password='zzz')

```

在没有做限制的条件下 sqlalchemy会返回一个元组
```py
instance_list = session.query(User.name,User.password)  #返回的是元组
# ('hzj', 'zzz')
# ('hzj', 'zzz')
# ('hzj', 'zzz')
for name,password in instance_list:
    print("User_name:  {},User_password:  {}".format(name,password))
# User_name:  hzj,User_password:  zzz
# User_name:  hzj,User_password:  zzz
# User_name:  hzj,User_password:  zz
instance_list = session.query(User.name,User.password)[:2]  # 返回列表
# instance_list = session.query(User.name,User.password).all() # 返回所有
print(instance_list)
```

### 给获取的字段名修改名称
```py
for new_name in session.query(User.name):
    print(new_name.name) 
# 这里的name 是我们之前创建User表时， 设定的变量名，但是在某些情况下
# 我们需要修改这个变量名，可以使用label和alias
# hzj
# hzj 
# hzj
for new_name in session.query(User.name.label("new_name")):
    print(new_name.new_name)
# hzj
# hzj
# hzj

```

### 类似sql的一些查询操作
```py
# 筛选字段数据
print(session.query(User.name).filter_by(password="qqq").first())
print(session.query(User.name).filter(User.password=="qqq").first())
print(session.query(User.name).filter(User.password=="qqq").filter(User.fullname=="hzj").first())
# like操作
print(session.query(User.name).filter(User.password.like('%qq%')))
# 不区分大小写的like操作
print(session.query(User.name).filter(User.password.ilike('%qq%')))
# https://docs.sqlalchemy.org/en/13/orm/tutorial.html#using-textual-sql
支持sql的一些操作

```