---
title: Python container build
date: 2021-05-21 11:46:25
categories:  [docker]
tags: [docker]
---


<!--more-->


# Python container build



# Python container build

构建镜像需要确认的几个点
- 镜像的初始状态 默认开放的端口等， 这样就不用在`docker run `的时候再去声明端口
- 需要挂载的文件 是在构建的时候就copy进去 还是在之后挂载进去
- docker只是一个单进程，是否需要supervisor 在单进程中托管多任务进度


打个比方，比如python的一份环境文件 `requirements.txt` 最好的方式就是在初始化的时候就加入，这样的话，你就只需要在dockerfile中COPY进去解决，产生的image-id 容器，只要运行他 肯定会有之前`requirements.txt`中的内容

再打个比方 mysql的初始化root密码，可以用dockerfile 打一个镜像，比如
```Dockerfile
FROM mysql:5.7
ENV MYSQL_ROOT_PASSWORD=my-secret-p
```
那么这个打出来的镜像 image-id 所有的mysql root初始密码都会是 my-secret-p


## simple example

### build a flask container

1. 创建文件
```bash
mkdir -p app/src
cd app & touch server.py requirements.txt
touch Dockerfile
```
2. flask 代码
```py
# server.py
from flask import Flask
server = Flask(__name__)

@server.route("/")
 def hello():
    return "Hello World!"

if __name__ == "__main__":
   server.run(host='0.0.0.0')
```
```py
# requirements.txt
Flask==1.1.1
```
```Dockerfile
# set base image (host OS)
FROM python:3.8

# set the working directory in the container
WORKDIR /code

# copy the dependencies file to the working directory
COPY requirements.txt .

# install dependencies
RUN pip install -r requirements.txt

# copy the content of the local src directory to the working directory
COPY src/ .

# command to run on container start
CMD [ "python", "./server.py" ]
```
3. the following structure
```py
app
├─── requirements.txt 
├─── Dockerfile
└─── src
     └─── server.py

```
4. run
```bash
docker build . -t py
docker run -itd py:latest
```


## build 

### 构建一个项目

```bash
# 打包项目上传到专门构建的服务器上
tar -zcvf xx.tar --exclude=py/venv py
```

> 构建dockefile
```Dockerfile
FROM python:3.7
WORKDIR /code
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY src/ .
EXPOSE 25
CMD [ "/bin/bash","init.sh" ]
```
初始化脚本

```bash
project="fpben"
html_file="/opt/fpben/html"
pdf_file="/opt/fpben/pdf"


init(){
  mkdir -p $html_file
  mkdir -p $pdf_file
}

start(){
        python3 -u newmail_2021_0306_v2.py
}

main(){
        start
}

main
```
src 目录
```bash
(venv) [root@dev app2]# tree src
src
├── init.sh
├── newmail_2021_0306_v2.py
└── newmail_old.py
```
运行时挂载文件
<span style="font-size:100px;color:red">重要,挂载的时候为了防止错误，最好挂载绝对路径</span>
```bash
docker run -itd -v /root/python_docker/app2/opt/fpben/html:/opt/fpben/html -v /root/python_docker/app2/log:/code/log -v /root/python_docker/app2/opt/fpben/pdf:/opt/fpben/pdf 804164d17b98
```

### 如果不需要挂载文件夹出来的话

```bash
# 打包项目上传到专门构建的服务器上
tar -zcvf xx.tar --exclude=py/venv py
```

> 构建dockefile
```Dockerfile
FROM python:3.7
WORKDIR /code
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY src/ .
EXPOSE 25
CMD [ "/bin/bash","init.sh" ]
```
初始化脚本

```bash
project="fpben"
html_file="/opt/fpben/html"
pdf_file="/opt/fpben/pdf"


init(){
     if [ ! -d "$html_file" ]; then mkdir -p $html_file; fi
     if [ ! -d "$pdf_file" ]; then mkdir -p $pdf_file; fi
}

start(){
        python3 -u newmail_2021_0306_v2.py
}

main(){
        init
        start
}

main
```
src 目录
```bash
(venv) [root@dev app2]# tree src
src
├── init.sh
├── newmail_2021_0306_v2.py
└── newmail_old.py
```
运行时挂载文件
<span style="font-size:100px;color:red">重要,挂载的时候为了防止错误，最好挂载绝对路径</span>
```bash
docker run -itd -v  804164d17b98
```
