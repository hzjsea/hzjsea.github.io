---
title: nginx优化配置
categories:
  - nginx
tags:
  - nginx
abbrlink: 12ec35da
date: 2020-03-19 03:31:05 
---
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [nginx优化配置](#nginx优化配置)
  - [server模块](#server模块)
    - [server模块配置](#server模块配置)

<!-- /code_chunk_output -->
<!-- more -->

# nginx优化配置

之前是nginx的一些基础配置，另外还要有Nginx的优化设置，根据不同的优化场景来调节nginx的配置，这里记录一些这些调节的内容
https://nginx.org/en/docs/http/ngx_http_core_module.html


## server模块

### server模块配置
![2020-03-19-15-36-00](http://noback.upyun.com/2020-03-19-15-36-00.png)

`调优参数都是经验积累，需要积累`

web应用中listen函数的backlog默认会给我们内核参数的net.core.somaxconn限制到128，而nginx定义的NGX_LISTEN_BACKLOG默认为511，所以有必要调整这个值。如：
net.core.netdev_max_backlog = 262144
我们线上服务器net.core.somaxconn都是默认的128,这个参数会影响到所有AF_INET类型socket的listen队列
Man 2 listen可以知道:
int listen(int s, int backlog);