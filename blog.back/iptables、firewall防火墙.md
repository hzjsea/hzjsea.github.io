---
title: iptables、firewall 防火墙
categories:
  - linux
tags:
  - linux
abbrlink: dd7b4b1a
date: 2020-01-05 12:03:16
---



<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [linux防火墙](#linux防火墙)
  - [iptables、firewall防火墙](#iptables-firewall防火墙)
    - [firewall](#firewall)
    - [iptables](#iptables)
  - [防火墙错误导致的原因](#防火墙错误导致的原因)

<!-- /code_chunk_output -->
<!--more-->
# linux防火墙
## iptables、firewall防火墙
Linux中有两种防火墙软件，ConterOS7.0以上使用的是firewall，ConterOS7.0以下使用的是iptables

<!-- more -->
### firewall
```bash
# 开启防火墙
systemctl start firewalld
# 关闭防火墙
systemctl stop firewalld
# 查看状态
systemctl status firewalld
firewall-cmd  --state
# 设置开机启动
systemctl enable firewalld
# 禁用开机启动
systemctl disabled firewalld
# 重启防火墙
firewall-cmd --reload
# 开放端口
firewall-cmd --zone=public --add-port=8080/tcp --permanet
# 查看已经开放的端口
firewall-cmd --list-ports 
firewall-cmd --list-all
# 关闭端口
firewall-cmd --zone=public --remove-port=8080/tcp --permanet\

```

https://juejin.im/post/5d538326f265da03ce39cf38

### iptables
```bash
# 安装iptables 
yum install  iptables
# 安装iptables-services
yum install iptables-services
# 开启防火墙
systemctl start iptables
# 关闭防火墙
systemctl stop iptables.services
# 防火墙状态 、开机启动 、 禁用开机启动、
status 、 enable 、disable
# 查看filter表的几条链规则(INPUT链可以看出开放了哪些端口)：
iptables -L -n
# 查看连标规则
iptables -t nat -L -n 
# 清除防火墙所有规则
iptables -F
# 给INPUT链添加规则（开放8080端口）
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
# 查找规则所在行
iptables -L INPUT --line-numbers -n
# 根据行号删除过滤规则(关闭8080)
iptables -D INPUT 1
```



## 防火墙错误导致的原因
1. 开通httpd后，当前机器下使用curl可访问，但外部机器访问ip+端口则无法访问
```bash
[root@localhost ~]# curl http://10.0.5.92:8888
curl: (7) Failed connect to 10.0.5.92:8888; No route to host
```
curl报如下错误：curl: (7) Failed connect to 172.16.225.43:7001; No route to host

解决办法
----> 在被访问机器下开放8888端口
----> iptables -l INPUT -p tcp --dport 8888 -j ACCEPTt 8888 -j ACCEPT