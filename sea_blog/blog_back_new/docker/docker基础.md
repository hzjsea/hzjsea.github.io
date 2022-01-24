---
title: docker基础
categories:
  - docker
tags:
  - docker
abbrlink: 5ae6092
date: 2020-02-04 17:26:54
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker](#docker)
  - [docker与虚拟机的区别](#docker与虚拟机的区别)
    - [虚拟机](#虚拟机)
    - [dokcer容器](#dokcer容器)
    - [区别](#区别)
  - [概念](#概念)
  - [镜像](#镜像)
    - [获取镜像](#获取镜像)
    - [提交一个容器为新的镜像(不提倡)](#提交一个容器为新的镜像不提倡)
    - [运行镜像](#运行镜像)
    - [查看镜像信息](#查看镜像信息)
    - [搜寻镜像](#搜寻镜像)
    - [删除镜像](#删除镜像)
  - [容器](#容器)
    - [创建容器与启动](#创建容器与启动)
    - [新建并启动容器](#新建并启动容器)
    - [守护状态运行](#守护状态运行)
    - [终止容器](#终止容器)
    - [进入容器](#进入容器)
    - [删除容器](#删除容器)
    - [导入与导出容器](#导入与导出容器)
    - [查看容器](#查看容器)
    - [复制容器内文件](#复制容器内文件)
  - [帮助命令](#帮助命令)
  - [清除容器](#清除容器)
  - [数据卷容器](#数据卷容器)
  - [错误合集](#错误合集)

<!-- /code_chunk_output -->
<!-- more -->

# docker

## docker与虚拟机的区别

### 虚拟机
虚拟机运行多个隔离应用时
![2020-02-04-18-07-59](http://noback.upyun.com/2020-02-04-18-07-59.png)

从下到上来看:
- 基础设施(Infrastructure)。它可以是你的个人电脑，数据中心的服务器，或者是云主机。
- 主操作系统(Host Operating System)。你的个人电脑之上，运行的可能是MacOS，Windows或者某个Linux发行版。
- 虚拟机管理系统(Hypervisor)。利用Hypervisor，可以在主操作系统之上运行多个不同的从操作系统。类型1的Hypervisor有支持MacOS的HyperKit，支持Windows的Hyper-V以及支持Linux的KVM。类型2的Hypervisor有VirtualBox和VMWare。
- 从操作系统(Guest Operating System)。假设你需要运行3个相互隔离的应用，则需要使用Hypervisor启动3个从操作系统，也就是3个虚拟机。这些虚拟机都非常大，也许有700MB，这就意味着它们将占用2.1GB的磁盘空间。它们还会消耗很多CPU和内存。
- 各种依赖。每一个从操作系统都需要安装许多依赖。如果你的的应用需要连接PostgreSQL的话，则需要安装libpq-dev；如果你使用Ruby的话，应该需要安装gems；如果使用其他编程语言，比如Python或者Node.js，都会需要安装对应的依赖库。
- 应用。安装依赖之后，就可以在各个从操作系统分别运行应用了，这样各个应用就是相互隔离的。

### dokcer容器
![2020-02-04-18-10-43](http://noback.upyun.com/2020-02-04-18-10-43.png)
相比于虚拟机来说  docker更加的简洁

从下到上
- 基础设施(Infrastructure)。
- 主操作系统(Host Operating System)。所有主流的Linux发行版都可以运行Docker。对于MacOS和Windows，也有一些办法”运行”Docker。
- Docker守护进程(Docker Daemon)。Docker守护进程取代了Hypervisor，它是运行在操作系统之上的后台进程，负责管理Docker容器。
- 各种依赖。对于Docker，应用的所有依赖都打包在Docker镜像中，Docker容器是基于Docker镜像创建的。
- 应用。应用的源代码与它的依赖都打包在Docker镜像中，不同的应用需要不同的Docker镜像。不同的应用运行在不同的Docker容器中，它们是相互隔离的


### 区别
Docker守护进程可以直接与主操作系统进行通信，为各个Docker容器分配资源；它还可以将容器与主操作系统隔离，并将各个容器互相隔离。虚拟机启动需要数分钟，而Docker容器可以在数毫秒内启动。由于没有臃肿的从操作系统，Docker可以节省大量的磁盘空间以及其他系统资源。

说了这么多Docker的优势，大家也没有必要完全否定虚拟机技术，因为两者有不同的使用场景。虚拟机更擅长于彻底隔离整个运行环境。例如，云服务提供商通常采用虚拟机技术隔离不同的用户。而Docker通常用于隔离不同的应用，例如前端，后端以及数据库。

## 概念
- 镜像(Image)
面向docker引擎的只读模版，包含了文件系统
- 容器(Container)
沙箱，利用容器来运行和隔离应用，容器与容器之间是相互隔离、互不可见的
<font color='red'>镜像自身是只读的。容器从镜像启动的时候，Docker会在镜像的最上层创建一个可写层。镜像本身将保持不变。</font>
- 仓库(Repository)
存放镜像的地方，类似于代码仓库github  gitee等等


![2020-02-06-19-51-47](http://noback.upyun.com/2020-02-06-19-51-47.png)

## 镜像

### 获取镜像
```bash
# 安装Ubuntu:latest镜像
[root@CodeSheep ~]# docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
5c939e3a4d10: Downloading  8.608MB/26.69MB
c63719cdbe7a: Download complete
19a861ea6baf: Download complete
651c9d2d6c4f: Downloadan zhuang 
# 安装Ubuntu:14.04镜像
docker pull ubuntu:14.04
# 从社区安装ubuntu镜像
docker pull dl.dockerpool.com:5000/ubuntu
```

### 提交一个容器为新的镜像(不提倡)
建议使用dockerfile创建新的镜像
```bash
docker commit CONTAINER  REPOSITORY:tag
root:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
612090a371f6        centos              "/bin/bash"         5 seconds ago       Up 5 seconds                            frosty_tereshkova
root:~# docker commit frosty_tereshkova new/centos:v4
sha256:2b405e30c679a448d9b913c19c48a4c435447655001abad264fcc3e7b2264312
```

### 运行镜像

```bash
docker run -t -i ubuntu /bin/bash
```

### 查看镜像信息

```bash
# 列出本地主机上所有镜像
# --------------------------
root:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/ubuntu    latest              ccc6e87d482b        6 days ago          64.2 MB

# REPOSITORY 表示镜像的仓库源
# TAG 表示镜像的标签
# IMAGE ID 表示镜像的ID
# CREATED 镜像的创建时间
# SIZE 镜像大小
# --------------------------

# 显示所有镜像的ID -a 显示所有镜像 -q显示ID
root:~# docker images -qa
41f1d7eba8f3
470671670cac
ccc6e87d482b
fce289e99eb9

# 显示镜像的摘要信息
root:~# docker images --digests

# 显示镜像详细信息
root:~# docker images --no-trunc

# 添加新的镜像标签
root:～# docker tag docker.io/ubuntu:latest docker-ubuntu-version:v2
root:~# docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
docker-ubuntu-version   v2                  ccc6e87d482b        6 days ago          64.2 MB
docker.io/ubuntu        latest              ccc6e87d482b        6 days ago          64.2 MB
docker.io/ubuntu        14.04               6e4f1fe62ff1        4 weeks ago         197 MB

# 查看镜像相信信息
docker inspect ccc6e87d482b 
docker imspect ccc

```
REPOSITORY表示镜像来自于哪个仓库
TAG 标签信息，版本信息
IMAMGE ID 镜像的ID号码，唯一
CREATED 创建时间
SIZE  镜像大小


### 搜寻镜像
```bash
docker search mysql
# --automated=false 仅显示自动创建的镜像
# --no-trunc=false 输出信息不截断显示
# -s --starts=0 指定仅显示评价为指定星级以上的镜像
```

### 删除镜像
```bash
docker rmi IMAGE[IMAGE...]
# IMAGE...可以是标签或者ID
# docker rmi -f IMAGE[IMAGE...] 强制删除

# 删除全部
docker rmi  -f $(docker images -qa)
```
当前镜像有被使用于某一个容器的时候，使用docker rmi IMAGE 会出现报错的内容,可以进行强制的删除，但这样的做法并不建议，可以先删除依赖于该镜像的容器，然后再删除该镜像


## 容器
### 创建容器与启动
使用create创建容器后，容器是处于停止状态的，可以使用start开启
```bash
# 创建容器
docker create -it ubuntu:test 
# 开启容器
docker start ID
# 重新开启容器
docker restart ID
```
### 新建并启动容器
```bash
# docker run [OPTIONS] IMAGE [COMMAND][ARG...]
# --name 指定容器名字，未指定则随机生成
# -t 给docker分一个伪终端 
# -i 提供交互模式 
# -d 后台运行容器，并返回容器ID
# -P 随机端口映射
# -p 指定端口映射 主机端口:容器端口
# -e 指定环境变量

docker run -i -t ubuntu:latest /bin/bash

# 区分-p 与 -P的区别
docker pull comcat 
docker run -it -d -p 8080:8080 tomcat
docker run -it -d -P tomcat
root:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
d5987c3d0257        tomcat              "catalina.sh run"   3 seconds ago       Up 2 seconds        0.0.0.0:32768->8080/tcp   xenodochial_hugle
ae1370718b31        tomcat              "catalina.sh run"   36 seconds ago      Up 35 seconds       0.0.0.0:8080->8080/tcp    upbeat_beaver
```
exit后容器处于终止状态

### 守护状态运行
正常情况下，退出容器后，容器处于终止状态。使用-d可以在后台运行容器
```bash
docker run -d ubuntu:latest /bin/bash -c "while true;do echo helloworld > xx.txt;sleep 1;done"
```

### 终止容器
使用stop终止容器的时候，会先发送一个SIGTERM信号，等待10s后发送SIGKILL信号终止容器
kill 会直接发送SIGKILL终止容器
```bash
# 终止容器
docker stop -t xxx  ID
docker stop --time xxx  ID
docker kill ID
```

### 进入容器
- attach 命令
attach可以进入-d开启的后台运行中的容器,但是多个窗口同时attach到同一个容器的时候，所有窗口会同步显示，当某个窗口命令阻塞的时候，其他窗口也无法执行操作了
```bash
docker attach ID
```
- exec 命令
通常使用这个
```bash 
docker exec -it ID /bin/bash 
```

### 删除容器
删除终止状态下的容器
```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
# -f --force=false 终止并删除一个正在运行的容器
# -l --link=false 删除容器的连接，但保留容器
# -v --volumes=false 删除容器挂载的数据卷

# 停止所有容器
docker stop $(dokcer ps -q)
# 删除所有容器
docker rm $(docker ps -aq) #需要先暂停
docker rm -f $(docker ps -aq) # 不需要暂停
```

### 导入与导出容器
```bash
# 导入
docker export ID > xx.tar
# 导出
cat xx.tar | docker import - test/ubuntu:v1
```

### 查看容器
```bash
# 查看活跃的容器
root:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4171c783b51f        470671670cac        "/bin/bash"         12 days ago         Up 2 seconds                            devtest
# 查看所有容器
root:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
7945b6135dda        ubuntu              "echo '“hello world'"   5 seconds ago       Exited (0) 4 seconds ago                       silly_kirch
# -l 显示最近创建的容器
# -n 显示最近n个创建的容器

# 查看容器日志
docker logs -f -t --tail  IMAGE

# 查看容器内运行进程
docker top IMAGE

#
```
status 状态表示是否在后台运行，up表示在后台运行  Exited表示终止

### 复制容器内文件
```bash
docker cp IMAGE:file_path now_path 
```

## 帮助命令
```bash
# 查看版本
docker version

# 显示docker信息，包括容器数量，镜像数量等等
[root@CodeSheep ~]# docker info
Client:
 Debug Mode: false

Server:
 Containers: 3
  Running: 0
  Paused: 0
  Stopped: 3
 Images: 1
 Server Version: 19.03.1
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs


# 查看日志
docker logs CONTAINER
```

## 清除容器
默认情况下，每个container在退出时，它的文件系统也会保存下来。这样一方面调试会方便些，因为你可以通过查看日志等方式来确定最终状态。另外一方面，你也可以保存container所产生的数据。但是当你仅仅需要短期的运行一个前台container，这些数据同时不需要保留时。你可能就希望docker能在container结束时自动清理其所产生的数据。 这个时候你就需要--rm这个参数了

```bash
sudo docker run --rm centos:latest
```

<font color='red'>注意：--rm 和 -d不能共用！</font>

## 数据卷容器
--volumes-from 顾名思义，就是从另一个容器当中挂载容器中已经创建好的数据卷。
如果你有一些持续更新的数据需要在容器之间共享，最好创建数据卷容器。 数据卷容器，其实就是一个正常的容器，专门用来提供数据卷供其它容器挂载的。 我们首先先创建一个数据卷容器

```bash
# 给postgres数据库创建一个数据卷容器
sudo docker run -d -v /dbdata --name dbdata training/postgres echo Data-only container for postgres

# 创建容器挂在数据卷
sudo docker run -d --volumes-from dbdata --name db1 training/postgres
```
数据卷容器本身是不需要保持运行的


**利用数据卷容器来备份、恢复、迁移数据卷**
首先使用 --volumes-from 标记来创建一个加载 dbdata 容器卷的容器，并从本地主机挂载当前到容器的 /backup 目录。命令如下：
```bash
sudo docker run --volumes-from dbdata -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```
容器启动后，使用了 tar 命令来将 dbdata 卷备份为本地的 /backup/backup.tar。 如果要恢复数据到一个容器，首先创建一个带有数据卷的容器 dbdata2。
```bash
sudo docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```
然后创建另一个容器，挂载 dbdata2 的容器，并使用 untar 解压备份文件到挂载的容器卷中
```bash
sudo docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf
/backup/backup.tar
```


## 错误合集

- 错误0
docker下载源出错
```bash
base                                                                                                                                                            | 3.6 kB  00:00:00     
http://download.docker.com/linux/centos/docker-ce.rep/repodata/repomd.xml: [Errno 14] HTTPS Error 404 - Not Found
Trying other mirror.
To address this issue please refer to the below wiki article  
```
这是因为添加源是国外源，有时候网络不对的情况下无法安装，建议添加阿里源
```bash
# 删除仓库内的国外源
cd /etc/yum.repos.d/
rm -rf docker-ce.repo download.docker.com_linux_centos_docker-ce.rep.repo 
# 添加阿里源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
- 错误1
没有开启docker服务
```bash
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running 

# 检测
root:~# systemctl status docker
* docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: http://docs.docker.com

# 开启服务
systemctl start docker
```

- 错误2
使用当前镜像的容器正在运行
```bash
root:~# docker rmi ccc6e87d482
Error response from daemon: conflict: unable to delete ccc6e87d482b (must be forced) - image is referenced in multiple repositories

# 强制删除
docker rmi -f ccc6e87d482

# 不建议强制删除，应当先删除所有依赖于该镜像的容器，再删除镜像
```

