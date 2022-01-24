---
title: netstat
categories:
  - linux
tags:
  - linux
abbrlink: 1372101d
date: 2020-01-05 11:51:48
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [netstat工具](#netstat工具)
  - [常用指令](#常用指令)

<!-- /code_chunk_output -->

<!--more-->


## netstat工具
netstat是在内核中访问网络相关信息的程序。
1. 提供TCP连接、TCP和UDP监听、进程内存管理的状态。
2. 监控TCP/IP网络的非常有用的工具，显示路由表、实际网络连接以及每一个网络接口设备的状态信息。
3. 使用netstat可以让用户知道有哪些网络连接正在运作，使用时如果不带参数，netstat显示活动的TCP连接

```bash
# 安装
yum install net-tools
# 使用
netstat 选项：
常用参数：
-a:显示所有的socket，包括监听的以及未监听的。
-n:不使用域名和服务名，而使用IP和端口号。
-l:仅列出在listen状态的网络服务。
-p:显示建立连接的程序名和PID。
-e:显示以太网统计。
-s:显示每个协议的统计。
-t：显示TCP协议连接情况。
-u:显示UDP协议连接情况。
-c:每隔一个固定时间，执行netstat命令。
-r：显示核心路由表。
-i:显示所有的网络接口信息<> 
```

### 常用指令
```bash
# 显示建立连接的程序名 PID IP 端口以及监听状态
[root@localhost ~]# netstat -pan | grep 8888
tcp6       0      0 :::8888                 :::*                    LISTEN      16711/httpd 
```

