---
title: docker镜像分层原理
categories:
  - docker
tags:
  - docker
abbrlink: 685470da
date: 2020-02-05 11:09:43
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker镜像分成原理](#docker镜像分成原理)
  - [base镜像](#base镜像)
  - [原理解析](#原理解析)
  - [docker镜像组成](#docker镜像组成)
  - [docker镜像加载](#docker镜像加载)
  - [镜像大小的问题](#镜像大小的问题)
  - [镜像分层的好处](#镜像分层的好处)
  - [创建镜像](#创建镜像)

<!-- /code_chunk_output -->
<!-- more -->

# docker镜像分成原理


## base镜像
base 镜像有两层含义：

- 不依赖其他镜像，从 scratch 构建。
- 其他镜像可以之为基础进行扩展。
如centos镜像


## 原理解析
了解几个概念
- 联合文件系统(Union File System):是一种把其它文件系统联合到一个联合挂载点的文件系统服务，它使用 branch 把不同文件系统的文件和目录"透明地"覆盖，形成一个单一一致的文件系统。这些 branch 或者是 read-only 的，或者是 read-write 的，所以当对这个虚拟后的联合文件系统进行写操作的时候，系统是真正写到了一个新的文件中。看起来这个虚拟后的联合文件系统是可以对任何文件进行操作的，但是其实它并没有改变原来的文件。这是因为 Union File System 用到了一个重要的资源管理技术：写时复制。
- 写时复制(copy-on-write，常被简写为 CoW)，也叫隐式共享，是一种提高资源使用效率的资源管理技术。它的思想是：如果一个资源是重复的，在没有对资源做出修改前，并不需要立即复制出一个新的资源实例，这个资源被不同的所有者共享使用。当任何一个所有者要对该资源做出修改时，复制出一个新的资源实例给该所有者进行修改，修改后的资源成为其所有者的私有资源。通过这种资源共享的方式，可以显著地减少复制相同资源带来的消耗，但是这样做也会在进行资源的修改时增加一部分开销。


<font color='red'>AUFS文件系统实例</font>
https://www.cnblogs.com/sparkdev/p/11237347.html



## docker镜像组成
在docker的image中,他其实是由一层一层的基础文件系统所叠加起来的，联合文件系统(UnionFS)他可以将几层目录挂载到一起，形成一个虚拟文件系统,docker 通过这些文件再加上宿主机的内核提供了一个 linux 的虚拟环境。每一层文件系统我们叫做一层 layer，联合文件系统可以对每一层文件系统设置三种权限，只读（readonly）、读写（readwrite）和写出（whiteout-able）

但是在docker的镜像中的每一层的基础文件系统都是只读的，我们可以把他们叫做base 最基础的。构建镜像的时候，从一个最基本的操作系统开始，每个构建的操作都相当于做一层的修改，增加了一层文件系统。一层层往上叠加，上层的修改会覆盖底层该位置的可见性，这也很容易理解，就像上层把底层遮住了一样。当你使用的时候，你只会看到一个完全的整体，你不知道里面有几层，也不清楚每一层所做的修改是什么。结构类似这样：
![2020-02-05-11-18-03](http://noback.upyun.com/2020-02-05-11-18-03.png)

当我们在从官方拉去镜像的时候，他的操作也会实现这一步叠加的过程，记住 像我们平时使用的ubuntu centos nginx这样的docker镜像都不是最原始的镜像，他们也都是一层一层的文件系统base叠加起来的,比如拉去ubuntu镜像
![2020-02-05-11-22-23](http://noback.upyun.com/2020-02-05-11-22-23.png)
他一共会拉去四层文件系统来完成整个ubuntu镜像的搭建，最后生成一个ubuntu的镜像，因此在外部看来他是单独的镜像文件，但其实他是由多个基础的文件系统通过联合文件系统叠加产生的。

## docker镜像加载
![2020-02-05-11-35-32](http://noback.upyun.com/2020-02-05-11-35-32.png)
- bootfs 在linux的开机过程中会用到bootfs,主要包含bootloader 和 kernel，bootloader 主要用于引导加载 kernel，当 kernel 被加载到内存中后kernel后，bootfs就会被umount掉。
- rootfs  rootfs (root file system) 包含的就是典型 Linux 系统中的/dev，/proc，/bin，/etc 等标准目录和文件,不同的linux发行版本，rootfs文件系统可能不通



![2020-02-05-12-04-07](http://noback.upyun.com/2020-02-05-12-04-07.png)
控制读写权限,传统的 Linux 加载 bootfs 时会先将 rootfs 设为 read-only，然后在系统自检之后将 rootfs 从 read-only 改为 read-write，然后我们就可以在 rootfs 上进行读写操作了。但 Docker 在 bootfs 自检完毕之后并不会把 rootfs 的 read-only 改为 read-write，而是利用 union mount（UnionFS 的一种挂载机制）将 image 中的其他的 layer 加载到之前的 read-only 的 rootfs层之上，每一层 layer 都是 rootfs 的结构，并且是read-only 的。所以，我们是无法修改一个已有镜像里面的 layer 的！只有当我们创建一个容器，也就是将 Docker 镜像进行实例化，系统会分配一层空的 read-write 的 rootfs ，用于保存我们做的修改。



## 镜像大小的问题
为什么tomcat镜像大小大于centos镜像大小?
![2020-02-05-12-13-04](http://noback.upyun.com/2020-02-05-12-13-04.png)
可以通过上面docker镜像的组成、加载两方面来解释
同样以kernel作为最底层文件系统，在centos镜像中，上层只放了一个centos的rootfs，而tomcat需要写入多层base文件系统，比如jdk8 tomcat 等等，因此tomcat的镜像大小大于centos的镜像大小

## 镜像分层的好处
<font color='red'>共享资源</font>
有多个镜像都从相同的base镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像，同时内存中也只需加载一份base镜像，就可以为所有容器服务了，饿而且镜像的每一层都可以被共享


## 创建镜像
- 基于已有镜像的容器创建
```bash
# 提交一个容器成为一个新的镜像
docker commit [OPTIONS] CONTAINER [RSPOSITORY[:TAG]]
# -a; --author="" 作者信息
# -m; --message="" 提交信息
# -p; --pause=true 提交时暂停容器运行
```
创建流程
```bash
# pull 一个新的image 
docker pull ubuntu
# 改变ubuntu 使其与原来image不同
docker run -it ccc6e87d482b /bin/bash
echo "update file " > test
exit
# 返回前记住当前容器的名字，一般都是主机名
# 提交当前容器为新的镜像文件
docker commit -m "update - add a file in ubuntu " -a "hzj" dc758c368fda test
# 查看当前镜像列表
root:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test                latest              41f1d7eba8f3        10 seconds ago      64.2 MB
docker.io/ubuntu    latest              ccc6e87d482b        6 days ago          64.2 MB
```
- 基于本地模版导入
```bash
# 下载模版压缩包
wget xx
# 根据压缩包导入成镜像
cat ubuntu-14.04.tar.gz | docker import - ubuntu:14.04 

# 保存镜像到本地文件
docker save -o ubuntu_latest.tar ubuntu:latest
# 载入镜像
docker load --input ubuntu_latest.tar
docker load < ubuntu_latest.tar
```
- 基于Dockerfile创建

<font color='red'>如何创建一个hello-world的docker镜像</font> 
1. 创建一个输出hello world的c文件
```c
#include<stdio.h>

int main()
{
  printf("hello world\n");
}
```
2. 编译c文件
```bash
gcc -static hello.c -o hello 
```
3. 编写dockerfile
```bash 
FROM scratch
ADD hello /
CMD ['/hello']
```
4. 新建helloworld镜像
```bash
docker -f /tmp/dockerfile -t helloworld . 
```