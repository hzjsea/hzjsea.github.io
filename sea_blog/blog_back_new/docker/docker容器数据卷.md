---
title: docker容器数据卷
categories:
  - docker
tags:
  - docker
abbrlink: 5f4d14d2
date: 2020-02-05 12:35:48
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker](#docker)
  - [数据管理](#数据管理)
    - [创建数据卷](#创建数据卷)
    - [查看是否挂载成功](#查看是否挂载成功)
    - [容器间数据共享传递](#容器间数据共享传递)
  - [报错](#报错)

<!-- /code_chunk_output -->
<!-- more -->
# docker
## 数据管理
数据卷是一个可提供使用的特殊目录，他绕过文件系统，可以提供很多有用的内容
- 数据卷可以在容器之间共享和重复使用
- 对数据卷的修改会立马生效
- 对数据卷的更新，不会影响镜像
- 卷会一直存在 直到没有容器使用

### 创建数据卷
```bash
# 查看数据卷
root:~# docker volume ls
DRIVER              VOLUME NAME
local               1c4c829dcedf5c071d6a14dea5cf9a40e5459a6d81d2a009ecef6c392156121c
local               2a8601766a8ec82c39132bc64c2c99546053cb4104bd94507f215b888e60464d
local               3fad677be0bbe16df041895a36b7ce088a57f6c75fcab163664522ddc405100d
local               3ff453f8da89ad256ee7971df11bc3297443b894584cec147c8637ce838a274b

# 查看volumes详细信息
docker volume inspect 1c4c829dcedf5c071d6a14dea5cf9a40e5459a6d81d2a009ecef6c392156121c
# 创建数据卷连接
docker run -it -v /宿主机绝对目录:/容器内目录  镜像名

# 运行时，挂载卷
# 无论哪一方修改文件，数据卷都会同步到另一端
# --privileged 关闭selinux
docker run -it -d -v /tmp/data/:/tmp/ --name test1 --privileged=true  centos

# 定义权限
doker run -it -v /宿主机绝对目录:/容器内目录:权限 镜像名

# 只读 ro - read-only
docker run -it -d -v /tmp/data/:/tmp/:ro --name test1 --privileged=true  centos
```

### 查看是否挂载成功
```bash
docker inspect test1 
```
![2020-02-05-13-13-45](http://noback.upyun.com/2020-02-05-13-13-45.png)

inspect会按照json的形式输出配置，其中mount表明了数据卷的挂载情况
RW:true表示可读写


### 容器间数据共享传递
从上面我们可以直到容器与本机之间的传递我们是有的是-v 或者dockerfile中的VOLUME进行挂载
在容器与容器之间，使用的是--volumes-from来进行文件之间的挂载,并且只要容器存在，容器数量上的减少和增加都不会使得挂载文件的消失
```bash
docker -it --name doc2 --volumes-from doc1 hzj/centos2
```
![2020-02-05-15-49-18](http://noback.upyun.com/2020-02-05-15-49-18.png)
![2020-02-05-15-53-27](http://noback.upyun.com/2020-02-05-15-53-27.png)
## 报错
1. 问题1
ls: cannot open directory 'tmp/': Permission denied
无法访问目录，权限拒绝。该问题通常在centos7下出现。或者一个容器启动成功后，里面的服务无法成功访问，这是因为centos7中的安全模块selinux把权限禁掉了，一般的解决方案有以下两种：
- 临时关闭selinux
直接在centos服务器上执行以下命令即可。执行完成以后建议重新docker run。
```bash
setenforce 0 
```
- 给容器加权限
在docker run时给该容器加权限，加上以下参数即可
```bash
# --privileged=true
docker run -it -d -v /tmp/data/:/tmp/ --name test1 --privileged=true  centos
```


2. 问题2
为啥没有这个命令
root@8d3aece482eb tmp]# getenforce
bash: getenforce: command not found
[root@8d3aece482eb tmp]#
[root@8d3aece482eb tmp]#
[root@8d3aece482eb tmp]# setenforce
bash: setenforce: command not found
[root@8d3aece482eb tmp]#

<font color='red'>因为这是最精简的centos,删除了所有跟内核无关的东西</font>