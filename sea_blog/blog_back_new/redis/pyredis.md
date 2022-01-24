---
title: pyredis使用
categories:
  - redis
tags:
  - redis
  - pyredis
  - python
abbrlink: cf566df7
date: 2020-06-01 10:44:08
---

![2020-10-28-17-47-35](http://noback.upyun.com/2020-10-28-17-47-35.png)
<!-- more -->


# pyredis
pyredis的使用和redis的使用类似，无非是生成了一个redis的对象，用对来来使用命令

<font color='red'>单次连接使用redis</font>

```bash

# ----> 单次连接redis
# 连接redis需要IP地址，运行端口，数据库，密码。如果本地redis没有密码可以直接使用默认值
conn = StrictRedis(host="localhost", port=6379, db=0, password=None)
# <class 'redis.client.Redis'>  调用Redis子类对象生成连接实例
from redis.client import Redis
print(type(conn))
# 连接成功之后调用实例对象的set方法设置值
conn.set("name", "Yang")
# 调用get方法获取值
print(conn.get("name"))
```


<font color='red'>开启redis池,维护redis池</font>

```bash
# ------> 维护一个redis池
# 连接池设置参数，max_connections最大连接数。其他不变
pool = ConnectionPool(host="10.0.43.102", port=6379, db=0, password=None, max_connections=5,decode_responses=True,encoding='utf-8')
# 生成的连接池实例传递给StrictRedis连接redis数据库
redis =  StrictRedis(connection_pool=pool)
# <class 'redis.client.Redis'>  一样的连接类型
# print(type(redis))
```



## redis使用上的问题

### redis输出内容为字节，不显示中文
这里需要对redis的连接代码做一些改变
```bash
# 单次连接
r = redis.StrictRedis(host='127.0.0.1', port=6379, password='123456',db=1, decode_responses=True,encoding='utf-8')
# redis池
pool = ConnectionPool(host="10.0.43.102", port=6379, db=0, password=None, max_connections=5,decode_responses=True,encoding='utf-8')
# 生成的连接池实例传递给StrictRedis连接redis数据库
redis =  StrictRedis(connection_pool=pool)
```


