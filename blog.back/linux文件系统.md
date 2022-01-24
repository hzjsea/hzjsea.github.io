---
title: linux文件系统2
categories:
  - linux
tags:
  - linux
abbrlink: cd527bc1
date: 2020-01-09 11:04:27
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux文件系统2](#linux文件系统2)
  - [/etc/hosts文件](#etchosts文件)

<!-- /code_chunk_output -->

<!-- more -->
# linux文件系统2

linux中有许多常用的文件，各自有各自主要的作用 ，怕忘记，这里做些笔记

## /etc/hosts文件
hosts —— the static table lookup for host name（主机名查询静态表）。 
hosts文件是Linux系统上一个负责ip地址与域名快速解析的文件，以ascii格式保存在/etc/目录下。hosts文件包含了ip地址与主机名之间的映射，还包括主机的别名。在没有域名解析服务器的情况下，系统上的所有网络程序都通过查询该文件来解析对应于某个主机名的ip地址，否则就需要使用dns服务程序来解决。通过可以将常用的域名和ip地址映射加入到hosts文件中，实现快速方便的访问。 
优先级 ： dns缓存 > hosts > dns服务

---> 配置

```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.5.91 lvs-master
10.0.5.93 lvs-slave
10.0.5.92 lvs-web1
#10.0.5.90 lvs-web2
10.0.5.90 www.baidu.com

# ip地址 主机名/域名 主机别名
```
相当于本地的域名解析，当你ping www.baidu.com的时候，会默认去ping 10.0.5.90