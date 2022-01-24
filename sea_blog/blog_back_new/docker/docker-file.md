---
title: docker-DokcerFile
categories:
  - docker
tags:
  - docker
abbrlink: 8d3c2c27
date: 2020-02-05 13:56:09
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [DokcerFile](#dokcerfile)
  - [dockerfile编写规则](#dockerfile编写规则)
  - [dockerfile执行顺序](#dockerfile执行顺序)
  - [docker编写关键字](#docker编写关键字)
    - [ADD和COPY 以及远程文件的添加](#add和copy-以及远程文件的添加)
    - [WORKDIR](#workdir)
    - [CMD与ENTRYPOINT](#cmd与entrypoint)
    - [ONBUILD使用](#onbuild使用)
  - [编写实例](#编写实例)
    - [一个挂载文件的dockerfile](#一个挂载文件的dockerfile)
    - [用dockerfile创建一个centos镜像](#用dockerfile创建一个centos镜像)
    - [docker开启mysql镜像](#docker开启mysql镜像)
    - [docker开启redis镜像](#docker开启redis镜像)
    - [docker安装stress压力测试软件](#docker安装stress压力测试软件)
  - [网址](#网址)

<!-- /code_chunk_output -->
<!-- more -->


# DokcerFile

dockerfile构建三步骤
dockerfile,docker build ,docker run

dockerfiley有很多具体的实例可以在docker hub上看到 ，比如
- centos
https://hub.docker.com/_/centos
```bash
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20191001"

CMD ["/bin/bash"] # 在进入容器后第一步执行的命令
```
- tomcat镜像
https://github.com/docker-library/tomcat/blob/807a2b4f219d70f5ba6f4773d4ee4ee155850b0d/8.5/jdk11/openjdk/Dockerfile

## dockerfile编写规则
1. 每条保留字指令都必须为大写字母且后面要跟至少一个参数
2. 指令按照顺序从上倒下，依次执行
3. #表示注释
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交

## dockerfile执行顺序
1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一跳指令直到所有指令都执行完成

## docker编写关键字
- FROM
声明基础镜像，如centos中声明的scratch
- MAINTAINER
镜像维护者的姓名和邮箱地址
- LABEL
镜像版本信息
- RUN
容器构建时需要运行的命令，可以用 \ 分行
- EXPOSE
暴露端口，比如tomcat镜像中的dockerfile 他expose了一个8080端口，当我们在用-P随机指定端口的时候，他会默认映射我们所暴露的端口，如8080
- WORKDIR
登陆之后的工作目录
- ENV
用来构建镜像过程中设置环境变量
- ADD
拷贝并解压到镜像中
- COPY
拷贝文件到镜像中，写法:
```bash
COPY src dest
COPY ["src","dest"] 
```
- VOLUME
容器数据卷,用于数据保存和持久化工作,写法:
```bash
VOLUME ["/data/","/data1/",] 
```
- CMD
dockerfile中可以有多个CMD命令,但只有最后一个生效，且CMD会被docker run之后的CMD替换掉，比如我们在启动一个镜像的时候会执行docker run -it centos /bin/bash 
指定一个容器要运行时，启动的命令,写法:
```bash
# shell 格式
CMD <命令>
# exec 格式
CMD ["可执行文件","参数1","参数2"]
# 参数列表格式
CMN ['参数1','参数2']
```
- ENTRYPOINT
指定一个容器启动时候要运行的命令，不会被替换
- ONBUILD
当构建一个被继承的Dockerfile时运行命令,父镜像在被子继承后，父镜像的onbuild被触发

### ADD和COPY 以及远程文件的添加 
ADD不但可以复制还可以解压
```bash
ADD xx.tar.gz /
COPY xx /tmp/data1/ 
```

<font color='red'>添加远程文件/目录 使用RUN curl | RUN wget </font>



### WORKDIR
WORKDIR表示工作目录
```bash
WORKDIR /test
```
如果没有test工作目录，则会自动创建
```bash
WORKDIR /test
WORKDIR demo1 
```
以上会生成的工作目录是/test/demo1

<font color='red'>尽量使用绝对目录</font>

### CMD与ENTRYPOINT
在tomcat的官方镜像包中，dockerfile里面的最后有这么一句CMD
```bash
CMD ["catalina.sh","run"] 
```
意思就是说在dokcer run没有加CMD的情况下启动时，会去运行CMD ["catalina.sh","run"] 这一串内容。
```bash
06-Feb-2020 05:29:22.623 INFO [main] org.apache.catalina.startup.Catalina.load Initialization processed in 389 ms
06-Feb-2020 05:29:22.639 INFO [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
06-Feb-2020 05:29:22.639 INFO [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet Engine: Apache Tomcat/8.5.50
06-Feb-2020 05:29:22.647 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
06-Feb-2020 05:29:22.653 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["ajp-nio-8009"]
06-Feb-2020 05:29:22.655 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in 32 ms 
```
但如果我们在后面加了如 /bin/bash
```bash
root:/tmp# docker run -it -P tomcat /bin/bash
root@2ee3ee482178:/usr/local/tomcat#
```
他就不回去运行这个catalina.sh这个程序

<font color='red'>ENTRYPOINT可以将参数累加</font>

```bash
FROM centos
RUN yum install curl -y
ENTRYPOINT  ["curl","-s","https://ip.cn/"]

docker run -it centos   ->  ["curl","-s","https://ip.cn/"]  
docker run -it centos -i  -> ["curl","-s","-i","https://ip.cn/"] 
```

<font color='red'>两种风格的编写格式</font>
![2020-02-06-20-53-20](http://noback.upyun.com/2020-02-06-20-53-20.png)

### ONBUILD使用
ONBUILD就是在继承父容器之后，如果子容器运行了，就会在父容器中运行ONBUILD后面的命令
```bash
FROM centos
RUN yum install -y curl
ENTRYPOINT ["curl","-s","https://ip.cn/"] 
ONBUILD RUN echo "son is run "
```
父容器为m1 子容器为m2,这里的继承就是说以父容器镜像为基本模版创建的子镜像后生成的子容器

## 编写实例

### 一个挂载文件的dockerfile
```bash
FROM centos
VOLUME ["data1/","data2/"]  # 这种定义方式无法像命令一样指定本地的文件
CMD echo "VOLUME TEST!"
CMD /bin/bash
```
根据编写的dockerfile构建自定义镜像
```bash
docker build -f /tmp/dockerfile -t hzj/centos . 
```
使用Inspect查看挂载的文件路径
```bash
docker inspect hzj/centos 
```
出现挂载的路径
```bash
"Mounts": [
            {
                "Type": "volume",
                "Name": "2a8601766a8ec82c39132bc64c2c99546053cb4104bd94507f215b888e60464d",
                "Source": "/var/lib/docker/volumes/2a8601766a8ec82c39132bc64c2c99546053cb4104bd94507f215b888e60464d/_data",
                "Destination": "/data2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }, 
```

### 用dockerfile创建一个centos镜像
初始centos镜像只有最基本的kernel内容，并没有其他的软件，比如vim，比如ifconfig等。使用dockerfile创建一个centos系统
```bash
FROM centos # 基于centos，网上写东西 再生成镜像  镜像的分层原理
MAINTAINER hzj<a1097690268@gmail.com> # 作者信息

ENV workdir /usr/local # 申明环境变量
WORKDIR $workdir # 设置登陆后的工作环境

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80 # 暴露端口

CMD echo $workdir
CMD echo "success"
CMD /bin/bash
```

```bash
docker  build  -f dockerfile -t hzj/centos:v3 .
```

### docker开启mysql镜像
```bash
docker pull mysql
docker run -p 12345:3306 --name mysql \
-v /conf/mysql/conf:/etc/mysql/conf.d \
-v /conf/mysql/logs:/logs \ 
-v /conf/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWD=123456 \ #初始化密码
-d mysql:5.6
```

### docker开启redis镜像
```bash
docker pull redis
docker run -p 6379:6379 \
-v /hzj/myredis/data:/data \
-v /hzj/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf \ 
-d redis:3.2 redis-server /usr/local/etc/redis/redis.conf \
--appendonly yes
```

### docker安装stress压力测试软件
```bash
FROM ubuntu
RUN apt-get update && apt-get install stress -y
ENTRYPOINT ["/usr/bin/stress"]
# ENTRYPOINT ["/bin/bash","-c","stress"]
CMD []
```
生成镜像
```bash
dokcer build -f dockerfile -t new/ubuntu-stress . 
```
利用镜像运行容器
```bash
# 不加参数 /usr/bin/stress
root:~# docker  run -it   new/ubuntu-stress
'stress' imposes certain types of compute stress on your system

Usage: stress [OPTION [ARG]] ...
 -?, --help         show this help statement
     --version      show version statement
 -v, --verbose      be verbose
 -q, --quiet        be quiet
 -n, --dry-run      show what would have been done
 -t, --timeout N    timeout after N seconds
     --backoff N    wait factor of N microseconds before work starts
 -c, --cpu N        spawn N workers spinning on sqrt()
 -i, --io N         spawn N workers spinning on sync()
 -m, --vm N         spawn N workers spinning on malloc()/free()
     --vm-bytes B   malloc B bytes per vm worker (default is 256MB)
     --vm-stride B  touch a byte every B bytes (default is 4096)
     --vm-hang N    sleep N secs before free (default none, 0 is inf)
     --vm-keep      redirty memory instead of freeing and reallocating
 -d, --hdd N        spawn N workers spinning on write()/unlink()
     --hdd-bytes B  write B bytes per hdd worker (default is 1GB)

Example: stress --cpu 8 --io 4 --vm 2 --vm-bytes 128M --timeout 10s

# 加参数 /usr/bin/stress -m 1
root:~# docker  run -it   new/ubuntu-stress -m 1
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
^Cstress: FAIL: [1] (415) <-- worker 6 got signal 2
stress: WARN: [1] (417) now reaping child worker processes
stress: FAIL: [1] (421) kill error: No such process
stress: FAIL: [1] (451) failed run completed in 2s
```

## 网址
docker官方提供的一个dockerfile编写的规范
https://docs.docker.com/engine/reference/builder/
docker镜像发布到docker-hub上面
https://coding.imooc.com/lesson/189.html#mid=11620