---
title: httpd服务
categories:
  - linux
tags:
  - linux
abbrlink: '4982550'
date: 2020-01-03 17:18:50
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [https服务](#https服务)
  - [apache http server](#apache-http-server)
    - [目录介绍](#目录介绍)
    - [配置信息](#配置信息)
  - [python3构建http服务](#python3构建http服务)
  - [维护&错误](#维护错误)
    - [http服务已开启，但是无法访问网站](#http服务已开启但是无法访问网站)
    - [检测端口是否被开启](#检测端口是否被开启)
    - [开放端口](#开放端口)
  - [地址](#地址)

<!-- /code_chunk_output -->
<!--more-->
# https服务
## apache http server
Apache HTTP Server（简称Apache）是Apache软件基金会的一个开放源码的网页服务器，可以在大多数计算机操作系统中运行，由于其多平台和安全性被广泛使用，是最流行的Web服务器端软件之一。它快速、可靠并且可通过简单的API扩展，将Perl/Python等解释器编译到服务器中。

<font color='blue'>  在Apache中，其配置文件目录为“/etc/httpd/conf/httpd.conf”，这里面包括设置网站资源的存放目录及一些相关的配置。</font>

```bash
# 安装http
yum install httpd 
# 开启服务
systemctl start httpd
# 检查httpd服务状态
systemctl status httpd
# 查看端口状态 netstat -anutp | grep  80
```

### 目录介绍
配置文件目录为“/etc/httpd/conf/httpd.conf”
```bash
[root@localhost httpd]# ls -1
conf  # 存放服务的主配置文件
conf.d # 存放apache的主页信息
conf.modules.d
logs
modules
run
```


### 配置信息
配置文件目录为“/etc/httpd/conf/httpd.conf”
```bash
[root@localhost httpd]# vi /etc/httpd/conf/httpd.conf
ServerRoot "/etc/httpd"  #apache配置文件的根目录
Timeout 60  #超时时间，即连接服务端在60秒内没有任何操作，即自动断开
Listen 80   #监听的端口
ServerAdmin root@localhost  #设置管理员，e-mail 地址
ServerName www.example.com:80  #服务器主机名.

DocumentRoot "/var/www/html"   #网站页面根目录,存放文档的地方
<Directory "/var/www/html"> 
Options Indexes FollowSymLinks #O目录浏览 #Followsymlinks:可以用连接，要是想要禁止显示文件目录，可以直接在‘indexes’前加‘-’。
    AllowOverride None
    Order allow,deny #目录与访问的控制
    Allow from all 
</Directory>

# 别名
Alias /icons/ "/var/www/icons/" #别名和别名目录
<Directory "/var/www/icons">
    Options Indexes MultiViews FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>

# 默认页面
</Directory> Options Indexes  #当一个目录没有默认首页时，允许显示此目录列表
<Directory />
    Options FollowSymLinks
    AllowOverride None
</Directory>
--
DirectoryIndex index.html index.html.var #指定默认首页
AddDefaultCharset UTF-8   #设置服务器的默认编码为： UTF-8
```



## python3构建http服务
```bash
python3 -m http.server 
```


## 维护&错误

### http服务已开启，但是无法访问网站

1. 在本地查看httpd服务
```bash
curl http://localhost:80  
```
2. 看端口，如果是在公司内部，查看是否有部分ip端口被封 比如80 8080 88 等等
如果是以上情况，修改httpd 配置文件中的监听端口，重启http服务即可
3. 永久关闭selinux ,需要重启
```bash
vi /etc/selinux/config
SELINUX=disabled
```
4. 临时关闭
```bash
setenforce 0 
```

可能会出现的报错信息
(13)Permission denied: make_sock: could not bind to address [::]:88
----> 关闭selinux即可

### 检测端口是否被开启
使用telnet登陆的方式测试服务器端口是否开放
```bash
# 未开放状态下的返回情况
➜  ~ telnet 10.0.5.92 8888
Trying 10.0.5.92...
telnet: Unable to connect to remote host: Connection refused

# 开放状态下的返回情况
➜  ~ telnet 10.0.5.91 8888
Trying 10.0.5.91...
Connected to 10.0.5.91.
Escape character is '^]'.
```


### 开放端口

端口未被开放
——---> iptables -I INPUT -p tcp --dport 8888 -j ACCEPT



## 地址
https://www.linuxidc.com/Linux/2017-05/143468.htm