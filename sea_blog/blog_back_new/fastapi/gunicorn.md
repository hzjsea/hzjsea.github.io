---
title: gunicorn使用
categories:
  - fastapi
tags:
  - fastapi
abbrlink: b862bd5d
date: 2020-04-29 18:27:12
---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [gunicorn使用](#gunicorn使用)
  - [使用gunicorn构建fastapi项目](#使用gunicorn构建fastapi项目)
  - [使用gunicorn.conf配置文件构建项目](#使用gunicornconf配置文件构建项目)

<!-- /code_chunk_output -->

<!-- more -->



# gunicorn使用


## 使用gunicorn构建fastapi项目
fastapi的启动主要使用了uvicorn,gunicorn主要是用来管控uvicorn的进程，并且可以指定多个进程

示例
```bash
gunicorn -w 4 -k uvicorn.workers.UvicornWorker
```
指定work 多个进程
指定class 运行的角色

<font color='red'>注意 fastapi必须使用uvicorn.workers.UvicornWorker这个角色，不然会出现not found 'send' 的错误</font>


## 使用gunicorn.conf配置文件构建项目
gunicorn不仅支持命令行，还支持使用配置文件构建项目

gunicorn.conf
```bash
# gunicorn.conf
# 并行工作进程数
workers = 4
# 指定每个工作者的线程数
threads = 2
# 监听内网端口5000
bind = '0.0.0.0:5000'
# 设置守护进程,将进程交给supervisor管理
daemon = 'false'
# 工作模式协程
worker_class = 'uvicorn.workers.UvicornWorker'
# 设置最大并发量
worker_connections = 2000
# 设置进程文件目录
pidfile = '/var/run/gunicorn.pid'
# 设置访问日志和错误信息日志路径
accesslog = '/var/log/gunicorn_acess.log'
errorlog = '/var/log/gunicorn_error.log'
# 设置日志记录水平
loglevel = 'warning'
```

dockerfile编写
```bash
FROM python:3.7

# MAINTAINER superman for hzj

LABEL version="0.2v"

WORKDIR /root/code

COPY ./BA ./

COPY ./requirements.txt ./

COPY ./guvicorn.conf ./

RUN pip install -r requirements.txt

CMD ["gunicorn","-c","guvicorn.conf","app:main"]
```

运行项目
```bash
docker build .
docker run -it xx /bin/bash
```