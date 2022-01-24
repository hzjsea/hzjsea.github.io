---
title: docker容器的资源限制
categories:
  - docker
tags:
  - docker
abbrlink: 8aa524f6
date: 2020-02-06 21:54:42
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [docker容器的资源限制](#docker容器的资源限制)
  - [使用stress进行压力测试](#使用stress进行压力测试)

<!-- /code_chunk_output -->
<!-- more -->


# docker容器的资源限制

在虚拟机中我们可以对所创建的机器在资源上进行限制，比如内存等等
同样的，在docker中也能对容器进行资源上的配置
```bash
# 分配400M的内存给容器  指定给memory的话是200M 但其实swap也会得到200M 因此是400M
docker run --memory=200M new/ubuntu-stress 
```

## 使用stress进行压力测试
```bash
# dockerfile
FROM ubuntu
RUN apt-get update && apt-get install stress -y
ENTRYPOINT ["/usr/bin/stress"]
# ENTRYPOINT ["/bin/bash","-c","stress"]
CMD []
```
先分配给他400M 然后使用stress做压力测试，stress 会开启进程，进程默认为256M 然后不断的开启不断的释放
```bash
# -m 开一个进程 --verbos 详细信息  默认开启256M --vm-bytes 指定开启多少大小的内存进程
docker run --memory=200M  new/ubuntu-stress -m 1 --verbos 

# 超范围运行
root:~# docker run  --memory=200M  new/ubuntu-stress -m 1 --verbose --vm-bytes 500M
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: dbug: [1] using backoff sleep of 3000us
stress: dbug: [1] --> hogvm worker 1 [6] forked
stress: dbug: [6] allocating 524288000 bytes ...
stress: dbug: [6] touching bytes in strides of 4096 bytes ...
stress: FAIL: [1] (415) <-- worker 6 got signal 9
stress: WARN: [1] (417) now reaping child worker processes
stress: FAIL: [1] (421) kill error: No such process
stress: FAIL: [1] (451) failed run completed in 0s
```
