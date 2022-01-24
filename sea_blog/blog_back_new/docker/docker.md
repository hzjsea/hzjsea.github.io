---
title: docker
date: 2020-12-07 11:23:15
categories: [docker,]  
tags: [docker,]
---


<!-- more -->

# docker


## docker备忘命令
```bash
# 关闭并删除所有容器
docker ps -a|awk '/ipes/{print $1}'|xargs docker stop
docker ps -a|awk '/ipes/{print $1}'|xargs docker rm
# 删除所有镜像
docker images | awk '{print $3}'  | xargs docker rmi -f
# 清空docker不在运行的镜像和容器
docker system prune # 慎用
```